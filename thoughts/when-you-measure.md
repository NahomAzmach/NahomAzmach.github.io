Two of my projects this year ended up teaching me the same thing from totally different directions: **the way you measure something can decide the answer before you even look at the data.**

In the [Addis Ababa heat project](#thermal-inequity), daytime satellite data said informal settlements were slightly *cooler* than the rest of the city. If I'd stopped there, the conclusion would have been "no heat penalty." Same sensor at night: informal areas ran up to about a degree hotter. The inequity was real the whole time. Daytime just couldn't see it.

In [Commute Watch](#commute-watch), I tried to check my AI's traffic labels against WSDOT's road sensors. They agreed about 10% of the time, which looked like proof the AI was wrong. Then I compared against WSDOT's corridor travel times instead, and the AI's labels lined up almost perfectly. The point sensor was just too narrow, often not even looking at the same stretch of road as the camera.

Both times, the first measurement I reached for gave a clean, confident, wrong-feeling answer. And both times the fix wasn't a better model. It was asking: *is this measurement even capable of seeing what I'm looking for?*

I think about that a lot now. A null result is only as good as the instrument that produced it.
