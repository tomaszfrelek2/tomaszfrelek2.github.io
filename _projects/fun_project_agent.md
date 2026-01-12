---
layout: page
title: Data Scientist Agentic System
description: An autonomous LLM-based agent for end-to-end exploratory and regression analysis using constrained tool use.
img: assets/img/ds_agent.jpg
importance: 3
category: project
related_publications: False
---

Paper, data, and code are available at my [github](https://github.com/tomaszfrelek2/ds_agent).

## Project Overview

This project addresses the gap between the linguistic reasoning of Large Language Models (LLMs) and the grounded statistical reasoning required for reliable data science. While modern LLMs can discuss patterns in natural language, they often lack the ability to verify claims through empirical modeling and frequently hallucinate numerical results. 

Our solution is an autonomous Data Scientist Agentic System capable of performing end-to-end data analysis from natural language research questions. By constraining the LLM to use specific, deterministic tools rather than free-form code generation, the system ensures that findings are grounded in actual computation.

---

## System Architecture

The system is built as a multi-agent pipeline using the LangGraph framework. It utilizes a hybrid model strategy: Llama-70b serves as the operational backbone for iterative logic, while Gemini-2.5-Flash acts as a supervisory judge for final evaluation.

The workflow is orchestrated by a central state object that persists context across four distinct functional nodes:
***Planner Agent ($f_{Plan}$):** Interprets the natural language query and dataset schema to select and sequence the appropriate tools.
***Tool Execution Layer:** Executes the selected deterministic operations on the dataset to produce intermediate statistical results.
***Reviewer Agent ($f_{Rev}$):** Acts as an internal quality gate, comparing results against the original query to decide whether to "Pass" or "Retry" the plan.
***Final Reporter Agent ($f_{Rep}$):** Synthesizes the verified results into a structured Markdown report.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/workflow.png" title="Agentic Workflow" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Workflow diagram.
</div>

---

## Data Analysis Toolkit

The Planner Agent accesses a specialized toolkit designed to perform precise, deterministic operations:

1.  **DescribeData:** Computes key statistics (Mean $\mu$, Median, $\sigma$, IQR) and identifies data quality issues such as missing values or duplicate constraints.
2.  **RunRegression:** Constructs a Random Forest Regressor for predictive queries. It automates preprocessing (Standard Scaling and One-Hot Encoding) and evaluates performance using the coefficient of determination:
    $$R^{2}=1-\frac{\sum(y_{i}-\hat{y}_{i})^{2}}{\sum(y_{i}-\overline{y})^{2}}$$
    The tool also extracts feature importance scores to explain the drivers of variance to the user.

---

## Experimental Performance

We evaluated our system using a Housing Prices Dataset (545 samples, 13 features) and a custom validation set of 20 queries.

### Quantitative Results
Our agentic system was compared against a baseline of direct LLM prompting (Llama-70b without tools).

|          Metric           | Baseline LLM (No Tools) | DS Agent (Our System) |
| :------------------------ | :---------------------- | :-------------------- |
| **Ground Truth Acc.**     |       20% (4/20)        |      90% (18/20)      |
| **Tool Selection Acc.**   |           N/A           |      100% (20/20)     |
| **Avg. Response Latency** |         47.38s          |         19.63s        |

**Key Findings:**
* **Accuracy:** The system yielded a 4.5x improvement in ground truth accuracy over direct prompting.
* **Efficiency:** The agent was 2.4x faster than the baseline. This gain comes from offloading math to specialized tools and reducing the number of input tokens by avoiding injecting the entire dataset into the LLM prompt.
* **Reliability:** While the baseline LLM often provided only conceptual descriptions of how it *would* solve a problem, our agent provided concrete results, such as an $R^2$ score of 0.6101.

### Qualitative Model Comparison
We qualitativly evaluated three backbone models to determine the best brain for the agent:

* **Gemini-2.5-Flash:** Highest quality output; the only model to spontaneously generate Markdown tables and demonstrate superior context-awareness during edge cases (e.g., identifying a simple "hi" as a greeting).
* **Llama-70b:** Provided concise, professional "executive-level" summaries but was significantly slower.
* **Llama-8b:** The fastest (~1.5s) but suffered from lower quality, verbosity, and hallucinations during edge cases.


<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/webapp.png" title="Chat Interface" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    An example of a conversation in the chat interface.
</div>
---

## Lessons Learned & Future Work

* **Design vs. Coding:** We realized the primary work was in the architectural design—determining agent steps and flexibility—rather than just coding.
* **Prompt Sensitivity:** LLMs are highly sensitive to prompt structure, requiring rigorous engineering to ensure consistent tool selection and behavior.
* **Compute Constraints:** While Gemini-2.5-Flash was the superior model, limited free compute credits necessitated using Llama-70b as the default operational model.
* **Generalization:** Future iterations will focus on expanding the tool set (e.g., classification, clustering) and testing the agent on broader data types like text, images, or time series.

---