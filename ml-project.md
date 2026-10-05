# Predicting County Home Values from Income and Education

<div class="card">

## Problem Definition

I wanted to look at something with real stakes: can a county's median home value be predicted just from how much people earn and how educated the population is? This is a regression problem. I'm predicting a continuous number, median home value in dollars, not a category.

The target variable is Median_Home_Value. People who might actually care about this include lenders assessing risk, local policymakers trying to understand housing affordability pressure, and real estate analysts looking at which counties are undervalued relative to their income and education levels.

I picked this topic because income and home value get talked about together constantly, but I wanted to actually test how much of the story they tell on their own, and where that story breaks down.

</div>

<div class="card">

## Background and Context

Housing economics research consistently points to income and education as two of the strongest predictors of home value, but the relationship isn't as clean as it sounds. Homeowners with a bachelor's degree have historically earned significantly more and owned more expensive homes than those without one (Florida Realtors, 2025). This suggested that education might carry a meaningful signal here beyond income alone.

At the same time, housing economists have found that income and education variables improve local assessment accuracy but still leave real variation unexplained, since local market dynamics and non economic factors matter too (as cited in work on property tax assessment accuracy). County level median home values also vary enormously nationwide, from under $100,000 to well over $800,000 depending on region (National Association of Realtors, 2026). That told me a national dataset like mine would need to handle a lot of spread.

**References:**

National Association of Realtors. (2026). *Median home prices and mortgage payments by county*. NAR Research and Statistics. https://www.nar.realtor/research-and-statistics/housing-statistics/county-median-home-prices-and-monthly-mortgage-payment

Florida Realtors. (2025, June 12). *How education is shaping homeownership*. https://www.floridarealtors.org/news-media/news-articles/2025/06/how-education-shaping-homeownership

[Authors to confirm]. (2026). *Tradeoffs are domain dependent: Improving accuracy and fairness in property tax assessments*. arXiv. https://arxiv.org/pdf/2605.15020

</div>

<div class="card">

## Data Description

I pulled this data live from the U.S. Census Bureau's American Community Survey 5 year estimates, using the Census API. I used the 5 year dataset instead of the 1 year version because 1 year estimates exclude any county under 65,000 people. Using it would have quietly cut out most of rural America and skewed the whole dataset toward big counties.

Each row represents one U.S. county. After pulling data and dropping rows with missing or invalid values, I ended up with 3,216 counties and county equivalents out of roughly 3,144 counties nationwide, since the Census Bureau also counts some independent cities and other county equivalents separately.

Missing values were minimal. Most counties report all four core variables, and the ones that didn't were dropped rather than estimated, which I come back to in the limitations section.

</div>

<div class="card">

## Data Understanding and Exploration

Before modeling anything, I looked at how these variables actually behaved. Income and population were both right skewed. A small number of very large or very wealthy counties pull the average upward, while most counties cluster lower. Home value showed the same pattern, which made sense once I saw the income skew, since the two are clearly related.

The correlation heatmap confirmed what I expected but also sharpened it. Income had a noticeably stronger relationship with home value than population did on its own. The scatter plot added a layer I hadn't initially appreciated. Counties with higher bachelor's degree rates weren't just wealthier, they tended to sit above the general income to home value trend line, suggesting education might carry information that income alone doesn't fully capture.

This is exactly why I included Bachelors_Rate as its own feature rather than assuming income would cover it.

</div>

<div class="card">

## Data Preparation and Feature Selection

I converted the raw Census numbers to numeric types, since they come back as text strings by default, and renamed the columns to something readable. I dropped any county missing a value in population, income, home value, or bachelor's count. I also filtered out any row where home value was recorded as zero or negative, which happens when the Census suppresses data for very small geographies.

I engineered one new feature, Bachelors_Rate, by dividing bachelor's degree count by total population. Raw bachelor's count would have just been a proxy for county size, not education level. A huge county with a low education rate could still have a high raw count. The rate version actually measures what I care about.

I selected three features for modeling: Median_Income, Total_Population, and Bachelors_Rate. I left out the raw bachelor's count specifically to avoid redundancy with the rate version.

For training, I used an 80/20 train test split with a fixed random seed so results are reproducible. I scaled all three features using StandardScaler, but importantly, I fit the scaler only on the training data and applied that same transformation to the test data. If I had fit the scaler on the full dataset before splitting, information from the test set would have leaked into training, making my evaluation artificially optimistic.

</div>

<div class="card">

## Baseline and Model Development

My baseline was a dummy model that just predicts the mean home value every time, regardless of input. This sounds almost too simple to count, but it's the right baseline. Any real model needs to beat just guessing the average to prove it's learning something.

I trained two real models.

Linear Regression was my first choice because it's interpretable. I can look directly at the coefficients and say exactly how much each feature moves the prediction, which matters for a problem like this where the why is as interesting as the what.

Random Forest Regressor was my second choice because it can pick up on non linear relationships and interactions between features that a straight line can't. Given what I saw in the exploration step, where education appeared to shift the income to home value relationship rather than just adding to it, a model that can capture that kind of interaction felt worth testing.

I didn't do extensive hyperparameter tuning beyond setting a reasonable tree depth (max_depth=8) and number of trees (200) for the Random Forest, mainly to avoid overfitting on a dataset with this much natural variance.

</div>

<div class="card">

## Model Evaluation and Selection

I evaluated all three models, baseline, Linear Regression, and Random Forest, using RMSE, MAE, and R².

RMSE, or root mean squared error, tells me in dollars roughly how far off my average prediction is, with larger errors penalized more heavily.

MAE, or mean absolute error, gives a more intuitive typical dollar error, without the extra penalty on big misses.

R² tells me what percentage of the variation in home value my model actually explains, versus just noise.

**Results:**

| Model | RMSE | MAE | R² |
|---|---|---|---|
| Baseline (Mean) | $139,133 | $90,559 | 0.001 |
| Linear Regression | $97,805 | $60,804 | 0.505 |
| Random Forest | $69,990 | $45,344 | 0.747 |

Random Forest clearly outperformed both the baseline and Linear Regression on every metric. It cut RMSE by about $27,800 compared to Linear Regression, and explained roughly 75 percent of the variation in county home values compared to about 50 percent for the linear model. I selected Random Forest as my final model. The gap between the two models tells me the relationship between these features and home value isn't a simple straight line, which lines up with what I found when I looked at feature importance below.

</div>

<div class="card">

## Model Interpretation and Insights

Looking at the Random Forest's feature importance, Median_Income carried the most weight by a wide margin, followed by Bachelors_Rate, with Total_Population mattering the least.

This genuinely surprised me, because it does not match what the correlation heatmap suggested earlier. On its own, Median_Income had almost no linear correlation with home value, while Bachelors_Rate showed a much stronger simple correlation. The explanation is that income's relationship with home value is not a straight line nationally. A high income county on the coast and a high income county in a lower cost region can have very different home values, so a simple correlation washes that out. Random Forest can split income into different ranges and combine it with the other features, which lets it pick up on that pattern in a way a linear correlation cannot.

The actual versus predicted plot shows the model tracking real home values closely for most counties, but it struggles the most with the most expensive counties, consistently underpredicting some of them by over $400,000 and overpredicting a few others by a similar amount. The residual plot confirms this. Most residuals cluster close to zero, but the spread widens noticeably at the high end, which tells me the model is missing something about what drives home values in the most expensive housing markets, almost certainly factors like land scarcity, coastal location, or local demand that are not captured by income, population, or education alone.

What I can conclude is that income and education together explain a meaningful share of county level home value variation, but they are clearly not the whole story, especially at the extremes. Local market conditions, housing supply, and geography almost certainly matter too, and this model does not see any of that.

</div>

<div class="card">

## Limitations, Ethics, and Reflection

This dataset has real gaps. It only captures income, population, and education, and says nothing about housing supply, zoning, local amenities, or regional cost of living differences, all of which genuinely drive home values. A model like this could be misleading if used on its own for something like loan risk assessment, since it would systematically miss counties where home values are high or low for reasons unrelated to income or education, such as tourist destinations or areas with strict zoning.

If this kind of model were used in a real decision making context, like flagging counties as overvalued or undervalued for investment, a false prediction could unfairly steer resources away from or toward a community based on an incomplete picture. I wouldn't consider this model appropriate for real financial decision making as is. It's better suited as an exploratory tool for understanding broad national patterns, not as a decision engine for any single county.

If I extended this project, I would want to add housing supply data, regional cost of living indexes, and maybe historical trend data instead of a single snapshot year, since none of that is currently represented.

</div>

<div class="card">

## Code and Transparency

Full notebook: [View on GitHub](ml_project.ipynb)

Data source: U.S. Census Bureau, American Community Survey 5 Year Estimates (2023), accessed via the [Census API](https://www.census.gov/data/developers/data-sets/acs-5year.html).

AI Usage Disclosure: I used Claude (Anthropic) throughout this project to help debug API authentication and environment setup issues, structure the data cleaning and modeling pipeline, and format and style the visualizations. I also used it to help organize this write up. All research question framing, feature selection reasoning, model selection logic, and interpretation of results reflect my own analysis and decisions.

</div>
