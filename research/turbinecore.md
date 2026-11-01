---
layout: page
title: TurbineCore
subtitle: World-model-based autonomous wind turbine condition monitoring
permalink: /research/turbinecore/
---

<p class="lead">
TurbineCore reads a wind turbine's own SCADA record, works out how healthy
it is, and decides how hard it should be allowed to run today.
</p>

<div class="btn-row">
{% if site.turbinecore_demo_url != "" %}<a class="btn" href="{{ site.turbinecore_demo_url }}" target="_blank" rel="noopener">Open the live demo</a>{% endif %}
{% if site.turbinecore_repo_url != "" %}<a class="btn btn-ghost" href="{{ site.turbinecore_repo_url }}" target="_blank" rel="noopener">Code on GitHub</a>{% endif %}
</div>
{% if site.turbinecore_demo_url != "" and site.turbinecore_demo_note != "" %}<p class="demo-note">{{ site.turbinecore_demo_note }}</p>{% endif %}

A wind turbine already reports everything it does: power, wind speed,
temperatures, pressures, vibration, pitch and yaw angles, alarms, and the
limits the controller was given. TurbineCore learns what a particular turbine
normally does with that record, notices when it stops behaving that way,
works out which component is drifting, says how likely a failure is over the
coming weeks, and then decides how hard the turbine should be allowed to run.

The decision is deliberately narrow: the only action is the power limit the
operator would send anyway. Holding a turbine below its rating costs energy
today and buys a lower chance of losing it for a week. The system prices
both sides of that trade with its own model of the turbine before
recommending anything.

## How it works

<div class="step-grid" markdown="0">
  <div class="step"><span class="step-n">1</span><h4>One vocabulary for five farms</h4><p>Every farm names its signals differently. Each channel is mapped onto a shared semantic description (component, physical quantity, unit, and whether it is a measurement or an operator command), so a model trained on four farms can read the fifth.</p></div>
  <div class="step"><span class="step-n">2</span><h4>The world model</h4><p>Given a turbine's recent history and the commands it is about to receive, it predicts every channel with an uncertainty band: a learned stand-in for the machine that answers "what would this turbine do if we capped it at 1.6&nbsp;MW tomorrow?"</p></div>
  <div class="step"><span class="step-n">3</span><h4>Anomaly detection</h4><p>A copy of the world model trained only on confirmed-healthy periods sets expectations; the gap between expectation and reality is the anomaly score, which surfaces slow drifts long before an alarm fires.</p></div>
  <div class="step"><span class="step-n">4</span><h4>Fault diagnosis</h4><p>The same gap, read channel by channel, says where the trouble is: gearbox, generator, converter, pitch or yaw system, down to subcomponent level.</p></div>
  <div class="step"><span class="step-n">5</span><h4>Failure probability</h4><p>The probability of failure within 7 to 60 days, recalibrated against recorded failures so that a stated 10&nbsp;% means roughly one in ten in practice.</p></div>
  <div class="step"><span class="step-n">6</span><h4>Written audit &amp; policy</h4><p>A locally hosted language model writes up each turbine-day for an engineer to challenge. Candidate power limits are simulated through the world model and scored on energy, failure risk, availability and cost; on most days the policy does nothing.</p></div>
</div>

## Results

### World model

Forecast accuracy of the world model over all channels of the held-out
blocks, against the persistence baseline ("the next value equals the last
one"). MASE below 1 means the model beats persistence.

<div class="stat-grid" markdown="0">
  <div class="stat"><span class="stat-value">0.928</span><span class="stat-label">R² at Penmanshiel<br><small>persistence 0.847</small></span></div>
  <div class="stat"><span class="stat-value">0.928</span><span class="stat-label">R² at Hill of Towie<br><small>persistence 0.832</small></span></div>
  <div class="stat"><span class="stat-value">0.71 / 0.72</span><span class="stat-label">MASE, Penmanshiel / Hill of Towie<br><small>persistence = 1</small></span></div>
  <div class="stat"><span class="stat-value">0.758</span><span class="stat-label">AUROC, failure within 15 days<br><small>63.9% recall at 5% false alarms</small></span></div>
</div>

<div class="table-wrap" markdown="0">
<table class="metrics two-label">
  <thead><tr><th>Farm</th><th>Model</th><th>R²</th><th>MAE</th><th>RMSE</th><th>MASE</th><th>Scored cells</th></tr></thead>
  <tbody>
    <tr class="best"><td>Penmanshiel</td><td>World model</td><td>0.9276</td><td>0.152</td><td>0.293</td><td>0.709</td><td>493,621</td></tr>
    <tr><td>Penmanshiel</td><td>Persistence</td><td>0.8474</td><td>0.214</td><td>0.425</td><td>1.000</td><td>493,621</td></tr>
    <tr class="best"><td>Hill of Towie</td><td>World model</td><td>0.9283</td><td>0.133</td><td>0.288</td><td>0.724</td><td>2,066,680</td></tr>
    <tr><td>Hill of Towie</td><td>Persistence</td><td>0.8316</td><td>0.184</td><td>0.442</td><td>1.000</td><td>2,066,680</td></tr>
  </tbody>
</table>
</div>
<p class="table-note">MAE and RMSE are on standardised channel values. Numbers are the precomputed results shown in the live demo.</p>

### Diagnosis, failure prediction and policy

<div class="card-grid two" markdown="0">
  <figure class="card figure-card">
    <img src="{{ '/assets/img/results/tc-diagnosis-scores.png' | relative_url }}" alt="Fault diagnosis accuracy, precision, recall and F1 on Kelmarsh, Penmanshiel, Farm D and Farm E" loading="lazy">
    <figcaption><strong>Fault diagnosis.</strong> Accuracy, precision, recall and F1 of component-level diagnosis on four wind farms.</figcaption>
  </figure>
  <figure class="card figure-card">
    <img src="{{ '/assets/img/results/tc-diagnosis-confusion.png' | relative_url }}" alt="Normalised confusion matrix over twelve turbine components" loading="lazy">
    <figcaption><strong>Which component?</strong> Normalised confusion matrix over twelve turbine components, from brake to yaw system.</figcaption>
  </figure>
  <figure class="card figure-card">
    <img src="{{ '/assets/img/results/tc-failure-auroc.png' | relative_url }}" alt="AUROC of failure prediction at 7, 15, 30, 45 and 60 days for TurbineCore, ANN, Cox-Time and logistic regression" loading="lazy">
    <figcaption><strong>Failure prediction.</strong> AUROC for failure within 7 to 60 days, against ANN, Cox-Time and logistic regression baselines.</figcaption>
  </figure>
  <figure class="card figure-card">
    <img src="{{ '/assets/img/results/tc-policy-cumulative.png' | relative_url }}" alt="Cumulative energy and availability gain of the learned policy at Penmanshiel and Hill of Towie" loading="lazy">
    <figcaption><strong>Power-limit policy.</strong> Cumulative gain over doing nothing in (a) energy, MWh, and (b) availability, turbine-days, across the decision period at Penmanshiel and Hill of Towie.</figcaption>
  </figure>
</div>

## Data and scope

Results come from recorded operation at five wind farms, with the decision
study carried on two of them. Public datasets include Kelmarsh, Penmanshiel
and Hill of Towie. The language model runs locally and no plant data leaves
the project's machines. Every figure in the demo is precomputed from result
files; nothing touches a live turbine.

## Related publications

<ul class="pub-list">
{% for p in site.data.publications %}{% include publication.html pub=p %}{% endfor %}
</ul>
