# Terminology

Track lead: Vadim Peretokin

## Overview

This track explores how applications can utilize the [Nordic FHIR terminology server](https://tx-nordics.fhir.org/fhir/r4/) (tx-nordics) for authoring FHIR content, and how AI agents can serve as terminology co-pilots - helping find codes, create mappings, and validate bindings.

No prior experience with terminology servers or AI agents is required. The track is structured as a progression from guided exercises to open hacking, so you can join at whatever level suits you.

## Goals

- Learn to use a FHIR terminology server for authoring: looking up codes, validating, expanding value sets, and translating between code systems
- Experience AI agents as terminology co-pilots that connect to terminology servers
- Produce concrete, reusable artifacts - ConceptMaps, ValueSets, or translated designations
- Identify gaps in the Nordic TX server's code system coverage

## Prerequisites

- A laptop with internet access
- (Optional) An AI coding tool you have a license for (e.g. [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview), [Cursor](https://www.cursor.com/), [GitHub Copilot](https://github.com/features/copilot)) - for the AI exercises. We'll help you set up if needed
- No prior terminology server or AI agent experience required

## Exercises

The exercises below are self-paced - start wherever you're comfortable and work through them at your own speed. The track lead will be walking around to help.

### Part 1: TX Server Basics

These exercises introduce the core FHIR terminology operations. You'll work directly with the [Nordic TX server](https://tx-nordics.fhir.org/fhir/r4/) using a REST client, browser, or curl.

1. `$lookup` - Given a SNOMED CT code, retrieve its display name and available designations. Does the server have translations for your language?

    ```
    GET https://tx-nordics.fhir.org/fhir/r4/CodeSystem/$lookup?system=http://snomed.info/sct&version=http://snomed.info/sct/45991000052106/version/20251130&code=73211009&property=designation
    ```

    This looks up "Diabetes mellitus" (73211009) using the Swedish SNOMED CT edition and returns `sv: diabetes` alongside the English designations. Try the Norwegian edition (`51000202101`) or the Danish edition (`554471000005108`) by replacing the module ID in the version parameter.

2. `$validate-code` - Check whether a code is valid in a code system. Try with a valid code, then an invalid one - what does the server return?

    ```
    GET https://tx-nordics.fhir.org/fhir/r4/CodeSystem/$validate-code?url=http://snomed.info/sct&version=http://snomed.info/sct/45991000052106/version/20251130&code=73211009
    ```

    This returns `"result": true`. Now try with a made-up code:

    ```
    GET https://tx-nordics.fhir.org/fhir/r4/CodeSystem/$validate-code?url=http://snomed.info/sct&version=http://snomed.info/sct/45991000052106/version/20251130&code=9999999999
    ```

    This returns `"result": false` with a message explaining the code is unknown.

<details>
<summary>More exercises: $expand and $translate</summary>

3. `$expand` - Expand a value set, optionally filtering by text. How does the result change with different filters?

    ```
    GET https://tx-nordics.fhir.org/fhir/r4/ValueSet/$expand?url=http://snomed.info/sct/45991000052106?fhir_vs=isa/73211009&count=5
    ```

    This expands all descendants of "Diabetes mellitus" using the Swedish edition and returns the first 5. Now add a text filter:

    ```
    GET https://tx-nordics.fhir.org/fhir/r4/ValueSet/$expand?url=http://snomed.info/sct/45991000052106?fhir_vs=isa/73211009&filter=type%202&count=5
    ```

    This narrows it down to 16 matches containing "type 2".

4. `$translate` - Use a ConceptMap to translate a code between systems. First, upload this sample ConceptMap that maps a few diabetes SNOMED codes to ICD-10:

    ```json
    {
      "resourceType": "ConceptMap",
      "url": "https://hl7.se/fhir/ConceptMap/snomed-to-icd10-diabetes-sample",
      "name": "SnomedToIcd10DiabetesSample",
      "status": "draft",
      "sourceUri": "http://snomed.info/sct",
      "targetUri": "http://hl7.org/fhir/sid/icd-10",
      "group": [
        {
          "source": "http://snomed.info/sct",
          "target": "http://hl7.org/fhir/sid/icd-10",
          "element": [
            {
              "code": "73211009",
              "display": "Diabetes mellitus",
              "target": [
                {
                  "code": "E14",
                  "display": "Unspecified diabetes mellitus",
                  "equivalence": "wider"
                }
              ]
            },
            {
              "code": "44054006",
              "display": "Type 2 diabetes mellitus",
              "target": [
                {
                  "code": "E11",
                  "display": "Type 2 diabetes mellitus",
                  "equivalence": "equivalent"
                }
              ]
            },
            {
              "code": "46635009",
              "display": "Type 1 diabetes mellitus",
              "target": [
                {
                  "code": "E10",
                  "display": "Type 1 diabetes mellitus",
                  "equivalence": "equivalent"
                }
              ]
            }
          ]
        }
      ]
    }
    ```

    Upload it with a PUT:

    ```
    PUT https://tx-nordics.fhir.org/fhir/r4/ConceptMap/snomed-to-icd10-diabetes-sample
    ```

    Then translate a code:

    ```
    GET https://tx-nordics.fhir.org/fhir/r4/ConceptMap/$translate?system=http://snomed.info/sct&code=44054006&target=http://hl7.org/fhir/sid/icd-10
    ```

    This should return E11 (Type 2 diabetes mellitus).

</details>

### Part 2: AI Agents as Terminology Co-Pilots

These exercises introduce AI agents that can query terminology servers. If you've never used an AI coding agent before, this is a gentle starting point.

1. "Find me a code" - Give your AI agent a clinical concept in natural language (e.g. "fasting blood glucose", "type 2 diabetes", "left hip replacement") and ask it to find the appropriate SNOMED CT or LOINC code. Then validate the result against the TX server using `$validate-code`. Did the AI get it right, or did it hallucinate?
2. AI-assisted ConceptMap - Take a small list of legacy codes (we'll provide samples, or bring your own) and ask the AI agent to propose SNOMED CT mappings for each. Validate the proposals against the TX server. How many were correct?
3. AI-assisted FSH authoring - Ask the AI agent to write a FHIR profile in FSH with terminology bindings - for example, a simple Observation profile for blood pressure with the right LOINC codes and SNOMED CT value set bindings. Validate the output with the IG publisher using tx-nordics as the terminology server.
4. Compare AI vs TX server - For a set of codes, compare what the AI suggests as the display name vs what the TX server returns from `$lookup`. Where do they diverge? This is a practical lesson in why you should validate AI-generated terminology.

### Part 3: Open Hacking

Bring your own problem, or pick from these:

- NPU codes - NPU (Nomenclature for Properties and Units) is available on the server (`http://npu-terminology.org`). Try looking up common lab codes like `NPU03835` (HbA1c) or `NPU22089` (Glucose):

    ```
    GET https://tx-nordics.fhir.org/fhir/r4/CodeSystem/$lookup?system=http://npu-terminology.org&code=NPU03835
    ```
- CSIRO Shrimp browser - [Shrimp](https://ontoserver.csiro.au/shrimp/) is a free SNOMED CT browser that works with any FHIR terminology server. Point it at the Nordic TX server by entering `https://tx-nordics.fhir.org/fhir/r4` in the endpoint field (top right), or use this direct link:

    [https://ontoserver.csiro.au/shrimp/?fhir=https://tx-nordics.fhir.org/fhir/r4](https://ontoserver.csiro.au/shrimp/?fhir=https://tx-nordics.fhir.org/fhir/r4)
- Build an MCP tool for the TX server - wrap the TX server's FHIR API as an [MCP server](https://modelcontextprotocol.io/) so any AI agent can query it natively. This could be a reusable artifact from the hackathon
- Validation pipeline - build a round-trip workflow where an AI proposes codes, the TX server validates them, and a human reviews the discrepancies

## Expected Outcomes

- Participants understand how to use a FHIR terminology server for authoring (lookup, validate, expand, translate)
- Participants have hands-on experience using AI agents to find and validate terminology
- Reusable artifacts uploaded to the Nordic TX server (ConceptMaps, ValueSets, translated designations)
- Documentation of patterns for connecting AI agents to TX servers (prompts, tool configs, pitfalls)
- Identified gaps in the Nordic TX server's code system coverage for Nordic countries

## Resources

- [Nordic FHIR Terminology Server](https://tx-nordics.fhir.org/fhir/r4/)
- [FHIR Terminology Services specification](https://hl7.org/fhir/R4/terminology-service.html) - reference for all terminology operations
- [Ontoserver documentation](https://ontoserver.csiro.au/docs/) - the software behind tx-nordics
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview) - AI coding agent with terminal access
- [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) - standard for connecting AI agents to tools
