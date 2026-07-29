# [Rehab Tool] Bed Selection Tool — Week 31

**Date:** 2026-07-29  
**ISO Week:** 31  
**Topic:** Bed Selection Tool (Index 6 — Topic cycle mod 12)  
**Target Users:** Admissions Coordinator, Charge Nurse, Bed Management, MFB Access/Intake Team  

---

## Problem Statement

Rehab bed assignment is often done from memory or via informal hallway conversation, leading to mismatches: a bariatric patient in a standard-weight-rated room, a high-elopement-risk patient in a distant hallway, or a ventilator-dependent admission landing in a room without appropriate infrastructure. This tool standardizes the decision by collecting 7–9 clinical parameters and outputting a prioritized bed type recommendation with rationale, conflict alerts, and a pre-placement checklist — all in under 60 seconds.

---

## Tool Design

| Element | Detail |
|---|---|
| **Use Context** | During admit/transfer acceptance, when bed assignment is needed |
| **Inputs** | Weight, program, medical acuity, isolation type, mobility/lift need, gender, 7 safety/equipment flags |
| **Logic** | Priority-ordered bed type matching; conflict detection for overlapping requirements |
| **Output** | Recommended bed type + flags + rationale + pre-placement checklist + Epic SmartPhrase |
| **Downstream Action** | Charge RN confirms availability, notifies therapy, updates bed board, documents in Epic |

### Bed Type Priority Hierarchy

1. **Vent-Capable Room** — highest clinical priority  
2. **Bariatric Room** — weight ≥ 350 lbs or bariatric lift specified  
3. **Airborne Isolation Room** — negative pressure required  
4. **Private Isolation Room** — contact/droplet/protective precautions  
5. **Behavioral/Safety Room** — elopement, aggression, or self-harm flags  
6. **High-Acuity / Step-Down Capable Bed** — IV meds, frequent monitoring, telemetry  
7. **Ceiling Lift-Equipped Room** — when not already captured by bariatric  
8. **Standard Rehab Bed** — default when no flags present  

### Conflict Detection

The tool flags potentially incompatible combinations requiring charge nurse escalation:
- Vent + Bariatric (room must meet both specs)  
- Airborne isolation + Behavioral concern (observation challenge)  
- Vent + Airborne isolation (negative pressure + vent infrastructure)  
- Bariatric + Ceiling lift (weight rating verification)  

---

## Epic SmartPhrase Pairing

Copy the generated output into Epic's admission note or bed placement documentation field. Suggested SmartPhrase name: `.MFBBEDREC`

**Template structure:**
```
BED SELECTION — REHAB ADMISSION
Program: [Rehab Program]
Recommended Bed Type: [Type]
Additional Requirements: [if any]
Medical Acuity: [low/moderate/high]
Isolation: [type or none]
Behavioral/Safety Flags: [list or None]
Equipment Needs: [list or Standard]
CONFLICTS: [if any]

Rationale: [auto-generated clinical rationale]

Pre-placement checklist completed per MFB Access protocol. Charge RN notified. Bed board updated.
Clinical judgment governs final placement. Document any deviation from recommendation.
```

---

## Testing Checklist

- [ ] **Bariatric trigger:** Enter weight 350 → badge shows "Bariatric Room"; weight 349 → Standard
- [ ] **Vent flag:** Check "Ventilator dependent" → badge shows "Vent-Capable Room"
- [ ] **Priority order:** Vent + bariatric → Vent-Capable Room as primary, Bariatric Room as additional need
- [ ] **Conflict detection:** Vent + Bariatric → conflict alert appears in red box
- [ ] **Isolation:** Select "Airborne" → "Airborne Isolation Room (Negative Pressure)" as bed type
- [ ] **Behavioral:** Check elopement OR aggression OR suicide → "Behavioral / Safety Room" appears
- [ ] **SmartPhrase copy:** Button copies text to clipboard
- [ ] **Reset:** Clears all fields and hides results
- [ ] **Print view:** Results section visible, buttons hidden
- [ ] **Mobile:** Renders in single column on screen < 520px wide
- [ ] **PHI banner:** Visible at top (sticky) at all times
- [ ] **No localStorage:** No PHI stored anywhere

---

## Next-Iteration Ideas

1. **Bed Inventory Integration** — Pull live bed board data (Epic API) to show available vs. occupied beds by type
2. **Shift Handoff Summary** — Generate pending placements summary for charge nurse shift change  
3. **Conflict Resolution Wizard** — When conflicts flagged, suggest workarounds  
4. **Roommate Matching Logic** — Add gender, program, behavioral compatibility for shared-room placement
5. **MFB Unit Map View** — Visual floor plan overlay showing room types and current status
6. **Audit Trail Export** — CSV export for quality/access analytics

---

## Standalone HTML Tool

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>MFB Rehab Bed Selection Tool</title>
<style>
:root {
  --navy: #1B3A5C;
  --navy-light: #2A5280;
  --gold: #C8952A;
  --gold-light: #E8B84B;
  --bg: #F4F6F9;
  --card: #FFFFFF;
  --border: #D0D8E4;
  --text: #1A2535;
  --muted: #5A6A80;
  --success: #1A6B3C;
  --warn: #8B5E00;
  --danger: #8B1A1A;
  --danger-bg: #FFF0F0;
  --warn-bg: #FFFAED;
  --success-bg: #F0FFF6;
}
* { box-sizing: border-box; margin: 0; padding: 0; }
body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  background: var(--bg);
  color: var(--text);
  font-size: 15px;
  line-height: 1.5;
}
.phi-banner {
  background: #C00;
  color: #fff;
  text-align: center;
  padding: 8px 16px;
  font-weight: 700;
  font-size: 13px;
  letter-spacing: 0.04em;
  position: sticky;
  top: 0;
  z-index: 100;
}
header {
  background: var(--navy);
  color: #fff;
  padding: 18px 24px 14px;
  display: flex;
  align-items: center;
  gap: 16px;
}
.mfb-logo {
  background: var(--gold);
  color: var(--navy);
  font-weight: 900;
  font-size: 18px;
  padding: 6px 10px;
  border-radius: 4px;
  letter-spacing: 0.05em;
  flex-shrink: 0;
}
header h1 { font-size: 20px; font-weight: 700; line-height: 1.2; }
header p { font-size: 12px; opacity: 0.75; margin-top: 2px; }
.container { max-width: 760px; margin: 0 auto; padding: 20px 16px 48px; }
.card {
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 8px;
  padding: 20px;
  margin-bottom: 18px;
}
.card h2 {
  font-size: 14px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  color: var(--navy);
  border-bottom: 2px solid var(--gold);
  padding-bottom: 6px;
  margin-bottom: 16px;
}
.grid { display: grid; grid-template-columns: 1fr 1fr; gap: 14px; }
@media (max-width: 520px) { .grid { grid-template-columns: 1fr; } }
.field { display: flex; flex-direction: column; gap: 4px; }
.field label { font-size: 13px; font-weight: 600; color: var(--navy); }
.field select, .field input {
  border: 1px solid var(--border);
  border-radius: 5px;
  padding: 8px 10px;
  font-size: 14px;
  color: var(--text);
  background: #fff;
  width: 100%;
  appearance: auto;
}
.field select:focus, .field input:focus {
  outline: 2px solid var(--gold);
  border-color: var(--gold);
}
.field .hint { font-size: 11px; color: var(--muted); }
.checkbox-group { display: flex; flex-direction: column; gap: 6px; }
.checkbox-group label {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 13px;
  font-weight: 400;
  color: var(--text);
  cursor: pointer;
}
.checkbox-group input[type="checkbox"] {
  width: 16px;
  height: 16px;
  accent-color: var(--navy);
  flex-shrink: 0;
}
.btn-row {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
  margin-top: 4px;
}
.btn {
  padding: 10px 22px;
  border-radius: 5px;
  border: none;
  font-size: 14px;
  font-weight: 700;
  cursor: pointer;
  transition: opacity 0.15s, transform 0.1s;
}
.btn:active { transform: scale(0.97); }
.btn-primary { background: var(--navy); color: #fff; }
.btn-primary:hover { background: var(--navy-light); }
.btn-secondary { background: var(--gold); color: var(--navy); }
.btn-secondary:hover { background: var(--gold-light); }
.btn-outline {
  background: transparent;
  border: 2px solid var(--border);
  color: var(--muted);
}
.btn-outline:hover { border-color: var(--navy); color: var(--navy); }
#results { display: none; }
.result-header {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 16px;
}
.bed-badge {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  background: var(--navy);
  color: #fff;
  padding: 6px 14px;
  border-radius: 20px;
  font-weight: 700;
  font-size: 14px;
}
.flags { display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 14px; }
.flag {
  padding: 3px 10px;
  border-radius: 12px;
  font-size: 12px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.04em;
}
.flag-red { background: #FFE4E4; color: var(--danger); border: 1px solid #FFBCBC; }
.flag-amber { background: #FFF4DB; color: var(--warn); border: 1px solid #FFD980; }
.flag-blue { background: #E4EAF5; color: var(--navy); border: 1px solid #B0C2DC; }
.flag-green { background: #E4F5EC; color: var(--success); border: 1px solid #A8DDB8; }
.rationale-list { margin: 0; padding-left: 18px; }
.rationale-list li { margin-bottom: 4px; font-size: 13px; }
.smartphrase-box {
  background: #F7F9FC;
  border: 1px solid var(--border);
  border-left: 4px solid var(--navy);
  border-radius: 4px;
  padding: 12px 14px;
  font-family: 'Courier New', Courier, monospace;
  font-size: 12px;
  white-space: pre-wrap;
  word-break: break-word;
  line-height: 1.6;
  margin-bottom: 12px;
}
.section-label {
  font-size: 11px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.07em;
  color: var(--muted);
  margin-bottom: 6px;
}
.conflict-box {
  background: var(--danger-bg);
  border: 1px solid #FFBCBC;
  border-radius: 5px;
  padding: 12px 14px;
  margin-bottom: 14px;
  font-size: 13px;
  color: var(--danger);
}
.conflict-box strong { display: block; margin-bottom: 4px; }
.checklist { display: flex; flex-direction: column; gap: 6px; }
.checklist-item {
  display: flex;
  align-items: flex-start;
  gap: 8px;
  font-size: 13px;
}
.checklist-item input[type="checkbox"] {
  margin-top: 2px;
  accent-color: var(--navy);
  flex-shrink: 0;
}
@media print {
  .phi-banner { position: static; }
  .btn-row { display: none; }
  #results { display: block !important; }
  body { background: #fff; }
  .card { border: 1px solid #ccc; page-break-inside: avoid; }
  .card:first-child { display: none; }
}
</style>
</head>
<body>

<div class="phi-banner">&#9888; DO NOT ENTER PATIENT-IDENTIFIABLE INFORMATION (PHI) — Clinical decision support only</div>

<header>
  <div class="mfb-logo">MFB</div>
  <div>
    <h1>Rehab Bed Selection Tool</h1>
    <p>Mary Free Bed Rehabilitation Hospital &nbsp;|&nbsp; Access &amp; Intake Workflow Support &nbsp;|&nbsp; Week 31</p>
  </div>
</header>

<div class="container">

  <!-- INPUT CARD -->
  <div class="card">
    <h2>Patient Clinical Parameters</h2>
    <div class="grid">

      <div class="field">
        <label for="weight">Patient Weight (lbs)</label>
        <input type="number" id="weight" min="50" max="1000" placeholder="e.g. 220" />
        <span class="hint">Bariatric threshold: ≥ 350 lbs</span>
      </div>

      <div class="field">
        <label for="program">Primary Rehab Program</label>
        <select id="program">
          <option value="">— Select —</option>
          <option value="tbi">TBI / Acquired Brain Injury</option>
          <option value="sci">Spinal Cord Injury</option>
          <option value="stroke">Stroke / Neuro</option>
          <option value="amp">Amputee</option>
          <option value="ortho">Orthopedic</option>
          <option value="cardiopulm">Cardiopulmonary</option>
          <option value="general">General Medical Rehab</option>
          <option value="other">Other</option>
        </select>
      </div>

      <div class="field">
        <label for="acuity">Medical Acuity Level</label>
        <select id="acuity">
          <option value="low">Low — Medically stable, routine monitoring</option>
          <option value="mod">Moderate — Daily vitals, oral meds, stable wound</option>
          <option value="high">High — IV meds/fluids, frequent monitoring, complex wound</option>
        </select>
      </div>

      <div class="field">
        <label for="isolation">Isolation Requirement</label>
        <select id="isolation">
          <option value="none">None</option>
          <option value="contact">Contact Precautions (MRSA, C. diff, etc.)</option>
          <option value="droplet">Droplet Precautions</option>
          <option value="airborne">Airborne / Negative Pressure</option>
          <option value="protective">Protective / Reverse Isolation</option>
        </select>
      </div>

      <div class="field">
        <label for="mobility">Lift / Mobility Assist Needs</label>
        <select id="mobility">
          <option value="indep">Independent or minimal assist</option>
          <option value="assist">2-person assist / gait belt</option>
          <option value="hoyer">Hoyer lift (portable)</option>
          <option value="ceiling">Ceiling lift required</option>
          <option value="bariatric_lift">Bariatric lift / sling system</option>
        </select>
      </div>

      <div class="field">
        <label for="gender">Patient Sex (for room pairing)</label>
        <select id="gender">
          <option value="">— Not specified —</option>
          <option value="M">Male</option>
          <option value="F">Female</option>
        </select>
      </div>

    </div>

    <div style="margin-top:18px;">
      <div class="field">
        <label style="margin-bottom:8px;">Behavioral / Safety Flags <span class="hint" style="font-weight:400;">(check all that apply)</span></label>
        <div class="checkbox-group">
          <label><input type="checkbox" id="vent" /> Ventilator dependent (current or likely on admit)</label>
          <label><input type="checkbox" id="elope" /> Elopement risk (confusion, agitation, dementia)</label>
          <label><input type="checkbox" id="aggress" /> Aggression risk (physical or verbal — behavioral concern)</label>
          <label><input type="checkbox" id="fall_high" /> High fall risk (MORSE ≥ 45 or equivalent)</label>
          <label><input type="checkbox" id="suicide" /> Suicide / self-harm precautions required</label>
          <label><input type="checkbox" id="wound_vac" /> Wound VAC or specialty wound equipment</label>
          <label><input type="checkbox" id="tele" /> Telemetry / cardiac monitoring needed</label>
        </div>
      </div>
    </div>

    <div class="btn-row" style="margin-top:20px;">
      <button class="btn btn-primary" onclick="evaluate()">&#9654; Evaluate &amp; Recommend</button>
      <button class="btn btn-outline" onclick="resetAll()">&#8635; Reset</button>
    </div>
  </div>

  <!-- RESULTS CARD -->
  <div id="results">
    <div class="card">
      <h2>Bed Selection Recommendation</h2>

      <div id="conflict-section" style="display:none;" class="conflict-box">
        <strong>&#9888; Conflict / Special Coordination Required</strong>
        <div id="conflict-text"></div>
      </div>

      <div class="result-header">
        <div>
          <div class="section-label">Recommended Bed Type</div>
          <div id="bed-type-badge" class="bed-badge">—</div>
        </div>
      </div>

      <div id="flags-row" class="flags"></div>

      <div class="section-label">Clinical Rationale</div>
      <ul id="rationale" class="rationale-list" style="margin-bottom:16px;"></ul>

      <div class="section-label">Pre-Placement Verification Checklist</div>
      <div id="checklist" class="checklist" style="margin-bottom:18px;"></div>

      <div class="section-label">Epic SmartPhrase — Bed Selection Note</div>
      <div id="smartphrase" class="smartphrase-box"></div>

      <div class="btn-row">
        <button class="btn btn-secondary" onclick="copyPhrase()">&#128203; Copy as Epic SmartPhrase</button>
        <button class="btn btn-outline" onclick="window.print()">&#128438; Print</button>
        <button class="btn btn-outline" onclick="resetAll()">&#8635; Reset</button>
      </div>
      <p style="font-size:11px;color:var(--muted);margin-top:10px;">Remove all patient identifiers before copying to Epic. This tool generates a template only — clinical judgment governs final placement.</p>
    </div>
  </div>

</div>

<script>
function evaluate() {
  const weight = parseFloat(document.getElementById('weight').value) || 0;
  const program = document.getElementById('program').value;
  const acuity = document.getElementById('acuity').value;
  const isolation = document.getElementById('isolation').value;
  const mobility = document.getElementById('mobility').value;
  const gender = document.getElementById('gender').value;

  const vent = document.getElementById('vent').checked;
  const elope = document.getElementById('elope').checked;
  const aggress = document.getElementById('aggress').checked;
  const fall_high = document.getElementById('fall_high').checked;
  const suicide = document.getElementById('suicide').checked;
  const wound_vac = document.getElementById('wound_vac').checked;
  const tele = document.getElementById('tele').checked;

  const bariatric = weight >= 350 || mobility === 'bariatric_lift';
  const behavSafety = elope || aggress || suicide;
  const isolRequired = isolation !== 'none';
  const highAcuity = acuity === 'high' || tele;
  const ceilingLift = mobility === 'ceiling' || mobility === 'bariatric_lift';

  // Conflict detection
  const conflicts = [];
  if (vent && bariatric) conflicts.push('Vent-capable AND bariatric equipment may both be required — confirm room meets both specifications with charge nurse.');
  if (isolRequired && !['none','contact'].includes(isolation) && behavSafety) conflicts.push('Isolation requirement + behavioral/safety concern: ensure isolation room allows continuous observation. May require 1:1 sitter.');
  if (vent && isolation === 'airborne') conflicts.push('Ventilator patient with airborne precautions: confirm negative-pressure capability of vent room.');
  if (bariatric && ceilingLift) conflicts.push('Bariatric weight class + ceiling lift: verify ceiling lift rated for patient weight in assigned room.');

  // Bed type determination (priority order)
  let bedType = 'Standard Rehab Bed';
  let bedPriorities = [];

  if (vent) bedPriorities.push('Vent-Capable Room');
  if (bariatric) bedPriorities.push('Bariatric Room');
  if (isolation === 'airborne') bedPriorities.push('Airborne Isolation Room (Negative Pressure)');
  if (isolRequired && isolation !== 'airborne') bedPriorities.push('Private Isolation Room');
  if (behavSafety) bedPriorities.push('Behavioral / Safety Room (Proximity to Station)');
  if (highAcuity) bedPriorities.push('High-Acuity / Step-Down Capable Bed');
  if (ceilingLift && !bariatric) bedPriorities.push('Ceiling Lift-Equipped Room');

  if (bedPriorities.length === 0) bedPriorities.push('Standard Rehab Bed');

  // Primary recommendation = highest priority
  bedType = bedPriorities[0];
  const additionalNeeds = bedPriorities.slice(1);

  // Build rationale
  const rationale = [];
  if (vent) rationale.push('Ventilator dependency requires room with accessible O₂/suction/power infrastructure and vent-trained nursing proximity.');
  if (bariatric) rationale.push(`Weight ≥ 350 lbs (or bariatric lift specified): bariatric-rated bed frame, mattress, and lift equipment required.`);
  if (isolation === 'airborne') rationale.push('Airborne precautions mandate negative-pressure private room.');
  else if (isolation === 'contact') rationale.push('Contact precautions require private room or cohorting with same pathogen; dedicated equipment.');
  else if (isolation === 'droplet') rationale.push('Droplet precautions require private room or ≥3 ft separation; door closed when possible.');
  else if (isolation === 'protective') rationale.push('Protective isolation: positive-pressure or private room with strict hand hygiene; minimize traffic.');
  if (elope) rationale.push('Elopement risk: room near nurses station, alarmed exits, visual line-of-sight preferred.');
  if (aggress) rationale.push('Aggression risk: proximity to staff, rapid-response accessibility; consider single-patient room.');
  if (suicide) rationale.push('Suicide/self-harm precautions: ligature-risk assessment of room required; 1:1 sitter may be indicated.');
  if (fall_high) rationale.push('High fall risk: low-rise bed with floor mat, call light within reach, ideally visible from hallway.');
  if (tele) rationale.push('Telemetry / cardiac monitoring required: confirm room has monitoring capability or proximity to telemetry tech.');
  if (wound_vac) rationale.push('Wound VAC: ensure room has adequate power outlets and clearance for pump placement.');
  if (ceilingLift) rationale.push('Ceiling lift required: room must be equipped with track system rated for patient weight.');
  if (acuity === 'high') rationale.push('High medical acuity: step-down capable room with closer nursing check-in frequency.');

  // Checklist items
  const checks = [
    'Confirm bed type availability with charge nurse / bed board',
    'Verify room equipment list matches patient needs (lift, vent hookups, monitors)',
    gender ? `Confirm room gender alignment (${gender === 'M' ? 'Male' : 'Female'} patient)` : 'Confirm room gender alignment',
    'Notify therapy team of bed location (distance from gym, elevator access)',
    'Ensure call-light and emergency call are operational',
    'Complete room readiness checklist before patient arrival',
    'Document bed type rationale in Epic admission note'
  ];
  if (vent) checks.push('Confirm respiratory therapy notified and vent setup completed pre-arrival');
  if (bariatric) checks.push('Bariatric equipment signed out and in room before transfer');
  if (isolRequired) checks.push('Post isolation signage and stock PPE outside room door');
  if (behavSafety) checks.push('Notify charge RN and behavioral health of safety flags; document in care plan');
  if (tele) checks.push('Activate telemetry order and notify monitoring tech of admit');
  if (additionalNeeds.length > 0) checks.push('Secondary bed requirements flagged — see conflicts/rationale above; escalate if single room cannot meet all needs');

  // Build flags
  const flags = [];
  if (vent) flags.push({text:'VENT', cls:'flag-red'});
  if (bariatric) flags.push({text:'BARIATRIC', cls:'flag-amber'});
  if (behavSafety) flags.push({text:'BEHAV SAFETY', cls:'flag-red'});
  if (isolRequired) flags.push({text: isolation.toUpperCase() + ' ISO', cls:'flag-amber'});
  if (highAcuity) flags.push({text:'HIGH ACUITY', cls:'flag-amber'});
  if (ceilingLift && !bariatric) flags.push({text:'CEILING LIFT', cls:'flag-blue'});
  if (fall_high) flags.push({text:'FALL RISK', cls:'flag-amber'});
  if (conflicts.length > 0) flags.push({text:'CONFLICT — SEE NOTES', cls:'flag-red'});
  if (flags.length === 0) flags.push({text:'STANDARD', cls:'flag-green'});

  // Build SmartPhrase
  const programLabel = {tbi:'TBI/Acquired Brain Injury',sci:'Spinal Cord Injury',stroke:'Stroke/Neuro',amp:'Amputee',ortho:'Orthopedic',cardiopulm:'Cardiopulmonary',general:'General Medical Rehab',other:'Specialty Rehab'}[program] || 'Rehab';
  const acuityLabel = {low:'low',mod:'moderate',high:'high'}[acuity];
  const isoLabel = {none:'No isolation required',contact:'Contact precautions',droplet:'Droplet precautions',airborne:'Airborne/negative-pressure precautions',protective:'Protective/reverse isolation'}[isolation];

  const phraseLines = [
    `BED SELECTION — REHAB ADMISSION`,
    `Program: ${programLabel}`,
    `Recommended Bed Type: ${bedType}`,
    additionalNeeds.length > 0 ? `Additional Requirements: ${additionalNeeds.join('; ')}` : null,
    `Medical Acuity: ${acuityLabel}`,
    `Isolation: ${isoLabel}`,
    `Behavioral/Safety Flags: ${[elope&&'elopement risk',aggress&&'aggression risk',suicide&&'self-harm precautions',fall_high&&'high fall risk'].filter(Boolean).join(', ') || 'None'}`,
    `Equipment Needs: ${[vent&&'ventilator hookup',bariatric&&'bariatric bed/lift',ceilingLift&&'ceiling lift',wound_vac&&'wound VAC',tele&&'telemetry monitoring'].filter(Boolean).join(', ') || 'Standard'}`,
    conflicts.length > 0 ? `CONFLICTS: ${conflicts.join(' | ')}` : null,
    ``,
    `Rationale: ${rationale.join(' ')}`,
    ``,
    `Pre-placement checklist completed per MFB Access protocol. Charge RN notified. Bed board updated.`,
    `Clinical judgment governs final placement. Document any deviation from recommendation.`
  ].filter(v => v !== null).join('\n');

  // Render
  document.getElementById('conflict-section').style.display = conflicts.length > 0 ? 'block' : 'none';
  document.getElementById('conflict-text').innerHTML = conflicts.map(c => `• ${c}`).join('<br>');

  document.getElementById('bed-type-badge').textContent = bedType;

  document.getElementById('flags-row').innerHTML = flags.map(f =>
    `<span class="flag ${f.cls}">${f.text}</span>`
  ).join('');

  document.getElementById('rationale').innerHTML = rationale.map(r => `<li>${r}</li>`).join('') || '<li>No special requirements identified. Standard bed appropriate.</li>';

  document.getElementById('checklist').innerHTML = checks.map(c =>
    `<div class="checklist-item"><input type="checkbox" /><span>${c}</span></div>`
  ).join('');

  document.getElementById('smartphrase').textContent = phraseLines;

  document.getElementById('results').style.display = 'block';
  document.getElementById('results').scrollIntoView({behavior:'smooth', block:'start'});
}

function copyPhrase() {
  const text = document.getElementById('smartphrase').textContent;
  navigator.clipboard.writeText(text).then(() => {
    const btn = event.target;
    const orig = btn.textContent;
    btn.textContent = '✓ Copied!';
    btn.style.background = '#1A6B3C';
    btn.style.color = '#fff';
    setTimeout(() => { btn.textContent = orig; btn.style.background = ''; btn.style.color = ''; }, 2000);
  }).catch(() => {
    const el = document.getElementById('smartphrase');
    const sel = window.getSelection();
    const range = document.createRange();
    range.selectNodeContents(el);
    sel.removeAllRanges();
    sel.addRange(range);
  });
}

function resetAll() {
  document.getElementById('weight').value = '';
  document.getElementById('program').value = '';
  document.getElementById('acuity').value = 'low';
  document.getElementById('isolation').value = 'none';
  document.getElementById('mobility').value = 'indep';
  document.getElementById('gender').value = '';
  ['vent','elope','aggress','fall_high','suicide','wound_vac','tele'].forEach(id => {
    document.getElementById(id).checked = false;
  });
  document.getElementById('results').style.display = 'none';
  window.scrollTo({top:0, behavior:'smooth'});
}
</script>

</body>
</html>

```

---

*Generated by MFB Rehab Tool Builder · ISO Week 31 · 2026-07-29*  
*Next week (Week 32): Liaison Communication Template (Index 7)*
