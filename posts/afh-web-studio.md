Adult Family Homes are small, licensed homes in Washington where a handful of residents live and get round-the-clock care. They're run by people who are great at care and usually have zero time for websites. And it shows: a lot of AFH sites are dated, hard to use on a phone, and don't turn a worried family member into a tour booking.

That's a very specific niche, and I think specific is good. So AFH Web Studio is a done-for-you web studio built only for Adult Family Homes: design, content and updates handled, so the owners can focus on care.

![AFH Web Studio's home page.](assets/img/afhstudio.webp)

## The studio site is the pitch

The studio's own site had to prove the point, so I treated it like a product:

- **Nine ready templates** with names that sound like real homes: Everbloom Care, Cedar Grove Living, Maple & Stone, Sunrise Orchard, Bluebird Haven, and more. Owners can preview any of them live in a modal without leaving the page.
- **Three plans** with clear pricing, so nobody has to "contact us for a quote" just to find out what it costs.
- **Case studies** from real builds.

## Plan My Site

My favorite part. Most AFH owners can't tell you what they want in design terms, but they can absolutely tell you what they *like* when they see it. So instead of a contact form, there's an **11-step visual questionnaire** that takes about five minutes: pick a palette (Warm & Earthy, Forest & Sage, Ocean & Teal...), a mood, the care services you offer, what your rooms are like.

Their answers turn into a clear design brief that gets sent over, and that's what I build from.

![Plan My Site: the visual questionnaire that replaces "tell us about your project."](assets/img/afhstudio-plan.webp)

It's the same lesson I keep relearning in other projects: don't ask people to describe things in your vocabulary. Give them choices in theirs.

## A real one: Homegrown AFH

Homegrown Adult Family Home is one of the client sites. The brief was warm and trustworthy without feeling clinical: "Live-in care that feels like family," a real tour of the home with photos of every room, the backyard and back porch (yes, even the pets), and care services spelled out plainly.

![Homegrown AFH, a client site built with React, Vite and Tailwind.](assets/img/rodas.webp)

![Everbloom, one of the homes behind the templates.](assets/img/everbloom.webp)

## Under the hood

The studio site is React + TypeScript with Tailwind and shadcn/ui on the front, a small Express API for plan requests, shared Zod schemas between client and server, and Vercel for hosting. Client sites are lighter: React, Vite and Tailwind, deployed on Vercel, so they're fast and cheap to keep running for a small business.

**Stack:** React, TypeScript, Tailwind CSS, shadcn/ui, Wouter, Express, Zod, Vite, Vercel.
