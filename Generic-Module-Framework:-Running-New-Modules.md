All new modules should be saved in a local directory that is passed to Synthea<sup>TM</sup> when it is run. For users of the [basic setup](https://github.com/synthetichealth/synthea/wiki/Basic-Setup-and-Running):

```
java -jar synthea-with-dependencies.jar -d path/to/modules/directory
```

For those using the [developer setup](https://github.com/synthetichealth/synthea/wiki/Developer-Setup-and-Running):

```
./run_synthea -d path/to/modules/directory
```

Synthea will scan the supplied directory for modules and include them in its output.