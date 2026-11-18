*Built with Salim Gloyd and Karun Mokha.*

Hausa is spoken by tens of millions of people, but it's what ML people call "low-resource": not much labeled data, not many models built for it. Our goal was simple to say. Take a recording of someone speaking Hausa, get back English text.

The bigger question underneath it was the part I found exciting: **can a model skip the written language entirely?** If you can go straight from speech in one language to text in another, then a language's spoken form might be enough, even where written resources are thin.

There are two ways to build this, so we built both.

```
Cascade:  [Hausa audio] → Whisper (fine-tuned) → [Hausa text] → NLLB-200 → [English]
Direct:   [Hausa audio] → Whisper, task="translate", LoRA → [English]
```

Neither trains from scratch. Both start from models already pretrained on speech and language in general (Whisper for speech, NLLB for translation) and fine-tune them for Hausa. We used `whisper-small` the whole way through so training could actually happen on one consumer GPU.

## The cascade

**Data.** The plan was Mozilla's Common Voice, until Mozilla moved it off the platform we were pulling from in October 2025. So we switched to Google's FLEURS Hausa (Nigeria) split, which had a comparable set of recordings.

**Fitting it on 8 GB.** Our GPU was an RTX 5050 with 8 GB of memory. So: batches of 2 clips, gradient accumulation over 4 (acting like a batch of 8 without the memory), and fp16 mixed precision to roughly halve memory use.

Three full passes over ~3,259 clips took about **4.3 hours**. Loss went from 13.7 down to 0.86, and word error rate on held-out audio dropped every epoch: 50.5% → 45.0% → **44.7%**.

![Cascade training: loss on the left, held-out WER on the right.](assets/img/hausa-curve.webp)

Numbers are abstract, so here's one real test clip:

| | |
|---|---|
| **Ground truth** | *An kwatanta faretin gine-ginen da ke yin sararin samaniyar Hong Kong da ginshiƙi mai walƙiya wanda aka bayyana ta gaban ruwan Victoria Harbor.* |
| **Base Whisper** | *Aung kwa tanta ferietun ginaginang deke yung sarulin samania Hong Kong dekin shiki mei wal kiya...* |
| **Our fine-tune** | *An kwatanta, feryatin gina-ginan da ke yin tsarin samaniya, Hong Kong, da ginshiki mai walƙiya wanda aka bayyana ta gaban ruwan Victoria Harbour.* |
| **Cascade → English** | *"The crystal structure of the skyline, Hong Kong, and the sparkling column that is depicted by the Victoria Harbour waterfront are illustrated."* |

Base Whisper is barely phonetically related to the answer. Ours gets nearly every word. The English is a little awkward, but Hong Kong's skyline, glittering buildings and Victoria Harbour all come through, even after passing through our imperfect transcription first. That gap is what the training bought us. The model is on the Hugging Face Hub as [`nahomazmach/whisper-small-ha`](https://huggingface.co/nahomazmach/whisper-small-ha).

## The direct pilot

The cascade's weakness is that errors compound: NLLB ends up translating our ASR's mistakes. So: what if we skip written Hausa?

Whisper can natively output English, so we LoRA-fine-tuned it in translate mode on real Hausa-audio → English-text pairs from McGill's NaijaS2ST. One practical headache: the training code would have downloaded the dataset's entire ~69 GB train split before using any of it. We used its metadata-only audit tooling to find which 2 of its 115 shards held the most usable pairs and pulled just those. A few GB instead of 69.

It was a small pilot: 256 examples, 50 steps, and LoRA meant training only 1.77 M of Whisper's 243 M parameters (0.7%).

Same Hong Kong clip, direct model:

> *"The government has been working on the development of the Hong Kong-based economic system in the region, which is the only way to help the country win the war."*

Perfectly fluent English. Totally unrelated to what was said. That's what a BLEU of 0.24 looks like: it learned to *sound* like English before it learned to *translate*.

## Head to head

| System | BLEU | chrF++ |
|---|---|---|
| Cascade, gold Hausa text (translation only) | ~22–25 | ~46–50 |
| Cascade, real ASR output (end to end) | **~8–10** | **~33–35** |
| Direct pilot | **0.24** | **14.39** |

![Cascade vs. direct pilot.](assets/img/hausa-compare.webp)

The first two rows are the *same* cascade. The only difference is whether NLLB got clean Hausa or our ASR's output. That drop is the cost of transcription errors compounding, and it's exactly the problem the direct approach is supposed to fix.

![How transcription quality affects translation quality.](assets/img/hausa-wer.webp)

## So, can ML skip the written language?

At this scale, not yet. The cascade wins by a mile. But the comparison isn't fair to the direct model, and I don't want to pretend it is: 256 examples vs. thousands. What the project does show clearly is *where* the cascade loses quality, which is the strongest argument for scaling the direct approach up properly.

Everything runs in a Colab-ready notebook (no GPU needed to explore it) with playable audio, training telemetry, and both models runnable side by side.

**Stack:** PyTorch + CUDA, Hugging Face Transformers / Datasets / PEFT (LoRA), Whisper-small, NLLB-200, FLEURS, NaijaS2ST, sacreBLEU, Jupyter/Colab.
