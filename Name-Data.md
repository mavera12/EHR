By default, the project contains English-style and Spanish-style names suitable for the United States in `src/main/resources/names.yaml` file.

You can change those names, or add names for other languages. The languages people speak determines their name. The language they speak is determined by their ethnicity. If you want to add other languages, you'll need to modify the `src/main/java/org/mitre/synthea/world/geographic/Demographics.java` file. Specifically, these two functions:

```java
public String ethnicityFromRace(String race, Person person) { ... }
public String languageFromEthnicity(String ethnicity, Person person) { ... }
```

## Names File Format

```yml
language:
  M: [MaleName1,MaleName2,...,MaleNameN]
  F: [FemaleName1,FemaleName2,...,FemaleNameN]
  family: [Surname1,...,SurnameN]

street:
  type: [Street,Straat,Rue]
  secondary: [Apt]
```

Languages currently supported (but not necessarily defined):

`english`,`spanish`,`french`,`french_creole`,`german`,`russian`,`portuguese`,`polish`,`greek`,`chinese`,`hindi`,`arabic`