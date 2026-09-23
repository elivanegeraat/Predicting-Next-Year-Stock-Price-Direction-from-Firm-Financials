# Predicting Next-Year Stock Price Direction from Firm Financials

**University assignment:** this project was my first assignment for the Introduction to Big Data in Economics module (ECU33143) at Trinity College Dublin. It was completed and submitted as coursework, and the full research report I submitted is included in this repository.

For the assignment, I built a random forest classifier to test whether US firms' accounting data and consumer sentiment in one year can predict whether their stock price rises or falls the following year. Most of the work is in the data engineering: turning raw SEC filings into a clean firm-year dataset linked to stock prices.

## Question

Can firm financials and economic sentiment in year t predict whether a company's stock price will be higher at the end of year t+1?

## Data

| Source | What I used it for |
|---|---|
| [SEC Financial Statement Data Sets](https://www.sec.gov/dera/data/financial-statement-data-sets) (2017Q4, 2018Q4, 2019Q4) | Accounting data from US company filings: revenues, assets, liabilities, cash flows, net income and more |
| [SEC company tickers file](https://www.sec.gov/files/company_tickers.json) | Linking each firm's SEC identifier (CIK) to its stock ticker |
| Yahoo Finance, via `yfinance` | Year-end stock prices for 2017 to 2020, used to build the target |
| [University of Michigan Consumer Sentiment Index](https://fred.stlouisfed.org/series/UMCSENT) (FRED) | A yearly measure of economic sentiment |

The SEC zip files are too large for GitHub, so they are not in this repository. They can be downloaded from the SEC link above.

## Method

**Building the firm-year panel**

- I kept only annual 10-K filings, so every observation is comparable.
- I selected 14 core accounting items and reshaped the data into one row per firm per fiscal year.
- I dropped any item missing for more than 25% of firms.

**Filling missing values**

Instead of filling gaps with a single overall median, I grouped firms into size quartiles by total assets and filled each missing value in three steps:

1. The median of firms in the same size group and year
2. If that was unavailable, the median for that year
3. As a last resort, the overall median

This means a small firm's missing values are filled using other small firms, not large ones.

**Transformations and ratios**

- I log-transformed heavily right-skewed variables such as assets and revenues.
- I built standard financial ratios, including leverage, current ratio, cash ratio, return on assets, profit and operating margins, cash flow to assets, capital intensity and interest coverage.
- I capped extreme ratio values at the 1st and 95th percentiles to limit the influence of outliers.

**Target and model**

- The target is 1 if the stock's year-end price in t+1 is higher than in year t, and 0 otherwise.
- I added yearly consumer sentiment as a standardised score.
- I trained a random forest with 500 trees on 70% of the data and tested it on the remaining 30%, keeping the share of price rises the same in both.
- I compared it with a baseline that always predicts the most common outcome in the training data.

## Results

| Model | Test accuracy |
|---|---|
| Baseline (always predict the most common outcome) | 54.5% |
| Random forest | 64.0% |

Classification report on the 222 test observations:

| Outcome | Precision | Recall | F1-score |
|---|---|---|---|
| Price fell (0) | 0.63 | 0.51 | 0.57 |
| Price rose (1) | 0.65 | 0.74 | 0.69 |

- The model correctly predicted 90 price rises and 52 price falls, but labelled 49 falls as rises and 31 rises as falls. It is better at spotting rises than falls.
- Firm size and balance sheet measures were the most important features, with leverage and consumer sentiment also contributing.
- The gain over the baseline is modest, which fits the idea that stock markets are largely, though not perfectly, efficient.

## Limitations

- The sample covers only three fiscal years in a relatively stable period, so the results may not hold in a crisis.
- The target is binary, so the model does not distinguish small price moves from large ones.
- The log transformation cannot handle negative values, which affects loss-making or distressed firms.
- Missing data may be related to firm health, which could bias the results even after imputation.
- The train/test split is random across firm-years, so the same firm and year can appear in both sets. Because consumer sentiment takes one value per year, it also tells the model which year an observation comes from, and the share of stocks that rose differed between years. Part of the accuracy gain may come from this rather than from firm fundamentals. Training on earlier years and testing on a later year would be a stricter test.

## How to run it

1. Download the 2017Q4, 2018Q4 and 2019Q4 zip files from the SEC Financial Statement Data Sets page.
2. Open the notebook in Google Colab and run the cells in order.
3. When prompted, upload the three SEC zip files, `company_tickers.json` and the UMCSENT CSV in one go.

## Files

- `stock_direction_random_forest.ipynb`: the full code
- `report.pdf`: the research report I submitted for the assignment
- `data/company_tickers.json`: CIK to ticker mapping
- `data/UMCSENT.csv`: monthly consumer sentiment, 2017 to 2019

## Tools

Python (pandas, numpy, scikit-learn, matplotlib, yfinance).
