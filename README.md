# Macro Market Direction Model
### Bank of Uganda, Petroleum Revenue Investment Reserve (PRIR)

**Author:** Ibaale Eric   
**Data Period:** January 1990 to December 2024  
**Language:** R (R Markdown)

---

## Project Overview

This project builds a **Macro Market Direction Model** for the  Bank of Uganda, Petroleum Revenue Investment Reserve. The fund holds a diversified portfolio made up of 40% Global Stocks, 30% Government Bonds, 20% Corporate Bonds, and 10% Emerging Market assets.

The model predicts, one month in advance, whether the next month will be **favorable** (portfolio return above historical median) or **unfavorable** (portfolio return at or below median). This is a binary classification problem solved using **logistic regression** in R.

The analysis draws on monthly macroeconomic and financial market data sourced from Bloomberg, covering major market regimes including the 2008 Global Financial Crisis, the 2020 COVID crash, and the 2022 inflation and rate hike cycle.

---

## Repository Structure

```
bou-macro-market-model/
│
├── BOU_Macro_Market_Direction.html      # Main analysis file
├── Book1.xlsx                          # Bloomberg dataset (1990 to 2024)
└── README.md                           # This file
```

---

## Dataset Description

The Excel workbook `Book1.xlsx` contains 420 monthly observations spanning January 1990 to December 2024. The first six rows contain Bloomberg metadata and are skipped during loading. The 10 market indicators in the dataset are:

| Variable | Full Name | Role |
|---|---|---|
| VIX | CBOE Volatility Index | Market fear gauge |
| FDTR | Federal Funds Target Rate | US monetary policy |
| DXY | US Dollar Index | Dollar strength |
| USGG10YR | US 10-Year Treasury Yield | Global benchmark rate |
| MXEF | MSCI Emerging Markets Index | EM equity performance |
| SPX | S&P 500 Index | Global equity sentiment |
| HYG | iShares High Yield Bond ETF | Corporate credit conditions |
| EURUSD | Euro to US Dollar Rate | Major currency pair |
| USGG2YR | US 2-Year Treasury Yield | Short-term policy expectations |
| CL1 | WTI Crude Oil (Front Month) | Commodity cycle indicator |

> **Note:** The original assignment specification includes 15 variables. Five of them (NAPMPMI, MOVE, LQD, GDBR10, EMBI+) were not available in the provided dataset and are documented as a limitation in the report.

> **Note on HYG:** The HYG ETF was launched in 2007, so this variable contains missing values for all months before that date. The analysis handles this by substituting the government bond return proxy for those periods.

---

## Methodology

### Feature Engineering

Five derived variables are created from the raw data to capture economic dynamics that raw levels alone may miss:

| Derived Variable | Formula | Economic Meaning |
|---|---|---|
| Credit Spread | USGG10YR minus HYG proxy | Measures credit stress in the market |
| Yield Curve Slope | USGG10YR minus USGG2YR | Signals economic growth expectations |
| VIX Change | VIX minus lagged VIX (1 month) | Captures volatility momentum |
| Dollar Strength | DXY minus lagged DXY (3 months) | Tracks currency momentum |
| Oil Dollar Ratio | CL1 divided by DXY | Reflects commodity and currency relationship |

### Target Variable Construction

The portfolio return is computed each month as a weighted combination of asset class returns:

```
Portfolio Return = 0.40 * SPX Return
                + 0.30 * Bond Proxy Return (inverted USGG10YR)
                + 0.20 * HYG Return
                + 0.10 * MXEF Return
```

Each month is then labelled:

- **1 (Favorable)** if the monthly return exceeds the historical median
- **0 (Unfavorable)** if the monthly return is at or below the historical median

### Model Setup

| Item | Detail |
|---|---|
| Model type | Binary logistic regression via `glm()` |
| Dependent variable | Favorable (0 or 1) |
| Independent variables | Top 8 predictors ranked by absolute correlation with target |
| Training period | January 1990 to December 2019 |
| Test period | January 2020 to December 2024 |
| Classification threshold | 0.5 (probability above 0.5 = Favorable) |

---

## Analysis Steps

The R Markdown report walks through the following stages in order:

1. **Data Loading and Preparation**: Reading the Excel file, skipping metadata rows, converting types, and renaming columns for clarity.

2. **Missing Value Assessment**: Counting and explaining gaps in the data, particularly for HYG.

3. **Summary Statistics**: Mean, standard deviation, minimum, and maximum for all 10 variables.

4. **Time Series Visualisation**: Line plots for all variables from 1990 to 2024 showing major market events.

5. **Feature Engineering**: Creating the five derived variables described above.

6. **Target Variable Construction**: Building the weighted portfolio return and labelling favorable and unfavorable months.

7. **Correlation Analysis**: Ranking predictors by their correlation with the target, and checking for multicollinearity among predictors.

8. **Time Period Analysis**: Splitting the data into Pre-2008, Post-Crisis (2009 to 2019), and Recent (2020 to 2024) sub-periods to assess stability of relationships.

9. **Crisis Period Analysis**: Examining how variables behaved during the 2008 Global Financial Crisis, 2020 COVID crash, and 2022 inflation shock.

10. **Logistic Regression Model**: Fitting the model, interpreting coefficients and odds ratios, and assessing statistical significance.

11. **Model Evaluation**: Confusion matrix, accuracy, precision, recall, F1 score, and ROC curve with AUC for both training and test sets.

12. **Business Insights**: Plain-language investment commentary explaining what the model means for portfolio management decisions.

---

## R Packages Required

Install all dependencies before knitting the report:

```r
install.packages(c(
  "readxl",
  "dplyr",
  "tidyr",
  "lubridate",
  "ggplot2",
  "gridExtra",
  "corrplot",
  "reshape2",
  "caret",
  "pROC",
  "knitr",
  "kableExtra"
))
```

---

## How to Run

1. Clone or download this repository.

2. Place `Book1.xlsx` and `BOU_Macro_Market_Direction.Rmd` in the same working directory.

3. Open `BOU_Macro_Market_Direction.Rmd` in RStudio.

4. Install the required packages listed above if you have not already done so.

5. Click **Knit** and select **Knit to HTML** to generate the full report.

The knitted HTML report will appear in the same folder as the `.Rmd` file.

---

## Key Findings

Based on the model results:

**Most Important Predictors**

The VIX (market fear index) is the single most powerful predictor. When VIX rises sharply, the portfolio tends to perform poorly across all four asset classes due to broad risk-off sentiment. The S&P 500 level, Yield Curve Slope, and Dollar Strength (DXY) are also consistently significant.

**Economic Logic**

The relationships uncovered by the model align with established financial theory:

- Rising VIX hurts equities and emerging markets through capital flight and risk aversion.
- An inverted or flat yield curve signals economic slowdown and is bad for the bond allocation.
- A strong dollar increases the cost of dollar-denominated debt in emerging markets and tightens global financial conditions, hurting both the equity and EM allocations.
- Strong equity momentum (rising SPX) benefits the 40% equity allocation directly and typically reflects broad risk appetite that lifts all asset classes.

**Model Limitations**

- HYG data is only available from 2007, creating gaps in the credit-related features for the earlier training period.
- Logistic regression assumes a linear relationship between predictors and log-odds, which may miss non-linear market dynamics.
- Five of the originally specified 15 variables were not available in the dataset.
- The model should be used as a decision support tool alongside qualitative judgement, not as a standalone trading signal.

---

## Variables to Monitor Monthly

Based on the model, the investment team should track these indicators each month:

1. VIX level and month-over-month change
2. Yield curve slope (10-year minus 2-year US Treasury yield)
3. US Dollar Index (DXY) trend and 3-month momentum
4. S&P 500 momentum
5. MXEF (Emerging Market equity index) trend

---

## License

This project was developed as an academic and professional internship exercise for the Bank of Uganda Petroleum Investment Fund. The dataset is proprietary Bloomberg data and is not licensed for redistribution.

---

*Report prepared by: **Ibaale Eric***  
