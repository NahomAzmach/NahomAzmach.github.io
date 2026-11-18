I built this one in a week. The Allen Institute's Mouse Brain Connectivity Atlas is an incredible public dataset: where neurons in one brain region send their projections, and how strongly. It's also very hard to *look at* if you're not already a neuroscientist.

So Synaptiviz (the Neuronal Connectivity Explorer) does two things: shows you the data in a few ways that make sense at a glance, and lets you ask questions about it in plain English.

▶ [Watch the demo on YouTube](https://www.youtube.com/watch?v=GDvpaCxxzcs)

![Top projection targets from a source region.](assets/img/syn-bar.webp)

## Three ways to see the same data

Pick a source region and the app pulls its projection data through the AllenSDK, then renders it three ways with Plotly:

- A **bar chart** of the strongest target regions.
- A **heatmap** for comparing strengths at a glance.
- A **network graph**: the source in the middle, targets around it, thicker lines for stronger connections.

<div class="pair">

![Heatmap view.](assets/img/syn-heat.webp)

![Network view, hippocampal formation as the source.](assets/img/syn-network.webp)

</div>

Same numbers every time. But a network graph answers "what does this region talk to?" way faster than a table does, and a bar chart answers "how much?" way faster than a network.

## Ask the brain a question

Every region gets a 1–2 sentence **AI tooltip** explaining what it does, generated with GPT and **cached in SQLite**, so each region only costs one API call ever.

There's also a natural-language box. Type something like *"Explain the relationships in the graph to me as if I'm new to neuroscience"* and the answer comes back grounded in the connectivity data you're actually looking at, not generic textbook text.

![Asking about the current graph in plain English.](assets/img/syn-ai.webp)

## The thread through my work

This one says out loud something that runs through a lot of my projects. From analyzing protein structures with ROSETTA in a computational biology lab, to a rental map, to this: I keep gravitating toward taking complex scientific or real-world data and turning it into something people can actually interact with. Tools shouldn't just present information. They should help you get to the insight.

**Stack:** Python, Flask, Plotly, AllenSDK, OpenAI GPT-3.5, SQLite, Bootstrap.
