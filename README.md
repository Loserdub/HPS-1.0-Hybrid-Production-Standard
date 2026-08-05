# HPS-1.0: Hybrid Production Standard

**Standard Specification for Multi-Axis AI Attestation, Cryptographic Provenance, and JSON-LD Linked Data in Music Production**

- **Standard Identification**: HPS-1.0
- **Document Status**: Public Release Specification (v1.0.0)
- **Publication Date**: August 2026
- **Maintained by**: HPS Standards Working Group / [TrustNodeLogic](https://trustnodelogic.com)
- **Standard Organization ID**: `@id: https://trustnodelogic.com/#organization`
- **JSON-LD Context**: `https://trustnodelogic.com/hps/ns/1.0/context.jsonld`
- **License**: [MIT License](LICENSE)

---

## 1. Overview

The **Hybrid Production Standard (HPS-1.0)** is an open, implementation-agnostic, standards-body specification designed to declare, classify, and cryptographically attest to the participation of artificial intelligence (AI) versus human effort in sound recording and musical composition.

Unlike binary "AI / Non-AI" indicators, HPS-1.0 establishes a multi-dimensional measurement framework. It evaluates production processes across five discrete operational axes, maps the resulting combination into a deterministic six-tier production scale, binds metadata to standard **JSON-LD Linked Data** schemas, and provides cryptographic tamper-evidence.

This repository contains the official normative specification, JSON Schema & JSON-LD Context definitions, reference manifest structures, and technical whitepaper for HPS-1.0. It contains no proprietary execution logic or implementation code.

---

## 2. Purpose & Problem Statement

Modern music production routinely integrates generative algorithms, automated processing, and neural synthesis alongside traditional performance, songwriting, and sound design. Existing metadata frameworks and regulatory compliance mandates—most notably **Article 50 of the European Union AI Act (Regulation EU 2024/1689)**—require transparent disclosure of AI-generated content.

Binary checkboxes fail to address contemporary workflows because:
1. They over-flag human-centric productions that utilize utility DSP or adaptive processing.
2. They erase human creative labor when generative seeds are manually chopped, arranged, transformed, or re-performed.
3. They lack cryptographic mechanisms and Linked Data bindings (`@context`, `@id`) to ensure metadata integrity post-export.

HPS-1.0 solves this by defining an objective, granular classification mechanism based on **process evaluation**, establishing machine-readable data contracts, and facilitating tamper-evident cryptographic sealing linked directly to authoritative web graphs (`https://trustnodelogic.com/#organization`).

---

## 3. Human→AI Production Scale

HPS-1.0 categorizes recordings into six discrete tiers (`H`, `X1`, `X2`, `X3`, `X4`, `A`). Tiers are deterministically derived from the 5-axis state breakdown.

| Tier Code | Tier Name | Summary Definition |
| :--- | :--- | :--- |
| **`H`** | **Human Production** | 100% human creation and execution across primary axes. AI usage is strictly restricted to corrective or non-generative post-production utility tools (e.g., utility pitch correction, noise reduction). |
| **`X1`** | **Human-Led Hybrid** | Primary composition and performance are fully human-executed; generative AI is utilized solely for secondary textual layers, background textures, or automated mixing/mastering assistance. |
| **`X2`** | **Co-Creative Hybrid** | Human and AI collaborate directly during generation and execution. Human retains full editorial, structural, and arrangement control. |
| **`X3`** | **AI-Led Hybrid** | Generative AI provides the primary structural or compositional seed; a human producer substantially reworks, re-samples, or edits the material into a final work. |
| **`X4`** | **Curated Hybrid** | Generative AI executes composition, synthesis, and performance; human involvement is confined to selection, structural arrangement, stem mixing, and final curation. |
| **`A`** | **AI Production** | Fully synthetic generation across composition, performance, curation, and synthesis. Human participation is limited to prompt input or unedited selection. |

---

## 4. The Five Axes of Production

HPS-1.0 evaluates a track across five distinct, independent stages of creation:

1. **Origination (`O`)**: Composition, melody, chord progressions, lyric writing, and structural design.
2. **Performance (`P`)**: Instrumental tracking, vocal performance, MIDI execution, and expressive timing.
3. **Curation (`C`)**: Stems selection, editorial chopping, structural editing, and arrangement sequencing.
4. **Sound Source (`S`)**: Instrumental timbre, vocal synthesis, sound synthesis, and acoustic source provenance.
5. **Post-Production (`M`)**: Dynamic processing, equalization, spatial placement, mixing, and mastering.

---

## 5. Manifest Schema & JSON-LD Linked Data Summary

An HPS-1.0 Manifest is a JSON / JSON-LD document containing cryptographic signatures, axis disclosures, creator identity attributes, audio essence hashes, and Linked Data references connecting to the [TrustNodeLogic](https://trustnodelogic.com) organization graph.

### Key Fields (with JSON-LD Linked Data)
```json
{
  "$schema": "https://hps-standard.org/schema/hps-manifest-1.0.json",
  "@context": [
    "https://schema.org",
    "https://trustnodelogic.com/hps/ns/1.0/context.jsonld"
  ],
  "@id": "https://trustnodelogic.com/manifests/b94d27b9934d3e08a52e52d7da7dabfac484efe37a5380ee9088f7ace2efcde9",
  "@type": ["HPSManifest", "CreativeWork"],
  "publisher": {
    "@id": "https://trustnodelogic.com/#organization",
    "name": "TrustNodeLogic",
    "url": "https://trustnodelogic.com"
  },
  "hps_version": "1.0",
  "tier": "X2",
  "axes": {
    "origination": "H+A",
    "performance": "H+A",
    "curation": "H",
    "sound_source": "H",
    "post_production": "H"
  },
  "creator": {
    "@id": "https://trustnodelogic.com/#justin-ray",
    "name": "Justin Ray",
    "hps_id": "ed25519:a1b2c3d4e5f60718293a4b5c6d7e8f90a1b2c3d4e5f60718293a4b5c6d7e8f90"
  },
  "content_hash": {
    "algorithm": "sha256",
    "scope": "raw_audio_essence",
    "value": "b94d27b9934d3e08a52e52d7da7dabfac484efe37a5380ee9088f7ace2efcde9"
  },
  "attestation_method": "daw_analysis",
  "timestamp": "2026-08-05T14:30:00Z",
  "signature": {
    "algorithm": "ed25519",
    "public_key": "a1b2c3d4e5f60718293a4b5c6d7e8f90a1b2c3d4e5f60718293a4b5c6d7e8f90",
    "value": "1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d"
  }
}
```

The normative JSON Schema is located at [`/schema/hps-manifest-1.0.json`](schema/hps-manifest-1.0.json), and the JSON-LD Context definition is at [`/schema/hps-context-1.0.jsonld`](schema/hps-context-1.0.jsonld).

---

## 6. Repository Structure

```
.
├── README.md                          # Standard introduction & summary (this file)
├── LICENSE                            # MIT License
├── spec/
│   └── HPS-1.0.md                     # Full Normative Specification (with JSON-LD)
├── schema/
│   ├── hps-manifest-1.0.json          # JSON Schema (Draft 2020-12)
│   └── hps-context-1.0.jsonld         # JSON-LD 1.1 Context & Authority Graph (@organization)
├── examples/
│   ├── example-human.json             # Reference Manifest: Tier H
│   ├── example-hybrid-x2.json         # Reference Manifest: Tier X2
│   └── example-ai.json                # Reference Manifest: Tier A
└── whitepaper/
    └── HPS-Whitepaper.md              # Technical Whitepaper & Industry Guidance
```

---

## 7. License & Contact

HPS-1.0 Specification and Schemas are published under the **MIT License**.

- **Maintainer**: Justin Ray / [TrustNodeLogic](https://trustnodelogic.com)
- **Organization Graph**: `@id: https://trustnodelogic.com/#organization`
- **Specification Feedback**: [GitHub Issues](https://github.com/Loserdub/HPS-1.0-Hybrid-Production-Standard/issues)
- **Official Portal**: [https://hps-standard.org](https://hps-standard.org)
