# [Rehab Tool] Dual Dx Screener — Week 28

**Date:** 2026-07-08 | **ISO Week:** 28 | **Topic:** Dual Diagnosis Screener (Index 3 of 12)
**Target User:** Rehab liaison nurses, access coordinators, intake clinicians

---

## Overview

Patients referred for inpatient rehab frequently carry psychiatric or substance use co-morbidities that affect admission readiness, program intensity, behavioral management, and care coordination. Without systematic pre-admission screening, these factors get missed — leading to inappropriate admissions, early discharges, or preventable safety events.

The **MFB Dual Dx Screener** provides structured behavioral health complexity triage at the PAS or day-of-admit stage. It outputs a risk tier (STANDARD / MONITOR / ELEVATED / COMPLEX), flags specific concerns, generates recommended pre-admission actions, and produces a ready-to-paste Epic SmartPhrase note.

---

## Tool Design

| Element | Detail |
|---|---|
| **Problem** | Undetected BH complexity at intake → unsafe or failed admissions |
| **User** | Liaison nurse, access coordinator, intake clinician |
| **Context** | PAS evaluation, day-of-admit review, pre-transfer triage |
| **Inputs** | Psych history, SUD, withdrawal risk, SI status, psych admit recency, behavioral incidents, psych consult status, cognitive factors, psychotropic meds, support system, engagement |
| **Logic** | Any red flag → COMPLEX; 3+ yellow flags → ELEVATED; 1–2 yellow → MONITOR; none → STANDARD |
| **Output** | Risk tier, flagged items, recommended pre-admission actions, Epic SmartPhrase |
| **Downstream** | Flag to charge nurse + medical director; request psych consult; delay transfer if red flags present |

**Red-flag triggers (any one → COMPLEX):**
- Active suicidal ideation / recent attempt
- High-risk or active withdrawal (CIWA/COWS concern)
- Inpatient psych hospitalization < 30 days ago
- Physical aggression during acute stay
- Required seclusion or physical restraint

---

## Epic SmartPhrase Pairing

**Trigger:** `.MFBDUALDXSCREEN`

Pair with:
- `.MFBPAS` (pre-admission screening note)
- `.MFBDAYOFADMIT` (day-of-admit checklist)
- Epic BH FYI flag on patient header when tier = ELEVATED or COMPLEX

**Workflow integration suggestion:**
> Run screener → copy SmartPhrase output → paste into liaison encounter note in Epic → mark "BH complexity" flag on access tracker → notify charge nurse for any ELEVATED or COMPLEX result.

---

## HTML Tool — Standalone File

See attached `2026-07-08_dual-dx-screener.html`.

- Single self-contained file (no CDN, no external dependencies)
- Mobile-responsive + print-friendly
- MFB navy/gold color system throughout
- "Copy as Epic SmartPhrase" and "Reset" buttons
- Persistent PHI banner (sticky, top of page)
- No PHI storage (zero localStorage use)
- Five input sections: Psych History · SUD · Current Behavioral Status · Behavioral Incidents · Support & Readiness

---

## Testing Checklist

- [ ] **STANDARD:** Check all "none/no known" options → verify STANDARD tier and minimal actions
- [ ] **COMPLEX — SI:** Select "Active SI or recent attempt" → verify RED flag, COMPLEX tier, mandatory psych-clearance actions
- [ ] **COMPLEX — Aggression:** Select "Physical aggression" → verify RED flag and 1:1 monitoring action
- [ ] **COMPLEX — Withdrawal:** Select "High / Active CIWA/COWS" → verify RED flag and delay-transfer action
- [ ] **COMPLEX — Recent psych admit:** Select "< 30 days ago" → verify RED flag and discharge-summary action
- [ ] **ELEVATED:** Select depression + alcohol SUD + limited support + 3+ psychotropic meds → verify ELEVATED tier with multiple yellow flags
- [ ] **SmartPhrase:** Calculate any combination → "Copy as Epic SmartPhrase" → paste into text editor, verify `.MFBDUALDXSCREEN` header and all data fields populated
- [ ] **Reset:** Click Reset → all inputs cleared, results hidden, scroll to top
- [ ] **Print:** Trigger window.print() → PHI banner visible, buttons hidden, results legible
- [ ] **Mobile (375px):** Check-groups collapse to single column; radio buttons wrap cleanly; tier banner readable

---

## Next-Iteration Ideas

1. **Numeric severity score:** Weighted point values per item (e.g., schizophrenia +2, BPD +1.5, historical SI +0.5) for a quantitative score alongside the tier
2. **Discharge planning linkage:** Auto-flag "requires outpatient BH linkage" to feed into the Discharge Readiness tool (Week 6)
3. **AUDIT-C / DAST-10 inline mini-screen:** Quick substance use questionnaire for when chart documentation is sparse
4. **Secure messaging template:** Auto-generate a brief team notification — "Re: arriving [DATE] — COMPLEX dual dx: [flags]" — for the Epic In Basket or Tiger Connect
5. **MAT medication capture:** When opioid/benzo dependence is flagged, prompt a specific MAT medication + dose field that flows to the SmartPhrase
6. **Aggregate CSV export:** "Export row" button outputs date + tier + flag count as a CSV row for tracking screening patterns over time without any PHI

---

```html
[FULL STANDALONE HTML — SEE ATTACHED FILE 2026-07-08_dual-dx-screener.html]
```

---

*MFB Rehab Tool Builder — auto-generated 2026-07-08*
*Next: Week 29 · Topic 4 — Ventilator Transfer Readiness*
