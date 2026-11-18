The most useful debugging habit I picked up this year has nothing to do with models. It's this: **before you blame the model, make sure the input is holding still.**

In [Commute Watch](#commute-watch) I spent way too long convinced Gemini was flaky. Same route, totally different scores, seconds apart. It turned out I was refetching live traffic camera images on every test. The traffic was changing, not the model. Froze the images to fixed bytes, ran it 8 times, identical every time.

Before that, the original version used a classifier that was confidently scoring descriptions like "too dark to determine conditions." The classifier was fine. What it was fed was garbage.

And in the [Hausa translation project](#hausa-speech), our small direct model produced beautiful, fluent English that had nothing to do with what was said. If you only read the output, you'd think it was working. Fluency isn't accuracy.

Three projects, same lesson. When an AI system does something weird, the model is the *last* place I look now:

1. Is the input actually the same between runs?
2. Is what I'm feeding it any good?
3. Am I measuring the thing I care about, or just something that looks like it?

Usually the answer is in one of those three, and I never have to touch the model at all.
