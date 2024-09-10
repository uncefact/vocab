# UN/CEFACT JSON Schema draft 2020-12 publication

This directory contains JSON schema artefacts. They are fully based on the JSON schema draft 2020-12.
This means that they fully comply with the requirements of OpenAPI 3.1.x.

In the [library](https://github.com/uncefact/spec-JSONschema/tree/main/JSONschema2020-12/library) directory,
the UN/CEFACT reference data models are provided in the library variant. This means that all contextualisations
of the base classes can be reproduced. Thus, in addition to the basic data types, it also contains the business
messages derived from the respective master messages. These can be further contextualised during implementation
if required. 

In the [snapshot](https://github.com/uncefact/spec-JSONschema/tree/main/JSONschema2020-12/snapshot) directory,
the UN/CEFACT reference data models and the message structures are provided in the snapshot variant. This means
that all contextualisations of the base classes have been fully applied. Each JSON schema artefact is
consequently a standalone variant that contains all the required data structures and code lists. These can be
further contextualised or extended as needed during implementation. 

In the [subset](https://github.com/uncefact/spec-JSONschema/tree/main/JSONschema2020-12/subset) directory, 
only the UN/CEFACT reference data models, but no message structures are provided in a subset variant. This means 
that all contextualisations of the base classes have been fully applied.  

In the [meta-library](https://github.com/uncefact/spec-JSONschema/tree/main/JSONschema2020-12/meta-library)
directory, the UN/CEFACT reference data models are provided in the library variant, with extended
meta-information. This means that an idividueal meta-schema and an additional vocabulary is applied.
The additional information is included to support mapping-processes and the JSON-LD vocabulary creation. 
