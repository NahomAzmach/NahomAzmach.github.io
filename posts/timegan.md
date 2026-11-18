Here's a problem I kept running into in energy and ML work: the data you most need is the data you almost never have. Real building energy data is sensitive, often proprietary, and badly imbalanced. If you want a model that handles a heatwave load surge or a grid anomaly, you need lots of examples of rare events, and you can't exactly wait years for them to happen.

So: generate them. This project is my implementation of **TimeGAN** (*Time-series Generative Adversarial Networks*, NeurIPS 2019), applied to building energy load.

## Why a normal GAN isn't enough

A regular GAN can make each hour of a fake day look plausible on its own. But energy load isn't a bag of independent hours. 3 PM depends on 2 PM. A surge ramps up, peaks, and decays. TimeGAN's whole idea is to learn those step-to-step dynamics, not just the distribution of values.

It does that with four networks working together:

<div class="flow">
<div><b>Embedder</b><span>real sequence → latent space</span></div>
<div><b>Recovery</b><span>latent → back to real values</span></div>
<div><b>Generator</b><span>noise → fake latent sequence</span></div>
<div><b>Discriminator</b><span>real or fake?</span></div>
</div>

Plus a **supervisor** that learns to predict the *next* latent step from the current one. That stepwise supervised loss is what pushes the generator to respect temporal dynamics.

## The bug in the original

While porting the original TensorFlow code to PyTorch, I noticed the supervised loss compared the supervisor's output to the *same* time step, not the next one. Which means the supervisor could get a perfect score by... copying its input. It never had to learn dynamics at all.

The paper says next step. So in my PyTorch version, the supervisor is trained the way the paper describes:

```python
def supervised_loss(H):
    # Stepwise supervised loss: supervisor predicts the NEXT latent step
    H_hat_supervise = model.supervisor(H)
    if shift:
        return mse(H_hat_supervise[:, :-1, :], H[:, 1:, :])   # paper: t → t+1
    return mse(H_hat_supervise[:, 1:, :], H[:, 1:, :])         # original TF: t → t
```

I kept it as a checkbox in the app ("Paper's next-step supervised loss") so you can untick it and reproduce the original behavior side by side. I'd rather show the difference than just assert it.

## Real buildings, real scenarios

Lots of TimeGAN implementations live as backend scripts running on abstract benchmark datasets. I wanted this one grounded in actual building physics, so it trains on real hourly load from the **Building Data Genome 2** dataset. Each day becomes one 24-hour sequence. You can pick a single building or pool every building of a type (each scaled by its own average load) to get far more days to train on.

The Streamlit app lets you pick a building, a scenario (**Baseline** or **Load Surge**), training iterations, sample count and a seed, then trains the four networks live with per-phase loss curves, and compares real vs. synthetic sequences with metrics and plots. There's a TensorFlow backend too, kept around as the original reference.

## From scenarios to descriptions

The newest piece is a **conditional generator**. Instead of training a separate model per scenario, one TimeGAN trains on every day, where each day is described by a condition vector: average load, peak hour, surge strength, weekend or not, and temperature. You describe the day you want, and it generates it.

To make the generator actually honor the conditions, it measures its own output (mean, peak hour, peak ratio) and gets penalized when they don't match:

```python
def _condition_loss(self, X_hat, C):
    # Compare in the same [0, 1] scale as the conditions so every feature weighs about equally
    n = self.measurable_features
    ...  # cmin / crange = the min and range of each condition
    measured = (self._measure(X_hat) - cmin) / crange
    return self._condition_loss_weight * torch.mean(torch.abs(measured - C[:, :n]))
```

## Where it's going

That condition vector is the input a plain-text feature would fill in. The plan is that you'll be able to just *describe* a scenario ("a hot August Tuesday with a big afternoon surge") and get matching sequences. The vector is the bridge between language and the generator.

**Stack:** PyTorch, TensorFlow (original reference), NumPy, pandas, Streamlit, Matplotlib, Building Data Genome 2.
