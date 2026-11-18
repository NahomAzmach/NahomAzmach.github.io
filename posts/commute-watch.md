Every weekday I drive from Marysville to Bellevue. If you know that stretch, you know the question isn't *if* it's bad, it's *how* bad and whether leaving ten minutes later fixes it.

This started because I set up [God's Eye View](https://github.com/bilawalsidhu/gods-eye-view), the spy-satellite-looking globe with live planes, ships and public cameras, and got kind of obsessed with the fact that there's just... live public camera data sitting out there. I added WSDOT traffic cameras to it for my own commute, and then caught myself opening a globe app every morning to squint at highway stills. Something else should be doing the squinting and just tell me "go now" or "wait 10 min."

So that's Commute Watch. Every few minutes on weekday mornings it reads the live WSDOT camera stills along my two usual routes, decides how bad each one looks, and pings my phone if things are getting worse.

<div class="stats">
<div><b>~1,958</b><span>public cameras on the browse map (WSDOT + Seattle DOT)</span></div>
<div><b>$0.0003</b><span>per route check, about 20¢ a month for my mornings</span></div>
<div><b>8 / 8</b><span>identical outputs on frozen images at temperature 0</span></div>
</div>

![The /explore page: type any two places in Washington and it checks the cameras along the way, live.](assets/img/commute.webp)

## Version 1: the cool model that scored garbage

The first build was around TypeSafe's JEV classifier, a tiny, cheap model built for yes/no-and-a-score decisions instead of full chat responses. I really liked the idea of it. The problem was upstream: JEV was scoring *descriptions* that a cheap vision model wrote about each image, and that vision model kept bailing with "too dark to determine conditions" on about half the cameras. So JEV was confidently scoring garbage.

Not JEV's fault. Garbage in, garbage out. Still a very cool architecture to learn, but I ripped it out.

## Version 2: one call, five words

The version running now does one thing per route: every camera image goes to Gemini 2.5 Flash-Lite in a single call, and the model has to put each image into one of five boxes: `clear`, `slow`, `congested`, `stopped`, or `unreadable`. That's it. No "rate this route 1 to 10."

Then plain code turns those labels into the route score. The model never does the math.

```ts
// lib/ai.ts: the route score is deterministic, the model only labels images
function computeSeverity(values: (number | null)[]) {
  const readable = values.filter((v): v is number => v !== null);
  if (readable.length === 0) return { score: 0, probability: 0 };

  const stoppedCount   = readable.filter((v) => v === 3).length;
  const congestedCount = readable.filter((v) => v === 2).length;
  const slowCount      = readable.filter((v) => v === 1).length;
  const badCount = stoppedCount + congestedCount;

  let score: number;
  if (stoppedCount >= 2 || (stoppedCount >= 1 && congestedCount >= 1) || congestedCount >= 3) {
    score = 3;            // needs more than one camera agreeing
  } else if (badCount >= 1) {
    score = 2;            // one bad camera = real but moderate flag
  } else if (slowCount >= 1) {
    score = 1;
  } else {
    score = 0;
  }
  ...
}
```

The top tier needs corroboration from more than one checkpoint on purpose. One "stopped" camera can just be a car at a red light, and I didn't want my phone telling me to reroute over a single misread.

Forcing five fixed categories per image turned out to be *way* more repeatable than asking the model to eyeball a whole route's severity, and it killed the useless "too dark" descriptions completely.

## The bug that wasn't the model

I burned an embarrassing amount of time chasing what looked like model flakiness. Same route, wildly different scores seconds apart. I was ready to blame Gemini.

Then I realized I was refetching the live images on every test. The traffic itself was changing between runs, not the model. Once I froze real camera images to fixed bytes and ran the classifier 8 times against the same bytes, I got identical output every single time.

> Before you blame the model, make sure the input is actually holding still.

## Teaching it what a red light is

Once it was stable, the remaining errors were the boring, real-world kind:

- A car stopped at a red light or stop sign was getting labeled `stopped`. I rewrote the label definitions so normal controlled stopping counts as `clear`.
- Ferry holding lots, weigh stations and rest areas are in WSDOT's camera catalog. An empty holding lot was reading as congestion, so those got filtered out.
- The map used one color per route, which hid *which* camera was the problem. Now every camera gets its own colored dot.

## Grounding it in things that aren't the model

I didn't love that the whole system leaned on one vision model's opinion, so I started feeding it real, cheap signals:

- **Frame differencing.** A pixel-level diff against the last time anyone checked that camera. Actual motion, instead of the model imagining "motion blur" in a single still.
- **Vehicle detection** with a pretrained YOLOv8n. This one had a deployment dead end worth writing down: I built it on `onnxruntime-web`'s wasm backend specifically to avoid native binaries. It worked perfectly locally, then died in production with `ERR_MODULE_NOT_FOUND` because Next.js's serverless file tracer couldn't follow its dynamic imports. I tried explicit includes, globs, a literal dynamic import in my own code. Nothing. Switched to `onnxruntime-node`, which is honestly Microsoft's own recommended path for server-side Node, and validated it against a reference PyTorch run on real WSDOT images before shipping.
- **WSDOT road sensors.** Real loop-detector readings shown as a "Sensor: ___" badge next to the AI's read. Early results are mixed, so it's a second opinion, not ground truth.

One thing I tried and dropped: an edge-density "how busy is this frame" score. Turns out guardrails and trees dominate edge energy, so an empty road with lots of scenery outscored a packed road with a plain background.

## The part where I tried to replace Gemini with my own model

The long-term idea is to distill all of this into a small classifier I own. So every classified checkpoint gets logged with its features (frame change, vehicle count, sensor reading, travel-time ratio, hour, camera) plus Gemini's label.

My first baseline was humbling. A from-scratch logistic regression, validated leave-one-camera-out:

- Class-balanced: **19–20% accuracy**, against an 80–85% "always say clear" baseline.
- Unweighted: ~83% accuracy, and 0% recall on everything except clear. It just learned to say clear.
- Going from 731 rows to 2,102 rows didn't help at all. So "it just needs more data" was off the table.

The interesting part was figuring out *why*. Gemini's "congested" calls only matched the WSDOT point sensor about 10% of the time, which looked damning. But WSDOT's corridor travel ratio (current drive time vs. typical) told the opposite story: 1.01 for clear, 1.05 slow, 1.12 congested, 1.18 stopped. Clean and monotonic. At a ratio of 1.2 or higher, congested + stopped jumps from ~4% of rows to 40%.

So Gemini's labels weren't noise. The point sensor is just too spatially narrow, and the data was starved: only 57% of checkpoints even had a travel-ratio match, and a month of *uniform* 24/7 collection meant very few rush-hour bad-traffic examples. Not a label problem. A coverage and collection problem.

Two fixes went out: I widened the travel-time match radius from 0.5 to 1.5 miles (coverage went from 77% to 88% of WSDOT cameras, checked against real data first), and switched to checking every 5 minutes during commute hours and every 30 otherwise.

Also, GitHub Actions' scheduled trigger never reliably fired on time. I saw gaps of 3 to 6 hours against a 10-minute schedule, across multiple incidents. I stopped tuning cron expressions GitHub wasn't honoring and moved everything to Upstash QStash.

## /explore, for everyone else stuck in WA traffic

There's a public `/explore` page now. Type any two places in Washington and it geocodes them (Nominatim), gets a real driving route (OSRM), finds WSDOT cameras along that path out of the ~1,700 in the catalog, spreads up to 8 along the route, and runs the same live check. No account, nothing saved, rate-limited per visitor so nobody runs up my bill.

One thing I chose *not* to do: Lynnwood's cameras run through a paid, per-camera-licensed video platform, not a public API. Embedding them would have routed random visitor traffic through the city's paid account with no clear authorization, so they're not in there. Just because something is reachable doesn't mean it's mine to use.

## Where it's headed

The newest experiment lets you click a camera to save its current still and then search saved moments in plain English (EmbeddingGemma, images in private Cloudflare R2, vectors in Supabase pgvector). It only searches frames someone chose to save. There's no fake history.

After that: a per-camera relative baseline for the visual signals, and eventually that distilled model, once the commute-hour data catches up.

**Stack:** Next.js, Upstash Redis + QStash, Gemini 2.5 Flash-Lite via Vercel AI Gateway, OSRM, Nominatim, ONNX Runtime (YOLOv8n), Supabase pgvector, ntfy.sh for push.
