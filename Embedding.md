In addition to standalone execution, Synthea<sup>TM</sup> can also be embedded in other applications.

## Dependency Configuration

Synthea releases (starting at version 2.5.0) will be published on [Maven Central](https://search.maven.org/search?q=synthea). Snapshots (starting at version 2.5.0-SNAPSHOT) are published on the [Sonatype snapshots](https://oss.sonatype.org/#nexus-search;quick~synthea) repository.

Below is an example of declaring a dependency on a Synthea release using a Maven pom.xml file:

```xml
<repositories>
  <repository>
    <id>maven.central</id>
    <url>http://repo.maven.apache.org/maven2</url>
    <releases>
      <enabled>true</enabled>
    </releases>
    <snapshots>
      <enabled>false</enabled>
    </snapshots>
  </repository>
</repositories>
<dependencies>
  <dependency>
    <groupId>org.mitre.synthea</groupId>
    <artifactId>Synthea</artifactId>
    <version>2.5.0</version>
  </dependency>
</dependencies>
```

Below is an example of declaring a dependency on a Synthea snapshot release using a Maven pom.xml file:

```xml
<repositories>
  <repository>
    <id>oss.sonatype.org-snapshot</id>
    <url>http://oss.sonatype.org/content/repositories/snapshots</url>
    <releases>
      <enabled>false</enabled>
    </releases>
    <snapshots>
      <enabled>true</enabled>
    </snapshots>
  </repository>
</repositories>
<dependencies>
  <dependency>
    <groupId>org.mitre.synthea</groupId>
    <artifactId>Synthea</artifactId>
    <version>2.6.0-SNAPSHOT</version>
  </dependency>
</dependencies>
```

## Documentation

Javadocs for the master branch are available at [https://synthetichealth.github.io/synthea/build/javadoc/index.html](https://synthetichealth.github.io/synthea/build/javadoc/index.html)

## File Output

In standalone use, Synthea generates files in an "output" subdirectory of the application working directory. The example below illustrates how to programmatically generate 10 patient records from a simple command line application.

```java
package com.example.generator;

import org.mitre.synthea.engine.Generator;
import org.mitre.synthea.helpers.Config;

public class Main {

    public static void main(String[] args) {
        Generator.GeneratorOptions options = new Generator.GeneratorOptions();
        options.population = 10;
        Config.set("exporter.hospital.fhir.export", "false");
        Config.set("exporter.practitioner.fhir.export", "false");
        Generator generator = new Generator(options);
        generator.run();
    }
    
}
```

In the above, the [`Config`](https://synthetichealth.github.io/synthea/build/javadoc/org/mitre/synthea/helpers/Config.html) class is used to set [Common Configuration](https://github.com/synthetichealth/synthea/wiki/Common-Configuration) options and the [`GeneratorOptions`](https://synthetichealth.github.io/synthea/build/javadoc/org/mitre/synthea/engine/Generator.GeneratorOptions.html) object is used to control the demographics of the population.

## JSON Integration

Synthea can also be used to create JSON patient records that are passed as a `String` to the invoking application. When used in this way, Synthea is invoked on a separate thread and generated records are retrieved from a blocking queue. The code below illustrates how to use Synthea in this way.

```java
// Configure options and override default file output configuration
Generator.GeneratorOptions options = new Generator.GeneratorOptions();
options.population = 1000;
Config.set("exporter.fhir.export", "false");
Config.set("exporter.hospital.fhir.export", "false");
Config.set("exporter.practitioner.fhir.export", "false");
Exporter.ExporterRuntimeOptions ero = new Exporter.ExporterRuntimeOptions();
ero.enableQueue(Exporter.SupportedFhirVersion.R4);

// Create and start generator
Generator generator = new Generator(options, ero);
ExecutorService generatorService = Executors.newFixedThreadPool(1);
generatorService.submit(() -> generator.run());

// Retrieve the generated records
int recordCount = 0;
while(recordCount < options.population) {
  try {
    String jsonRecord = ero.getNextRecord();
    recordCount++;
    processRecord(jsonRecord);
  } catch (InterruptedException ex) {
    break;
  }
}

// Shutdown the generator
generatorService.shutdownNow();
```

Note the use of the [`Config`](https://synthetichealth.github.io/synthea/build/javadoc/org/mitre/synthea/helpers/Config.html) class to disable file generation. If this is omitted, files will also be written to the "output" subdirectory.