---
layout: page
title: Implementation of Awareness Into Epidemiological Models
description: Adapting an extended SEIR model to simulate how short and long-term population awareness impacts the spread of COVID-19, H1N1, and Seasonal Flu.
img: assets/img/virus.jpg
importance: 5
category: project
related_publications: False
---

### Introduction: Awareness in Disease Modeling
Code, report, and data is available at my [github](https://github.com/tomaszfrelek2/Disease-Modeling)

Since the inception of epidemiology, mathematical models have been integral to visualizing disease progression and ascertaining the influence of various transmission factors. The standard approach utilizes **SEIR models**, which compartmentalize a population into **S**usceptible, **E**xposed, **I**nfected, and **R**ecovered groups.

However, traditional SEIR models often rely on static parameters for transmission rates, neglecting a critical dynamic variable: human behavior. A population's awareness of a disease—driven by media coverage, fear of death, or government mandates—drastically alters how individuals interact.cFor example, during the H1N1 pandemic, awareness of high risks to children prompted school closures, changing contact patterns.

This project implements a dynamical system originally proposed by [Weitz et al.](https://www.pnas.org/doi/10.1073/pnas.2009911117), which integrates awareness-driven behavior changes into an extended SEIR framework. By adding compartments for Hospitalization ($H$) and Death ($D$), the model adjusts the transmission rate based on the population's sensitivity to recent (short-term) and cumulative (long-term) fatalities.

We recreated this system in Python to simulate three distinct scenarios:
1.  **COVID-19:** A novel, high-mortality pandemic with extreme public awareness.
2.  **H1N1 (Swine Flu):** A pandemic strain with significant media attention but lower mortality than COVID-19.
3.  **Seasonal Flu (2013-2014):** A standard recurring outbreak with low public fear and awareness.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/covid_data.png" title="Figure 1: COVID-19 Daily Deaths Model. " class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/swine_data.png" title="Figure 2: H1N1 Weekly Hospitalizations Model" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/flu_data.png" title="Figure 3: Seasonal Flu Weekly Hospitalizations Model" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Comparison of model performance across three diseases. Left: COVID-19 (high awareness). Middle: H1N1 (medium awareness). Right: Seasonal Flu (low awareness). Red line is predicted infection rate. Blue data points are irl infection statistics.
</div>

### Methodology

The core of the Weitz et al. model is the modified transmission equation, which reduces the infection rate as deaths rise. The rate of change for susceptible individuals ($\dot{S}$) is defined as:

$$ \dot{S}=-\frac{\beta SI}{[1+(\delta/\delta_{c})^{k}+(D/D_{c})^{k}]} $$

Here, the denominator represents the "awareness factor".
* **$\delta_c$ (Short-term awareness):** The half-saturation constant representing the number of daily deaths/hospitalizations a population tolerates before reducing contact.
* **$D_c$ (Long-term awareness):** The cumulative total of deaths/hospitalizations the population tolerate.
* **$k$:** The sharpness of the public's reaction to these metrics.

Crucially, these parameters are **inversely proportional** to awareness: a lower $\delta_c$ or $D_c$ value means the population has a lower threshold for tragedy, indicating *higher* awareness and faster behavioral changes.

### Discussion of Results

Our Python implementation confirmed that awareness parameters strongly correlate with the public "fear factor" of each disease, though the model struggled with secondary outbreaks caused by extraneous social dynamics.

#### 1. COVID-19: High Awareness, Accurate Initial Fit
The model fit the COVID-19 data exceptionally well for the first 250 days of the pandemic.
* **Parameters:** We found $\delta_c = 25e-7$ and $D_c = 4000e-7$. These were the lowest values among all three simulations, mathematically confirming that the population was extremely sensitive to COVID-19 deaths.
* **Limitations:** The model failed to predict the large spike in November 2020. This spike is historically attributed to Halloween gatherings and pandemic fatigue—social factors that a pure awareness model does not explicitly track.

#### 2. H1N1: Medium Awareness, Extraneous Factors
For H1N1, we substituted death data with hospitalization data due to the low mortality rate.
* **Parameters:** The best fit yielded $\delta_c = 90e-7$ and $D_c = 8000e-7$. These values are higher than COVID-19 but lower than the seasonal flu, accurately reflecting H1N1's status as a "media-sensationalized" pandemic.
* **Limitations:** The model accurately captured the first outbreak but missed the secondary spike in October. This secondary wave coincided with the start of the school year—a major transmission vector for H1N1, which affected children disproportionately.

#### 3. Seasonal Flu: Low Awareness, Timing Discrepancies
The 2013-2014 seasonal flu simulation required the highest parameter values ($\delta_c = 108e-7$, $D_c = 16500e-7$), indicating the population was largely desensitized to standard flu risks.
* **The "Lag" Error:** While the model captured the general shape of the curve, it predicted the peak approximately **25 days earlier** than it actually occurred.
* **Hypothesis:** This error likely stems from using hospitalization as a proxy for death. In the model, we treated hospitalization as having the same "gravitas" as death. In reality, people are less terrified of flu hospitalization than COVID-19 death. By over-weighting the fear response to hospitalization, the model predicted the population would react (and flatten the curve) sooner than they actually did.

### Conclusion & Future Work

Ultimately, our research demonstrates that a population's awareness is a quantifiable and critical factor in epidemiological modeling. The consistency of the awareness parameters—lowest for COVID-19 and highest for the seasonal flu—aligns with the historical reality of public sentiment during each outbreak. The simulations successfully accounted for significant portions of the infection curves for all three diseases, validating that awareness-driven behavior changes can shift the shape of epidemics.

However, the discrepancies in our results highlight the complexity of modeling real-world behavior. The model's inability to predict secondary spikes caused by social events (like holidays or school terms) suggests that awareness alone is insufficient for predicting long-term trends. Furthermore, the timing error in the seasonal flu model suggests that future iterations should incorporate a weighting constant. This constant would differentiate the psychological impact of hospitalization versus death, rather than treating them as equivalent triggers for behavioral change.