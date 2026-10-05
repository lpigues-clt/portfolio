# Predicting County Home Values from Income and Education

<div class="card">

## Problem Definition

I wanted to look at something with real stakes: can a county's median home value be predicted just from how much people earn and how educated the population is? This is a **regression problem** — I'm predicting a continuous number (median home value in dollars), not a category.

The target variable is **Median_Home_Value**. People who might actually care about this: lenders assessing risk, local policymakers trying to understand housing affordability pressure, and real estate analysts looking at which counties are undervalued relative to their income and education levels.

I picked this because income and home value get talked about together constantly, but I wanted to actually test how much of the story they tell on their own, and where that story breaks down.

</div>

<div class="card">

## Background and Context

Housing economics research consistently points to income and education as two of the strongest predictors of home value, but the relationship isn't as clean as it sounds. Homeowners with a bachelor's degree have historically earned significantly more and owned more expensive homes than those without one (Florida Realtors, 2025), which suggested education might carry a meaningful signal here beyond income alone.

At the same time, housing economists have found that income and education variables improve local assessment accuracy but still leave real variation unexplained, since local market dynamics and non-economic factors matter too (as cited in work on property tax assessment accuracy). County-level median home values also vary enormously nationwide, from under $100,000 to well over $800,000 depending on region (National Association
