# UN/CEFACT JSON Schema draft 2020-12 subset publication

## Introduction

This directory contains JSON schema artefacts. They are fully based on the JSON schema draft 2020-12.
This means that they fully comply with the requirements of OpenAPI 3.1.x.  

The UN/CEFACT reference data models are provided in the subset variant. No message (business document) 
structures are provided. All contextualisations of the base classes have been fully applied. 

## Table of contents
<!-- TOC depthFrom:1 depthTo:3 withLinks:1 updateOnSave:1 orderedList:0 -->

- [Basic schema design rules, external file](../meta-library/README.md#basicPrinciples)
  - [Basic Components](../meta-library/README.md#basicComponents)
    - [Primitive data types](../meta-library/README.md#pdt)
	- [Unqualified data types](../meta-library/README.md#udt)
    - [Qualified data types](../meta-library/README.md#qdt)	
- [JSON filenames and sub-folder structure](#jsonFilenames)
- [What is this subset library ?](#subsetLibrary)
  - [Links to other export variants](#exportLinks)

<!-- /TOC -->

## What is this subset library ?

<a name="subsetLibrary" />
The subset library export includes all Reference Data Model data types. 

### Links to other export variants

<a name="exportLinks" />

In the [library](https://github.com/uncefact/spec-JSONschema/tree/main/JSONschema2020-12/library) directory, 
the UN/CEFACT reference data models are provided in the library variant without the meta-schema extension.

In the [meta-library](https://github.com/uncefact/spec-JSONschema/tree/main/JSONschema2020-12/meta-library) directory, 
the UN/CEFACT reference data models are provided in the library variant with additional the meta-schema extension information.

In the [snapshot](https://github.com/uncefact/spec-JSONschema/tree/main/JSONschema2020-12/snapshot) directory, 
the UN/CEFACT reference data models and the message structures are provided in the snapshot variant. This means 
that all contextualisations of the base classes have been fully applied. Each JSON schema artefact is 
consequently a standalone variant that contains all the required data structures and code lists. These can be 
further contextualised or extended as needed during implementation. 
