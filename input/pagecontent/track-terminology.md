Track lead: Vadim Peretokin

### Overview

This track explores how applications can utilize the [Nordic FHIR terminology server](https://tx-nordics.fhir.org/fhir/r4/) (tx-nordics) for authoring FHIR content, and how AI agents can serve as terminology co-pilots - helping find codes, create mappings, and validate bindings.

No prior experience with terminology servers or AI agents is required. The track is structured as a progression from guided exercises to open hacking, so you can join at whatever level suits you. Use of LLMs (ChatGPT, Claude, etc.) is welcomed and encouraged, but not necessary (for part 1).

### Goals

- Learn to use a FHIR terminology server for authoring: looking up codes, validating, expanding value sets, and translating between code systems
- Experience AI agents as terminology co-pilots that connect to terminology servers
- Produce concrete, reusable artifacts - ConceptMaps, ValueSets, or translated designations
- Identify gaps in the Nordic TX server's code system coverage

### Prerequisites

- A laptop with internet access
- [Postman](https://www.postman.com/downloads/) client for part 1 (free)
- An AI coding tool (e.g. [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview), [Cursor](https://www.cursor.com/), [GitHub Copilot](https://github.com/features/copilot)) - for the AI exercises. We'll help you set up if needed. This is needed for part 2 (requires a paid license)
- No prior terminology server or AI agent experience required

### Exercises

The exercises below are self-paced - start wherever you're comfortable and work through them at your own speed. The track lead will be walking around to help.

#### Part 1: TX server basics

These exercises introduce the core FHIR terminology operations. You'll work directly with the [Nordic TX server](https://tx-nordics.fhir.org/fhir/r4/) using a REST client, browser, or curl.

1. `$lookup` - Given a SNOMED CT code, retrieve its display name and available designations. Does the server have translations for your language?

    ```
    GET https://tx-nordics.fhir.org/fhir/r4/CodeSystem/$lookup?system=http://snomed.info/sct&version=http://snomed.info/sct/45991000052106/version/20251130&code=73211009&property=designation
    ```

    This looks up "Diabetes mellitus" (73211009) using the Swedish SNOMED CT edition and returns `sv: diabetes` alongside the English designations. Try the Norwegian edition (module `51000202101`, version `20251215`) or the Danish edition (module `554471000005108`, version `20250930`) by replacing the module ID and version date in the version parameter.

2. `$validate-code` - Check whether a code is valid in a code system. Try with a valid code, then an invalid one - what does the server return?

    ```
    GET https://tx-nordics.fhir.org/fhir/r4/CodeSystem/$validate-code?url=http://snomed.info/sct&version=http://snomed.info/sct/45991000052106/version/20251130&code=73211009
    ```

    This returns `"result": true`. Now try with a made-up code:

    ```
    GET https://tx-nordics.fhir.org/fhir/r4/CodeSystem/$validate-code?url=http://snomed.info/sct&version=http://snomed.info/sct/45991000052106/version/20251130&code=9999999999
    ```

    This returns `"result": false` with a message explaining the code is unknown.

    Try it yourself:

    - What happens if you also send a `display` parameter with the wrong display name for code `73211009`? Does the server catch it? (hint: add `&display=Hypertension` to the valid code query)
    - Can you validate that a code belongs to a specific value set instead of just the code system? Try using the `ValueSet/$validate-code` endpoint with the diabetes descendants value set from exercise 3 below. Does `44054006` (Type 2 diabetes mellitus) belong? What about `38341003` (Hypertension)?
    - Try validating an NPU code. The system URL for NPU is `http://npu-terminology.org` - can you check if `NPU03835` (HbA1c) is valid?

<details markdown="1" style="border: 1px solid #aaa; border-radius: 4px; padding: 0.5em; margin-bottom: 1em;">
<summary style="font-weight: bold; cursor: pointer; padding: 0.5em; margin: -0.5em; background-color: #f4f4f4; border-radius: 4px;">More exercises: $expand and $translate</summary>

3. `$expand` - Expand a value set, optionally filtering by text. How does the result change with different filters?

    ```
    GET https://tx-nordics.fhir.org/fhir/r4/ValueSet/$expand?url=http://snomed.info/sct/45991000052106?fhir_vs=isa/73211009&count=5
    ```

    This expands all descendants of "Diabetes mellitus" using the Swedish edition and returns the first 5. Now add a text filter:

    ```
    GET https://tx-nordics.fhir.org/fhir/r4/ValueSet/$expand?url=http://snomed.info/sct/45991000052106?fhir_vs=isa/73211009&filter=type%202&count=5
    ```

    This narrows it down to 16 matches containing "type 2".

    Try it yourself:

    - Add `&includeDesignations=true` to the expand query. What extra information do you get back? Can you spot translations in your language?
    - Try adding `&displayLanguage=sv` (or `no`, `da`) to the query. How does the output change?
    - Can you expand a different hierarchy? Try replacing `73211009` (Diabetes mellitus) with `38341003` (Hypertension) to find all types of hypertension. How many are there?
    - What happens if you use `&offset=5&count=5` - can you paginate through results?

4. `$translate` - Use a ConceptMap to translate a code between systems. First, upload this sample ConceptMap that maps a few diabetes SNOMED codes to ICD-10:

    ```json
    {
      "resourceType": "ConceptMap",
      "id": "snomed-to-icd10-diabetes-sample",
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

5. Build your own ConceptMap - Now create a [ConceptMap](https://hl7.org/fhir/R4/conceptmap.html) from scratch. Map these SNOMED CT codes to ICD-10:

    | SNOMED CT code | Display |
    |---|---|
    | 38341003 | Hypertensive disorder |
    | 48146000 | Diastolic hypertension |
    | 56218007 | Systolic hypertension |

    Figure out the appropriate ICD-10 codes for each, use the sample ConceptMap from exercise 4 as a template, and upload it with a PUT. Test it with `$translate` - does the server return the ICD-10 code you expected?

</details>

#### Part 1b: Authoring IGs with national terminology

These exercises show how to use a national SNOMED CT edition when building a FHIR Implementation Guide. You'll write FSH, build the IG, and verify that the Nordic TX server was used automatically through the FHIR terminology ecosystem.

1. Clone and build - Clone this hackathon IG and build it:

    ```
    git clone https://github.com/vadi2/vitalis-hackathon-2026-ig.git
    cd vitalis-hackathon-2026-ig
    ./_genonce.sh
    ```

    The build should succeed. The IG publisher uses `tx.fhir.org` by default, which knows how to route requests to national terminology servers like the Nordic TX server.

2. Add a Swedish SNOMED code - Create a new FSH file (e.g. `input/fsh/diabetes-condition.fsh`) with an example Condition that uses a code from the Swedish SNOMED CT edition:

    ```fsh
    Instance: DiabetesExample
    InstanceOf: Condition
    Usage: #example
    * subject = Reference(Patient/example)
    * code = http://snomed.info/sct|http://snomed.info/sct/45991000052106#73211009 "Diabetes mellitus"
    * clinicalStatus = http://terminology.hl7.org/CodeSystem/condition-clinical#active
    ```

    Note the version in the code system URI - `http://snomed.info/sct|http://snomed.info/sct/45991000052106` pins it to the Swedish edition.

3. Build and check TX logs - Rebuild the IG with `./_genonce.sh`. After the build completes, open `output/qa-tx.html` in your browser. This shows every terminology request the IG publisher made. Can you find the request that validated your Swedish SNOMED code? Which server handled it - was it routed to the Nordic TX server?

    Try it yourself:

    - Break it on purpose - Change the display text in your Condition to something wrong (e.g. `#73211009 "Hypertension"`) and rebuild. What does the QA report say? Check both `output/qa.html` and `output/qa-tx.html`.
    - Try a different national edition - Create another example using the Norwegian (`http://snomed.info/sct/51000202101`) or Danish (`http://snomed.info/sct/554471000005108`) edition. Does the build still validate successfully?
    - Add an NPU-coded Observation - Create an Observation example that uses an NPU code for a lab result:

        ```fsh
        Instance: HbA1cExample
        InstanceOf: Observation
        Usage: #example
        * status = #final
        * code = http://npu-terminology.org#NPU03835 "Hb(Fe;B)—Haemoglobin A1c(Fe); subst.fr. = ?"
        * subject = Reference(Patient/example)
        * valueQuantity = 48 'mmol/mol'
        ```

        Does the build validate the NPU code? Check `qa-tx.html` to see if it reached the Nordic TX server.
    - Create a ValueSet - Write a ValueSet in FSH that includes descendants of a SNOMED concept from a national edition, then bind it to a profile element. Does the IG publisher expand and validate it correctly?

        ```fsh
        ValueSet: DiabetesTypesSE
        Id: diabetes-types-se
        Title: "Diabetes Types (Swedish edition)"
        * include codes from system http://snomed.info/sct|http://snomed.info/sct/45991000052106
            where concept is-a #73211009 "Diabetes mellitus"
        ```

#### Part 2: AI Agents as terminology assistants

These exercises introduce AI agents that can query terminology servers. If you've never used an AI coding agent before, this is a gentle starting point.

1. "Find me a code" - Give your AI agent a clinical concept in natural language (e.g. "fasting blood glucose", "type 2 diabetes", "left hip replacement") and ask it to find the appropriate SNOMED CT or NPU code. Then validate the result against the TX server using `$validate-code`. Did the AI get it right, or did it hallucinate?
   1. Now, ask the AI agent to look up codes by giving it the sample URL from Part 1. Ask it to look up codes again. Does it do a better job now?

   Note: tx-nordics indexes SNOMED CT (international + Nordic editions) and NPU; it does not currently host LOINC, so LOINC codes won't validate against this server.
2. AI-assisted ConceptMap - Take the sample legacy codes below (or bring your own) and ask the AI agent to propose SNOMED CT mappings for each. Validate the proposals against the TX server with `$validate-code`. How many were correct?

    | Legacy code | Description |
    |---|---|
    | LAB-001 | Fasting blood glucose |
    | LAB-002 | HbA1c |
    | LAB-003 | Total cholesterol |
    | DX-101 | High blood pressure |
    | DX-102 | Adult-onset diabetes |
    | DX-103 | Chest pain on exertion |
    | DX-104 | Iron deficiency |
    | PROC-201 | Total knee replacement, left |
    | PROC-202 | Removal of gallbladder |
    | PROC-203 | Insertion of cardiac pacemaker |
3. Compare AI vs TX server - For a set of codes, compare what the AI suggests as the display name vs what the TX server returns from `$lookup`. Where do they diverge? This is a practical lesson in why you should validate AI-generated terminology.

#### Part 3: Open hacking

Bring your own problem, or pick from these:

- NPU codes - NPU (Nomenclature for Properties and Units) is available on the server (`http://npu-terminology.org`). Try looking up common lab codes like `NPU03835` (HbA1c) or `NPU22089` (Glucose):

    ```
    GET https://tx-nordics.fhir.org/fhir/r4/CodeSystem/$lookup?system=http://npu-terminology.org&code=NPU03835
    ```
- CSIRO Shrimp browser - [Shrimp](https://ontoserver.csiro.au/shrimp/) is a free SNOMED CT browser that works with any FHIR terminology server. Point it at the Nordic TX server by entering `https://tx-nordics.fhir.org/fhir/r4` in the endpoint field (top right), or use this direct link:

    [https://ontoserver.csiro.au/shrimp/?fhir=https://tx-nordics.fhir.org/fhir/r4](https://ontoserver.csiro.au/shrimp/?fhir=https://tx-nordics.fhir.org/fhir/r4)
- Connect an AI agent to the TX server via MCP - [fhir-mcp](https://github.com/xSoVx/fhir-mcp) is an existing MCP server that supports `terminology.lookup`, `terminology.expand`, and `terminology.translate`. Point it at the Nordic TX server by setting `TERMINOLOGY_BASE_URL=https://tx-nordics.fhir.org/fhir/r4` and use it from Claude Code or Cursor to look up codes conversationally
- Autonomous codesystem mapping - Set up an AI agent on an autonomous loop to map a large codesystem (1000+ codes) to a standard terminology like SNOMED CT. The agent iterates through codes in batches, searches for matches, validates them against the TX server, and writes the results to a ConceptMap - all without manual intervention. This is a practical pattern for migrating legacy codesystems at scale
- Validation pipeline - build a round-trip workflow where an AI proposes codes, the TX server validates them, and a human reviews the discrepancies

### Expected outcomes

- Participants understand how to use a FHIR terminology server for authoring (lookup, validate, expand, translate)
- Participants have hands-on experience using AI agents to find and validate terminology
- Reusable artifacts uploaded to the Nordic TX server (ConceptMaps, ValueSets, translated designations)
- Documentation of patterns for connecting AI agents to TX servers (prompts, tool configs, pitfalls)
- Identified gaps in the Nordic TX server's code system coverage for Nordic countries

### Resources

- [Nordic FHIR Terminology Server](https://tx-nordics.fhir.org/fhir/r4/)
- [FHIR Terminology Services specification](https://hl7.org/fhir/R4/terminology-service.html) - reference for all terminology operations
- [Ontoserver documentation](https://ontoserver.csiro.au/docs/) - the software behind tx-nordics
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview) - AI coding agent with terminal access
- [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) - standard for connecting AI agents to tools

### Results

See the [Results](results.html#terminology) page.
