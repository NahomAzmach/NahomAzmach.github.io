A lot of my side projects keep pulling me back toward Ethiopia: a farmer marketplace, a church website for an Ethiopian Orthodox community here in Washington. This one goes straight to Addis Ababa.

The question: **Are informal settlements in Addis hotter than the rest of the city, and is that getting worse as the city grows?** Urban heat is a fairness problem. The people least able to adapt usually live where it's hottest. Addis is growing fast, and its informal settlements are spreading into the fringe where city meets countryside.

Every recent study of heat in Addis I could find did the same thing: hand-crafted satellite formulas (NDVI, NDBI and friends) plus standard regression. None of them isolated an informal-settlement effect. I wanted to try something different.

![2024. Left: Sentinel-2 true color. Center: my predicted fabric class (red = informal). Right: daytime land-surface temperature.](assets/img/thermal-panels.webp)

## Let a foundation model describe the ground

Instead of building spectral indices by hand, I used **Google's AlphaEarth** satellite embeddings. It's a foundation model that turns every 10 m patch of Earth into 64 numbers. Then I did the least fancy thing possible on top of it:

1. Exported AlphaEarth embeddings, Landsat + MODIS temperature, Sentinel-2 greenery, and elevation from Google Earth Engine, clipped to Addis's built-up area.
2. Hand-drew about **70 polygons** in QGIS, labeling informal vs. everything else on 2024 imagery.
3. Trained a classifier on each pixel's 64 numbers.

A plain linear model beat both an MLP and XGBoost, which is apparently a known trait of these embeddings: the hard work is already done inside the 64 numbers. Cross-validated AUC: **0.938**.

<div class="stats">
<div><b>93.8%</b><span>cross-validated AUC, informal vs. formal fabric</span></div>
<div><b>31% → 42%</b><span>informal prevalence, 2017 → 2024</span></div>
<div><b>18%</b><span>of the built city flagged as new growth at the fringe</span></div>
</div>

## Train once, apply twice

To compare 2017 and 2024 honestly, I trained the model **once** on 2024 labels and applied the frozen model to both years. If I'd relabeled each year, any change could just be me labeling differently. This way, a change in the map reflects a change on the ground.

![Settlement change, 2017 → 2024. Red is new informal growth.](assets/img/thermal-change.webp)

## The twist

I went in expecting informal fabric to be hotter. During the **day**, it isn't. It's slightly *cooler*: −0.41 °C in 2017, −0.38 °C in 2024, even after controlling for elevation and greenery.

If I'd stopped there, the honest write-up would've been "no heat penalty found."

## The twist on the twist

Then I extended the analysis to **night** using MODIS, and the sign flipped. In 2017 informal fabric runs significantly *hotter* after dark: +0.30 °C per pixel, and **+1.13 °C** in a stricter "pure cell" test (p ≈ 10⁻¹¹).

![Adjusted informal effect by time of day and sensor. Below zero = informal is cooler, above zero = hotter.](assets/img/thermal-daynight.webp)

Daytime Landsat and daytime MODIS agree informal is cooler. The *same* MODIS sensor flips positive at night. Same instrument, so it isn't a sensor artifact. Dense fabric stores heat during the day and releases it at night, which is exactly when people need to cool down and can't.

And it's the **established core**, not the new fringe, that traps the heat. In 2024 the core trends +0.34 °C hotter at night while the new fringe shows nothing. The city is growing into its own heat problem.

![2024 nighttime heat gap: established core vs. new fringe.](assets/img/thermal-core.webp)

> Whether you detect urban-heat inequity depends on the time of day you measure it.

That's the real finding. The AI classifier is a nice step up from index-based mapping, but the day-vs-night contrast is the thing I'd want someone to take away.

## What I'm not claiming

I tried to be strict with myself here:

- Surface temperature isn't air temperature.
- MODIS night is coarse (1 km), and the 2024 core effect is marginal (p = 0.06).
- The labels are mine: 70 self-drawn polygons.
- It's correlational, one city, one season.

All of that is in the paper. The whole pipeline is ten numbered scripts, from Earth Engine export to the core-vs-fringe test, and the trained models, labels and figures are in the repo. The 1 GB embedding exports aren't committed, but script 01 regenerates them.

**Stack:** Google Earth Engine, AlphaEarth embeddings, Landsat, MODIS, Sentinel-2, QGIS, scikit-learn, XGBoost, confounder-adjusted OLS, LaTeX.
