###Viewing Rules and Attributes Relationships

A diagram showing the relationships between various rules and attributes can be generated with Graphviz. Since there are so many rules and attributes being modeled in Synthea, it can often be useful to see a visual representation of how these different aspects interact with each other.

####Running Synthea Graphviz
To begin, first clone the [Synthea Graphviz](https://github.com/synthetichealth/synthea_graphviz) repo into the same folder as the Synthea repo. Synthea Graphviz must be in the same folder as Synthea for it to work properly.

	cd file_path_of_synthea
	git clone https://github.com/synthetichealth/synthea_graphviz.git

In the Synthea Graphviz folder, follow the instructions on the [README](https://github.com/synthetichealth/synthea_graphviz/blob/master/README.md) for the project.

The flowchart diagram will be generated in the Synthea Graphviz folder, named `synthea_rules.png`.
