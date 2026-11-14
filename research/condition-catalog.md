# Condition catalog: Case pool by Environment

Answers [#6](https://github.com/spowers42/reachy-emt-history/issues/6): which conditions belong in the Case pool for each Environment (Urban/Suburban, Rural, Wilderness), with a suggested minimum Provider Level for each.

Scope: medical presentations that can be worked out from the history, plus a few trauma-adjacent ones where the history does most of the work. Physical exam and treatment are out of scope, per map #1. Vitals stay available as Case data on request.

Terminology: "Altered Mental Status" and "Acute Mountain Sickness" are always written out. The bare abbreviation is never used.

## Sources and how each is used

| Tag | Source | Used for |
|---|---|---|
| **[NEMS]** | NHTSA, *2021 National EMS Education Standards* (v2.2). Medicine pp. 38-44, Trauma > Environmental Emergencies p. 47, Special Patient Populations > Obstetrics p. 50. Official URL: https://www.ems.gov/assets/EMS_Education-Standards_2021_FNL.pdf. ems.gov returned 403 to automated fetches, so the text was read from a byte-identical republication at https://www.safehomealabama.gov/wp-content/uploads/2021/11/EMS_Education_Standards_2021_v22.pdf | Which conditions each Provider Level is taught, and to what breadth/depth |
| **[NREMT]** | NREMT, *Examination Content* handbook page (footer "Version 2026.3"): https://nremt.org/Handbooks/Certification-Examinations/Examinations-Content | How heavily each level's exam weights medical content |
| **[NEMSIS]** | NEMSIS TAC, *National EMS Data Report 2022* (served as `NEMSIS-End-of-Year-Report-2023.pdf`): https://nemsis.org/wp-content/uploads/2025/02/NEMSIS-End-of-Year-Report-2023.pdf, p. 7 (urbanicity) and p. 8 (primary impression) | Real-world frequency and Environment mix |
| **[WMS-*]** | Wilderness Medical Society Clinical Practice Guidelines, *Wilderness & Environmental Medicine*. Titles and citations were confirmed on PubMed (list below). | Which Wilderness Conditions to include |
| **[AAFP-Alt]** | AAFP Practice Guidelines summary of the WMS 2019 altitude guideline, *Am Fam Physician* 2020;101(8):505: https://www.aafp.org/pubs/afp/issues/2020/0415/p505.html | History discriminators for altitude illness |

WMS guidelines used (PubMed search on 2026-11-14):
- **[WMS-Alt]** Acute Altitude Illness: 2024 Update. WEM 2024;35(1_suppl):2S-19S. PMID 37833187
- **[WMS-Heat]** Heat Illness: 2024 Update. WEM 2024;35(1_suppl):112S-127S. PMID 38425235
- **[WMS-Hypo]** Accidental Hypothermia: 2019 Update. WEM 2019;30(4S):S47-S69. PMID 31740369. This was the newest hypothermia guideline I could find.
- **[WMS-Frost]** Frostbite: 2024 Update. WEM 2024;35(2):183-197. PMID 38577729
- **[WMS-Drown]** Drowning: 2024 Update. WEM 2024;35(1_suppl):94S-111S. PMID 38379489
- **[WMS-Light]** Lightning Injuries: 2014 Update. WEM 2014;25(4 Suppl):S86-95. PMID 25498265. This was the newest lightning guideline I could find.
- **[WMS-Viper]** Pit Viper Envenomations in the US and Canada: 2026 Update. WEM 2026;37(4):506-520. PMID 42217172
- **[WMS-EAH]** Exercise-Associated Hyponatremia: 2019 Update. WEM 2020;31(1):50-62. PMID 32044213
- **[WMS-Water]** Water Treatment: 2024 Update. WEM 2024;35(1_suppl):45S-66S. PMID 38379474
- **[WMS-Aval]** Avalanche and Nonavalanche Snow Burial: 2024 Update. WEM 2024;35(1_suppl):20S-44S. PMID 37945433
- **[WMS-Eye]** Eye Injuries and Illnesses: 2024 Update. WEM 2024;35(1_suppl):67S-77S. PMID 38425236
- **[WMS-Pain]** Acute Pain in Austere Environments: 2024 Update. WEM 2024;35(2):198-218. PMID 38651342

**Fetch limits:** PubMed abstract pages would not load for automated fetches. Only the titles and citations above were confirmed. The clinical detail on Wilderness Conditions comes from [AAFP-Alt] for altitude illness and from standard EMS and wilderness teaching for everything else. Those rows are marked *(clinical background)* and need review by a subject-matter expert (SME) before they become prompt content.

## Key findings

1. **NREMT does not publish a condition list.** Since the EMT exam restructure, the EMT exam is built around phase-of-care domains: Scene Size-Up and Safety 15-19%, Primary Assessment 39-43%, Secondary Assessment 5-9%, Patient Treatment and Transport 20-24%, Operations 10-14%. The AEMT and Paramedic exams still have a "Medical/Obstetrics/Gynecology" domain, at 25-29% and 24-28% [NREMT]. NREMT says its content is guided by the Scope of Practice Model and the Education Standards [NREMT]. **The Education Standards are therefore the authoritative source for which conditions each level should know.**
2. **The Standards show what each level learns, and how deeply.** Each topic carries a (Breadth, Depth) code: Breadth is Simple, Foundational or Comprehensive; Depth is Simple, Fundamental or Complex [NEMS, legend p. 27]. Example: Acute coronary syndrome is EMT (F,F), AEMT (C,F), Paramedic (C,C). Several topics appear only at Paramedic: bowel obstruction, pancreatitis, meningitis, adrenal and thyroid disease, ectopic pregnancy at (C,C), and toxidromes [NEMS pp. 38-44, 50]. This gives a defensible rule for setting minimum Provider Level (next section).
3. **NEMSIS uses the same Environment buckets.** The 2022 national data codes each incident's location as Urban, Suburban, Rural or Wilderness, using USDA Urban Influence Codes. Among 911 calls with patient contact, the split was Urban 87.5%, Rural 5.9%, Suburban 5.3%, Wilderness 1.4% [NEMSIS p. 7]. **Note:** NEMSIS "Wilderness" is a USDA geographic code, not an austere or backcountry setting. Our Wilderness Environment means remote, delayed-care settings. The two only partly overlap.
4. **Real 911 impressions are dominated by vague complaints.** The top primary impressions were Weakness 14.2%, Injury unspecified 9.9%, Change in Mental Status 7.4%, Acute Pain NEC 6.5%, Generalized Abdominal Pain 6.4%, Syncope and Collapse 5.6%, Chest Pain 4.9%, Respiratory Distress 4.7%, Mental Illness 3.8%, Back Pain 3.6%, Shortness of Breath 3.1%, Light-Headedness 3.0%, Anxiety 2.9% (top-20 share) [NEMSIS p. 8]. **Implication:** the Case pool should weight Weakness, Syncope, Altered Mental Status and abdominal pain heavily. These are the complaints where history taking decides the Field Impression.
5. **The Education Standards already cover the core environmental emergencies.** Drowning, temperature-related illness, bites and envenomations, and lightning injury are taught at EMT (F,F), AEMT (F,F) and Paramedic (C,C) [NEMS p. 47]. Altitude illness, exercise-associated hyponatremia, frostbite detail and avalanche burial are *not* named in the Standards. They come only from the WMS guidelines, which is the basis for treating them as "Wilderness Conditions."

## Rule for suggested minimum Provider Level

The rule, for the Case generator and the Debrief:

- **EMT Student / EMT:** the condition appears in the EMT column at Breadth F or C. The Trainee is expected to recognize the presentation and complete SAMPLE/OPQRST. Students get complete, in-order coverage, per map #1.
- **AEMT Student / AEMT:** the condition is in the EMT column at (S,S) only, or the AEMT column raises it to C breadth (e.g. ACS, diabetic emergencies, asthma/COPD, pulmonary edema). AEMT Debriefs can expect a short Differential.
- **Paramedic Student / Paramedic:** the condition appears only in the Paramedic column, or at Paramedic (C,C) where EMT and AEMT are (S,S). Debriefs expect a ranked Differential.
- **Higher levels inherit everything below them.** A Paramedic should still get "easy" Cases, with a deeper Differential expected in the Debrief.
- Wilderness Conditions not named in the Standards follow a separate default, explained in the Wilderness section.

Abbreviations in the tables: **CC** = chief complaint. **Discriminators** = the SAMPLE/OPQRST answers that point toward the condition. **Min level** = suggested minimum Provider Level, followed by the (Breadth, Depth) code for that level from [NEMS].

---

## 1. Core pool (all Environments)

Every Environment draws from this pool. Environment-specific weighting and extras follow in sections 2-4.

### Cardiovascular [NEMS p. 39]

| Condition | Typical CC | Key history discriminators | Common mimics | Min level |
|---|---|---|---|---|
| Acute coronary syndrome | "Chest pressure," indigestion, weakness (elderly, diabetic, women) | O: exertion or rest; P: not changed by position or breathing; Q: pressure or squeezing; R: arm, jaw, back; T: >15 min; S: diaphoresis, nausea, dyspnea; M: nitro, antiplatelets; P: CAD, DM, HTN, smoking | GERD, biliary colic, anxiety, PE, aortic dissection, pericarditis | EMT (F,F) |
| Heart failure / pulmonary edema | Shortness of breath, "can't lie flat" | Orthopnea, paroxysmal nocturnal dyspnea, more pillows, weight gain or ankle swelling, missed diuretic, salty meal; P: CHF, MI | COPD exacerbation, pneumonia, ACS | EMT (F,F); AEMT (C,F) |
| Aortic dissection / AAA | Sudden tearing chest or back pain; abdominal or back pain with syncope (AAA) | O: abrupt, maximal at onset; Q: tearing or ripping; R: between shoulder blades or migrating; P: HTN, Marfan, stimulant use; older male smoker (AAA) | ACS, kidney stone (AAA), musculoskeletal back pain | EMT (F,F) |
| Pulmonary embolism (thromboembolism) | Sudden shortness of breath, pleuritic chest pain | Recent surgery, long travel, immobilization, estrogen/OCP, pregnancy or postpartum, cancer, prior DVT; unilateral calf pain; hemoptysis | Pneumonia, pneumothorax, ACS, anxiety | EMT (F,F) |
| Hypertensive emergency | Headache, blurred vision | Missed antihypertensives, stimulants; headache with vision change or chest pain | Stroke, preeclampsia (if pregnant), migraine | EMT (S,S); AEMT (F,S); **Paramedic (C,C)** |
| Symptomatic dysrhythmia | "Heart racing," palpitations, near-syncope | Sudden on/off (SVT), caffeine or stimulants, new meds, electrolyte history (dialysis, diuretics); P: WPW, prior ablation | Anxiety/panic, hypoglycemia, hyperthyroid, dehydration | Paramedic (C,C) ("cardiac rhythms," "WPW, Brugada, long QT") |
| Pericarditis | Sharp chest pain, worse lying flat | Better leaning forward, recent viral illness | ACS, PE | Paramedic (F,F) |

### Respiratory [NEMS p. 44]

| Condition | Typical CC | Key history discriminators | Common mimics | Min level |
|---|---|---|---|---|
| Asthma exacerbation | "Can't catch my breath," wheeze | Trigger (allergen, URI, cold, exercise), rescue inhaler use frequency, prior intubation or ICU stay, steroid courses | Anaphylaxis, foreign body, pulmonary edema | EMT (F,F) |
| COPD exacerbation | Worse shortness of breath, more sputum | Smoking history, home O2, sputum color change, recent URI, missed meds | CHF, pneumonia, PE, pneumothorax | EMT (F,F) |
| Pneumonia | Cough, fever, shortness of breath | Days of fever, chills, productive cough, pleuritic pain; elderly may present only with Altered Mental Status or Weakness | PE, CHF, sepsis from another source | EMT (F,F) |
| Spontaneous pneumothorax | Sudden one-sided chest pain and shortness of breath | Tall thin young male, smoker, COPD/bullae, onset at rest | PE, ACS, musculoskeletal pain | EMT (F,F) |
| Upper airway: croup / epiglottitis / foreign body | Child "barky cough"; drooling; choking episode | Croup: night onset, URI, age 6 mo-3 yr; epiglottitis: rapid, high fever, unvaccinated; foreign body: witnessed choking while eating or playing | Each other, anaphylaxis, peritonsillar abscess | EMT (F,F) |
| Bronchiolitis (infant) | Infant breathing fast, poor feeding | RSV season, URI prodrome, prematurity | Asthma, pneumonia, foreign body | EMT (F,F) |

### Neurology [NEMS p. 42]

| Condition | Typical CC | Key history discriminators | Common mimics | Min level |
|---|---|---|---|---|
| Stroke / TIA | Family: "face drooping," "talking funny," sudden weakness | **Last known well** time; sudden onset; anticoagulants; afib, HTN, prior stroke; TIA = resolved symptoms | **Hypoglycemia**, post-ictal (Todd's) paralysis, Bell's palsy, complex migraine, intoxication | EMT (F,F) |
| Seizure | "Had a seizure," found down | Witness description and duration, incontinence or tongue bite, post-ictal confusion; P: epilepsy, missed doses, alcohol withdrawal, recent head injury, pregnancy (eclampsia), fever (child) | Syncope with myoclonus, hypoglycemia, pseudo-seizure, dysrhythmia | EMT (F,F) |
| Headache (incl. subarachnoid hemorrhage, migraine) | Headache | "Worst of my life," thunderclap onset, exertion or sex at onset (SAH); aura, prior identical pattern (migraine); fever + stiff neck (meningitis); CO exposure | Hypertensive emergency, meningitis, CO poisoning, sinusitis | EMT (F,F); Paramedic (C,C) |
| Altered Mental Status (undifferentiated) | "Not acting right" / Change in Mental Status | Baseline mental status from family; onset speed; med changes; glucose history; ingestion; infection signs; fall or head strike | **The workup itself is the Differential**: AEIOU-TIPS-style causes (hypoglycemia, stroke, sepsis/UTI in elderly, overdose, post-ictal, hypoxia, head injury) | EMT ("Decreased level of responsiveness" (S,S)); Differential depth rises with level |
| Dementia vs. delirium | Elderly "more confused" | Acute vs. gradual change, new med, UTI symptoms, constipation or retention, fall | Stroke, hypoglycemia, sepsis | EMT (S,S) |
| Brief Resolved Unexplained Event (BRUE) | Parent: "baby stopped breathing / turned blue" | Age < 1 yr, duration < 1 min, resolved; feeding at time (reflux), prior episodes | Seizure, choking, abuse | EMT (F,F) |

### Endocrine / metabolic [NEMS p. 40]

| Condition | Typical CC | Key history discriminators | Common mimics | Min level |
|---|---|---|---|---|
| Hypoglycemia | Altered Mental Status, sweaty, "acting drunk" | Insulin or sulfonylurea; missed meal, exercise, alcohol; rapid onset; prior episodes | Stroke, intoxication, seizure, psychiatric | EMT (F,F); AEMT (C,F) |
| DKA / hyperglycemia (HHS in elderly) | Nausea, vomiting, abdominal pain, "breathing fast" | Polyuria, polydipsia over days; missed insulin or pump failure; infection; new-onset in children | Gastroenteritis, sepsis, appendicitis, toxic ingestion | EMT (F,F) |
| Adrenal crisis / thyroid storm | Weakness, vomiting / palpitations, fever | Chronic steroid use stopped abruptly; known thyroid disease, missed meds | Sepsis, dysrhythmia, stimulant toxicity | **Paramedic** (S,S): low priority |

### Immunology & infectious [NEMS p. 41]

| Condition | Typical CC | Key history discriminators | Common mimics | Min level |
|---|---|---|---|---|
| Allergic reaction / anaphylaxis | Hives, throat tight, shortness of breath | Exposure (food, sting, new med) minutes to hours before; prior reactions; carries epi; ACE inhibitor (angioedema) | Asthma, panic attack, vasovagal, angioedema | EMT (F,F); AEMT (C,C) |
| Sepsis | Weakness, fever, Altered Mental Status | Infection source (UTI, pneumonia, wound, line), immunocompromise, chemo, nursing-home resident, recent procedure | Dehydration, DKA, adrenal crisis, pneumonia | EMT (S,S); AEMT (F,F); Paramedic (C,C) |
| Meningitis | Headache, fever, stiff neck | Dorm or barracks living, rash, rapid onset, photophobia; unvaccinated | SAH, migraine, flu | **Paramedic** (C,C) |

### Abdominal / GI / GU / GYN [NEMS pp. 38, 40, 50]

| Condition | Typical CC | Key history discriminators | Common mimics | Min level |
|---|---|---|---|---|
| GI hemorrhage (upper/lower) | Vomiting blood / "coffee grounds," black stools, Weakness, Syncope | NSAIDs, anticoagulants, alcohol, liver disease, prior ulcer, varices | Iron/bismuth (dark stools), epistaxis swallowed blood | EMT (F,F) |
| Kidney stone | Severe flank pain, "can't sit still" | Colicky flank-to-groin pain, hematuria, prior stones | **AAA (older patient)**, pyelonephritis, appendicitis, ectopic | EMT (S,S); AEMT (F,S); Paramedic (C,C) |
| Biliary colic / cholecystitis | RUQ or epigastric pain | After fatty meal, radiates to right shoulder, recurrent; "female, forty, fertile" | ACS (inferior MI), pancreatitis, peptic ulcer | **Paramedic** ("Liver and biliary" (F,F)) |
| Pancreatitis | Epigastric pain to back | Alcohol, gallstones, vomiting, better leaning forward | ACS, AAA, ulcer | **Paramedic** (S,S) |
| Bowel obstruction | Crampy abdominal pain, vomiting | No flatus or stool, prior abdominal surgery, hernia | Gastroenteritis, constipation | **Paramedic** (C,C) |
| Appendicitis / peritonitis | Abdominal pain | Periumbilical pain migrating to RLQ, anorexia, fever | Ectopic, ovarian torsion, kidney stone, gastroenteritis | EMT ("other GI to be determined locally" (S,S)); Paramedic ("Peritonitis" (S,S)) |
| Dialysis complication | Weakness, shortness of breath | Missed dialysis session, fluid overload, potassium (dysrhythmia risk), access bleeding | CHF, ACS | EMT (S,S); AEMT (F,S); Paramedic (C,C) |
| Vaginal bleeding (non-pregnant) / ovarian emergency | Pelvic pain, heavy bleeding | LMP, pad count, sudden one-sided pain (torsion), STI risk | Ectopic, appendicitis, kidney stone | EMT ("Vaginal bleeding" (F,F)); Paramedic ("Ovarian emergencies" (F,F)) |

### Obstetrics [NEMS p. 50]

| Condition | Typical CC | Key history discriminators | Common mimics | Min level |
|---|---|---|---|---|
| Ectopic pregnancy | Lower abdominal pain, Syncope, spotting | **LMP / missed period**, possibility of pregnancy, prior ectopic, PID, IUD, IVF; shoulder-tip pain | Appendicitis, ovarian cyst/torsion, miscarriage, kidney stone | EMT ("vaginal bleeding in the pregnant patient" (S,S)); AEMT (F,F); Paramedic (C,C) |
| Spontaneous abortion / miscarriage | Bleeding and cramping in early pregnancy | Gestational age, passage of tissue | Ectopic | AEMT (F,F) |
| Placenta previa / abruption | Third-trimester bleeding (painless vs. painful) | Painless bright red bleeding (previa) vs. painful, rigid abdomen, trauma, cocaine, HTN (abruption) | Labor, uterine rupture | AEMT (F,F) |
| Preeclampsia / eclampsia | Headache, visual change, swelling; seizure | > 20 weeks or postpartum, new HTN, RUQ pain | Hypertensive emergency, seizure disorder, migraine | AEMT (F,F); Paramedic (C,C) |
| Imminent / normal delivery | "My water broke," contractions | G/P, gestational age, contraction interval, urge to push, prior fast labors, prenatal care | (History-only Case is fine; delivery itself is out of scope) | EMT (F,F) |
| Hyperemesis gravidarum, postpartum depression | Vomiting in early pregnancy; low mood after birth | Gestational age, weight loss; weeks postpartum, sleep, thoughts of self-harm | Gastroenteritis, DKA; psychiatric | **Paramedic** (S,S) |

### Toxicology [NEMS p. 44]

| Condition | Typical CC | Key history discriminators | Common mimics | Min level |
|---|---|---|---|---|
| Opioid toxicity | Found unresponsive / Altered Mental Status | Drug paraphernalia, prior overdose, recent release from jail or rehab (lost tolerance), chronic pain meds | Hypoglycemia, stroke, head injury, sedative overdose | EMT (S,S) |
| Alcohol intoxication / withdrawal | "Drunk," or shaking/sweating/seeing things | Last drink time, prior withdrawal seizures or DTs | **Hypoglycemia**, head injury, hepatic encephalopathy, sepsis | EMT (F,F); Paramedic (C,C) |
| Intentional / accidental ingestion (poisons) | "Took pills," child got into cabinet | What, how much, when, why (suicide screen), co-ingestants | Many; toxidrome depth is Paramedic | EMT (F,F) |
| Carbon monoxide poisoning | Headache, nausea, "flu" in multiple people | **Others in the house sick**, better when away from home, heater/generator/car in garage, winter | Viral illness, migraine, food poisoning | EMT (S,S); **Paramedic (C,C)** |
| Toxidromes (sympathomimetic, anticholinergic, cholinergic, serotonin syndrome) | Agitation, hyperthermia, Altered Mental Status | Specific agent, new serotonergic med, stimulant use; farm chemicals for cholinergic | Sepsis, heat stroke, thyroid storm, psychiatric | **Paramedic** (C,C) |

### Psychiatric / behavioral [NEMS p. 43]

| Condition | Typical CC | Key history discriminators | Common mimics | Min level |
|---|---|---|---|---|
| Suicidal ideation / depression | "I want to die," family worried | Plan, means, prior attempts, substance use, recent loss | Intoxication, medical causes of Altered Mental Status | EMT (F,F); AEMT (C,C) |
| Anxiety / panic attack | Chest tightness, tingling hands, "can't breathe" | Prior identical episodes, stressor, perioral tingling | **ACS, PE, hypoglycemia, hyperthyroid, asthma**: a mimic Case is high value | EMT (F,F) |
| Acute psychosis | Hearing voices, paranoid | Known psychiatric disorder, missed antipsychotics, drug use | Delirium, hypoglycemia, stimulant toxicity, head injury | EMT (F,F) |

### Hematology & other [NEMS pp. 39-40, 42]

| Condition | Typical CC | Key history discriminators | Common mimics | Min level |
|---|---|---|---|---|
| Sickle cell crisis | Severe pain (chest, back, limbs) | Known sickle cell disease, triggers (cold, dehydration, infection, altitude); fever; chest pain = acute chest syndrome | Drug-seeking (beware bias), ACS, PE | EMT (S,S); AEMT (F,F) |
| Syncope (undifferentiated) | "Passed out," Syncope and Collapse | Prodrome (vasovagal), position (orthostatic), exertional syncope (cardiac: red flag), palpitations, new BP meds, GI bleed, pregnancy | Seizure, hypoglycemia, dysrhythmia, ectopic, AAA | EMT (no single row; spans cardiovascular and neurology) |
| Weakness (undifferentiated, elderly) | "Just weak," "not feeling right" | **Highest-frequency NEMSIS impression** [NEMSIS p. 8]. Hidden causes: UTI/sepsis, silent ACS, dehydration, hypoglycemia, electrolyte problems, GI bleed, stroke | All of the above | EMT. Use as a "find the real diagnosis" Case; best suited to AEMT and above |
| Epistaxis | Nosebleed | Anticoagulants, HTN, duration, posterior bleed (runs down throat) | GI bleed | EMT (S,S) |
| Non-traumatic back pain | Back pain | Red-flag history: IV drug use and fever (epidural abscess), cancer, saddle anesthesia / incontinence (cauda equina), AAA risk | AAA, kidney stone, pyelonephritis | Paramedic ("Disorders of the spine" (F,F)) |

### History-heavy, trauma-adjacent

| Condition | Why it fits | Key history discriminators | Mimics | Min level |
|---|---|---|---|---|
| Elderly ground-level fall | NEMSIS "Injury unspecified" 9.9% is the #2 impression [NEMSIS p. 8]. The history question is *why* they fell | Syncope before the fall vs. a trip; anticoagulants (head bleed risk); long lie | Syncope, stroke, hypoglycemia, dysrhythmia, UTI | EMT |
| Minor head injury on anticoagulants | History determines risk | Mechanism, LOC, vomiting, anticoagulant, headache worsening over hours | Stroke, intoxication | EMT ("Life threats"/"Skull fractures" in Trauma) |
| Intimate partner violence / elder or child maltreatment | History inconsistent with the injury | Story changes, delay in seeking care, caregiver answers for the patient | Accidental injury | EMT ("Recognizing and reporting abuse" (S,S)); Paramedic expanded (S,S) |

---

## 2. Urban/Suburban

**Pool:** the full core pool (section 1). Frequencies should loosely follow NEMSIS, since Urban is 87.5% of real 911 activations [NEMSIS p. 7].

Urban/Suburban adds plausible circumstances to the core pool, not new diagnoses. These circumstance hooks are *(authoring judgment)*. The Standards say "determined locally," so the pool is meant to be locally tunable:
- Opioid and stimulant overdoses, sympathomimetic toxidrome; overdose after recent release from incarceration.
- Nursing-home and assisted-living calls: Weakness, Altered Mental Status from UTI/sepsis, dementia vs. delirium, falls.
- Dialysis-center and missed-dialysis Cases.
- Psychiatric emergencies, homelessness and poverty (listed under Patients with Special Challenges [NEMS p. 51]).
- Environmental emergencies in a city: heat illness during a heat wave in an elderly person without AC; CO poisoning from a generator during an outage; cold exposure in a homeless patient (temperature-related illness, EMT (F,F) [NEMS p. 47]).

## 3. Rural

**Pool:** the core pool, plus circumstances that change the history. NEMSIS Rural is 5.9% of 911 activations [NEMSIS p. 7]. None of the sources lists rural-specific *conditions*, so the additions below are *(authoring judgment, needs SME review)*:
- **Longer time since onset.** Delayed or prolonged transport means Cases where symptom duration and "what have you taken for it" matter more. Example: ACS that started 6 hours ago and the patient "waited to see if it got better."
- **Agricultural toxic exposures.** Organophosphate/carbamate pesticide exposure (cholinergic toxidrome, **Paramedic** (C,C) [NEMS p. 44]); CO from barn or shop heaters, engines in enclosed spaces. Silo gas and anhydrous ammonia exposure are other options.
- **Bites and envenomations.** Pit viper bite (EMT (F,F) [NEMS p. 47]; [WMS-Viper]), black widow spider bite, bee sting with anaphylaxis.
- **Heat illness in outdoor laborers**, cold exposure, lightning injury.
- **Tick-borne illness** (Lyme, Rocky Mountain spotted fever). The Standards list these under "vector-borne diseases," **Paramedic** (S,S) [NEMS p. 41].
- **Older population, fewer resources:** missed medications because of pharmacy distance, untreated chronic disease (CHF, COPD, DM).
- **Rural maternal care:** imminent delivery, preeclampsia with no prenatal care.

## 4. Wilderness

**Pool:** a reduced core pool (conditions that plausibly occur on a hike or expedition, e.g. ACS on exertion, asthma, anaphylaxis, hypoglycemia, seizure, ectopic, appendicitis), plus the Wilderness Conditions below.

**Wilderness context the history should surface** *(authoring judgment)*: elevation and ascent profile, last water source, food and fluid intake, time since symptom onset, sleeping altitude, gear and clothing, exposure duration, companions with the same symptoms, and how far care is.

**Default level rule for conditions not named in the Standards:** EMT for recognition and a full history. Paramedic Students and Paramedics are expected to discriminate *between* Wilderness Conditions in the Differential (e.g. HACE vs. hyponatremia vs. hypoglycemia). *This rule is a judgment call.* NREMT certification does not include wilderness credentials (WEMT, WFR); see Open questions.

| Wilderness Condition | Typical CC | Key history discriminators | Common mimics | Min level | Source |
|---|---|---|---|---|---|
| **Acute Mountain Sickness** | Headache, nausea, fatigue at altitude | Recent ascent; risk high for unacclimatized people > 2,500 m, possible from ~2,000 m. **Sleeping altitude matters more than altitude reached awake.** Rapid ascent (e.g. Kilimanjaro in < 7 days); prior altitude illness raises risk. Symptoms: headache, nausea/vomiting, fatigue, dizziness, insomnia. WMS uses no fixed altitude threshold for diagnosis | Dehydration, exhaustion, hangover, CO poisoning (stove in tent), migraine, **exercise-associated hyponatremia**, viral illness | EMT | [WMS-Alt]; [AAFP-Alt] |
| **High Altitude Cerebral Edema (HACE)** | Companions: "stumbling," "confused" | Usually progresses from Acute Mountain Sickness. **Ataxia, confusion, Altered Mental Status**; typically higher altitude unless HAPE is also present | Hypoglycemia, hyponatremia, hypothermia, stroke, intoxication, head injury | EMT recognition; Paramedic Differential | [WMS-Alt]; [AAFP-Alt] |
| **High Altitude Pulmonary Edema (HAPE)** | "Can't keep up," shortness of breath, cough | Reduced exercise tolerance and exertional dyspnea first, then dyspnea at rest, cough, pink frothy sputum. Prior HAPE is a strong risk factor. Can occur without Acute Mountain Sickness *(clinical background)* | Pneumonia, asthma, PE (dehydration + immobility), bronchitis ("Khumbu cough") | EMT | [WMS-Alt]; [AAFP-Alt] |
| **Heat exhaustion / heat stroke** | Weakness, collapse, confusion in heat | Exertion in heat and humidity, fluid intake, acclimatization, meds (diuretics, anticholinergics, stimulants). **Altered Mental Status separates heat stroke from heat exhaustion** *(clinical background)* | **Exercise-associated hyponatremia**, hypoglycemia, cardiac event, sympathomimetic toxidrome | EMT (F,F) | [NEMS p. 47]; [WMS-Heat] |
| **Exercise-associated hyponatremia** | Nausea, headache, confusion after long exertion | Endurance event or long hike, **drinking large volumes of water**, weight gain or "puffy," no salt intake *(clinical background)* | Heat illness, HACE, Acute Mountain Sickness, hypoglycemia | EMT recognition (not named in [NEMS]); Paramedic Differential | [WMS-EAH] |
| **Accidental hypothermia** | Shivering, clumsy, "acting strange" | Exposure duration, wet clothing, wind, immersion, alcohol, exhaustion, age extremes; shivering stops as it worsens *(clinical background)* | Hypoglycemia, intoxication, head injury, stroke, HACE | EMT (F,F) | [NEMS p. 47]; [WMS-Hypo] |
| **Frostbite** | Numb, white/waxy fingers or toes | Cold exposure duration, wet or tight boots, prior frostbite, rewarming and **refreezing** history *(clinical background)* | Frostnip, non-freezing cold injury (trench foot), Raynaud's | EMT ("temperature-related" (F,F)) | [WMS-Frost] |
| **Lightning injury** | Found down after storm, confusion, ear pain | Storm exposure, companions affected, transient paralysis (keraunoparalysis), amnesia, tympanic membrane rupture *(clinical background)* | Seizure, cardiac arrest from other cause, fall trauma | EMT (F,F) | [NEMS p. 47]; [WMS-Light] |
| **Drowning (submersion), post-rescue** | Coughing, shortness of breath after submersion | Submersion duration, water temperature, coughing that persists. WMS 2024 says modifiers like "near," "dry," "secondary" drowning should not be used | Hypothermia, spinal injury from diving, seizure as cause | EMT (F,F) | [NEMS p. 47]; [WMS-Drown] |
| **Pit viper envenomation** | Snakebite, pain and swelling | Time of bite, snake description (do not encourage capture), progression of swelling, bleeding gums, metallic taste, anticoagulants *(clinical background)* | Dry bite, anxiety reaction, allergic reaction | EMT (F,F) | [NEMS p. 47]; [WMS-Viper] |
| **Other bites/stings (bee/wasp anaphylaxis, black widow, scorpion)** | Pain, cramping, hives | Exposure, prior reactions, epi auto-injector | ACS or acute abdomen (black widow can mimic) *(clinical background)* | EMT (F,F) | [NEMS p. 47] |
| **Avalanche / snow burial** (survivor, history from companions) | Shortness of breath, cold | Burial duration, air pocket, hypothermia | Trauma, hypothermia | EMT, low priority (trauma-heavy) | [WMS-Aval] |
| **Waterborne GI illness** (e.g. giardiasis) | Diarrhea, cramps, bloating days into a trip | Untreated water source, onset days later *(clinical background)* | Food poisoning, appendicitis, dehydration | EMT, low priority | [WMS-Water] |
| **Eye: snow blindness (UV keratitis)** | Painful gritty eyes hours after snow or glacier travel | No sunglasses, delayed onset *(clinical background)* | Foreign body, acute glaucoma | EMT, low priority | [WMS-Eye] |
| **CO poisoning in tent/snow cave** | Headache, nausea in group | Stove used inside a closed tent | **Acute Mountain Sickness** (a valuable mimic Case) | EMT (S,S); Paramedic (C,C) | [NEMS p. 44] |

Not included (trauma-dominant, exam-driven): spinal cord protection (WMS Spinal Cord Protection: 2024 Update, PMID 38379496) and wound management. These fail the history-taking scope test.

---

## Suggested pool weighting (draft for #7)

*(Authoring judgment, informed by [NEMSIS] frequencies)*: about 60% "bread-and-butter" complaints (chest pain, dyspnea, Altered Mental Status, abdominal pain, Syncope, Weakness, diabetic, allergic, stroke, seizure); about 25% mimic-rich Cases for higher Provider Levels; about 15% Environment-specific additions. Wilderness may want more, e.g. about 40% Wilderness Conditions.

## Open questions surfaced

1. **Wilderness Provider Level ladder.** NREMT levels don't certify wilderness competence. Should Wilderness Cases assume a WEMT/WFR-style add-on, or stay at the "EMT recognition / Paramedic Differential" default above?
2. **NEMSIS "Wilderness" ≠ our Wilderness.** Should the app's Environment definitions in `GLOSSARY.md` state that Wilderness means remote, delayed-care settings, not the USDA code?
3. **"Determined locally" conditions.** The Standards leave room for local additions. Should the Case pool be configurable per program or region, e.g. desert vs. alpine, or regional envenomations?
4. **Pediatric and geriatric weighting.** The Standards fold these into every section. Should age be a separate random axis (#7)?
5. **SME review.** The *(clinical background)* rows need an EMS/wilderness educator to check them before they are used as prompt content. Primary-text verification of WMS recommendations was not possible here because PubMed abstracts were not fetchable.
6. **Sensitive topics.** Should sexual assault, IPV, suicide and maltreatment Cases be opt-in, given that this is a voice robot used for low-stakes practice?
