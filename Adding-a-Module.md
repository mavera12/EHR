**We strongly encourage new modules be developed using the [Generic Module Framework](https://github.com/synthetichealth/synthea/wiki/Generic-Module-Framework). Java modules should be used only when the GMF does not support your specific use case.**

Modules implement the core functionality of Synthea. Each module models a specific aspect of a patient's medical history. Some examples of modules currently implemented include:

* [Lifecycle](https://github.com/synthetichealth/synthea/blob/master/src/main/java/org/mitre/synthea/modules/LifecycleModule.java) - birth, death, height, weight, and other vital signs
* [Encounters](https://github.com/synthetichealth/synthea/blob/master/src/main/java/org/mitre/synthea/modules/EncounterModule.java) - medical encounters are performed, diagnoses are made, and the results are recorded
* [Cardiovascular Disease](https://github.com/synthetichealth/synthea/blob/master/src/main/java/org/mitre/synthea/modules/CardiovascularDiseaseModule.java) - coronary heart disease, myocardial infarction, cardiac arrest, stroke, atrial fibrillation
* [Immunizations](https://github.com/synthetichealth/synthea/blob/master/src/main/java/org/mitre/synthea/modules/Immunizations.java) - vaccines and immunization schedules

See the existing modules in `lib/modules` for more in-depth examples beyond what is provided on this page.

To start writing a module, create a new Java class file in the `src/main/org/mitre/synthea/modules` folder with the following skeleton:

```java
package org.mitre.synthea.modules;

import org.mitre.synthea.engine.Module;

public class NewModule extends Module {

  @Override
  public boolean process(Person person, long time) {

    // java modules will never "finish"
    return false;
  }

}
``` 

And add a reference to the module in src/main/java/org/mitre/synthea/engine/Module.java :: loadFiles : 

```java
  private static Map<String, Module> loadModules() {
    Map<String, Module> retVal = new ConcurrentHashMap<String, Module>();

    retVal.put("Lifecycle", new LifecycleModule());
    retVal.put("Cardiovascular Disease", new CardiovascularDiseaseModule());
    retVal.put("Quality Of Life", new QualityOfLifeModule());
    retVal.put("Health Insurance", new HealthInsuranceModule());

    retVal.put("Name of new module", new NewModule()); /// YOUR MODULE HERE

```

### Attributes and Events

There are two primary ways that Synthea keeps track of the conditions and events that have occurred for a patient - an attributes map and an event list. The attributes map is part of the `Person` class, and attributes indicating the presence or value of a condition/measurement can be stored in this map. For example, `"BMI" => 25.4` or `"hypertension" => false`. 

Every patient also has an event list object from `EventList.java`.  This event list keeps track of when certain conditions are acquired and whether or not they have been processed yet, which can be used during encounters to narrow down the events that need to be recorded.

When writing module rules and determining when to track an attribute with the event list or the attributes map, follow these guidelines:

* Always include the presence/absence/measurement of a condition in the attributes map when it is acquired or the status of it changes
* If a new attribute is being added, be careful that the name does not clash with the name of another attribute being tracked to avoid unintentional overwriting
* If the time something occurs is important, use the event list to record it, such as with the scheduling of [encounters](https://github.com/synthetichealth/synthea/blob/master/src/main/java/org/mitre/synthea/modules/EncounterModule.java)
* There can be other uses for the event list as well, so feel free to deviate from these guidelines 

### Record Functions

Most modules will produce attributes and conditions that will be recorded in a patient's electronic health record. To record conditions, functions need to be defined in the module that will be called upon when a medical encounter is performed. Follow these guidelines when defining these functions:

* Place these functions at the end of a module and separate them from the rest of the code with a comment
* These functions only need `person` and `time` parameters
* The function should handle any logic and ensure that an attribute is only recorded under the correct circumstances - assume the function can be called at any point during patient generation
* The function itself should call an instance method for `person.record`, which actually saves information. The list of instance methods to be called are located in `src/main/java/org/mitre/synthea/world/concepts/HealthRecord.java`

For more information on records, check out the [Records Wiki page](https://github.com/synthetichealth/synthea/wiki/Records).

### Final Note
Additional methods and other data structures that will be used by the rules and record functions can also be added. It is strongly recommended to adhere to these guidelines, but slight deviations are acceptable as well.

Writing unit tests for new modules is also strongly encouraged.