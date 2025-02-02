# UN/CEFACT JSON-LD Web Vocabulary - Technical Specification

## Abstract
Emerging technologies depend upon common semantics in order to achieve scalability, when these semantics are managed as web vocabularies and each term has a specific meaning it can be composed dynamically in any order, in this world the dictionary is more important than the document.  The aim of this project is to publish the UN/CEFACT's vocabulary as JSON-LD creating a semantic anchor for supply chain standards, including all classes, properties and code lists.


## Introduction

The JSON-LD project aims to let not just humans, but also machines understand the semantics of CEFACT. 

### Linked Data
Machines build “knowledge graphs” by linking data together. This is done by use of RDF (Resource Description Framework). RDF specifies expressing data as so-called triples, defining subject-predicate-object. A super simple example of a knowledge graph might be: 
* The author's name is Nis. 
* Nis lives in Denmark.

From this, a machine would be able to build this kind of simple knowledge graph: Author - name is - Nis - lives in - Denmark. And by parsing this graph, determine for example that "the author lives in Denmark".

All subjects, predicates and (some) objects are identified on the web by a IRI (Uniform Resource Identifier). An IRI identifies _the thing_, as opposed to a URL which locates a page describing the _the thing_. While this is important to machines, there can be a bit of a clash here as to how humans best read data. For example `<https://schema.org/author> <https://schema.org/givenName> Nis.` is hardly as readable as ”Presenter’s name is Nis”. 

### JSON-LD
Luckily, RDF comes in many flavors. The one which we will focus on here is JSON-LD, which among its advantages is that it is useful for both humans and machines. Also, JSON-LD is based on JSON, which practically is the grammar of any modern API. 

JSON-LD works by injecting a "context" and some other linked data aspects into a normal JSON. All injections are prefixed with an “@” indicating a JSON-LD keyword. 

Let’s consider an example: 

```
{
	"@context": "https://edi3.org/specs/edi3-transport/develop/context.jsonld",
	"consignment": {
		"bookingNumber": "123456789",
		"@id": "https://www.maersk.com/tracking/123456789",
		"includedConsignmentItem": [
			{
				"consignmentItem": {
					"information": "Mangos and bananas",
					"grossWeight": {
						"Value": "12000", "Unit": "Kgs"
					}
				}
			}
		],
		"utilizedTransportEquipment": [
			{
				"transportEquipment": {
					"identification": "MSKU0134962",
        				"@id": "https://app.bic-boxtech.org/containers?search=MSKU0134962"
				}
			}
		]
	}
}
```

Most of this is just a basic json: a consignment with some consignmentItems and a transportEquipment. All expressed in somewhat nice CEFACT lingo which is useful for humans, but just meaningless strings to a computer. The JSON-LD parts `@context` and `@id` changes that. 

The `@id` tags inject IRIs to properly identify the consignment and transportEquipment, respectively referencing appropriate APIs from the carrier and BIC.  

The `@context` links to a jsonld file, defining the semantic meaning of each element of the JSON (note that the context does not have to be externalized to a referenced file like this, but can also just be included directly within the json data file). Here's what `https://edi3.org/specs/edi3-transport/develop/context.jsonld` might look like: 

```
{
	"@context": {
		"consignment": {
			"@id": "https://edi3.org/specs/edi3-transport/develop/vocab/Consignment",
			"@type": "@id"
		},
  		"includedConsignmentItem": "https://edi3.org/specs/edi3-transport/develop/vocab/Consignment#ConsignmentItem",
		"consignmentItem": "https://edi3.org/specs/edi3-transport/develop/vocab/ConsignmentItem",
  		"utilizedTransportEquipment": "https://edi3.org/specs/edi3-transport/develop/vocab/Consignment#utilizedTransportEquipment",
		"transportEquipment": {
			"@id": "https://edi3.org/specs/edi3-transport/develop/vocab/TransportEquipment",
			"@type": "@id"
		}
	}
}
```

Here, the @context adds mapping from the human terms in the JSON to IRIs formally defining the semantics used. For example, `https://edi3.org/specs/edi3-transport/develop/vocab/Consignment` is the IRI for Consignment. 

From this, a computer is able to build a model like this (here serialized as N-Quads): 

```
<https://www.maersk.com/tracking/123456789> <https://edi3.org/specs/edi3-transport/develop/vocab/Consignment#ConsignmentItem> _:b1 .
<https://www.maersk.com/tracking/123456789> <https://edi3.org/specs/edi3-transport/develop/vocab/Consignment#utilizedTransportEquipment> _:b3 .
_:b0 <https://edi3.org/specs/edi3-transport/develop/vocab/Consignment> <https://www.maersk.com/tracking/123456789> .
_:b1 <https://edi3.org/specs/edi3-transport/develop/vocab/ConsignmentItem> _:b2 .
_:b3 <https://edi3.org/specs/edi3-transport/develop/vocab/TransportEquipment> <https://app.bic-boxtech.org/containers?search=MSKU0134962> .
```

### Non-Breaking Retro Fitting 
A clever aspect of JSON-LD is that it can be retrofitted “on top” of legacy JSONs. Adding the JSON-LD (`@`-prefixed) tags will not break your APIs. 

For example, the following legacy JSON (which is much less aligned to CEFACT) will continue working, but generate the exact same machine model. Note that the `@context` in this example is embedded into the JSON itself:  

```
{
	"@context": {
		"shipment": {
			"@id": "https://edi3.org/specs/edi3-transport/develop/vocab/Consignment",
			"@type": "@id"
		},
      	"goods": "https://edi3.org/specs/edi3-transport/develop/vocab/Consignment#ConsignmentItem",
		"goodsItem": "https://edi3.org/specs/edi3-transport/develop/vocab/ConsignmentItem",
      	"containers": "https://edi3.org/specs/edi3-transport/develop/vocab/Consignment#utilizedTransportEquipment",
		"container": {
			"@id": "https://edi3.org/specs/edi3-transport/develop/vocab/TransportEquipment",
			"@type": "@id"
		}
	},
	"shipment": {
		"bookingNumber": "123456789",
		"@id": "https://www.maersk.com/tracking/123456789",
		"goods": [{
				"goodsItem": {
					"information": "Mangos and bananas",
					"grossWeight": {"Value": "12000", "Unit": "Kgs"}
				}
			}],
		"containers": [{
				"container": {
					"boxNb": "MSKU0134962",
                  	"@id": "https://app.bic-boxtech.org/containers?search=MSKU0134962"
				}
			}]
	}
}
```


### Background
Informal draft work 2018-2021.  

### Web semantics 
A high-level introduction to web semantics and JSON-LD. 

## Methodology
How is this standard produced.

## Architecture
UN/CEFACT vocabulary in a broader context. What does the world need from UN? What does UN/CEFACT need from the world? 


## Scope
This project will deliver a high-quality JSON-LD vocabulary published to an open, developer friendly and well-known unece domain and maintained throughout the ongoing development of the CCL, RDMs, and code lists. The vocabulary will be both human readable and machine readable and will support the international community in the development of interoperable APIs, IoT streams, and Verifiable Credentials. 

In order to support that outcome, the project will deliver
* A technical specification that describes the JSON-LD structure and architecture. This work is already 90% completed as a technical guidance note from the RDM2API project – please refer to “draft-rdm2api-json-ld-ndr-docx at https://uncefact.unece.org/pages/viewpage.action?pageId=43384856
* A human and machine readable JSON-LD vocabulary on a unece web domain. These works are already 90% completed and a draft vocabulary is available at https://service.unece.org/trade/uncefact/vocabulary/uncefact/ (human-readable) and https://service.unece.org/trade/uncefact/vocabulary/uncefact.jsonld (machine-readable)
* A publishing mechanism that allows the UNECE secretariat to continue to easily update the vocabulary as CCL, RDM, and code list changes happen.

It has been agreed that the scope will NOT cover 
* API and Schemas
* Data Modelling work as we are re-using existing work rather than creating new standards or subsets of existing.


## Deliverables 

1. A PDF / Word document which covers the project lifecycle including; Public Review logs and the final document ready for official publication.
2. Code in the UN/CEFACT GitHub (https://github.com/uncefact/vocab) to produce the JSON-LD vocabulary outputs
3. Delivery of an official and maintained release of the JSON-LD vocabulary published on the UNECE website (https://service.unece.org/trade/uncefact/vocabulary/uncefact.jsonld)


## Maintenance

UN/CEFACT release their reference data models twice a year (usually around July and December) the intention for the JSON-LD project is that the json-ld vocab would be produced as a deliverable from the release of Buy Ship Pay (BSP) reference data model, this should be automated via scripts.

1. BSP RDM Release 
2. JSON Schema from BSP 
3. JSON-LD release to UN/CEFACT GitHub

This process means that the maintenance of semantics and data elements are managed within the UNECE following the official maintenance and release schedules, JSON-LD vocabulary is then produced from the release and tagged at the same version for stability.


## Requirements

This specification is part of a suite of documents that collectively provides the necessary tools and methods for data modellers to produce high-quality API designs based on UN/CEFACT semantics.

The UN/CEFACT vocabulary is currently published as a CSV file (the reference data models) and variously as CSV, XML, PDF or HTML (the code lists).  The core purpose of this specification is to define the naming and design rules for consistent publishing of both the reference data models and code lists as JSON-LD vocuabularies. This is the foundation specification that makes UN/CEFACT semantics accessible and consumable for web developers. This specification will have achieved its purpose when UN/CEFACT semantics are published and consumable in a similar way to other well-established vocabularies such as schema.org.  

Within this primary goal, there are several more detailed requirements

1. unambiguous. The NDR must define unambiguous rules for publishing UN/CEFACT constructs such as ABIEs, ASBIES, BBIEs, etc as JSON-LD vocabulary constructs.
2. governed. The UN/CEFACT RDMs and code lists are updated on a regular basis (roughly once per 6 months). The JSON-LD publishing process should allow updates to the vocabularies (not a new duplicated vocabulary) at each version increment. 
3. de-duplication. In JSON-LD a "property" such as "consignment.consignor"is a primary entity and has attributes like "domain" (ie which classes may include this property) and "range" (ie what is the value domain of this property). In the UN/CEFACT RDMs the "class" is the primary entity and properties can only belong to a class. Furthermore, it is common for the RDM to define several versions of the same class intended for use in different contexts (eg "consignment" and "referenced.consignment"). There is usually a significant overlap between the properties of these classes. This means that the same semantic vocabulary item occurs multiple times. The JSON-LD vocabulary must de-duplicate without losing the usage context.
4. developer friendly. The published output must be readable and consumable by any developer that is familiar with JSON-LD and should not require any understanding of UN/CEFACT library management terms and processes (eg they should not need to know what an ABIE is). Schema.org provides the most widely used JSON-LD vocabulary in use today and so is a good guide for what the published UN/CEFACT output should look like.
5. maintained and supported. A dedicated team of active maintainers lead by the JSON-LD focal point; maintainers are registered UN/CEFACT experts and manage issues and request. Contributors abide by the UN/CEFACT Open Development Progress (ODP), Intellectual Property Rights Policy (IPR). 


## Naming & Design Rules

### RDM mapping

The current version of vocabulary was automatically generated from the CEFACT Buy-Ship-Pay Reference Data Model xls file, following the rules listed below.

* ABIEs are grouped by `Object Class Term Qualifier(s)` + `Object Class Term` as RDFS Classes
  * but 'Referenced' qualifier is taken out (see [#63](https://github.com/uncefact/spec-jsonld/issues/63))
* BBIEs are grouped by `Property Term Qualifier(s)` + `Property Term` + `Datatype Qualifier(s)` + `Representation Term` as RDFS Properties
  * TDED is empty, Property Term Qualifier is empty, Datatype Qualifier is not empty
    * `Datatype Qualifier(s)` + `Property Term` + `Representation Term` (Datatype Qualifier doesn't contain Property Term - if the property term isn't a part of the DTQ, both of them are used)
    * `Object Class Term Qualifier(s)` + `Object Class Term` + `Datatype Qualifier(s)` + `Representation Term`(Datatype Qualifier equals Property Term - to distinct it from a property with the same key, but no DTQ defined, see UN01002112)
    * `Datatype Qualifier(s)` + `Property Term` + `Representation Term` (Datatype Qualifier contains Property Term, but doesn't equal to it - use DTQ to distinct properties from ones with the same property term)
  * TDED is not empty, Datatype Qualifier is empty
    * `Object Class Term Qualifier(s)` + `Object Class Term` + `Representation Term`(TDED is specified but the DTQ isn't, the object class data is being used to create a meaningful property key)
  * TDED is not empty, Datatype Qualifier is not empty
    * `Datatype Qualifier(s)` + `Property Term Qualifier(s)` + `Property Term` + `Representation Term` (property term isn't a part of the DTQ, both of them are used)
    * `Object Class Term Qualifier(s)` + `Object Class Term` + `Datatype Qualifier(s)` + `Representation Term`(Datatype Qualifier equals Property Term - to distinct it from a property with the same key, but no DTQ defined, see UN01002112)
    * `Datatype Qualifier(s)` + `Property Term Qualifier(s)` + `Property Term` + `Representation Term` (Datatype Qualifier contains Property Term, but doesn't equal to it - use DTQ to distinct properties from ones with the same property term)
  * Representation Term - `Text` is omited, `Identifier` is replaced by `Id`
  * If as a result property name is `id` - rename it to `identifier`. If it's `type` - prepend `Object Class Term`. (see https://github.com/uncefact/spec-jsonld/issues/144#issuecomment-1333493717) 
* ASBIEs are grouped by `Property Term Qualifier(s)` + `Property Term` + `Associated Object Class` as RDFS Properties

At the last phase of the project we switched from the CEFACT Buy-Ship-Pay Reference Data Model xls file to [JSON Schema](https://github.com/uncefact/spec-JSONschema/blob/main/JSONschema2020-12/meta-library/BuyShipPay/D22A/UNECE-BSPContextCCL.json), but kept the NDRs and used "title" property to get Terms and Qualifiers for NDRs.

#### De-duplication

The above grouping rules may lead to the deduplication of several CEFACT BIEs into a single class or property. For example both _SupplyChain_Consignment_ and _Referenced_SupplyChain_Consignment_ BIEs get merged into one _Consignment_ class.

Such deduplication is necessary to make the RDFS modelling guidelines to be unambiguous.

#### Primary identifier mapping

Some CEFACT BIEs have explicit primary identifier properties, for example, Referenced_SupplyChain_Consignment.Identification.Identifier. These properties are omitted in the RDF vocabulary, as RDF data model makes the primary identifier an inherent attribute for each entity in the graph.

### UN/CEFACT metadata

We provide and publish the machine-readable RDF representation of The CEFACT Buy-Ship-Pay RDM Business Information Elements, preserving their types, inheritance heirarchy and metadata. All rdfs classes and properties in edi3 vocabulary are linked with corresponding BIEs by their identifier. This link can be used to implement a software which automatically maps CEFACT RDM messages to RDF format. So that interoperability between existing systems which use CEFACT RDM and new Linked Data based systems is preserved.

The example rdfs property from the edi3 vocabulary, with linked CEFACT RDM BIEs:
```json
{
  "@id": "edi3:consignorTradeParty",
  "@type": "rdfs:Property",
  "rdfs:domain": "edi3:Consignment",
  "rdfs:range": "edi3:Party",
  "edi3:cefactElementMetadata": [
    {
      "@id": "cefact:Referenced_SupplyChain_Consignment.Consignor.Trade_Party",
      "@type": "edi3:AssociationBIE", 
      "edi3:cefactUNId": "cefact:UN01011054",
      "edi3:cefactBieDomainClass": "cefact:Referenced_SupplyChain_Consignment.Details",
      "edi3:cefactBusinessProcess": "Buy-Ship-Pay"
    },
    {
      "@id": "cefact:SupplyChain_Consignment.Consignor.Trade_Party",
      "@type": "edi3:AssociationBIE", 
      "edi3:cefactUNId": "cefact:UN01004212",
      "edi3:cefactBieDomainClass": "cefact:SupplyChain_Consignment.Details",
      "edi3:cefactBusinessProcess": "Buy-Ship-Pay"
    },
  ]
}
```
 
### Business domain granularity

The vocabulary terms are annotated with the logical business domain to which this term belongs to:

```json
{
  "@id": "edi3:consignorTradeParty",
  "@type": "rdfs:Property",
  "edi3:businessDomain": "Trade"
}
```

TODO: The formal process of assigning the business domain to the vocabulary terms is to be decided.

### Versioning

The vocabulary is officially updated twice a year, every 6 months, following the maintenance cycle of the CEFACT BSP RDM.  These releases will be tagged in the repository to show stable versions.

Each BIE is annotated with the date when it was created, the current active\deprecated status, and the date of deprecation.

```json
{
    "@id": "SupplyChain_Consignment.Consignor.Trade_Party",
    "@type": "edi3:AssociationBIE", 
    "@edi3:cefactUNId": "cefact:UN01004212",
    "edi3:currentStatus":"deprecated",
    "edi3:createdDate": "01-04-2017",
    "edi3:deprecatedDate": "21-03-2020"
}
```

Each rdfs class and property in the vocabulary is annotated the same way. The rdfs class or property can only be deprecated when all the RDM BIEs it is linked to are deprecated.

Every time when the vocabulary is updated from the new version of BSP RDM, the new JSON-LD context file for this vocabulary is created and published at the new permanent URL. e.g https://edi3.org/vocab/2020.09/context.json

TODO: the exact URL for the context is to be decided.


### Approach for Dealing with Semantic Issues 

Exposing the UN/CEFACT vocab openly has allowed a larger community of contributors using advanced tooling to look through the UN/CEFACT semantic model. This has generated a number of issues such as semantic inconsistencies, term duplication, unfortunate naming, etc. These issues are inherited from the upstream semantic model, and the vocab project team discussed how to deal with them, essentially either: 
1. "Forking" the model by doing a one-off transformation, allowing ourselves to fix the semantic issues directly in the graph. This would allow the team to quickly improve the semantic quality of the vocab, but at the expense of deviating from the source over time; and version incrementing would require significant manual involvement re-applying semantical fixes.
2. Accept that the vocab has a number of inherited issues, pass back the valuable feedback from the community to the modelling teams and wait for issues to be fixed at the root. Only issues caused by the NDR are considered in the project scope to be addressed. 

After a thorough discussion, the decision was made to go with the second choice. We will not "fork" the model but stick with a fully automated transformation process - but also a number of inherited semantic issues. The arguments for this decision included:
- Full alignment to other implementations of the UN/CEFACT model (including semantic issues).
- Fixing the issues at the root has broader benefits, not only to users of linked data. 
- Lowering project responsibility; modelling is not what this project is about. 
- Less work for the project; dealing with semantic issues is limited to passing feedback upstream. 


### Code list representation

Domain-specific parts of the data model may be governed and published separately, some of these vocabularies are called "code lists". Such vocabularies sometimes have fairly flat and simple organization, for ex. iso-3166 country codes. But others may have a quite complex hierarchical structure with additional metadata, for ex. WCO Harmonized System nomenclature. 

In this section, we describe the recommended format for publishing code lists using rdf and json-ld data model. The vocabulary definitions are represented as [flattened json-ld](https://www.w3.org/TR/json-ld/#flattened-document-form) graph:

```json
{
  "@context": "https://edi3.org/vocab/2020.09/context.json",
  "@graph": [
    { "@id": "iso:AU", "rdfs:label": "Australia" },
    { "@id": "iso:US", "rdfs:label": "United States of America" },
    ...
  ]
}
```

RDF graph data model and json-ld representation may be the best format for machine-readable vocabularies available today. Some prominent features are:

* Standardized way to cross-reference, reuse and extend terms from multiple separately governed vocabularies
* Support for internationalized strings
* Supports hierarchical model of classes and properties
* Consistent library of simple data types like bool, int, date and time
* json-ld is designed to be easily interpreted by human developers, compared to older formats like XML



#### Motivation

TODO: does it belong here?

To avoid interoperability disruptions, it is important for communication systems to have a consistent and up-to-date data model to operate on. Given that code lists are regularly updated, maintaining interoperability between several separately developed business applications is a challenging task. 

Unfortunately, maintainers often publish code lists in proprietary or machine-unfriendly formats like XLS, PDF and HTML, which require tedious human processing to translate and implement in business logic. It would be beneficial to have an authoritative source of vocabulary definitions in a machine-readable format to enable automated processing and allow existing systems to have always up-to-date and consistent views on the data they produce/consume.

TODO: we could discuss somewhere automated mechanisms to distribute updates of machine-readable vocabularies. Differences between push\pull approaches, for ex. CDN vs infrastructure based on WEBSUB hubs?

#### Identifiers

Business application data usually reference entities defined in the code list vocabulary by its identifier. For example "AU" is an identifier of Australia, defined by iso-3166.

While arbitrary string like "AU" may be a good enough identifier in many scenarios, [best practices for data on the web](https://www.w3.org/TR/dwbp/#DataIdentifiers) is to use HTTP URLs as primary identifiers. The advantages of HTTP URLs are namespacing and discovery, briefly highlighted below.

##### Namespaces

Sometimes several concurrent code lists exist, which describe similar concepts, for ex. [Vehicle Plate Country codes](https://en.wikipedia.org/wiki/International_vehicle_registration_code) vs iso-3166 country codes. Quite often business application data have to use a mix of multiple codelists, rendering the used identifiers ambiguous.

To resolve the identifiers ambiguity, we recommend using HTTP URLs based on the domain name which is under control of the authoritative group which maintains the code list vocabulary. For example in place of UNECE rec.21 code "1A", the http url "https://www.unece.org/uncefact/rec21#1A" can be used.

For human convenience, most RDF syntaxes support URL shortening. For example, the JSON-LD representation can use default vocabulary or namespace prefix defined in the context:

Example: default vocabulary makes "1A" to expand to the full url "https://www.unece.org/uncefact/rec21#1A"

```json
{
 "@context": { 
   "@vocab": "https://www.unece.org/uncefact/rec21#",
   "typeCode": {"@type": "@vocab"}
  },
  "@id": "http://maersk.com/packages/171346",
  "typeCode": "1A" 
}
```

Example: prefix definition makes "rec21:1A" to expand to the full url "https://www.unece.org/uncefact/rec21#1A"

```json
{
 "@context": { 
   "@vocab": "https://edi3.org/vocab#",
   "rec21": "https://www.unece.org/uncefact/rec21#",
   "typeCode": {"@type": "@id"}
  },
  "@id": "http://maersk.com/packages/171346",
  "typeCode": "rec21:1A" 
}
```

##### Documentation discovery

It is recommended that dereferencing vocabulary term identifier URL in the web browser result (or redirect to) the page, where the human-readable definition of this term can be found.


#### Data modelling

Entities in the code list vocabulary might have associated metadata, such as human-readable definitions, comments, symbolic representation, and links to other related entities. Each entity can be seen as a node in the RDF graph with an assigned primary identifier (HTTP URL), and other datatype or identifier nodes linked to it. In the Subject-Predicate-Object RDF representation, the kg/m² measurement unit can be defined as:

```turtle
<rec20:kilogram_per_square_meter> <rdfs:comment> "Unit of surface density, areic mass" .
<rec20:kilogram_per_square_meter> <edi3:unitSymbol> "kg/m²" .
<rec20:kilogram_per_square_meter> <edi3:uneceRec20Code> "28" .
```

Which corresponds to the json-ld

```json
{
  "@id": "rec20:kilogram_per_square_meter",
  "rdfs:comment": "Unit of surface density, areic mass",
  "edi3:uneceRec20Code": "28",
  "edi3:unitSymbol": "kg/m²"
}
```

Predicates used to associate an entity with related metadata have their own primary identifier (HTTP URL). In the example above the context is omitted, but implied that _rdfs:comment_ and _edi3:uneceRec20Code_ are abbreviated HTTP URL identifiers of predicates (properties), which are defined in the rdfs and edi3 vocabularies (see rdfs properties definition below).

##### RDF Schema (RDFS) explainer

[RDF Schema](https://www.w3.org/TR/rdf-schema/) provides the mechanisms for describing groups of related resources (entities) and the relationships between these resources. The RDF Schema class and property system is similar to the type systems of object-oriented programming languages, and modelling languages like UML.

While an abstract linked graph view on the data may be useful for simple cases, the RDF Schema provides familiar and powerful semantics of Classes and Properties on top it. 

###### Classes

Continuing from the example of UNECE rec.20 codes for measurement units, we can divide entities into classes NormativeUnit, NormativeEquivalentUnit, InformativeUnit, which are all subclasses of MeasurementUnit. To achieve that, we should first define a class, and then we can assign that class to the entity (instance of that class). On the RDF level, classes just like instances have unique identifiers and can be seen as nodes in the graph.

Example: Base class and its specific subclasses definition
```turtle
<edi3:MeasurementUnit> <rdf:type> <rdfs:Class> .
<edi3:NormativeUnit> <rdf:type> <rdfs:Class> .
<edi3:NormativeUnit> <rdfs:subClassOf> <edi3:MeasurementUnit> .
<edi3:NormativeEquivalentUnit> <rdf:type> <rdfs:Class> .
<edi3:NormativeEquivalentUnit> <rdfs:subClassOf> <edi3:MeasurementUnit> .
<edi3:InformativeUnit> <rdf:type> <rdfs:Class> .
<edi3:InformativeUnit> <rdfs:subClassOf> <edi3:MeasurementUnit> .
```

Now all measurement units can be declared being an instance of appropriate class:
```turtle
<rec20:kilogram_per_square_meter> <rdf:type> <edi3:NormativeUnit> .
<rec20:fahrenheit> <rdf:type> <edi3:NormativeEquivalentUnit> .
...
```

###### Properties

On the RDF graph level, predicates (properties) just like classes or instances have unique identifiers and can be seen as nodes in the graph. Unlike classes or class instances, the property identifiers can also appear in the predicate position (link between the nodes), specifying the semantic of the relationship between subject and object. RDFS vocabulary allows to define a of node as a property, and expresses restrictions on the valid types of the subject and object which is allowed to be linked by this property (domain and range).

Example: define a property and its domain and range
```turtle
<edi3:unitSymbol> <rdf:type> <rdfs:Property> .
<edi3:unitSymbol> <rdfs:domain> <edi3:MeasurementUnit> .
<edi3:unitSymbol> <rdfs:range> <xsd:string> .
```

Now the property can be used to associate measurement unit with its symbol
```turtle
<rec20:kilogram_per_square_meter> <edi3:unitSymbol> "kg/m²" .
```

##### Inferencing

The advanced data consumers can apply RDFS inferencing engine to enrich the input graph data with the additional links (triples), which was omitted but implied by the rdfs class hierarchy and property domain\range provided in the vocabulary.

For example, the kilogram_per_square_meter was defined as instance of NormativeUnit:

```turtle
<rec20:kilogram_per_square_meter> <rdf:type> <edi3:NormativeUnit> .
```

And after applying RDFS inferencing, the new association will be added, saying that kilogram_per_square_meter is also an instance of the base MeasurementUnit class:

```turtle
<rec20:kilogram_per_square_meter> <rdf:type> <edi3:MeasurementUnit> .
```

While RDFS inferencing is powerful, it is also computationally complex and hard to implement. It is not recommended to rely on the data consumer inferencing capability in the published data and vocabularies. It is up for the particular use case to decide on, but generally, it is safer to explicitly declare all types\classes on the instances, and use more generic properties instead of more specific sub-properties.

##### RDFS for existing codelists

Some existing code lists combine multiple entity attributes to be "flattened" into a list of unique identifiers. For example, UNECE Rec.21 for package types assigns a code "BO" to "Bottle, non-protected, cylindrical", and "XH" to "Bag, textile, water-resistant". The brute-force way to express this codelist in a machine-readable way would be 1) assign the full HTTP URL to each code, 2) associate it with a human-readable description and 3) publish it as flattened graph JSON-LD.

Example: simplest json-ld representation of UNECE Rec.21

```json
{
  "@context": {
    "rec21": "https://unece.org/codelists/rec21#",
    "rdfs": "http://www.w3.org/2000/01/rdf-schema#"
  },
  "@graph": [
    {
      "@id": "rec21:1A",
      "rdfs:comment": "Drum, steel",
    }, 
    {
      "@id": "rec21:1B",
      "rdfs:comment": "Drum, aluminium",
    },
    ... 
  ]
}
```

While the example given above is valid and fulfils the requirement of making the code list machine-readable in many use cases, it can be improved. Proper use of RDFS annotations can make the code list vocabulary significantly more convenient to maintain, comprehend and implement in the business logic.

The entities in the rec.21 vocabularies can be quite naturally interpreted as types of package, so they can be declared to be instances of _rdfs:Class_. Also, the primary identifier can be made more human-friendly:

```json
{
  "@id": "rec21:Drum_steel",
  "@type": "rdfs:Class",
  "rdf:value": "1A"
}
```

Now the business data producer can assign appropriate rec.21 package class to the subject of interest:

```json
{
  "@id": "http://maersk.com/packages/b646-629",
  "@type": "rec21:Drum_steel"
}
```

Many entities in the rec.21 vocabulary could be seen as subclasses of the generic base class, for example all package types listed below can be made subclasses of generic _Pallet_ base class:

```
Pallet, CHEP 100 cm x 120 cm
Pallet, AS 4068-1993
Pallet, ISO T11
```

The appropriate class heirarchy can help maintainers to organize and visualize the vocabulary and allow business logic applications to choose the generalization level they need to operate on. 

Some entities in the rec.21 vocabulary mix class-level abstraction with properties, such as water resistance or physical dimensions. It would be more natural to define the properties that business data could use to express such attributes:

```json
{
  "@id": "rec21:width",
  "@type": "rdfs:Property",
  "rdfs:domain": "rec21:BasePackage",
  "rdfs:range": "xsd:decimal",
  "rdfs:comment": "physical width of the package, in millimeters"
}
```

In some cases, part of the vocabulary such as base classes and properties could be extracted to form the stable core vocabulary, while keeping other more specific and volatile subclasses and instances to be governed and published separately. Such distinction might be beneficial for maintaining long-term interoperability between code list users.


## Equivalence Vocabs
There are many other vocabularies in the world beyond UN/CEFACT. Some of them have semantic overlap with parts of the UN/CEFACT vocabulary. This project is scoped to indiscriminately transform the full B-S-P model, including parts which may already be covered by de-facto semantic standards. This section lists other vocabularies which implementers are encouraged to consider. These are included for information only, and should not be interpreted as preference in any direction; there may be reasons to choose either UN/CEFACT or alternative established vocabularies, depending on circumstances. 

### schema.org
Probably the most broadly used vocabulary on the internet. It has an overlap on foundational elements such as Identifier, Organization, and Address.

### GeoSPARQL
| UNCEFACT Class | Mapping | GeoSPARQL Simple Features Class |
|---|---|---|
| uncefact:Polygon | owl:equivalentClass | sf:Polygon | 
| uncefact:LinearRing | owl:equivalentClass | sf:LinearRing | 
| uncefact:GeographicalPoint | owl:equivalentClass | sf:Point |
| uncefact:GeographicalMultiPoint | owl:equivalentClass | sf:MultiPoint |
| uncefact:GeographicalLine | owl:equivalentClass | sf:Line |
| uncefact:GeographicalMultiCurve | owl:equivalentClass | sf:MultiCurve |
| uncefact:GeographicalSurface | owl:equivalentClass | sf:Surface |
| uncefact:GeographicalMultiSurface | owl:equivalentClass | sf:MultiSurface |

Considerations on GeoSPARQL are discussed on this ticket: https://github.com/uncefact/vocab/issues/54. 
