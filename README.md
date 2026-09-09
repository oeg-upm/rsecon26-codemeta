# Making Research Software FAIR with CodeMeta

<!-- QUALITY_BADGE_START -->
[![Software quality](https://img.shields.io/badge/FAIRness-24%25-red "score: 24% | passed: 10 | failed: 31 | errors: 1")](RSFC_REPORT.md)
<!-- QUALITY_BADGE_END -->

Mini-site for the **RSECon26 workshop** “Making Research Software FAIR with CodeMeta”.

The site is intentionally built with plain HTML and CSS so that it can be published directly with **GitHub Pages** and edited easily by workshop organisers.

## Workshop structure

The workshop is a **180-minute interactive session** combining short presentations, live demonstrations, discussions, and hands-on activities.

| Duration | Session |
| ---: | --- |
| **5 min** | **Welcome**|
| **10 min** | **Introduction to FAIR research software (Software Heritage) and CodeMeta** — |
| **20 min** | **Demonstration of the CodeMeta tool ecosystem** — GitHub Actions, CFF conversions, AutoCodeMeta, metadata quality assurance, and related tools|
|  | Show dashboards of the OSPO project and software catalogue, if available. |
| **15 min** | **Discussion on tools for CodeMeta generation** |
| **40 min** | **Hands-on exercise 1 — Generating your own CodeMeta file** |
|  | Generate `codemeta.json`. |
|  | Detect metadata pitfalls. |
|  | Implement GitHub Actions. |
|  | Archive software using the CodeMeta file and repository into Software Heritage. |
|  | Collect participant feedback. |
| **20 min** | **Break** |
| **15 min** | **Introduction to mappings and mapping methodology**  |
|  | Introduction to the use of LLMs for metadata mappings. |
| **40 min** | **Hands-on exercise 2 — Bring your schema: mapping metadata from other schemas into CodeMeta** |
|  | Participants will be divided into groups. |
|  | Select a source schema and approximately 8–12 representative properties. |
|  | Understand the semantics, expected values/types, cardinality, and examples of the selected properties. |
|  | Find possible CodeMeta correspondences and classify the mappings. |
|  | Identify mappings requiring transformations, ambiguous mappings, and properties without a satisfactory CodeMeta correspondence. |
|  | Investigate 2–3 difficult cases, optionally using an LLM, and compare its recommendations with the group's decisions. |
|  | Each group presents one interesting or problematic mapping for discussion. |
| **15 min** | **Wrap-up and discussion** |

## Tools & resources

The following tools and resources are useful during the workshop:

- [CodeMeta](https://codemeta.github.io/) — Vocabulary and project documentation.
- [AutoCodeMeta](https://autocodemeta.linkeddata.es/) — Generate CodeMeta metadata.
- [RSMetaCheck](https://github.com/SoftwareUnderstanding/RsMetaCheck) — Research software metadata checks.
- [sw-metadata-bot](https://github.com/SoftwareUnderstanding/sw-metadata-bot) — Automated metadata feedback.
- [Software Heritage](https://www.softwareheritage.org/) — Software preservation and archiving.
- [Mapping template](resources/mapping-template.csv) — CSV template for Exercise 2.
- [LLM mapping prompt](resources/llm-mapping-prompt.md) — Common prompt for uncertain mappings.
- [SSSOM Methodology](https://github.com/codemeta/codemeta/tree/sssom-methodology/crosswalks-sssom) — Draft methodology proposed by the CodeMeta group for creating mappings.
- [SOMEF](https://github.com/KnowledgeCaptureAndDiscovery/somef) — Tool for extracting metadata from repositories; it can export metadata as CodeMeta files.
- [RSFC](https://github.com/oeg-upm/rsfc) — Tool for assessing the FAIRness of research software.


## Acknowledgements

The workshop materials acknowledge the following projects:

OSCARS, which has received funding from the European Commission's Horizon Europe Research and Innovation programme under grant agreement No. 101129751.

EVERSE, which has received funding from the European Commission's Horizon Europe Research and Innovation programme under grant agreement No. 101129744.

OSTRAILS, which has received funding from the European Commission's Horizon Europe Research and Innovation programme under grant agreement No. 101130187.

FAIR2ADAPT, which has received funding from the European Commission's Horizon Europe Research and Innovation programme under grant agreement No. 101188256.