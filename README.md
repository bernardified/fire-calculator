# FIRE Calculator — Singapore Edition

An interactive, browser-based FIRE (Financial Independence, Retire Early) calculator built for Singapore-specific mechanics: CPF OA/SA/RA, SRS, CPF LIFE, and HDB/condo mortgage.

## Live Demo

Hosted on GitHub Pages: `https://bernardified.github.io/fire-calculator`

## Features

- **Two FIRE scenarios** — Barista FIRE (part-time income) and Chubby FIRE (full lifestyle spend, no work)
- **FIRE Number calculator** — binary search finds the minimum liquid portfolio needed at retirement for each scenario
- **Monthly compounding simulation** — year-by-year projection table through age 85
- **CPF mechanics** — OA/SA merge to RA at 55, FRS cap with excess flowing to liquid, CPF LIFE from 65
- **SRS mechanics** — contributions until a configurable stop year, PMT-amortised withdrawal over 10 years from age 62
- **Lump sums** — add one-off injections or withdrawals at any future year
- **Chart visualisation** — NW trajectory (Barista vs Chubby) and stacked asset breakdown
- **Save / Load / Clear profile** — persists your inputs in `localStorage` so you can return later
- **Shareable** — opens with empty fields by default; friends fill in their own numbers

## CPF Assumptions

| Rule | Value |
|------|-------|
| OA growth | 2.5%/yr |
| SA / RA growth | 4.0%/yr |
| FRS base (2025) | S$213,000 |
| FRS escalation | 3.5%/yr |
| CPF LIFE — Standard | 7.5% of RA/yr |
| CPF LIFE — Escalating | 6.0% start, +2%/yr each Jan |
| CPF LIFE — Basic | 6.5% of RA/yr |
| Mortgage ends | Age 65 |

## Other Assumptions

| Item | Value |
|------|-------|
| Liquid portfolio growth | 5.0%/yr |
| SRS growth | 5.0%/yr |
| Expense inflation | 3.0%/yr |
| SWR reference | 3.5% |
| SRS withdrawal period | 10 years (from age 62) |

## Inputs

| Field | Description |
|-------|-------------|
| Date of Birth | Used to compute ages for CPF events |
| Retirement Year | When accumulation stops and drawdown begins |
| Current Liquid Assets | Investable portfolio today (SGD) |
| Current SRS Balance | SRS account balance today |
| CPF OA / SA | Current OA and SA balances |
| Monthly Liquid Investment | New money added to liquid portfolio pre-retirement |
| Monthly SRS Contribution | Typically S$1,275/month (annual cap S$15,300) |
| Stop SRS Contribution Year | Year to stop SRS top-ups (defaults to retirement year) |
| Monthly CPF OA / SA Contributions | From employment (employer + employee) |
| Monthly Mortgage from OA | Instalment deducted from OA |
| Monthly Lifestyle Spend | Barista base expenses in today's dollars |
| Chubby Monthly Spend | Higher spend for Chubby FIRE scenario |
| Barista Part-time Income | Offset income in today's dollars |
| Stop Barista Income Year | Year part-time income goes to zero |
| Lump Sums | One-off injections (+) or withdrawals (−) at a specific year |

## Usage

1. Open `index.html` in any modern browser (no server required)
2. Fill in your details and click **Calculate**
3. Review the FIRE numbers, projection table, and charts
4. Click **Save Profile** to persist your inputs for next time

## Deploying to GitHub Pages

1. Push `index.html` to the root of a GitHub repo
2. Go to **Settings → Pages → Source: main / root**
3. Your calculator is live at `https://<username>.github.io/<repo>/`

No build step, no dependencies to install — it's a single self-contained HTML file.
