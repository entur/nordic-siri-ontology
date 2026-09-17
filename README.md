# Nordic SIRI Ontology

The **Nordic SIRI Profile** as a machine-readable overlay on top of the
generated SIRI base ontology. This repository holds the profile layer only —
service scope, message structure, profile constraints, and cross-standard
references — and imports the CEN-owned base rather than re-deriving it.

> **Status:** Active. The dedicated ontology layer for the Nordic SIRI Profile,
> maintained in the Entur organisation. It mirrors the split used for NeTEx,
> where [`nordic-netex-ontology`](https://github.com/entur/nordic-netex-ontology)
> is the dedicated ontology, separate from the documentation repository.

## Purpose

- **For humans:** A precise, navigable reference for how SIRI services,
  objects, and constraints relate to each other, and how they reference
  planned NeTEx objects.
- **For machines:** A foundation for automated validation (SHACL),
  documentation generation, and tooling integration.

## Architecture

The SIRI base vocabulary is **generated** — projected deterministically from
the official SIRI XSD by the [siri-ontology-generator](#base-ontology-generated)
(intended to be CEN-owned). This repository builds the Nordic layer on top:

```
base/ (siri.ttl + modules)          ← generated SIRI base — vendored snapshot (CEN-owned)
└─ siri-nordic.ttl                  ← Nordic SIRI Profile: scope, governance, rules
   ├─ siri-nordic-vocab.ttl         ← Nordic vocabulary (nordic:)
   ├─ siri-nordic-baseline.ttl      ← positive membership allowlist (in-profile classes/fields)
   ├─ siri-nordic-model.ttl         ← curated service catalogue, containment & element specs
   ├─ siri-transmodel-alignment.ttl ← SIRI ⇄ Transmodel (skos)
   └─ siri-netex-bridge.ttl         ← SIRI ⇄ NeTEx planned-object references
      └─ <your-layer>.ttl           ← Organisation, service, codespace, …
```

**Design principle:** The generated base stays a faithful, mechanical projection
of the standard. The Nordic layer only *tightens* (SHACL, planned), *annotates*,
and *aligns* — it never renames or forks the base. SIRI is **separate from NeTEx
by design** — SIRI is real-time data that *references* planned NeTEx objects;
those cross-references live in `siri-netex-bridge.ttl`, while the canonical NeTEx
definitions live in [`nordic-netex-ontology`](https://github.com/entur/nordic-netex-ontology).

## Base ontology (generated)

The base is produced by `siri-ontology-generator`, which projects the SIRI XSD
into RDF/OWL and splits it into per-module documents (`siri.ttl` root plus
`siri-core`, `siri-framework`, `siri-model`, `siri-et`, `siri-sx`, `siri-vm`,
`siri-fm`, `siri-acsb`, `siri-ifopt`, `siri-gml`, …). Terms keep their SIRI
identity in the single `siri:` namespace; only the documents are split.

**Naming philosophy:** term names follow the SIRI XSD (and the standard RDF
convention): **PascalCase classes** (`siri:EstimatedVehicleJourney`) and
**lowerCamelCase properties** (`siri:monitoredVehicleJourney`). Transmodel
governs *alignment*, not naming — the mapping lives in
`siri-transmodel-alignment.ttl` via `skos:exactMatch` / `skos:closeMatch`, so
SIRI keeps its own identity.

> **TODO (living branch):** `base/` is a checked-in snapshot of the generator's
> `output/` (SIRI v2.2, `v2.2-6-g7b1463b`), so the repository is self-contained
> and loadable today. Once the generator is hosted (CEN), replace the manual
> snapshot with an automated ingest that refreshes `base/` from the published
> output.

## Files

| File | Contents |
|------|----------|
| `base/` | Vendored snapshot of the generated SIRI base — the `siri.ttl` root plus 20 per-module documents. Projected from the SIRI XSD (v2.2); not hand-edited. |
| `siri-nordic.ttl` | Profile definition (`profile:NordicSIRI`, NSP), data governance, and the Nordic Profile constraint rules (ET/VM/SX/ID/DATA). Imports the generated base, vocab + baseline. |
| `siri-nordic-vocab.ttl` | Nordic-invented vocabulary in the `nordic:` namespace (profile meta-classes, structural containment predicates, cross-reference/element description vocabulary, navigational chains, constraint vocabulary, membership & provenance, NeTEx bridge property). |
| `siri-nordic-baseline.ttl` | Positive membership allowlist — which SIRI classes and fields are in the profile, with cardinality, governing `profile:` scope and `nordic:provenance nordic:Baseline`. Seeded once from `siri-nordic-model.ttl`; henceforth the profile inherits from CCB decisions, not the documentation. |
| `siri-nordic-model.ttl` | Curated structural overlay: service catalogue, delivery/object containment, shared objects, enumerations, key element specifications, service chains, cross-service links, communication patterns, and documentation pointers. |
| `siri-transmodel-alignment.ttl` | `skos:exactMatch` / `skos:closeMatch` alignment from generated SIRI classes to Transmodel concepts. |
| `siri-netex-bridge.ttl` | Which planned NeTEx objects each SIRI class references, plus the per-element reference mapping (`LineRef → netex:Line`, `StopPointRef → netex:Quay`, …). |

## Extension model

Downstream layers import and build on top without modifying this repository:

```
siri-nordic.ttl                    ← Base overlay + Nordic Profile (this repo)
└─ siri-entur.ttl                  ← Entur governance + service sub-profiles
   └─ <service>.ttl                ← Per-service constraints (Anshar ET/VM/SX, …)
```

Each layer can:
- **Tighten** — Add stricter SHACL shapes (`sh:minCount`, `sh:maxCount 0`)
- **Extend** — Define new classes, references, or domain properties
- **Link** — Reference generated `siri:`, `netex:` and `nordic:` terms in its own rules

## Planned work (SHACL)

Nordic Profile constraints are currently expressed as **rules in prose**
(`siri-nordic.ttl`, `nordic:constraint`). The primary quality task — mirroring
how `nordic-netex-ontology` works — is to promote these to **executable SHACL
shapes** so validation tools can run them directly against SIRI data:

| Constraint | SHACL expression | Example |
|------------|------------------|---------|
| Excluded | `sh:maxCount 0` | Element not used in the Nordic SIRI Profile |
| Allowed | `sh:maxCount 1` | Optional element |
| Required | `sh:minCount 1; sh:maxCount 1` | Element the profile requires |
| Type check | `sh:class` | Reference must point to the correct class |

Shape naming: `profile:NSP_{ClassName}Shape` (mirrors NeTEx's
`profile:NP_{ClassName}Shape`).

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
| `nordic:` | `https://siri-cen.eu/nordic#` |
| `profile:` | `https://siri-cen.eu/profile#` |
| `doc:` | `https://siri-cen.eu/doc#` |
| `netex:` | `https://netex-cen.eu/ontology#` |
| `sh:` | `http://www.w3.org/ns/shacl#` |
| `tm-journeys:` | `https://w3id.org/transmodel/journeys#` |
| `tm-fac:` | `https://w3id.org/transmodel/facilities#` |

## Tools

The ontology can be consumed by any standard RDF/SHACL tooling, e.g.:

- **pySHACL** — Validate SIRI data against profile shapes (once SHACL is added)
- **Apache Jena** — SPARQL queries
- **TopBraid / Protégé** — Visual exploration and editing
- **Custom scripts/agents** — Import the `.ttl` files via `owl:imports` or load directly

## Source

The ontology is derived from the Nordic SIRI Profile documentation
(`Services/`, `Objects/`, `Guides/`), from which the classes, references,
enumerations, and profile constraints modelled here were extracted.

## Further reading

- [W3C RDF Primer](https://www.w3.org/TR/rdf11-primer/) — Introduction to RDF and Turtle syntax
- [W3C SHACL Specification](https://www.w3.org/TR/shacl/) — Shapes Constraint Language
- [Nordic NeTEx Ontology](https://github.com/entur/nordic-netex-ontology) — The NeTEx counterpart this repo mirrors
- [Transmodel](https://www.transmodel-cen.eu/) — The conceptual model behind SIRI and NeTEx
