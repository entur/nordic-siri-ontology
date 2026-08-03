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
siri.ttl                ← SIRI base model + Nordic SIRI Profile (this repo)
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
| `siri.ttl` | OWL classes for SIRI services (ET, SX, VM, FM), shared objects (EstimatedVehicleJourney, EstimatedCall, VehicleActivity, PtSituationElement, …), enumerations, key element specifications, communication patterns, NeTEx bridges, and Transmodel alignment. |

## Content overview

| Section | What it holds |
|---------|---------------|
| Profile | `profile:NordicSIRI` (NSP) — Nordic localisation of SIRI 2.0 (CEN/TS 15531) |
| Relationship to NeTEx | Bridging references to planned NeTEx objects (Line, Route, ServiceJourney, Quay, StopPlace, DatedServiceJourney, …) via `skos:exactMatch`/`closeMatch` |
| Data governance | Real-time producer vs. central aggregator (Entur) roles |
| SIRI services | `SIRI_ET`, `SIRI_SX`, `SIRI_VM`, `SIRI_FM` with their delivery structures |
| Shared objects | The core real-time objects and their properties |
| Enumerations | Controlled vocabularies (occupancy, alteration, progress, …) |
| Key element specifications | Element-level detail (cardinality, semantics) |
| Communication patterns | Request/response and publish/subscribe patterns |
| Transmodel alignment | `skos:exactMatch`/`closeMatch` to Transmodel concepts |

## Extension model

Downstream layers can import and build on top without modifying this repository:

```
siri.ttl                           ← Base + Nordic Profile (this repo)
└─ siri-entur.ttl                  ← Entur governance + service sub-profiles
   └─ <service>.ttl                ← Per-service constraints (Anshar ET/VM/SX, …)
```

Each layer can:
- **Tighten** — Add stricter SHACL shapes (`sh:minCount`, `sh:maxCount 0`)
- **Extend** — Define new classes, references, or domain properties
- **Link** — Reference URIs from this repo in your own shapes and rules

## Planned work (quality)

This ontology currently expresses Nordic Profile constraints as **prose**
(`rdfs:comment "Mandatory in Nordic Profile"`, `siri:description "… mandatory …"`).
The primary quality task — mirroring how `nordic-netex-ontology` works — is to
express these as **executable SHACL shapes** so validation tools can run them
directly:

| Constraint | SHACL expression | Example |
|------------|------------------|---------|
| Excluded | `sh:maxCount 0` | Element not used in the Nordic SIRI Profile |
| Allowed | `sh:maxCount 1` | Optional element |
| Required | `sh:minCount 1; sh:maxCount 1` | Element the profile requires |
| Type check | `sh:class` | Reference must point to the correct class |

Suggested shape naming: `profile:NSP_{ObjectName}Shape` (mirrors NeTEx's
`profile:NP_{ClassName}Shape`). A natural follow-up is to split `siri.ttl` into
a base vocabulary file plus a `siri-nordic.ttl` SHACL file, matching NeTEx's
`netex.ttl` + `netex-nordic.ttl` two-file layout.

## Technology

| Vocabulary | Role |
|------------|------|
| RDF/OWL | Classes and properties |
| SHACL | Profile constraints as validatable shapes (planned) |
| SKOS | Definitions, notation, and cross-vocabulary mapping |
| Turtle (.ttl) | Serialisation format |

## Prefixes

| Prefix | Namespace |
|--------|-----------|
| `siri:` | `https://siri-cen.eu/ontology#` |
| `profile:` | `https://siri-cen.eu/profile#` |
| `doc:` | `https://siri-cen.eu/doc#` |
| `netex:` | `https://netex-cen.eu/ontology#` |
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
