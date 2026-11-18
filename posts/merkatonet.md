Merkato in Addis Ababa is one of the biggest open-air markets in Africa. MerkatoNet borrows the name for a simple idea: connect Ethiopian farmers directly to buyers, and give farmers the same price information the middlemen have.

The farmer who grows the teff often knows the least about what teff is selling for in Addis today. That information gap is money left on the table.

## Built for the phone people actually have

The obvious version of this is a slick web app. But a slick web app assumes a smartphone and a data plan, and a lot of the people this is for have neither in the field. So the core feature works over plain **SMS**.

Text `PRICE` and you get current teff prices back. `STOP` and `START` handle subscriptions, `HELP` lists commands:

```ts
const message = Body.trim().toLowerCase();

if (message === 'price' || message === 'prices') {
  response = 'Current teff prices in Addis Ababa:\n' +
             'White (Magna): 6,800 ETB/quintal\n' +
             'Mixed: 5,950 ETB/quintal\n' +
             'Red (Sergegna): 5,300 ETB/quintal';
} else if (message === 'stop') {
  response = 'You have been unsubscribed from all MerkatoNet alerts. Reply START to resubscribe.';
}
```

*(Those prices are demo data. The format is the point: real units, ETB per quintal, the varieties people actually trade.)*

Phone numbers are validated with a country code (`+251...`), and if Twilio credentials aren't set, SMS gets mocked so you can still develop locally without paying for messages.

## The web side

For buyers and anyone with a smartphone, there's the full app:

- **Market prices** with current and historical data plus insights.
- **Product listings** you can browse and search.
- **Price alerts** you subscribe to per commodity.
- **English and Amharic**, the whole interface, not just a translated homepage.
- Sign-in with **phone number** or Google through Firebase, because a phone number is a more universal identity than an email address here.

## What I'd do differently

I'd start from the SMS side and build up, not the other way around. The web app was the fun part to build, but the SMS interface is the part that actually matches who the platform is for. Designing for the most constrained user first would've shaped everything else better.

**Stack:** React, TypeScript, Tailwind, shadcn/ui, React Query, Wouter, Express, PostgreSQL + Drizzle ORM, Firebase Auth, Twilio, i18next.
