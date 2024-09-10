# UN/CEFACT JSON Schema publication compatible with OpenAPI 3.0.x

This directory contains JSON schema artefacts. Although they are based on JSON schema draft 2020-12, they use certain constructs that are compatible with OpenAPI 3.0.x requirements.


In the [library](https://github.com/uncefact/spec-JSONschema/tree/main/compatibility/library) directory, the UN/CEFACT reference data models are provided in the library variant. This means that all contextualisations of the base classes can be reproduced. Thus, in addition to the basic data types, it also contains the business messages derived from the respective master messages. These can be further contextualised during implementation if required. 

In the [snapshot](https://github.com/uncefact/spec-JSONschema/tree/main/compatibility/snapshot) directory, the UN/CEFACT reference data models and the message structures are provided in the snapshot variant. This means that all contextualisations of the base classes have been fully applied. Each JSON schema artefact is consequently a standalone variant that contains all the required data structures and code lists. These can be further contextualised or extended as needed during implementation. 