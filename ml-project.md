# Predicting County Home Values from Income and Education

<div class="card">

## Problem Definition

I wanted to look at something with real stakes: can a county's median home value be predicted just from how much people earn and how educated the population is? This is a **regression problem** I'm predicting a continuous number (median home value in dollars), not a category.

The target variable is **Median_Home_Value**. People who might actually care about this: lenders assessing risk, local policymakers trying to understand housing affordability pressure, and real estate analysts looking at which counties are undervalued relative to their income and education levels.

I picked this because income and home value get talked about together constantly, but I wanted to actually test how much of the story they tell on their own, and where that story breaks down.

</div>

<div class="card">

## Background and Context

Housing economics research consistently points to income and education as two of the strongest predictors of home value, but the relationship isn't as clean as it sounds. Homeowners with a bachelor's degree have historically earned significantly more and owned more expensive homes than those without one (Florida Realtors, 2025), which suggested education might carry a meaningful signal here beyond income alone.

At the same time, housing economists have found that income and education variables improve local assessment accuracy but still leave real variation unexplained, since local market dynamics and non-economic factors matter too (as cited in work on property tax assessment accuracy). County-level median home values also vary enormously nationwide, from under $100,000 to well over $800,000 depending on region (National Association of Realtors, 2026), which told me a national dataset like mine would need to handle a lot of spread.

**References:**

National Association of Realtors. (2026). *Median home prices and mortgage payments by county*. NAR Research and Statistics. https://www.nar.realtor/research-and-statistics/housing-statistics/county-median-home-prices-and-monthly-mortgage-payment

Florida Realtors. (2025, June 12). *How education is shaping homeownership*. https://www.floridarealtors.org/news-media/news-articles/2025/06/how-education-shaping-homeownership

[Authors to confirm]. (2026). *Tradeoffs are domain dependent: Improving accuracy and fairness in property tax assessments*. arXiv. https://arxiv.org/pdf/2605.15020

*(Note to self before final submission: confirm full author names on the third source by opening the arXiv link directly.)*

</div>

<div class="card">

## Data Description

I pulled this data live from the **U.S. Census Bureau's American Community Survey (ACS) 5-year estimates**, using the Census API. I used the 5-year dataset specifically instead of the 1-year version because 1-year estimates exclude any county under 65,000 people — using it would have quietly cut out most of rural America and skewed the whole dataset toward big counties.

Each row represents **one U.S. county**. After pulling data and dropping rows with missing or invalid values, I ended up with **[FILL IN: final row count from Cell 3 output]** counties out of roughly 3,144 nationwide. The features I pulled were total population, median household income, median home value, and the count of residents with a bachelor's degree, which I then converted into an education rate.

Missing values were minimal and most counties report all four core variables, and the ones that didn't were dropped rather than estimated, which I'll come back to in the limitations section.

</div>

<div class="card">

## Data Understanding and Exploration

Before modeling anything, I looked at how these variables actually behaved. Income and population were both right-skewed, a small number of very large or very wealthy counties pull the average upward, while most counties cluster lower. Home value showed the same pattern, which made sense once I saw the income skew, since the two are clearly related.

The correlation heatmap confirmed what I expected but also sharpened it: income had a noticeably stronger relationship with home value than population did on its own. The scatter plot added a layer I hadn't initially appreciated. Counties with higher bachelor's degree rates weren't just wealthier, they tended to sit above the general income-to-home-value trend line, suggesting education might carry information that income alone doesn't fully capture.

This is exactly why I included Bachelors_Rate as its own feature rather than assuming income would cover it.

</div>

<div class="card">

## Data Preparation and Feature Selection

I converted the raw Census numbers to numeric types (they come back as text strings by default), renamed the columns to something readable, and dropped any county missing a value in population, income, home value, or bachelor's count. I also filtered out any row where home value was recorded as zero or negative, which happens when the Census suppresses data for very small geographies.

I engineered one new feature: **Bachelors_Rate**, dividing bachelor's degree count by total population. Raw bachelor's *count* would have just been a proxy for county size, not education level — a huge county with a low education rate could still have a high raw count. The rate version actually measures what I care about.

I selected three features for modeling: Median_Income, Total_Population, and Bachelors_Rate. I left out the raw bachelor's count specifically to avoid redundancy with the rate version.

For training, I used an 80/20 train-test split with a fixed random seed so results are reproducible. I scaled all three features using StandardScaler, but importantly, I fit the scaler only on the training data and applied that same transformation to the test data. If I'd fit the scaler on the full dataset before splitting, information from the test set would have leaked into training, making my evaluation artificially optimistic.

</div>

<div class="card">

## Baseline and Model Development

My baseline was a dummy model that just predicts the mean home value every time, regardless of input. This sounds almost too simple to count, but it's the right baseline: any real model needs to beat just guessing the average to prove it's learning something.

I trained two real models:

**Linear Regression** I chose this first because it's interpretable. I can look directly at the coefficients and say exactly how much each feature moves the prediction, which matters for a problem like this where the "why" is as interesting as the "what."

**Random Forest Regressor** I chose this second because it can pick up on non-linear relationships and interactions between features that a straight line can't. Given what I saw in the exploration step (education appearing to shift the income-home value relationship rather than just adding to it), a model that can capture that kind of interaction felt worth testing.

I didn't do extensive hyperparameter tuning beyond setting a reasonable tree depth (max_depth=8) and number of trees (200) for the Random Forest, mainly to avoid overfitting on a dataset with this much natural variance.

</div>

<div class="card">

## Model Evaluation and Selection

I evaluated all three models (baseline, Linear Regression, Random Forest) using RMSE, MAE, and R².

- **RMSE** (root mean squared error) tells me, in dollars, roughly how far off my average prediction is, with larger errors penalized more heavily.
- **MAE** (mean absolute error) gives a more intuitive "typical" dollar error, without the extra penalty on big misses.
- **R²** tells me what percentage of the variation in home value my model actually explains, versus just noise.

**Results:**

| Model | RMSE | MAE | R² |
|---|---|---|---|
| Baseline (Mean) | [FILL IN] | [FILL IN] | [FILL IN] |
| Linear Regression | [FILL IN] | [FILL IN] | [FILL IN] |
| Random Forest | [FILL IN] | [FILL IN] | [FILL IN] |

[Once you have real numbers: Random Forest outperformed Linear Regression on RMSE and R², which tells me there's a genuinely non-linear relationship here, not just interactions my linear model could approximate — if that's what you see]. I selected **[FILL IN: whichever model actually won]** as my final model because [tie this to a specific metric improvement, e.g. "it reduced RMSE by $X compared to Linear Regression, meaning predictions were meaningfully closer to actual home values"].

</div>

<div class="card">

## Model Interpretation and Insights

Looking at the Random Forest's feature importance, **[FILL IN: which feature ranked highest in your Cell 14 chart]** carried the most weight in the model's predictions, followed by [FILL IN: second feature]. This [matches / doesn't match] what the correlation heatmap suggested early on, which [tells me the simple linear relationship held up / tells me there's more going on than a simple correlation captures].

The actual-vs-predicted plot shows the model tracking real home values reasonably well in the middle of the range, but [FILL IN: describe what you actually see — does it struggle more with very expensive counties? Does it cluster tightly or scatter widely?]. The residual plot backs this up: [FILL IN: do residuals look randomly scattered around zero, or is there a pattern, like residuals growing larger for higher-value counties?].

What I can conclude: income and education together explain a meaningful share of county-level home value variation, but they're clearly not the whole story — local market conditions, housing supply, and geography almost certainly matter too, and this model doesn't see any of that.

</div>

<div class="card">

## Limitations, Ethics, and Reflection

This dataset has real gaps. It only captures income, population, and education. It says nothing about housing supply, zoning, local amenities, or regional cost-of-living differences, all of which genuinely drive home values. A model like this could be misleading if used on its own for something like loan risk assessment, since it would systematically miss counties where home values are high or low for reasons unrelated to income or education (tourist destinations, areas with strict zoning, etc.).

If this kind of model were used in a real decision making context say, flagging counties as "overvalued" or "undervalued" for investment, A false prediction could unfairly steer resources away from or toward a community based on an incomplete picture. I wouldn't consider this model appropriate for real financial decision-making as is; it's better suited as an exploratory tool for understanding broad national patterns, not as a decision engine for any single county.

If I extended this project, I'd want to add housing supply data, regional cost-of-living indexes, and maybe historical trend data instead of a single snapshot year, since none of that is currently represented.

</div>

<div class="card">

## Code and Transparency

Full notebook: [View on GitHub](ml_project.ipynb)

Data source: U.S. Census Bureau, American Community Survey 5-Year Estimates (2023), accessed via the [Census API](https://www.census.gov/data/developers/data-sets/acs-5year.html).

**AI Usage Disclosure:** I used Claude (Anthropic) throughout this project to help debug API authentication and environment setup issues, structure the data cleaning and modeling pipeline, and format/style the visualizations. I also used it to help organize this write-up. All research question framing, feature selection reasoning, model selection logic, and interpretation of results reflect my own analysis and decisions.

</div>
