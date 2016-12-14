Currently, our goal is to improve existing Synthea modules and to guide the development of new modules and claims data. We are seeking more information and assistance with the following:

* [Existing Modules](#existing-modules)   
* [New Modules](#new-modules)
* [Claims Data](#claims-data)

***

# Existing Modules
**We are seeking review of individual modules by medical students and medical professionals.** Full-size, graphical representations of each module are available in our [Module Gallery](https://github.com/synthetichealth/synthea/wiki/Module-Gallery).

W need **quantitative** feedback. Our models are driven by statistics, incidence, and prevalence rates. "How much?", "How often?", "At what ages?", and "When, historically?" are all good questions to ask. For starters, here are some questions about existing modules that we need answered:

## Asthma

See a visual representation of this module [here](https://synthetichealth.github.io/synthea/graphviz/Asthma.png).

#### What does a "typical" asthma patient look like?
* How often do they have asthma attacks?
* What medicines are they taking, in what doses, and how often?
* How often do they visit a doctor or an emergency department?

#### What is the prevalence of different asthma severities?
* The prevalence of intermittent. mild, moderate, and severe cases?

#### What are the typical tests and observations used to evaluate a patient's asthma?
* FEV1/FVC test, PEF test, others?
* What are the results of these tests?
* Can you map these to specific [LOINC codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#loinc-codes)?

#### What are the significant, quantitative risk factors for asthma?
* For example: race, obesity, smoking, gender?
* Can these be quantified? (e.g. "smoking increases your risk of asthma by 50%")

#### What does asthma claims data look like?
* Cost of outpatient visits, ED visits?
* Cost of medications?


## Allergic Rhinitis

See a visual representation of this module [here](https://synthetichealth.github.io/synthea/graphviz/Allergic%20Rhinitis.png).

#### What does the comorbidity of asthma, allergic rhinitis, and eczema look like (the "atopic triad")?
* Do patients typically have two of these comorbid conditions? three? 
* What do typical treatments look like when patients have these comorbid conditions?
* We still need a [new module](#new-modules) for Eczema.

#### Is our model of immunotherapy treatment realistic?
* Do more people get immunotherapy? less? For how long?
* Is this prohibitively expensive for some patients? What percentage, and who?

#### What are the significant, quantitative risk factors for allergic rhinitis?
* For example: race, obesity, smoking, gender, family history?
* Can these be quantified? (e.g. "males are 20% more likely than females")

#### What does allergic rhinitis claims data look like?
* Cost of outpatient visits, ED visits?
* Cost of treatments (e.g. immunotherapy)?
* Cost of medications?

## Colorectal Cancer

See a visual representation of this module [here](https://synthetichealth.github.io/synthea/graphviz/Colorectal%20Cancer.png).

#### How frequent and how likely are various exams?
* We only model colonoscopies. What about sigmoidoscopies? Other examinations?
* What factors would cause a doctor to give one type of exam instead of another?


#### What other tests are performed on patients diagnosed with colorectal cancer?
* Blood work? Additional stool samples? Biopsies?
* What are the results of these tests?
* Can you map these to specific [LOINC codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#loinc-codes)?

#### Are our treatments for different stages of cancer (e.g. chemotherapy, diverting colostomy) appropriate and realistic?
* What other treatments might be commonly used?
* What factors would cause a doctor to recommend one treatment instead of another?

#### What does colorectal cancer claims data look like?
* Cost of outpatient visits, ED visits?
* Cost of treatments and procedures (e.g. chemotherapy, partial colectomy)?
* Cost of medications?

## Diabetes

#### What do lab tests for dialysis look like?
* Labs before a dialysis session? After a dialysis session?
* Electrolytes (Na, Cl, etc.)?
* What are the results of these tests?
* Can you map these to specific [LOINC codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#loinc-codes)?

## Heart Disease

#### What are common tests performed on patients with heart disease?
* AST/ALT hepatic panels? others?
* What are the typical results of these tests?
* How frequently are these tests performed?
* Can you map these to specific [LOINC codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#loinc-codes)?

#### We need a better model of dual antiplatelet therapy. What does this look like?
* How long do we give therapy for? Why?
* What tests and results are associated with this therapy?
* What medications and procedures are associated with this therapy?

#### How has treatment of heart disease evolved over time?
* Can you identify moments of significant evolution in treatment (e.g. "after 1990, a new treatment was used instead")? 
* What were this historical standards of care?

## Injuries

See a visual representation of this module [here](https://synthetichealth.github.io/synthea/graphviz/Injuries.png).

#### Fractures in the elderly should trigger an osteoporosis workup. What does that look like?
* What tests are run and what are the results?
* If osteoporosis is found, what is the recommended course of treatment?

## Pregnancy

See a visual representation of this module [here](https://synthetichealth.github.io/synthea/graphviz/Pregnancy.png).

#### What tests are performed as part of routine pregnancy care?
* How often are these tests performed?
* What are the results?
* Can you map these to specific [LOINC codes](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework%3A-Basics#loinc-codes)?

## Vaccines

#### How does the patient's age affect the vaccines he/she receives?
* At present, what vaccines would we expect to see in an 18 year old patient?
* A 45 year old patient?
* A 70 year old patient?
* An infant?

#### What are the specific doses for each vaccine?
* Are there multiple ways each vaccine can be administered? Injection? Oral? Nasal spray?

***

# Claims Data

In Synthea v1.0 we did not model claims data of any kind. For **ALL** modules, we would like to know:

#### What does claims data for this module look like?
* Cost of outpatient visits, ED visits, specialists?
* Cost of treatments (e.g. physical therapy)?
* Cost of procedures (e.g. surgery)?
* Cost of medications?

***

# New Modules

We are seeking new [generic modules](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework) and models for the following:

* Contraceptive prescriptions (birth control)
* Eczema
* Allergies to medications
* Food allergies
* Vaccines
