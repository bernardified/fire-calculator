# FIRE Calculator (Singapore)

A single-file, browser-based FIRE planner built around Singapore's CPF and SRS rules. It projects your liquid portfolio, SRS and CPF accounts (OA / SA / MA / RA) month by month from today to age 95. It then solves for the liquid portfolio you need on retirement day under four scenarios.

- **File:** `index.html` (everything is self-contained; the only external dependency is Chart.js 4.4.1 from cdnjs)
- **Run:** open `index.html` in any modern browser. No server or build step is needed.
- **Save:** the 💾 Save button stores your inputs in browser `localStorage` under the key `fire_calc_profile`. Saved data lives only in that browser on that machine.

---

## Scenarios: spend level × part-time work

|                  | No work           | With part-time work  |
|------------------|-------------------|----------------------|
| **Base spend**   | 🌱 Regular FIRE   | ☕ Barista FIRE      |
| **Chubby spend** | 🍾 Chubby FIRE    | 💼 Chubby Barista    |

The four scenarios share everything except two things: the monthly spend after retirement, and whether part-time income (with its CPF effects) is switched on. Accumulation, SRS, CPF, mortgage, lump sums and growth rates are identical across all four.

**Regular FIRE is the planning number.** It answers "can I stop working entirely?" The part-time scenarios show how much work income reduces the target. Treat them as upside, not as the base case.

Each card shows:

- **FIRE number:** the minimum liquid portfolio at the start of the retirement year that never goes negative through age 95.
- **On track / Shortfall:** your projected liquid at the start of the retirement year minus that FIRE number. The on-track flag comes from the full simulation itself, not from the gap.

---

## Inputs

### Personal
| Field | Notes |
|---|---|
| Date of birth | Sets every age-based rule. Age is computed as `year − birth year`. |
| Target retirement year | Salary, SRS contributions and mortgage cash from savings stop at the start of this year. |

### Current assets
| Field | Notes |
|---|---|
| Portfolio growth rate | Annual return on the liquid portfolio (default 7%). This is the biggest lever on every FIRE number. |
| Liquid assets | Investments and cash. |
| SRS balance | Current SRS balance. |
| SRS growth rate | Blended return on the whole SRS balance (default 5%). Weight it down for any uninvested SRS cash, which earns about 0.05%. |
| SRS withdrawal age | Locked to the statutory retirement age when you **first contributed**: before 1 Jul 2022 → 62; 1 Jul 2022 – 30 Jun 2026 → 63; from 1 Jul 2026 → 64. |
| CPF OA / SA / MA balances | Current balances. MA is needed for the Basic Healthcare Sum (BHS) overflow logic. |

### Monthly contributions (pre-retirement)
| Field | Notes |
|---|---|
| Monthly liquid investment | What you add to the liquid portfolio each month. |
| Monthly SRS contribution | Warns if the annual total exceeds the $15,300 cap. |
| Stop SRS contribution year | The **first year with no contributions**. Defaults to the retirement year. |
| Monthly gross salary | **The only CPF input.** All CPF contributions are calculated from it (see CPF rules below). Held flat in nominal terms until retirement. |
| Monthly mortgage from OA | Fixed nominal amount. Paid from OA first; any shortfall comes from cash. |
| Mortgage end year | Defaults to the year you turn 64. |

### Retirement spending and part-time work
| Field | Notes |
|---|---|
| Base monthly spend | Today's dollars, inflated at 3%/yr. Used by Regular and Barista. |
| Chubby monthly spend | Today's dollars, inflated at 3%/yr. Used by Chubby and Chubby Barista. |
| Part-time work type | **Employee** or **Self-employed** (see below). |
| Part-time monthly income | Today's dollars, inflated at 3%/yr. For an employee, enter the gross wage. For self-employed, enter net trade income (commissions minus business expenses) as a monthly average. |
| Stop part-time work year | Income is $0 after this year. |

### CPF LIFE plan
| Plan | Payout model |
|---|---|
| Standard | 7.5% of RA per year, fixed |
| Escalating | 6.0% of RA per year to start, rising 2%/yr |
| Basic | 6.5% of RA per year, fixed |

### Lump sums
These are one-off events applied in **January** of the given year. Positive amounts are injections; negative amounts are withdrawals. Each lump sum has a destination:

| Destination | Behaviour |
|---|---|
| **Cash** | Goes into the liquid portfolio. |
| **CPF OA** | Goes into OA before 55. Use it for a CPF refund from a property sale or a SAVER CPF Top-Up transfer. From 55, it goes to cash, because the model treats OA as withdrawable after the RA is formed. |
| **CPF SA** | A cash top-up, capped at the current year's FRS. From 55 to 64 it goes to RA, capped at the ERS (2 × FRS at 55). From 65, it goes to cash. Any excess over a cap stays as cash, and a warning appears in the milestones. |

Withdrawals (negative amounts) always come from cash, whatever the destination.

---

## How the engine works

The simulation runs monthly from January of the current year to December of the year you turn 95. Each month runs in this order:

1. **One-time events (in January)**
   - **Age 55:** OA + SA form the RA, up to the FRS at 55. Anything above the FRS goes to liquid (withdrawable). SA is closed.
   - **SRS withdrawal age:** the SRS balance is amortised into 120 equal monthly draws, using the SRS growth rate.
   - **Age 65:** RA converts to CPF LIFE using the selected plan's payout rate. Escalating payouts rise 2% each following January.
2. **Mortgage:** paid from the OA balance first. Any shortfall is paid in cash from liquid, both before and after retirement. The first year this happens is reported as "OA Runs Out".
3. **Cash flows**
   - **Before retirement:** add the monthly liquid investment, add the SRS contribution (until the stop year), and credit CPF from salary.
   - **After retirement:** liquid pays spending (inflated) plus any mortgage shortfall, minus income. Income is SRS draws + CPF LIFE + part-time take-home pay (part-time scenarios only). Part-time CPF is credited as described below.
4. **Lump sums (in January),** routed by destination.
5. **Growth,** compounded monthly from the annual rates:

   | Account | Annual rate |
   |---|---|
   | Liquid | Portfolio growth rate |
   | SRS | SRS growth rate |
   | OA | 2.5% |
   | SA, RA, MA | 4.0% |
   | Inflation | 3.0% |

6. **MediSave overflow:** any MA balance above the BHS spills over. Before 55 it goes to SA up to the current FRS, then to OA. From 55 to 64 it goes to RA up to the FRS at 55, then to liquid. From 65, the overflow is assumed to be used up by MediShield Life and Integrated Shield premiums, so it isn't credited anywhere.

### CPF rules (2026)

**Salary CPF.** Contributions are calculated on salary up to the **$8,000/month Ordinary Wage ceiling**:

| Age | Employee | Employer | OA | SA / RA | MA | Source |
|---|---|---|---|---|---|---|
| ≤ 35 | 20% | 17% | 23% | 6% | 8% | Official |
| 36 – 45 | 20% | 17% | 21% | 7% | 9% | Official |
| 46 – 50 | 20% | 17% | 19% | 8% | 10% | Official |
| 51 – 55 | 20% | 17% | 15% | 11.5% | 10.5% | Official |
| 56 – 60 | 18% | 16% | 12% | 11.5% | 10.5% | Official totals; split approximate |
| 61 – 65 | 12.5% | 12.5% | 3.5% | 11% | 10.5% | Official totals; split approximate |
| 66 – 70 | 7.5% | 9% | 1% | 5% | 10.5% | Official totals; split approximate |
| > 70 | 5% | 7.5% | 1% | 1% | 10.5% | Official totals; split approximate |

All rates are percentages of wages. For ages above 55, the OA / RA / MA split is approximate: it assumes MA stays around 10.5% and that the 2026 rate increases go to RA.

**Routing of contributions:**
- **Before 55:** contributions go to OA, SA and MA. **SA contributions keep going into SA even once SA is above the FRS.**
- **55 to 64:** the RA share goes to RA up to the FRS at 55. The rest, and the OA share, go to liquid.
- **65 and above:** the RA share raises the CPF LIFE payout by `amount × payout rate ÷ 12`.

**Part-time CPF**

- **Employee:** full CPF on the wage, using the table above and the $8,000 ceiling. The employee share is deducted, so only take-home pay offsets spending. Both shares are credited to your accounts.
- **Self-employed:** compulsory MediSave only, paid out of your own income. There is no employer share and nothing goes to OA or SA.

| Annual net trade income | < 35 | 35 – 44 | 45 – 49 | 50+ |
|---|---|---|---|---|
| ≤ $6,000 | 0 | 0 | 0 | 0 |
| $6,000 – $12,000 | 4% | 4.5% | 5% | 5.25% |
| $12,000 – $18,000 | linear phase-in (CPF formula) | " | " | " |
| > $18,000 | 8% | 9% | 10% | 10.5% |
| Annual cap | rate × $96,000 | " | " | " |

**Retirement sums**

| Sum | Model |
|---|---|
| FRS | $213,000 (2025), grown 3.5%/yr. Each year's current FRS is used for the SA overflow and SA top-up caps; the FRS at 55 is used for the RA. |
| ERS | 2 × FRS at 55 |
| BHS | $79,000 (2026), assumed to grow 4.5%/yr until the year you turn 65, then fixed for life |

### SRS
- Contributions go in monthly until the stop year and grow at the SRS rate.
- From January of the withdrawal-age year, the balance is paid out as 120 equal monthly draws, amortised at the SRS rate. The final draw falls in the year you reach withdrawal age + 9.
- **SRS Tax Check** milestone: takes 50% of the annual SRS draw, adds part-time taxable income if part-time work overlaps the draw window, and flags when the total exceeds the $20,000 band taxed at 0%. It ignores personal reliefs, so it errs on the cautious side. CPF LIFE payouts are tax-exempt and are excluded.

### FIRE number
For each scenario, the engine saves the full account state at 1 January of the retirement year. It then re-runs **the same simulation** from that state with different starting liquid values, and binary-searches (40 iterations, rounded up to the nearest $1,000) for the smallest liquid balance that never goes negative in any month through age 95.

Because it uses the same engine, the FIRE number can't drift out of step with the year-by-year table.

---

## Outputs

- **FIRE cards (2×2):** the FIRE number and on-track or shortfall figure for each scenario.
- **Info boxes:**
  - projected liquid at the start of retirement
  - FRS at 55
  - estimated CPF LIFE payout, with and without part-time work
  - part-time income at retirement, gross or NTI and take-home, adjusted for inflation
- **Chart:**
  - **Trajectory:** liquid balance for all four scenarios, with each FIRE target shown as a dashed line.
  - **Asset Breakdown:** stacked Liquid / SRS / OA / SA / MA / RA for the scenario selected in the table tabs.
- **Key milestones:**
  - retirement
  - OA running out (with and without part-time work)
  - SRS start, SRS tax check and final SRS draw
  - the age 55 RA formation
  - CPF LIFE start
  - mortgage end
  - lump-sum cap warnings
- **Year-by-year table:** one tab per scenario. Shows key years plus every 5th year. Columns: Liquid, SRS, OA, SA, MA, RA, Total NW, Liquid Draw and Other Income.

---

## Known limitations and simplifications

- **Salary** is flat in nominal terms. Bonuses (Additional Wages, such as the 13th month and AVC) and the $102k annual wage ceiling aren't modelled.
- **SAF SAVER Plan:** any CPF Top-Up Account balance, and the SAVER Retirement Account payout, must be entered manually as lump sums. Use CPF OA for the Top-Up transfer and Cash for the payout.
- **CPF extra interest** (+1% on the first $60k, +2% on the first $30k after 55) isn't modelled, so CPF growth is slightly understated.
- **After 55**, OA is treated as fully withdrawable to liquid. In reality, withdrawals above the FRS depend on your property pledge and BRS arrangements.
- **CPF LIFE** is simplified to a flat payout rate on the RA at 65. Real payouts depend on cohort, gender and CPF's actuarial tables.
- **Ceilings and caps** (the $8,000 wage ceiling, $15,300 SRS cap and $20k tax band) are held flat in nominal terms forever.
- **Part-time income** is treated as a smooth monthly figure. Commissions are lumpy in reality. Income tax on part-time earnings isn't deducted. Voluntary CPF contributions by self-employed people aren't modelled.
- **Mortgage** is a fixed nominal payment; interest-rate changes aren't modelled.
- **Returns** are deterministic, with no sequence-of-returns risk. A FIRE number from a flat 7% return is a point estimate, not a safe withdrawal guarantee.
- **Healthcare** costs beyond MA overflow absorption aren't modelled separately. They are assumed to sit inside the spend inputs.

---

## Annual maintenance checklist

Update these constants when CPF, IRAS or MOM publish new figures, usually in January:

| Constant / location | Current value |
|---|---|
| `CPF_OW_CEILING` | 8,000 |
| `BHS_2026` / `BHS_GROWTH` | 79,000 / 4.5% |
| `frs2025` in `getParams()` | 213,000 (grown 3.5%/yr) |
| `cpfRates()` | 2026 contribution and allocation table |
| `seMedisaveAnnual()` | 2026 self-employed MediSave rates |
| SRS withdrawal-age dropdown | 62 / 63 / 64; add 65 once the retirement age rises to 65 (legislated by 2030) |
| CPF LIFE payout rates in `getParams()` | 7.5% / 6.0% / 6.5% |
| SRS cap check | $15,300 |

---

## Changelog

### 2026-09-24
- **SRS:** growth rate and withdrawal age are now inputs (previously a fixed 5% and age 62). Added the SRS tax check and the cap warning. Fixed the stop-year off-by-one.
- **CPF:** manual OA/SA contribution inputs replaced with a single salary input. CPF is calculated from 2026 rates with the $8k wage ceiling. Added an MA balance and BHS overflow routing.
- **Scenarios:** Barista/Chubby replaced with a 2×2 grid (spend level × part-time work), with four FIRE numbers, table tabs and chart lines.
- **Lump sums:** added a destination of Cash, CPF OA or CPF SA, with FRS and ERS caps.
- **Mortgage:** any shortfall after OA runs out is now paid from liquid. Previously it disappeared.
- **Part-time CPF:** added the employee and self-employed modes.
- **FIRE number:** now re-runs the main engine from the retirement-day state, replacing the old duplicated post-retirement loop.
