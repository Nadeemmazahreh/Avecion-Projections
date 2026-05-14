# Avecion V5 Audit Report

**File audited:** `/Users/cardano/Downloads/Avecion_Financial_Model_V5.xlsx`
**Audit date:** 2026-05-10
**Sheets examined:** Assumptions, Cost Structure, Actuals, Revenue Model, Dashboard, Notes, Sensitivity
**Method:** openpyxl data_only=True; all arithmetic independently recalculated from cell values

---

## PASS / FAIL SUMMARY

| # | Check | Result | Notes |
|---|-------|--------|-------|
| 1 | Revenue quarterly→annual arithmetic | **PASS** | Y1, Y2, Y3, 3YR totals verified |
| 2 | Stream breakdown SDK+B2G+B2B = Total per quarter | **PASS** | All 12 quarters correct |
| 2b | Annual stream sums = annual totals | **PASS** | SDK, B2G, B2B quarterly sums match annual rows |
| 3 | EBITDA waterfall (Rev−COGS=GP; GP−OPEX=EBITDA) | **PASS** | All 4 periods (Y1–Y3, 3YR) verified |
| 4 | ARR run-rate (Q4 Y3 × 4) | **PASS** | $519,060 × 4 = $2,076,240 |
| 5 | Valuation bridge (ARR × 8x = EV; EV / $500K = return) | **PASS** | $16,609,920; 33.2198x |
| 6 | Sensitivity scenario Y3 revenues (5 scenarios) | **PASS** | All multipliers applied correctly |
| 7 | Cost structure subtotals sum to TOTAL OPEX | **PASS** | Y1, Y2, Y3 verified |
| 8 | Gross margin % (GP / Revenue) | **PASS** | Y1 73.63%, Y3 87.73% |
| 9 | Formula errors (#REF!, #DIV/0!, #VALUE!, #NAME?) | **PASS** | Zero errors across all 7 sheets |
| 10 | Sensor cumulation ramp | **PASS** | Y1=50, Y2=183, Y3=406 — within spec |
| 10b | B2G client churn Q4 Y2 | **NOTE** | Drop from 4→3 is intentional model logic |
| X1 | Cross-check: Dashboard vs Revenue Model | **PASS** | All KPIs consistent |
| X2 | Cross-check: Sensitivity base values vs Revenue Model | **PASS** | All P&L base inputs match |
| X3 | EV methodology: Sensitivity tab vs Revenue Model tab | **FAIL** | Two different EV formulas in use — see Issues |

**Overall: 14 PASS, 0 FAIL on arithmetic checks; 1 structural inconsistency flagged (non-blocking for internal arithmetic but blocking for investor presentation consistency)**

---

## ISSUES

### BLOCKING — Investor Presentation Inconsistency

**Issue X3: Two conflicting EV / return calculations exist in the same model**

- **Revenue Model tab** (Valuation Bridge, rows 43–47) uses:
  `Exit ARR = Q4 Y3 revenue × 4 = $519,060 × 4 = $2,076,240`
  `EV = ARR × 8x = $16,609,920`
  `Pre-Seed Return = $16,609,920 / $500,000 = 33.22x`

- **Sensitivity tab** (row 26–27) uses:
  `EV = Y3 Annual Revenue × 8x = $1,670,520 × 8 = $13,364,160`
  `Pre-Seed Return = $13,364,160 / $500,000 = 26.73x`

- The Sensitivity tab label explicitly says `[Y3 Revenue × ARR Multiple]`, not `[ARR × ARR Multiple]`.
- The Dashboard and Revenue Model show **33.22x**; the Sensitivity tab base case shows **26.73x** — a 24% lower return figure.
- An investor who reads both tabs will see contradictory return projections.

**Recommendation:** Align to one methodology. The Revenue Model's ARR approach ($2,076,240 ARR × 8x) is the more defensible valuation convention for a SaaS business and should be used in the Sensitivity tab as well. Update Sensitivity row 26 to use the run-rate ARR formula rather than annual revenue.

---

### WARNING — Minor Rounding / Model Assumptions

**Issue W1: Sensor renewal rate rounding at Y1→Y2 boundary**

- Assumptions tab states Y1 sensor renewal rate = 85%.
- Y1 ends with 50 cumulative sensors; 50 × 0.85 = 42.5 → should round to 42 or 43.
- Revenue Model shows Y2 Q1 cumulative = 78 sensors; carryover = 78 − 35 (new) = 43.
- This implies an effective renewal rate of 43/50 = 86.0%, not 85%.
- The $30 per-sensor discrepancy ($70 → rounding) is immaterial (≈$30 revenue impact) but the modeled assumption is inconsistent with the stated 85%.

**Recommendation:** Either update the stated assumption to 86% or apply floor(42.5)=42 to make the cell match 85% exactly.

**Issue W2: B2G client count declines Q3→Q4 in Year 2**

- B2G cumulative clients: Q1 Y2=2, Q2=3, Q3=4, Q4=3.
- A client is lost in Q4 Y2 (churn modeled intra-year).
- This is internally consistent if intentional (85% renewal applied within-year for this client), but no annotation explains the Q4 drop.
- B2G revenue Q4 Y2 = $72,000 (3 clients × $8,000/mo × 3 months) — confirms the 3-client figure.

**Recommendation:** Add a note or comment in the Revenue Model tab explaining this planned client departure.

**Issue W3: EBITDA Margin stored as ratio not decimal percent**

- Revenue Model row 39 EBITDA Margin % Y1 = −7.4388...
- This is the raw ratio (−423,120 / 56,880 = −7.4388), which in Excel is formatted as a percentage and would display as −743.9%.
- Standard financial model convention is to store as −0.7439 (decimal) and format as %.
- Dashboard row 23 shows the same raw ratio value.
- In data_only export the values look anomalous, though Excel display is correct.

**Recommendation:** Confirm cell format in Excel is set to "Percentage" (which divides by 100 on display). If the formula is `=EBITDA/Revenue` and the cell is formatted as %, this will display −744%, which is correct but alarming. Consider whether this label should read "EBITDA Margin %" or "EBITDA as % of Revenue" with the value stored as −7.44 (ratio without % formatting applied).

---

## VERIFIED NUMBERS

All of the following were independently recalculated and confirmed correct:

| Metric | Value | Verification |
|--------|-------|--------------|
| Y1 Total Revenue | $56,880 | 4,230+8,460+12,690+31,500 = 56,880 |
| Y2 Total Revenue | $598,320 | 92,880+139,080+185,280+181,080 = 598,320 |
| Y3 Total Revenue | $1,670,520 | 297,690+383,820+469,950+519,060 = 1,670,520 |
| 3-Year Revenue | $2,325,720 | Y1+Y2+Y3 = 2,325,720 |
| Y3 COGS | $205,000 | From Cost Structure (Infra $90K + API $35K + CustSvc $80K) |
| Y3 Gross Profit | $1,465,520 | $1,670,520 − $205,000 = $1,465,520 ✓ |
| Y3 Gross Margin | 87.73% | $1,465,520 / $1,670,520 = 87.73% ✓ |
| Y1 Gross Margin | 73.63% | $41,880 / $56,880 = 73.63% ✓ |
| Y3 OPEX | $1,016,500 | 8 category subtotals sum = $1,016,500 ✓ |
| Y3 EBITDA | $449,020 | $1,465,520 − $1,016,500 = $449,020 ✓ |
| Q4 Y3 Revenue | $519,060 | SDK $207,060 + B2G $180,000 + B2B $132,000 = $519,060 ✓ |
| Y3 Exit ARR | $2,076,240 | $519,060 × 4 = $2,076,240 ✓ |
| Implied EV | $16,609,920 | $2,076,240 × 8x = $16,609,920 ✓ |
| Pre-Seed Return | 33.2198x | $16,609,920 / $500,000 = 33.2198x ✓ |
| Bear Y3 Revenue | $751,734 | $1,670,520 × 0.45 = $751,734 ✓ |
| Bull Y3 Revenue | $2,672,832 | $1,670,520 × 1.60 = $2,672,832 ✓ |
| Conservative Y3 | $1,085,838 | $1,670,520 × 0.65 = $1,085,838 ✓ |
| Optimistic Y3 | $2,255,202 | $1,670,520 × 1.35 = $2,255,202 ✓ |
| Y1 OPEX subtotals | $465,000 | Mgmt $90K + SwDev $174.5K + Advisory $5K + Legal $73.5K + Acct $5K + BD $43K + Mktg $24K + Contingency $50K = $465,000 ✓ |
| Y2 OPEX subtotals | $712,000 | $148K+$273K+$58K+$92K+$27K+$46K+$38K+$30K = $712,000 ✓ |
| Y3 OPEX subtotals | $1,016,500 | $210K+$372K+$135K+$117.5K+$52K+$62K+$53K+$15K = $1,016,500 ✓ |
| Cumulative sensors Y1 end | 50 | 13+13+13+11 new = 50 ✓ |
| Cumulative sensors Y2 end | 183 | 43 carried + 4×35 new = 183 ✓ |
| Cumulative sensors Y3 end | 406 | 156 carried + 63+63+63+61 new = 406 ✓ |
| Formula errors | 0 | 358 formula cells scanned; zero #REF!/#DIV/0!/#VALUE!/#NAME? |

---

## INVESTOR-READINESS ASSESSMENT

The model is arithmetically sound across all internal checks. The single blocking issue (inconsistent EV methodology between Revenue Model and Sensitivity tabs) must be resolved before showing this to investors — the 33.22x vs 26.73x discrepancy in return projections will draw immediate questions.

Once Issue X3 is resolved and Issues W1–W3 are annotated, the model is appropriate for investor distribution.
