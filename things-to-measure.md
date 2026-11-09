# Things to measure: ten findings and the channels that power them

`biomarkers.md` establishes *which* biomarkers exist and why. This file is the layer above it: the
**conditions the app detects**, the **published criteria it detects them with**, and the reduced
channel list that falls out. `weaknesses.md` §3 says no biomarkers are decided — this is the
decision, and the thing that decides it is the finding list, not the sensor list.

Working principle, extending `searching-for-correlations.md`: **a channel earns its place only if a
finding consumes it.** 33 channels with no finding list means nothing constrains the set. Ten
findings constrain it to ~22.

---

## 0. How to read the citations

Every rule below carries a source key resolving to §9. Because `searching-for-correlations.md` is
right that LLMs invent PMIDs and thresholds, this file uses a deliberate convention:

- **No PMIDs or DOIs unless highly confident.** Author/title/journal/year/volume:pages is
  checkable without risking a fabricated identifier.
- **⚠ marks a specific number or claim I could not verify to the digit.** The reference is real;
  the exact threshold needs eyes on the source before it goes in the rule engine.
- **Everything unmarked is a standard clinical cut-point** restated from a named guideline.

Nothing here is in the engine until someone opens the source. §8 is the verification queue.

---

## 1. The selection test

A condition is in scope only if it passes all four:

1. **Named in NASA's risk list** — hazard traceability for the judges.
2. **Published multi-marker criteria** — the rule is citable, not invented.
3. **Powered by channels we can produce** — measured, reported, or honestly-labelled simulated.
4. **Has an unambiguous crew action** — if the app can only say "tell the ground," it is not
   decision support, and under a Mars comm delay it is not even that.

Test 4 is the one that cuts hardest, and it is why the lab-heavy end of `biomarkers.md` Tier 3
mostly doesn't survive.

---

## 2. The ten findings

| # | Finding | Hazard | Detection basis | Action class |
|---|---|---|---|---|
| 1 | Orthostatic intolerance / CV deconditioning | Gravity | AAS/AAN + consensus OH & POTS criteria | Stand test, fluid+salt load, resistive exercise |
| 2 | Dehydration & electrolyte loss | Closed env | Body-mass delta + urine indices + sweat Na⁺ | Drink *this much, now* |
| 3 | Acute infection / immune dysregulation | Radiation | NEWS2 / qSOFA / SIRS + latent-virus shedding | Medical kit protocol, reduce workload, escalate |
| 4 | Hypoxia / CO₂ exposure | Closed env | BTS oxygen targets + ISS ppCO₂ headache data | Check scrubber, ventilate, relocate |
| 5 | Sleep deficiency / circadian misalignment | Isolation | Actigraphy + PVT + AASM sleep-duration consensus | Light protocol, nap window, schedule shift |
| 6 | Behavioral health decline | Isolation | PHQ-9 / GAD-7 / POMS + PVT lapses + sleep debt | Comms window, workload reduction, flag |
| 7 | Negative energy balance / nutrition | Closed env | Body-mass trend + ADA glucose cut-points + 25-OH-D | Caloric target, supplement |
| 8 | Bone demineralization + renal stone risk | Gravity | IOF/IFCC turnover markers + AUA stone panel | ARED load increase, fluid target |
| 9 | SANS / cephalic fluid shift | Gravity | Visual acuity + Amsler, triggered by fluid index | Run the eye test, trend, escalate |
| 10 | Radiation dose / SPE | Radiation | Cumulative dose vs. NASA-STD-3001 limits | Get to shelter |

**Distance from Earth is deliberately not a finding.** It is architectural: offline-first,
comm-delay-aware escalation, self-treatment guidance. Saying so explicitly reads as deliberate;
inventing an eleventh finding for it would not.

---

## 3. Finding detail

### 1. Orthostatic intolerance / cardiovascular deconditioning

**Rule.** Orthostatic hypotension = systolic fall ≥20 mmHg *or* diastolic fall ≥10 mmHg within 3
min of standing or ≥60° head-up tilt [FREEMAN-2011, AAS-1996]. Orthostatic tachycardia = heart-rate
rise ≥30 bpm within 10 min of standing in the absence of OH [FREEMAN-2011]. Supporting trend
signals: resting-HR drift and RMSSD decline against that astronaut's own baseline, with RMSSD
computed per the ESC/NASPE standard so the number means the same thing across devices
[TASKFORCE-1996]. Shock index (HR/SBP) > 0.9 ⚠ as an acute flag [RADY-1994, ALLGOWER-1967].

**Why in space.** Microgravity removes the hydrostatic gradient the baroreflex is tuned to; plasma
volume drops and the problem surfaces on standing — at re-entry, or on first standing up on the
Moon or Mars. Long-duration flight is markedly worse than short-duration ⚠ [MECK-2001,
BUCKEY-1996].

**Channels.** `heart_rate`, `hrv_rmssd`, `bp_systolic`, `bp_diastolic`, `orthostatic_response`.

**Action.** Prompt the stand test (this is the one finding where the app *creates* its own missing
input — good demo beat, and it is exactly the `missing` field in `json-shape.md` §4). Then fluid +
salt loading, resistive exercise compliance, and a pre-landing flag.

### 2. Dehydration and electrolyte loss

**Rule.** Acute body-mass loss >2% of baseline as the dehydration threshold [NATA-2000]; urine
specific gravity >1.020 as a concurrent index [ARMSTRONG-1994]. During EVA, sweat rate and sweat
Na⁺ from the patch, reported **baseline-relative per astronaut** because local-to-whole-body
extrapolation has real regional variation. Confirmatory: HR↑ with BP↓ and a falling pulse pressure.

**Why in space.** Headward fluid shift blunts thirst, and today in-suit loss is reconstructed after
the fact from pre/post body mass and drink-bag volume [LEACH-1996 ⚠ for the fluid-compartment
numbers]. Sweat Na⁺/Cl⁻ and collected sweat volume are the well-validated wearable analytes
[GAO-2016, KOH-2016, BANDODKAR-2019]; sweat glucose, lactate and cortisol correlate poorly with
blood and must not be presented as blood equivalents.

**Channels.** `body_mass`, `urine_specific_gravity`, `fluid_intake`, `sweat_rate`, `sweat_sodium`,
`heart_rate`, `bp_systolic`.

**Action.** A volume and a deadline — "drink 600 mL in the next 30 min." The only finding in the set
whose action is fully quantitative, which makes it the best one to show on stage.

**Honesty note.** A two-electrode conductance sensor measures electrodermal activity, i.e.
sudomotor activation, **not** mL/h. If we demo that, the channel is `electrodermal_activity` under
autonomic arousal, not hydration. See `hydrogel-usecases.md` for the full correction.

### 3. Acute infection / immune dysregulation

**Rule.** NEWS2 as the primary aggregate: respiratory rate, SpO₂, air-or-oxygen, systolic BP,
pulse, consciousness, temperature; aggregate ≥5 = urgent review, ≥7 = emergency, and any single
parameter scoring 3 = red regardless of total [RCP-NEWS2-2017]. qSOFA ≥2 of {RR ≥22, altered
mentation, SBP ≤100 mmHg} as the deterioration flag [SINGER-2016]. SIRS pattern {temp >38.0 or
<36.0 °C, HR >90, RR >20} where no WBC count is available [BONE-1992].

**Why in space.** Radiation and chronic stress dysregulate immunity, and the best-evidenced
in-flight readout is not a cytokine number but **latent virus reactivation** — EBV, VZV and CMV
shedding measurable in saliva, more frequent on longer missions, with shedders showing elevated
plasma cytokines [MEHTA-2017, CRUCIAN-2018].

**Channels.** `temp_core_equiv`, `heart_rate`, `respiratory_rate`, `spo2`, `bp_systolic`,
`latent_virus_shedding`.

**Action.** Medical-kit protocol, workload reduction, and escalation — NEWS2 already maps onto the
green/amber/red colour coding the challenge asks for, which is most of the UI work done for free.

### 4. Hypoxia / CO₂ exposure

**Rule.** SpO₂ target range 94–98% on air; below 94% warrants investigation and <90% is significant
hypoxaemia [BTS-2017]. Rising respiratory rate and HR alongside falling SpO₂ is the composite.
Cabin ppCO₂ is a **separate environmental channel** that modifies the rule rather than being a
vital: reported headache incidence on ISS rises with ppCO₂ ⚠ [LAW-2014], and ISS operational ppCO₂
limits have been progressively lowered ⚠ [NASA-STD-3001, OCHMO-TB].

**Why in space.** CO₂ accumulation is a documented, recurring ISS problem with a crew-visible
symptom profile (headache, lethargy, impaired cognition) that is easy to misattribute to fatigue.

**Channels.** `spo2`, `respiratory_rate`, `heart_rate`, `cabin_co2`, plus a reported symptom flag.

**Action.** Check the scrubber, increase ventilation, relocate within the vehicle. Fast, physical,
unambiguous.

**Demo value.** This is the one acute finding that can fire from a real sensor on stage: breathe
near an SCD40 and ppCO₂ climbs in front of the audience. Right now finding 10 is the only other
time-critical beat, and that one has to be faked.

### 5. Sleep deficiency and circadian misalignment

**Rule.** Sleep duration, onset and efficiency from wrist actigraphy, a validated method for sleep/
wake estimation [ANCOLI-ISRAEL-2003, AASM-ACTIGRAPHY-2018]. Objective alertness from a daily
5-minute PVT, scoring mean reaction time and lapses (RT >500 ms) [DINGES-1985, BASNER-2011].
Supporting: resting-HR elevation and RMSSD decline vs. baseline.

**Threshold anchoring — read this before setting cut-points.** The general-population consensus is
≥7 h for adults [WATSON-2015], but ISS crews average **6.09 h against 8.5 h scheduled**, with heavy
hypnotic use [BARGER-2014]. If "nominal" is >7.5 h, every real astronaut reads abnormal and the
colour coding becomes noise. Anchor to flight data and to the individual's own baseline, and weight
the **trend** over the absolute.

**Circadian phase.** `biomarkers.md` currently infers phase from sleep + light, which is an
inference. Core body temperature minimum is the gold-standard phase marker [CZEISLER-1999], and
zero-heat-flux thermometry is validated against oesophageal core temp ⚠ [TEUNISSEN-2011]. Light
drives phase with a known dose-response [ZEITZER-2000], which is what makes the lighting
recommendation defensible rather than folk advice.

**Channels.** `sleep_duration`/`onset`/`efficiency`, `light_exposure`, `pvt_reaction_time`,
`activity_counts`, `heart_rate`, `hrv_rmssd`, `temp_core_equiv`.

**Action.** Light exposure protocol, protected nap window, schedule shift request.

### 6. Behavioral health decline

**Rule.** Daily validated self-report — PHQ-9 (cut-points 5/10/15/20 for mild/moderate/moderately
severe/severe, ≥10 the usual screening threshold) [KROENKE-2001] and GAD-7 (5/10/15)
[SPITZER-2006], or a POMS/PANAS-style mood scale for daily use [MCNAIR-1971]. Paired with
**objective** signals: PVT lapses [BASNER-2011], accumulated sleep debt [BARGER-2014], and salivary
cortisol expressed as a **diurnal slope** rather than a single absolute value, which is what avoids
circadian false positives [ADAM-2009].

**Why in space.** Behavioural conditions are a top risk for a Mars-class mission [NASA-BMED], and
analog confinement shows protracted hypokinesis with altered sleep duration and timing
[BASNER-2013].

**Channels.** `mood_score`, `pvt_reaction_time`, `sleep_duration`, `salivary_cortisol_slope`.

**Flagged exploratory.** Speech-time / vocalisation duration from a mechano-acoustic patch
[LEE-2020] is a genuinely interesting objective withdrawal proxy with the audio-privacy objection
structurally removed — but "speech time tracks mood" is weakly validated. Ship it baseline-relative
and labelled exploratory, never diagnostic. Same for `electrodermal_activity`.

### 7. Negative energy balance and nutrition

**Rule.** Body-mass trend as the primary channel — slope per week against baseline, not a single
reading. Glucose cut-points from ADA/IHSG: hypoglycaemia Level 1 <70 mg/dL (3.9 mmol/L), Level 2
<54 mg/dL (3.0 mmol/L) [IHSG-2017, ADA-SOC]; fasting 100–125 mg/dL impaired, ≥126 mg/dL (7.0
mmol/L) diagnostic [ADA-SOC]. 25-OH vitamin D against clinical sufficiency, read as change from
preflight.

**Why in space.** Under-eating is the default failure mode and it accelerates bone and muscle loss;
ISS crews land measurably lower in 25-OH-D than they launched despite supplementation, and urinary
bone resorption markers run well above preflight [SMITH-2005, SMITH-2012]. In-flight insulin
resistance has been reported ⚠ [HUGHSON-2016]. Body mass needs an inertial device in microgravity,
not a scale [HRF-SLAMMD].

**Channels.** `body_mass`, `glucose`, `fluid_intake` (doubles as the intake log),
`vitamin_d_25oh`.

**Action.** A caloric target and a supplement instruction. Tie every glucose reading to meal and
exercise timing, or a normal low reading will alarm for no reason.

### 8. Bone demineralization and renal stone risk

**Rule — two halves, and the second is the mission-threatening one.**

*Bone.* Resorption markers expressed as **percent change from that astronaut's own preflight
baseline**, which is how the spaceflight literature reports them — in-flight urinary resorption
markers run on the order of tens of percent above preflight ⚠ [SMITH-2005]. CTX and P1NP are the
IOF/IFCC reference markers [VASIKARAN-2011]; absolute nM cut-points have no grounding here.
Densitometry-confirmed loss runs roughly 1–1.5%/month at hip and spine without countermeasures ⚠
[LEBLANC-2000, LANG-2004].

*Stones.* Standard stone-risk panel: 24-h urine volume (<2.0 L/day as risk ⚠), urinary calcium,
and citrate [PEARLE-2014-AUA]. Spaceflight raises this specifically because bone-derived calcium is
excreted into a low-volume, often dehydrated urine ⚠ [WHITSON-1997].

**Why it's ranked high.** On a Mars transit an obstructing stone is a mission-threatening event with
no surgical option. This is the clearest case in the whole set where prevention beats detection.

**Exercise compliance is a biomarker here, not a side note.** Resistive exercise plus adequate
nutrition is the countermeasure [SMITH-2012], which makes logged ARED/treadmill minutes and loads
the most actionable number in the domain — and it is what makes the gamification idea in `idea.md`
medically meaningful rather than decorative.

**Channels.** `urinary_ntx_pct_baseline`, `urine_calcium_volume`, `exercise_compliance`,
`fluid_intake`, `body_mass`, `vitamin_d_25oh`.

### 9. SANS / cephalic fluid shift

**Rule — a two-tier structure, and it is the best structural idea we have.** A continuous
`cephalic_fluid_index` (segmental bioimpedance ratio, calf vs. thoracic, as percent change from
preflight supine baseline) **triggers** the validated screen rather than being diagnostic itself.
The screen is pure software: distance visual acuity on a logMAR chart [BAILEY-LOVIE-1976] and an
Amsler grid, which is close to what in-flight vision screening actually consists of today.

**Honesty on both tiers.** The Amsler grid has limited sensitivity and a normal result is not
evidence of absence [CROSSLAND-2007] — so it escalates, it does not clear. And there is **no
validated cut-point for a bioimpedance fluid index in microgravity**; it is baseline-relative trend
only, and a ratio specifically to avoid absolute calibration. Do not report mmHg of anything.

**Why in space.** Long-duration flight produces optic disc oedema, globe flattening, choroidal folds
and hyperopic shift [MADER-2011, LEE-2020-SANS], headward fluid shift is the suspected driver, and
jugular stasis has been observed in flight [MARSHALL-GOEBEL-2019]. Some of it does not fully
reverse, making it one of the few spaceflight risks that is permanent.

**Channels.** `cephalic_fluid_index` (research tier), `visual_acuity`, `amsler_grid`.

### 10. Radiation dose and solar particle event

**Rule.** Cumulative effective dose and dose rate against career and annual limits from
NASA-STD-3001 ⚠ (see §8 — our own `biomarkers.md` and the current standard disagree on the career
number, and that is exactly the kind of figure a judge will know). SPE flag from an external feed as
a boolean event channel.

**Why it earns its place twice.** It is hazard #1 in the challenge, it is the input that makes the
immune and inflammation story *causal* rather than asserted, and it gives the environment selector
in `idea.md` something real to switch on: transit cruise dose rates are far above LEO ⚠
[ZEITLIN-2013] and Mars surface rates differ again ⚠ [HASSLER-2014]. Personal dosimetry with time
resolution is already flight practice [BERGER-2023].

**Channels.** `radiation_dose`, `dose_rate`, `spe_flag`.

**Action.** Get to the shelter. The only finding in the set with a single-verb action and a hard
deadline — which is why it belongs in the pitch video even though the data has to be synthetic.

---

## 4. The channel list (22, down from 33)

★ = one of the six indicators the challenge names explicitly. These must be first-class tiles in the
UI even where they are low-information, because the brief scores against them.

**Continuous**

| Channel | Unit | Source | Feeds |
|---|---|---|---|
| `heart_rate` ★ | bpm | measured | 1, 3, 4, 5 |
| `hrv_rmssd` | ms | derived | 1, 5 |
| `respiratory_rate` | br/min | derived | 3, 4 |
| `spo2` | % | measured | 3, 4 |
| `temp_core_equiv` ★ | °C | measured | 3, 5 |
| `activity_counts` | counts/epoch | measured | 5 |
| `light_exposure` | lux | measured | 5 |
| `radiation_dose` / `dose_rate` | mSv, µSv/h | measured | 10 |
| `cabin_co2` | mmHg | measured (env) | 4 |
| `cephalic_fluid_index` | % vs baseline | measured (research) | 9 |
| `sweat_rate` / `sweat_sodium` | mL/h, mmol/L | measured (research, EVA) | 2 |

**Prompted / daily**

| Channel | Unit | Source | Feeds |
|---|---|---|---|
| `bp_systolic` ★ / `bp_diastolic` (+ `map`) | mmHg | measured | 1, 2, 3 |
| `orthostatic_response` | Δbpm, ΔmmHg | derived | 1 |
| `glucose` ★ | mg/dL, mmol/L | measured | 7 |
| `fluid_intake` | mL/day | reported | 2, 8 |
| `urine_specific_gravity` | unitless | measured/reported | 2 |
| `sleep_duration` ★ / `onset` / `efficiency` | h, clock, % | derived | 5, 6 |
| `mood_score` ★ | validated scale | reported | 6 |
| `pvt_reaction_time` | ms, lapses | measured (software) | 5, 6 |
| `exercise_compliance` | min, kg, W | reported/device | 1, 8 |

**Weekly / monthly**

| Channel | Unit | Source | Feeds |
|---|---|---|---|
| `body_mass` | kg | measured | 2, 7, 8 |
| `visual_acuity` / `amsler_grid` | logMAR, pass/map | reported (software) | 9 |
| `urinary_ntx_pct_baseline` | % vs preflight | measured | 8 |
| `urine_calcium_volume` | mg/day, mL/day | measured | 8 |
| `vitamin_d_25oh` | ng/mL | measured | 7, 8 |
| `latent_virus_shedding` | detect / no-detect | measured | 3 |
| `salivary_cortisol_slope` | nmol/L, slope | measured | 6 |

**Event-driven**

| Channel | Unit | Source | Feeds |
|---|---|---|---|
| `spe_flag` | bool | external feed | 10 |
| `hs_ctni` | ng/L | measured | 1 (event only) |

**Exploratory, labelled as such in the UI**

`electrodermal_activity`, `speech_time`. Neither gates a finding; both are baseline-relative trends
shown next to the validated channels.

---

## 5. What was cut, and why

| Cut | Reason |
|---|---|
| Ferritin, PTH, serum calcium, haematocrit/haemoglobin | Monthly-or-slower labs that no finding acts on. Fails test 4. |
| IL-6 / TNF-α / CRP panel | The real spaceflight picture is chronic low-grade dysregulation, not a storm. Latent-virus shedding carries the immune story with a far more robust detect/no-detect signal. |
| Intraocular pressure | Needs a pneumotonometer and usually ground readout. The acuity + Amsler screen covers the same domain in software. |
| Urine citrate / oxalate as separate channels | Folded into `urine_calcium_volume` as one stone-risk panel. |
| Monthly cognitive battery | Redundant with a daily PVT at hackathon scope. Re-add only if gamification needs a per-domain score [BASNER-2015]. |
| ISF glucose GFET | Commercial CGM already owns this. No differentiation. |
| Prefrontal EEG θ/α | No path to a defensible in-suit claim in the time available. |
| Joint-kinematics strips | Real literature behind it, but an IMU does most of it cheaper, so the "only we can do this" argument is thin. Keep only if gamification needs rep counting. |
| Serotonin / dopamine, non-invasive ICP as a number, continuous troponin or cytokine streaming, voice sentiment | Already excluded in `biomarkers.md` §6. Staying excluded. |

The threshold work is the real bottleneck and it scales with channel count, so this cut is worth
roughly a third of that effort.

---

## 6. Hazard coverage check

| NASA hazard | Findings |
|---|---|
| Space radiation | 10 (dose, SPE), 3 (immune, latent virus) |
| Isolation & confinement | 5 (sleep, circadian), 6 (behavioral) |
| Gravity fields | 1 (orthostatic), 8 (bone, stones), 9 (SANS) |
| Hostile closed environment | 2 (hydration), 4 (hypoxia/CO₂), 7 (nutrition) |
| Distance from Earth | Architectural, not a finding — offline-first, comm-delay-aware escalation, self-treatment guidance |

Against the health problems the challenge names: immune changes (3), bone density loss (8),
cardiovascular events (1), inadequate nutrition (7), behavioral health (6). All five have a real
finding, not just a channel.

---

## 7. What is actually live for the demo

`idea.md` wants one physical sensor proving the pipeline end to end. Two is better, and the second
one is cheap:

1. **PPG + temperature module** (MAX30102-class, or a chest strap with an open BLE HR profile) —
   yields `heart_rate`, `hrv_rmssd`, `spo2`, `respiratory_rate` and skin temp from one device, so
   the live trend chart is driven by real data rather than a replay. Drives findings 1, 3, 5.
2. **SCD40 cabin CO₂** — the only acute, time-critical finding that can fire from a real sensor in
   the room. Breathe near it, ppCO₂ climbs, finding 4 fires with a physical action. Without this,
   finding 10 is the only urgent beat in the pitch and its data is synthetic.

Everything else is reported, simulated-from-baseline, or manual entry, **with the source labelled in
the UI** — `json-shape.md` already has the `source` field for exactly this. A judge will respect
"two channels are live, the rest are labelled" far more than an all-synthetic dashboard claiming to
be live.

**One naming problem to fix first.** `temp_core_equiv` fed by a MAX30205 is skin temperature wearing
a core-temperature name, which `biomarkers.md` §3 already warns against. Either rename it
`skin_temp` and treat it as trend-only, or commit to zero-heat-flux thermometry ⚠ [TEUNISSEN-2011] —
which also upgrades the circadian phase in finding 5 from an inference to a measurement.

---

## 8. Verification queue

Nothing marked ⚠ goes into the rule engine until someone opens the source. In rough priority order:

1. **NASA career radiation dose limit.** `biomarkers.md` §3 says 1000 mSv career with "some agencies
   use 600 mSv." Current NASA-STD-3001 Vol 1 and the 2021 NASEM review of NASA's radiation limits
   point toward a single 600 mSv career limit, which would make our own doc wrong on hazard #1.
   Resolve this first — it is the most quotable number in the project.
2. **ISS operational ppCO₂ limit** and the headache dose-response in [LAW-2014]. Finding 4's
   threshold depends on it.
3. **Bone resorption percent-above-preflight range** from [SMITH-2005] and [SMITH-2012], and the
   %/month BMD figure from [LANG-2004].
4. **Stone-risk cut-points** (urine volume, calcium, citrate) from the AUA guideline, plus the
   spaceflight-specific numbers in [WHITSON-1997].
5. **Orthostatic intolerance incidence** after long- vs. short-duration flight [MECK-2001].
6. **Zero-heat-flux core-temp agreement limits** [TEUNISSEN-2011] — needed before we claim core
   temperature at all.
7. **Mars transit and surface dose rates** [ZEITLIN-2013, HASSLER-2014] for the environment selector.
8. **Shock index threshold** [RADY-1994] — low stakes, but it is in a rule.

Per `json-shape.md` §2, each channel's dictionary entry carries a `citations` array. That is what
makes the "every threshold hardcoded from cited research" line in the pitch literally checkable by a
judge, so the keys below should land in those arrays verbatim.

---

## 9. References

**Cardiovascular / autonomic**

- `FREEMAN-2011` — Freeman R. et al. Consensus statement on the definition of orthostatic
  hypotension, neurally mediated syncope and the postural tachycardia syndrome. *Clin Auton Res*
  21(2):69–72 (2011).
- `AAS-1996` — Consensus Committee of the American Autonomic Society and the American Academy of
  Neurology. Consensus statement on the definition of orthostatic hypotension, pure autonomic
  failure, and multiple system atrophy. *Neurology* 46(5):1470 (1996).
- `TASKFORCE-1996` — Task Force of the European Society of Cardiology and the North American Society
  of Pacing and Electrophysiology. Heart rate variability: standards of measurement, physiological
  interpretation, and clinical use. *Circulation* 93(5):1043–1065 (1996).
- `RADY-1994` — Rady M.Y. et al. Shock index: a re-evaluation in acute circulatory failure.
  *Resuscitation* / *Am J Emerg Med* (1994). ⚠ exact venue and threshold to confirm.
- `ALLGOWER-1967` — Allgöwer M., Burri C. Schockindex. *Dtsch Med Wochenschr* 92:1947–1950 (1967).
- `BUCKEY-1996` — Buckey J.C. Jr. et al. Orthostatic intolerance after spaceflight. *J Appl Physiol*
  81(1):7–18 (1996).
- `MECK-2001` — Meck J.V. et al. Marked exacerbation of orthostatic intolerance after long- vs.
  short-duration spaceflight in veteran astronauts. *Psychosom Med* 63(6):865–873 (2001). ⚠

**Acute illness scoring**

- `RCP-NEWS2-2017` — Royal College of Physicians. National Early Warning Score (NEWS) 2:
  standardising the assessment of acute-illness severity in the NHS. RCP London (2017).
- `SINGER-2016` — Singer M. et al. The Third International Consensus Definitions for Sepsis and
  Septic Shock (Sepsis-3). *JAMA* 315(8):801–810 (2016).
- `BONE-1992` — Bone R.C. et al. Definitions for sepsis and organ failure and guidelines for the use
  of innovative therapies in sepsis. ACCP/SCCM Consensus Conference. *Chest* 101(6):1644–1655 (1992).
- `BTS-2017` — O'Driscoll B.R. et al. BTS guideline for oxygen use in adults in healthcare and
  emergency settings. *Thorax* 72(Suppl 1):ii1–ii90 (2017).

**Immune / spaceflight**

- `MEHTA-2017` — Mehta S.K., Crucian B.E. et al. Latent virus reactivation in astronauts on the
  International Space Station. *npj Microgravity* 3:11 (2017).
- `CRUCIAN-2018` — Crucian B.E. et al. Immune system dysregulation during spaceflight: potential
  countermeasures for deep space exploration missions. *Front Immunol* 9:1437 (2018).
- NASA Evidence Report: Risk of Crew Adverse Health Event Due to Altered Immune Response,
  NTRS 20150008177.

**Hydration / sweat**

- `NATA-2000` — Casa D.J. et al. National Athletic Trainers' Association position statement: fluid
  replacement for athletes. *J Athl Train* 35(2):212–224 (2000).
- `ARMSTRONG-1994` — Armstrong L.E. et al. Urinary indices of hydration status. *Int J Sport Nutr*
  4(3):265–279 (1994).
- `GAO-2016` — Gao W. et al. Fully integrated wearable sensor arrays for multiplexed in situ
  perspiration analysis. *Nature* 529:509–514 (2016).
- `KOH-2016` — Koh A. et al. A soft, wearable microfluidic device for the capture, storage, and
  colorimetric sensing of sweat. *Sci Transl Med* 8(366):366ra165 (2016).
- `BANDODKAR-2019` — Bandodkar A.J. et al. Battery-free, skin-interfaced microfluidic/electronic
  systems for simultaneous electrochemical, colorimetric, and volumetric analysis of sweat.
  *Sci Adv* 5(1):eaav3294 (2019).
- `LEACH-1996` — Leach C.S. et al. Regulation of body fluid compartments during short-term
  spaceflight. *J Appl Physiol* 81(1):105–116 (1996). ⚠

**Sleep / circadian / cognition**

- `BARGER-2014` — Barger L.K., Czeisler C.A. et al. Prevalence of sleep deficiency and use of
  hypnotic drugs in astronauts before, during, and after spaceflight. *Lancet Neurol*
  13(9):904–912 (2014). ISS mean sleep 6.09 h vs 8.5 h scheduled.
- `ANCOLI-ISRAEL-2003` — Ancoli-Israel S. et al. The role of actigraphy in the study of sleep and
  circadian rhythms. *Sleep* 26(3):342–392 (2003).
- `AASM-ACTIGRAPHY-2018` — Smith M.T. et al. Use of actigraphy for the evaluation of sleep disorders
  and circadian rhythm sleep-wake disorders: an AASM clinical practice guideline. *J Clin Sleep Med*
  14(7):1231–1237 (2018).
- `WATSON-2015` — Watson N.F. et al. Recommended amount of sleep for a healthy adult: a joint
  consensus statement of the AASM and Sleep Research Society. *Sleep* 38(6):843–844 (2015).
- `DINGES-1985` — Dinges D.F., Powell J.W. Microcomputer analyses of performance on a portable,
  simple visual RT task during sustained operations. *Behav Res Methods Instrum Comput*
  17(6):652–655 (1985).
- `BASNER-2011` — Basner M., Dinges D.F. Maximizing sensitivity of the psychomotor vigilance test
  (PVT) to sleep loss. *Sleep* 34(5):581–591 (2011).
- `BASNER-2015` — Basner M. et al. Development and validation of the Cognition test battery for
  spaceflight. *Aerosp Med Hum Perform* 86(11):942–952 (2015).
- `CZEISLER-1999` — Czeisler C.A. et al. Stability, precision, and near-24-hour period of the human
  circadian pacemaker. *Science* 284(5423):2177–2181 (1999).
- `ZEITZER-2000` — Zeitzer J.M. et al. Sensitivity of the human circadian pacemaker to nocturnal
  light: melatonin phase resetting and suppression. *J Physiol* 526(Pt 3):695–702 (2000).
- `TEUNISSEN-2011` — Teunissen L.P.J. et al. Non-invasive continuous core temperature measurement by
  zero heat flux. *Physiol Meas* 32(5):559–570 (2011). ⚠ agreement limits to confirm.

**Behavioral health**

- `KROENKE-2001` — Kroenke K., Spitzer R.L., Williams J.B.W. The PHQ-9: validity of a brief
  depression severity measure. *J Gen Intern Med* 16(9):606–613 (2001).
- `SPITZER-2006` — Spitzer R.L. et al. A brief measure for assessing generalized anxiety disorder:
  the GAD-7. *Arch Intern Med* 166(10):1092–1097 (2006).
- `MCNAIR-1971` — McNair D.M., Lorr M., Droppleman L.F. Profile of Mood States. Educational and
  Industrial Testing Service (1971).
- `ADAM-2009` — Adam E.K., Kumari M. Assessing salivary cortisol in large-scale, epidemiological
  research. *Psychoneuroendocrinology* 34(10):1423–1436 (2009).
- `BASNER-2013` — Basner M. et al. Mars 520-d mission simulation reveals protracted crew hypokinesis
  and alterations of sleep duration and timing. *PNAS* 110(7):2635–2640 (2013).
- `NASA-BMED` — NASA Human Research Roadmap. Risk of Adverse Cognitive or Behavioral Conditions and
  Psychiatric Disorders (BMed), and its Evidence Report.
  <https://humanresearchroadmap.nasa.gov/>

**Nutrition / metabolic**

- `SMITH-2005` — Smith S.M. et al. The nutritional status of astronauts is altered after long-term
  space flight aboard the International Space Station. *J Nutr* 135(3):437–443 (2005).
- `SMITH-2012` — Smith S.M. et al. Benefits for bone from resistance exercise and nutrition in
  long-duration spaceflight: evidence from biochemistry and densitometry. *J Bone Miner Res*
  27(9):1896–1906 (2012).
- `IHSG-2017` — International Hypoglycaemia Study Group. Glucose concentrations of less than
  3.0 mmol/L should be reported in clinical trials. *Diabetes Care* 40(1):155–157 (2017).
- `ADA-SOC` — American Diabetes Association. Standards of Care in Diabetes (current year),
  *Diabetes Care* — classification/diagnosis and hypoglycaemia sections.
- `HUGHSON-2016` — Hughson R.L. et al. Increased postflight carotid artery stiffness and inflight
  insulin resistance resulting from 6-mo spaceflight in male and female astronauts.
  *Am J Physiol Heart Circ Physiol* 310(5):H628–H638 (2016). ⚠
- `HRF-SLAMMD` — NASA HRP ISS Facilities Overview (HRF-1/HRF-2) — Space Linear Acceleration Mass
  Measurement Device.

**Bone / renal**

- `VASIKARAN-2011` — Vasikaran S. et al. Markers of bone turnover for the prediction of fracture
  risk and monitoring of osteoporosis treatment: a need for international reference standards.
  *Osteoporos Int* 22(2):391–420 (2011).
- `LEBLANC-2000` — LeBlanc A. et al. Bone mineral and lean tissue loss after long duration space
  flight. *J Musculoskelet Neuronal Interact* 1(2):157–160 (2000). ⚠
- `LANG-2004` — Lang T. et al. Cortical and trabecular bone mineral loss from the spine and hip in
  long-duration spaceflight. *J Bone Miner Res* 19(6):1006–1012 (2004). ⚠
- `PEARLE-2014-AUA` — Pearle M.S. et al. Medical management of kidney stones: AUA guideline.
  *J Urol* 192(2):316–324 (2014). ⚠ cut-points.
- `WHITSON-1997` — Whitson P.A. et al. Renal stone risk assessment during Space Shuttle flights.
  *J Urol* 158(6):2305–2310 (1997). ⚠

**Vision / SANS / fluid shift**

- `MADER-2011` — Mader T.H. et al. Optic disc edema, globe flattening, choroidal folds, and hyperopic
  shifts observed in astronauts after long-duration space flight. *Ophthalmology*
  118(10):2058–2069 (2011).
- `LEE-2020-SANS` — Lee A.G. et al. Spaceflight associated neuro-ocular syndrome (SANS) and the
  neuro-ophthalmologic effects of microgravity: a review and an update. *npj Microgravity* 6:7 (2020).
- `MARSHALL-GOEBEL-2019` — Marshall-Goebel K. et al. Assessment of jugular venous blood flow stasis
  and thrombosis during spaceflight. *JAMA Netw Open* 2(11):e1915011 (2019).
- `BAILEY-LOVIE-1976` — Bailey I.L., Lovie J.E. New design principles for visual acuity letter
  charts. *Am J Optom Physiol Opt* 53(11):740–745 (1976).
- `CROSSLAND-2007` — Crossland M., Rubin G. The Amsler chart: absence of evidence is not the same as
  evidence of absence. *Br J Ophthalmol* 91(3):391–393 (2007).
- NASA Human Research Roadmap — SANS risk page and Evidence Report.
  <https://humanresearchroadmap.nasa.gov/Risks/risk.aspx?i=105>

**Radiation**

- `NASA-STD-3001` — NASA Space Flight Human-System Standard, Volume 1: Crew Health. ⚠ career and
  annual dose limits — see §8 item 1.
- `NASEM-2021` — National Academies. Space Radiation and Astronaut Health: Managing and
  Communicating Cancer Risks (2021).
- `BERGER-2023` — Berger T. et al. The importance of time-resolved personal dosimetry in space: the
  ISS Crew Active Dosimeter. *Life Sciences in Space Research* (2023).
- `ZEITLIN-2013` — Zeitlin C. et al. Measurements of energetic particle radiation in transit to Mars
  on the Mars Science Laboratory. *Science* 340(6136):1080–1084 (2013). ⚠ dose rate.
- `HASSLER-2014` — Hassler D.M. et al. Mars' surface radiation environment measured with the Mars
  Science Laboratory's Curiosity rover. *Science* 343(6169):1244797 (2014). ⚠ dose rate.

**Environmental / CO₂**

- `LAW-2014` — Law J. et al. Relationship between carbon dioxide levels and reported headaches on
  the International Space Station. *J Occup Environ Med* 56(5):477–483 (2014). ⚠ thresholds.
- `OCHMO-TB` — NASA OCHMO Medical Technical Briefs: Bone Loss (OCHMO-TB-030), Radiation Protection
  (OCHMO-TB-020), Exercise Overview (OCHMO-TB-031), Spaceflight Experience & Medical Care
  (OCHMO-TB-033). <https://www.nasa.gov/ochmo/>

**Hardware precedent**

- CHeCS ISS Medical Hardware Catalog v10.0, NASA NTRS 20110022379 — BP/ECG unit, HMS contents.
- `LEE-2020` — Lee K. et al. Mechano-acoustic sensing of physiological processes and body motions
  via a soft wireless device placed at the suprasternal notch. *Nat Biomed Eng* 4:148–158 (2020).
