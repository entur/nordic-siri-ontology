# Nordic SIRI Ontology

Machine-readable ontology for the SIRI standard and the Nordic SIRI Profile.
Models services, objects, relationships, enumerations, and profile constraints
as RDF/OWL in Turtle format.

> **Status:** Prep / personal area. This is the extracted, dedicated ontology
> layer for the Nordic SIRI Profile. It is being refined here before being
> promoted to an Entur-org repository. It mirrors the split used for NeTEx,
> where [`nordic-netex-ontology`](https://github.com/entur/nordic-netex-ontology)
> is the dedicated ontology, separate from the documentation repository.

## Purpose

The ontology serves two purposes:

- **For humans:** A precise, navigable reference for how SIRI services,
  objects, and constraints relate to each other, and how they reference
  planned NeTEx objects.
- **For machines:** A foundation for automated validation (SHACL),
  documentation generation, and tooling integration.

## Scope

This repository contains only the **shared Nordic SIRI foundation** — the
real-time model agreed across the Nordics. It is deliberately separated from
organisation-specific layers (Anshar service sub-profiles, codespace
conventions, data ownership), which belong in downstream repositories that
import this foundation via `owl:imports` or as a git submodule.

```
siri.ttl                ← SIRI base vocabulary (unofficial placeholder, pending CEN)
siri-nordic.ttl         ← Nordic SIRI Profile (this repo)
```

**Design principle:** Any layer built on top can tighten constraints (via
SHACL), but this repository remains stable and reusable regardless of who
consumes it. SIRI is **separate from NeTEx by design** — SIRI is real-time
data that *references* planned NeTEx objects. Cross-references are declared
here as bridging links; the canonical NeTEx definitions live in
[`nordic-netex-ontology`](https://github.com/entur/nordic-netex-ontology).

## Files

| File | Contents |
|------|----------|
| `siri.ttl` | **Unofficial placeholder** for a SIRI base vocabulary. There is no canonical CEN-published SIRI ontology yet; this file only reserves the `https://siri-cen.eu/ontology#` namespace and the base slot in the two-file layout until CEN provides one. |
| `siri-nordic.ttl` | The Nordic SIRI Profile: OWL classes for SIRI services (ET, SX, VM, FM), shared objects (EstimatedVehicleJourney, EstimatedCall, VehicleActivity, PtSituationElement, …), enumerations, key element specifications, executable SHACL profile-constraint shapes (`profile:NSP_*`), communication patterns, NeTEx bridges, and Transmodel alignment. |

This mirrors [`nordic-netex-ontology`](https://github.com/entur/nordic-netex-ontology)'s
`netex.ttl` + `netex-nordic.ttl` split. Note that **neither standard has an
official, up-to-date base ontology** — CEN publishes no canonical RDF/OWL for
SIRI or NeTEx. The only existing formal source is an old, outdated Transmodel
ontology. Both `siri.ttl` and NeTEx's `netex.ttl` are therefore placeholders,
and the `-nordic.ttl` profile files currently self-contain the base terms they
need until canonical CEN ontologies exist.

## Content overview

| Section | What it holds |
|---------|---------------|
| Vocabulary declarations | Ontology declaration plus typed definitions of every custom `siri:`/`doc:`/`profile:` term (meta-classes and properties), following `nordic-netex-ontology`'s reuse-before-invention principle |
| Profile | `profile:NordicSIRI` (NSP) — Nordic localisation of SIRI 2.0 (CEN/TS 15531) |
| Relationship to NeTEx | Bridging references to planned NeTEx objects (Line, Route, ServiceJourney, Quay, StopPlace, DatedServiceJourney, …) via `skos:exactMatch`/`closeMatch` |
| Data governance | Real-time producer vs. central aggregator (Entur) roles |
| SIRI services | `SIRI_ET`, `SIRI_SX`, `SIRI_VM`, `SIRI_FM` with their delivery structures |
| Shared objects | The core real-time objects and their properties |
| Enumerations | Controlled vocabularies (occupancy, alteration, progress, …) |
| Key element specifications | Element-level detail (cardinality, semantics) |
| Profile constraints | Executable SHACL shapes (`profile:NSP_{ObjectName}Shape`) enforcing NSP rules beyond XSD |
| Communication patterns | Request/response and publish/subscribe patterns |
| Transmodel alignment | `skos:exactMatch`/`closeMatch` to Transmodel concepts |

## Extension model

Downstream layers can import and build on top without modifying this repository:

```
siri.ttl                           ← SIRI base vocabulary (placeholder, pending CEN)
└─ siri-nordic.ttl                 ← Nordic SIRI Profile (this repo)
   └─ siri-entur.ttl               ← Entur governance + service sub-profiles
      └─ <service>.ttl             ← Per-service constraints (Anshar ET/VM/SX, …)
```

Each layer can:
- **Tighten** — Add stricter SHACL shapes (`sh:minCount`, `sh:maxCount 0`)
- **Extend** — Define new classes, references, or domain properties
- **Link** — Reference URIs from this repo in your own shapes and rules

## Quality & constraints

Nordic Profile constraints are expressed as **executable SHACL shapes**
(`profile:NSP_{ObjectName}Shape`, mirroring NeTEx's `profile:NP_{ClassName}Shape`),
so validation tools (pySHACL, Apache Jena, TopBraid) can run them directly:

| Constraint | SHACL expression | Example (NSP) |
|------------|------------------|---------------|
| Required | `sh:minCount 1; sh:maxCount 1` | `Source`, `Affects` mandatory on `PtSituationElement` |
| Fixed value | `sh:hasValue` | `IsCompleteStopSequence` = `true` (ET) / `false` (VM) |
| Enumerated | `sh:in ( … )` | `Progress` restricted to `open`/`closed` |
| Format | `sh:pattern` | `SituationNumber` = `CODESPACE:SituationNumber:ID` |
| Length | `sh:maxLength` | `Summary` ≤ 160 characters |
| Exclusive choice | `sh:xone` | exactly one of `FramedVehicleJourneyRef` / `VehicleJourneyRef` |

Each shape's `sh:path` points to a named element path (`siri:{Class}_{Element}`),
keeping every constraint traceable to a base-schema element. Conditional rules
that need SPARQL logic (e.g. SX-001 closing windows, SX-009 mode/submode
pairing) are retained as `rdfs:comment` on the relevant shape.

**Remaining follow-ups:** express the conditional rules as `sh:sparql`
constraints, and — once CEN publishes an official SIRI base ontology — replace
the `siri.ttl` placeholder with (or align it to) that source.

## Technology

| Vocabulary | Role |
|------------|------|
| RDF/OWL | Classes and properties |
| SHACL | Profile constraints as executable validation shapes |
| SKOS | Definitions, notation, and cross-vocabulary mapping |
| Turtle (.ttl) | Serialisation format |

## Prefixes

| Prefix | Namespace |
|--------|-----------|
| `siri:` | `https://siri-cen.eu/ontology#` |
| `profile:` | `https://siri-cen.eu/profile#` |
| `doc:` | `https://siri-cen.eu/doc#` |
| `sh:` | `http://www.w3.org/ns/shacl#` |
| `netex:` | `https://netex-cen.eu/ontology#` |
| `dcterms:` | `http://purl.org/dc/terms/` |
| `tm-commons:` | `https://w3id.org/transmodel/commons#` |
| `tm-journeys:` | `https://w3id.org/transmodel/journeys#` |
| `tm-fac:` | `https://w3id.org/transmodel/facilities#` |
| `tm-org:` | `https://w3id.org/transmodel/organisations#` |

## Source

The ontology is derived from the Nordic SIRI Profile documentation in
[`profile-documentation-siri`](https://github.com/hfjelstad/Profile_Documentation_SIRI)
(`Services/`, `Objects/`, `Guides/`), where `LLM/siri-ontology.ttl` is the
generated superset this file was extracted from.

## Further reading

- [W3C RDF Primer](https://www.w3.org/TR/rdf11-primer/) — Introduction to RDF and Turtle syntax
- [W3C SHACL Specification](https://www.w3.org/TR/shacl/) — Shapes Constraint Language
- [Nordic NeTEx Ontology](https://github.com/entur/nordic-netex-ontology) — The NeTEx counterpart this repo mirrors
