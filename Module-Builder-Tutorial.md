
This tutorial and exercise will introduce and walk through the modeling of diseases using the Synthea Module Builder.  The session will highlight Synthea (“Synthetic Health”), an open-source software package that generates synthetic patient records. The software models and simulates over 500 clinical concepts, with a growing list of community written modules.

After completing this tutorial, you will be able to:

1.  Explore and Modify the Disease Modules
2.  Understand the pertinent elements of disease life-cycles to include in modeling for Synthea
3.  Understand the concepts of disease modeling within Synthea


Have fun and remember to ask for help if you get stuck!


# 1. Starting a New Module

We will start the tutorial with a step by step guide on building a new module. We will need to start by navigating to https://synthetichealth.github.io/module-builder/.  Choose **Examplitis** as the initial module.  From there, 

- Open a new module by clicking on **New Module**
- Navigate to **Module Remarks** and title your module **My CHF module**
- Navigate to **+ Add State**
- Navigate to the **State Type** drop down menu and choose **ConditionOnset**
- Open separate window to utilize for SNOMED searches.  (Example Site: browser.ihtsotools.org)
- Search for an appropriately broad parent SNOMED code (eg 42343007)
- Add your chosen SNOMED code to the **Code** section in the State Editor
- Label the SNOMED code by clicking on the **Display** **SNOMED Code** place-holder
- Title your ConditionOnset State by clicking on the **New_State** place-holder and titling it **CHF Condition Start**
- Navigate to the **Initial** state
- Within the State Editor for the **Initial** state, navigate to **Transition Type** and choose **Direct**.  Choose **CHF Condition Start** in the **Transition To** drop down.
- Navigate to and click on **+ Add State**
- Navigate to and click on **State Type** and choose **Encounter**
- Choose **ambulatory** in **Encounter Class** drop down
- In Separate browser, search for appropriate SNOMED code to label the encounter (Recommend searching for **encounter** SNOMEDs – e.g. 185347001)
- Navigate to the State Editor and label the encounter **CHF Initial Workup Encounter**
- Navigate to back to **CHF Condition Start** and navigate to **Transition Type**.  Choose **Direct** in the drop-down option.  Choose **CHF Initial Workup Encounter** in the **Transition To** drop down.

You have now successfully created two states linked together at the initial start of the disease module. 

# 2. Building a Complex CHF Module: Modeling Prevalence Rate

Continue building your module in a similar fashion by adding the following states (click on **+ add state** for each new state):

- State Type = **delay**.  Label this state **age delay**.  Type **50** within state editor, under **exact quantity**. Choose **years** in **exact unit** drop down.  Keep Transition Type as **Direct**. 
- State type = **Symptom**.  Label this state **CHF symptom onset 1**.  Under Symptom, type **shortness of breath**.  Under Cause, type **Congestive Heart Failure**.  Keep Transition Type as **Direct**.
- State type = **delay**.  Label this state **chance of CHF**.  Type **1** under **exact quantity**.  Choose **years** in **exact unit**.   Choose **Distributed** as Transition Type.  Within Distributed Transition, choose **chance of CHF**.  Weight this with **0.99905**.  Click on the **+/-** to add a nested transition option.  Choose **CHF symptom onset 1**.  Under weight, type **0.00095**
- Navigate to Initial State.  Change **Transition To** to **age delay** state.  
- Navigate to **age delay** state.  Change **Transition To** to **chance of CHF** state.
- Navigate to **CHF symptom onset 1** state.  Change **Transition To** to **CHF Condition Start** state.

# 3. Building a Complex CHF Module: Modeling Encounters & Diagnostic Work-Ups

Keeping with the same model, you should now have a module that has an age delay, a state to prevalence rates, an example symptom of SOB, and the condition onset of CHF.  Next, we will model a sample encounter.

- Navigate to **+ Add State**.  State type = encounter.  Label this state **CHF initial workup encounter**. Encounter class is **ambulatory**.  Label the **Reason** as **chf**.  Under codes, choose an appropriate problem based SNOMED code (e.g. 185347001).  

- Navigate BACK to the **CHF Condition Start** encounter.  Within **Target Encounter**, type the EXACT text of the just-added encounter, **CHF initial workfup encounter**.  Under assign to attribute, type the exact text recently added in the **Reason** section of the encounter, **chf**. 

- Navigate to **+ Add State**.  State type = **Observation**.  Label this state **BNP lab workup**.  Under category, choose **laboratory**.  In **unit**, type **pg/mL**.  Within the nested **Codes** section, under **Code**, you will need to insert an appropriate LOINC code.  We recommend using an appropriate LOINC browser/search, such as: loinc.org.  In this case, choose 42637-9 as an ORDERABLE lab.  Under **Display**, type the exact label of the lab test **Natriuretic peptide B**.   Within the Value section, select **Change to Range**.  Type the LOW range to be 2, and the HIGH range to 100.  

- Navigate to **+ Add State**.  State type = **EncounterEnd**.  Label this **CHF initial workup encounter end**.  Under NUBC code, search for the appropriate NUBC discharge code.  In this case, the patient is discharged home, so choose code **01**.  Within Transition to, choose **End**.  

You have now completed the initial CHF module modeling.  You should have an encounter that begins with an age delay, has a prevalence rate modeled, that leads to symptom progression, the start of CHF, and an initial work-up with a sample laboratory test ordered.




# 4.  Advanced CHF Module Building

The last phase of the tutorial, should there be enough time, is to attempt to re-create the **Congestive Heart Failure** module on your own.  Critical additional elements to pay close attention to include the following:

- Multiple laboratory orders within one section
- Understanding and leveraging hard-coded Vitals (e.g. BUN, Creatinine, etc.)
- Starting Medications.  Leveraging RxNorm values (recommend [https://rxnav.nlm.nih.gov/])
Building and leveraging **CONDITIONAL** Transition Types (see “Maintaining CHF” State)
- Building and leveraging **DISTRIBUTED** Transition Types (see “Possible CHF Flare” State)
- Understanding an Inpatient clinical workflow (See start of inpatient section, with “acute Inpatient CHF flare” and subsequent sections.

