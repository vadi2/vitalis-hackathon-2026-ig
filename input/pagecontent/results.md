This page consolidates the outcomes from all three tracks of the Vitalis Hackathon 2026. Each track's results are also available on its own page; this page is here to make it easy to read everything that came out of the day in one place.

### Track 1: Agentic Patient Access

(See the [Agentic Patient Access track](track-agentic-patient-access.html) for the full track description.)

#### Christian Hilmersson
*[A C Hilmersson Consulting AB](https://achilmersson.se), Sweden*

Christian registered late and decided to learn the track by doing it: he opened 1177 in the browser, grabbed an appointment-flow request from DevTools, sanitised it, and built a small local app that takes the raw 1177 JSON and converts it to FHIR. The interesting twist is that the model runs entirely on his own machine - a 4B-parameter Qwen model picked because it supports tool calling, where smaller open-weights models often don't. He then wrapped a UI around it with a chat panel, so once the data is converted you can ask things like "summarise my bookings" without scrolling through JSON. Two ingestion paths are supported (paste a pre-sanitised file, or pipe a fresh curl through the sanitiser script first).

#### Rickard Ötvös
*[A C Hilmersson Consulting AB](https://achilmersson.se), Sweden*

Rickard took a different angle from inside Cambio: rather than scraping 1177, he pointed an AI coding agent at Cambio's existing REST APIs - the older system-to-system endpoints that already expose much of the same data - and used it to upgrade and translate the responses into a FHIR-like shape, reusing FHIR resources where they already aligned and filling the rest from the REST payloads. The output was generated fast but is not yet validated; the takeaway is that the same agent-driven facade pattern works equally well against a vendor's existing integration surface, not just patient portals.

#### Jens Kristian Villadsen
*[Trifork](https://trifork.com), Denmark*

Jens Kristian built [c3po-initiative/1177](https://github.com/c3po-initiative/1177), a read-only HAPI FHIR R4 proxy that fronts three Swedish 1177 services and exposes them as standard FHIR. The proxy authenticates against the Inera QA environment using HTTP Basic (personnummer + portal password), performs a SAML/Shibboleth login dance against the shared Inera IDP for each upstream SP, and joins the responses into a single FHIR API per authenticated patient. It maps `journalen` (journal records, HTML fragments inside JSON envelopes) to `DocumentReference`, `bokadetider` (appointment booking) to `Appointment`, and `e-tjanster` (patient inbox) to `Communication`/`DocumentReference`. The whole thing was assembled in under five hours of AI-assisted coding, and Jens Kristian's takeaway was that there are no technical limitations to building a FHIR facade for 1177 - it is entirely a governance question. The repo follows the pattern previously established for the Danish [Dhroxy](https://github.com/c3po-initiative/dhroxy) project.

#### Mikael Rinnetmäki
*[Sensotrend Oy](https://sensotrend.com), Finland*

Mikael worked on a implementing the same access framework on [OmaKanta](https://www.kanta.fi/en/mykanta), the patient portal of the Finnish national centralized health data registry.

Claude got pretty far with implementation. It was able to find many types of information and map those to correct FHIR resource types. It required some guidance to find the rest of the information. This seems to be due to the Kanta portal currently being split to two instances, a legacy and a new one, with most information being available only on one instance.

Eventually Claude got derailed attempting to implement authentication properly.

It may be worthwhile to try again, from a clean slate.

The repo used in this exercise contains personal health information and is not shared publicly.

### Track 2: Terminology

(See the [Terminology track](track-terminology.html) for the full track description.)

#### Anna Rossander
*[VGR](https://www.vgregion.se), Sweden*

Anna worked through the Part 1 hands-on exercises against the Nordic TX server and then spent the rest of her day in the corridor conversations the track was really there to enable - comparing notes with the other terminology folks in the room. That kind of cross-organisation conversation is hard to schedule and easy to underrate, and it's exactly the sort of value a hackathon room is supposed to generate.

#### Joakim Berg
*[Västra Götalandsregionen](https://www.vgregion.se), Sweden*

Joakim also worked through Part 1 and dropped in on the Part 2 AI-assisted demos before pivoting to his real deliverable: progressing the Swedish base profile work that has a hard deadline the following week. As part of that he started investigating a test data factory tool, with the idea of generating profile-conformant example resources automatically - exactly the gap that the Comparing Profiles track also flagged as needed across Nordic IGs. Plus the usual unsung documentation work that keeps a base profile actually usable.

#### Ádám Zoltán Kövér
*[Felleskatalogen](https://www.felleskatalogen.no), Norway*

Ádám built [nlk-test-ig](https://github.com/adamzkover/nlk-test-ig): a test IG with conversion of a national, NPU-based laboratory terminology (with non-FHIR API and data export) to FHIR.

#### Nikolai Ryzhikov
*[Health Samurai](https://www.health-samurai.io), Portugal*

Nikolai demonstrated [Termbox](https://www.health-samurai.io/termbox), a FHIR terminology server from Health Samurai. The demo walked through browsing code systems and value sets, executing terminology operations (with `$lookup` available today and `$translate` plus closure support in active development), and loading terminologies from multiple sources - FHIR packages, NPM packages from the registry, atom syndication feeds, or a single config file declaring all dependencies. A particularly useful view shows the full dependency graph for a value set, which helps diagnose problems when a specific dependency fails to resolve.

#### Kate Ebrill
*[CSIRO](https://www.csiro.au), Australia*

Kate used Claude with an MCP server pointed at the TX server to validate the code systems in a Clinical Practice Guideline IG, generate the eligibility group rules and criteria, and wire the result into Australia's existing CarePlan/SmartForm template - the exercise also surfaced gaps in AU Core that now go on the backlog. Spot checks against EBM-on-FHIR content came out correct, though the full output still needs human review by Michael Lawley.

#### Joonatan Vuorinen
*[Duodecim Publishing Company Ltd.](https://www.duodecim.fi), Finland*

Joonatan built an MCP server backed by an OHDSI Athena export that reads guideline text (a lower-back-pain CPG was the demo) and resolves the concepts mentioned in it. The interesting part is the `$translate` step on top: Athena already carries cross-codesystem mappings between SNOMED CT, ATC, RxNorm, and others, so an agent can answer "is this drug a member of this concept group?" by navigating Athena's graph with subsumption - no need to materialise transitive closures or maintain hundreds of hand-curated value sets. The idea is to let the agent extract preliminary codes from guideline prose and then let the terminology service do the subsumption.

#### Thomas Tveit Rosenlund
*[Helsedirektoratet](https://www.helsedirektoratet.no), Norway*

Thomas worked all day on a long-standing issue with a Norwegian IG that uses terms only available in the Norwegian SNOMED CT edition. The original symptom was that Norwegian display names rendered correctly in the IG narrative but were missing from value set expansions, producing QA errors that the Norwegian edition could not be found. After the upstream changes that syndicate the Norwegian edition into the build, the expansions now resolve and the rendering looks better - but one error remains around a missing display name that Thomas could not yet track down. Progress, but not a clean run.

#### Michael Lawley
*[CSIRO](https://www.csiro.au), Australia*

Michael kicked off [fhir-syndication-ig](https://github.com/FHIR/fhir-syndication-ig) ([published](https://fhir.github.io/fhir-syndication-ig/)), a FHIR Implementation Guide specifying the Atom-based terminology syndication feed format originally developed for Australia's National Clinical Terminology Service (NCTS). The IG documents the feed and entry shape, field semantics, cross-field constraints, the controlled vocabularies used in `<category>`, and three extension namespaces (NCTS ASF, SNOMED CT, and Ontoserver). It enables terminology servers to advertise the code systems, value sets, concept maps, and packages they publish - along with version, publication date, and download links - so consumer servers can discover and either automatically ingest or fetch the content on demand. SNOMED International already publishes a feed of their RF2 content using this format, and any Ontoserver instance can publish a feed of what it contains. Nikolai Ryzhikov ([Health Samurai](https://www.health-samurai.io)) also contributed, with input from Mark Czotter and Gábor Nagy (both [IQVIA](https://www.iqvia.com), Hungary).

#### Vadim Peretokin
*[Peretokin Consulting](https://vadimperetok.in), Sweden*

Vadim's day job at the hackathon was running the Terminology track itself, so the mapping work ran in the background on his laptop while he facilitated: a team of Claude agents chewed through 2000 SNOMED CT procedure codes during the day without him touching the keyboard for most of it. The full task is around 60,000 SNOMED CT procedure codes to the ~15,000 codes in [ICHI](https://www.who.int/standards/classifications/international-classification-of-health-interventions) (WHO's International Classification of Health Interventions), driven by a clinical-information problem: if no mapping exists, clinicians end up entering ICHI directly at the point of capture and lose the clinical-level detail SNOMED would have preserved. The setup is a `while true` shell loop running Claude agents in batches of 50, each emitting FHIR ConceptMap entries with source, relationship (narrower/wider/equivalent/related), target, and the agent's rationale - all going to a Google Sheet for human modellers to review before any production use.

### Track 3: Comparing National and EU FHIR Base Profiles

(See the [Comparing Profiles track](track-comparing-profiles.html) for the full track description.)

Participants discussed the problem and agreed that the problem of comparing profiles and evaluating whether they are compatible is foreseen to be a real issue and something the current FHIR tooling does not address properly.

The group drew inspiration from Gino Canessa's [Cross-Version Extensions project](https://github.com/GinoCanessa/fhir-cross-version-source) (see also the DevDays [presentation](https://www.devdays.com/wp-content/uploads/2025/06/250605_GinoCanessa_CrossVersionExtensions.pdf) and [video](https://www.youtube.com/watch?v=AFTnGTd-yWs)) for FHIR versions.

#### Aleksandr Kislitsyn - FHIR Schema approach
*[Health Samurai](https://www.health-samurai.io), Slovenia*

Aleksandr worked on an [implementation](https://github.com/HealthSamurai/fhir-profile-diff) based on [FHIR Schema](https://github.com/fhir-schema/fhir-schema) to compare profiles.

Identifying "hard differences" like conflicting cardinalities or bindings is important and trivial with this kind of a tool. However, there

Aleksandr's implementation also includes Claude to provide AI-based analysis of the compatibility of profiles. It was surprisingly eye-opening.

#### Pétur Þór Valdimarsson - StructureDefinition merge approach
*[Heybaberiba AB](https://heybaberiba.se), Sweden*

Pétur explored a different angle: take two implementation guides, compare them, and produce a third implementation guide that describes the compatibility between the first two.

His prototype takes profile A and profile B, merges them with some post-processing, and uses extensions to describe similarities (and differences) in fields. The result is a normal implementation guide that can be built with the IG publisher, containing a "conflict profile" that summarises the relevant fields from either source and highlights where they break or where the profiling does something unnecessary (e.g. actively excluding an element that could just be left unsupported). The tool also drills into references - for example, checking whether the referenced organisation profiles are themselves compatible.

The work is rough but promising; the next step is wiring up the IG theme to actually render the custom extensions so the compatibility annotations show up in the published output.

#### Automated creation of example instances
(Captured collectively during discussion of the trakc results)

Each FHIR implementation guide should have examples that demonstrate the use of profiles. However, it is rare that national base profile specifications have an extensive set of examples covering each constraint and extension.

If we could get the implementation guides to include extensive example resources, we could validate those example resources against profiles from other implementation guides and thereby evaluate compatibility of the implementation guides.

The participants made some attempts to create resource instances automatically from profile definitions. The work did not proceed to a demonstratable level.
