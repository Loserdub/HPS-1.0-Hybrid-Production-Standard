# HPS-1.0 Specification: Hybrid Production Standard

**Normative Specification for Multi-Axis AI Attestation, Process Classification, and Cryptographic Provenance in Music Production**

- **Standard Identifier**: HPS-1.0
- **Document Version**: 1.0.6 (Engine-Synchronized Specification)
- **Date**: September 2026
- **Status**: Published Specification
- **Publisher**: HPS Standards Working Group / [TrustNodeLogic](https://trustnodelogic.com)
- **Contact & Inquiries**: [trustnodelogic.com/contact.html](https://trustnodelogic.com/contact.html) · `trustnodelogic@gmail.com`
- **Normative Schema**: `https://hps-standard.org/schema/hps-manifest-1.0.json`
- **JSON-LD Context**: `https://hps-standard.org/ns/1.0/context.jsonld`
- **Reference Implementation**: [hps-attestation-engine](https://github.com/Loserdub/hps-attestation-engine) · Live: [trustnodelogic.web.app](https://trustnodelogic.web.app)

> ### Intellectual Property & Licensing Notice
> - **Specification Documentation**: Copyright © 2026 Justin Ray / TrustNodeLogic. Published under [CC BY-ND 4.0](https://creativecommons.org/licenses/by-nd/4.0/).
> - **Schemas & Data Interchange Formats**: Licensed under the [MIT License](https://opensource.org/licenses/MIT) for open industry interoperability.
> - **Proprietary Technology & Patent Reservation**: This specification defines open data formats, manifest schemas, and classification rules. Nothing herein grants any right, title, license, or interest in or to any patent, trade secret, or proprietary implementation of the TrustNodeLogic attestation engine, forensic detection heuristics, or acoustic watermarking algorithms.
> - **Trademarks**: "HPS", "Hybrid Production Standard", and "TrustNodeLogic" are trademarks of TrustNodeLogic. No trademark license is granted.

---

## 1. Introduction & Scope

### 1.1 Purpose
The Hybrid Production Standard (HPS-1.0) defines an open, implementation-agnostic framework for classifying and attesting to human and artificial intelligence (AI) contributions across the music creation process.

HPS-1.0 addresses the limitation of binary AI disclosure flags ("AI-generated: Yes/No") by establishing a multi-axis evaluation model. The standard measures **process, not tool presence**, allowing precise classification across five discrete creative stages. It provides a formal JSON Manifest schema, deterministic tier classification rules, JSON-LD Linked Data context bindings, cryptographic signature requirements, acoustic watermark embedding, multi-container embedding specifications, Merkle revision audit chains, and DAW forensic parser definitions.

### 1.2 Self-Attestation Boundary
HPS-1.0 is a **self-attestation system with a tamper-evidence layer**, not third-party certification. Concretely:
- The Ed25519 signature proves the manifest was not altered *after* signing by the keyholder. It does **not** prove the axis values are factually true.
- Axis values are user-declared. Auto-detected evidence from DAW forensics is a suggestion the user may override (override events are recorded in the `overrides` array for audit transparency).
- The PCM hash proves audio integrity *after* signing, not provenance of how the audio was created.

Implementors MUST NOT describe HPS output as "certified" or use language implying third-party verification of axis truthfulness. Correct terminology: "self-attested," "declared," "tamper-evident."

### 1.3 Normative References
The following standards contain provisions that, through reference in this text, constitute normative provisions of HPS-1.0:

- **RFC 8032**: *Edwards-Curve Digital Signature Algorithm (Ed25519)*, IETF.
- **FIPS PUB 180-4**: *Secure Hash Standard (SHA-256)*, NIST.
- **RFC 8259**: *The JavaScript Object Notation (JSON) Data Interchange Format*, IETF.
- **W3C JSON-LD 1.1**: *A JSON-based Serialization for Linked Data*, W3C.
- **Schema.org**: *CreativeWork & Organization Vocabularies*, W3C Community Group.
- **DDEX ERN 4.3**: *Electronic Release Notification Message Suite Standard*, DDEX.
- **ID3v2.4.0**: *ID3 Informal Standard for Informal Metadata Container*, ID3.org.
- **RIFF WAV**: *Multimedia Programming Interface and Data Specifications 1.0*, IBM/Microsoft.
- **ISO 3901**: *International Standard Recording Code (ISRC)*.
- **ISO 27729**: *International Standard Name Identifier (ISNI)*.
- **GS1 UPC/EAN**: *Universal Product Code / European Article Number*.

---

## 2. Definitions & Terminology

For the purposes of this specification, the following terms apply:

- **Attestation**: A signed, self-declared statement describing the production process used to create a specific audio recording.
- **Audio Essence**: The raw, uncompressed Pulse Code Modulation (PCM) audio sample data of a recording, extracted exclusively from the WAV `data` subchunk, excluding all RIFF container headers, metadata chunks, and injected watermark data.
- **Axis**: A discrete stage of the music production workflow evaluated independently (`Origination`, `Performance`, `Curation`, `Sound Source`, `Post-Production`).
- **Classifier**: The deterministic function that maps a 5-tuple of axis states into exactly one production tier (`H`, `X1`, `X2`, `X3`, `X4`, `A`). Evaluation is strictly order-dependent; the first matching rule terminates evaluation.
- **DAW Forensics**: The automated inspection of a raw DAW project file (`.als`, `.logicx`, `.flp`, `.ptx`, `.cpr`, `.song`, `.rpp`, `.dawproject`) to extract plugin inventory, sample references, tempo, clip bounds, and track hierarchy for axis suggestion.
- **Fingerprint Engine**: The forensic plugin identification subsystem that classifies detected tools via multi-tier matching (binary identification, contextual lexical matching, and behavioral heuristics) to produce classification evidence and axis suggestions.
- **Generative AI**: Algorithmic or neural network systems capable of producing novel musical structures, stems, lyrics, vocal performances, or synthesized sound sources based on training data or prompt conditioning.
- **HPS Manifest**: The canonical JSON / JSON-LD structure containing the declared axis states, derived tier, creator identity, industry identifiers, audio essence hash, Merkle revision chain, and digital signature.
- **Merkle Audit Chain**: An append-only, hash-linked revision log embedded in the manifest (`revision_history`, `merkle_root`) that enables downstream verifiers to confirm no intermediate metadata edits occurred after initial attestation.
- **Parse Confidence**: A standardized fidelity tier (`high`, `medium`, `partial`, `none`) reported by DAW parsers to indicate the reliability of extracted session data.
- **Tamper-Evidence**: The cryptographic property ensuring that any alteration to the audio essence or manifest payload invalidates the digital signature.
- **Utility DSP**: Non-generative digital signal processing (e.g., static equalizers, compressors, parametric reverb, surgical notch filtering, pitch correction used solely for intonation) that MUST be classified as `H` and never as `A` or `H+A` regardless of internal ML optimization.

---

## 3. Human→AI Production Scale

HPS-1.0 establishes a six-tier production scale reflecting the spectrum of human agency and machine generation.

```
       [H] -------------- [X1] -------------- [X2] -------------- [X3] -------------- [X4] -------------- [A]
Human Production   Human-Led Hybrid    Co-Creative Hybrid    AI-Led Hybrid      Curated Hybrid     AI Production
 (AI Utility Only)  (Human Lead/Perf)  (Direct Co-Creation) (AI Seed/Human Rework) (AI Gen/Human Edit) (100% Synthetic)
```

### 3.1 Tier Definitions

#### Tier H: Human Production
- **Definition**: Composition, performance, curation, and sound sources executed entirely by human creators.
- **AI Allowance**: Generative AI strictly prohibited for content generation. Non-generative utility DSP permitted (see §4.2).
- **Normative Condition**: $O = \text{H}$, $P = \text{H}$, and $hCount \ge 4$.

#### Tier X1: Human-Led Hybrid
- **Definition**: Primary composition and performance are human-driven. AI assistance limited to secondary elements, atmospheric textures, or automated mixing/mastering.
- **AI Allowance**: Secondary generative tools permitted for texture or post-production. Core melody, chords, lyrics, and lead performance remain strictly human.
- **Normative Condition**: $O = \text{H}$, $P = \text{H}$, and $hCount < 4$.

#### Tier X2: Co-Creative Hybrid
- **Definition**: Human creators and AI systems actively collaborate during origination or performance. The producer maintains real-time editorial and directional control.
- **AI Allowance**: Generative assistance on composition or performance, provided AI does not dominate more than two axes.
- **Normative Condition**: $(O = \text{H+A} \lor P = \text{H+A}) \land aCount \le 2$.

#### Tier X3: AI-Led Hybrid
- **Definition**: Generative AI creates the primary compositional or structural seed, subsequently edited, re-arranged, re-sampled, or substantially reworked by a human producer.
- **AI Allowance**: Primary composition or stem generation via AI, subject to substantial human curation or post-processing.
- **Normative Condition**: Default classification for all hybrid configurations not matching Tiers X1, X2, or X4.

#### Tier X4: Curated Hybrid
- **Definition**: Generative AI executes composition, performance, and sound synthesis; human involvement confined to selection, structural arrangement, stem mixing, and final curation.
- **AI Allowance**: AI generates all raw musical assets. Human agency restricted to editorial curation ($C \in \{\text{H}, \text{H+A}\}$).
- **Normative Condition**: $O = \text{A}$, $P = \text{A}$, and $C \in \{\text{H}, \text{H+A}\}$.

#### Tier A: AI Production
- **Definition**: Generative AI executes composition, performance, curation, and synthesis. Human input limited to text prompts, initial seed parameters, or unedited selection.
- **AI Allowance**: Full synthetic generation. No manual human performance, composition, or structural editing.
- **Normative Condition**: $O = \text{A}$, $P = \text{A}$, $C = \text{A}$, and $aCount \ge 4$.

---

## 4. The Five Axes of Production

Every HPS-1.0 attestation requires independent evaluation across five production stages.

```
+-----------------------------------------------------------------------------------+
|                                 THE 5 HPS AXES                                    |
+-------------------+--------------------+--------------------+---------------------+
| 1. Origination    | Composition        | Melody, chords, lyrics, structure seed     |
| 2. Performance    | Tracking & Exec    | Vocals, instruments, MIDI execution         |
| 3. Curation       | Editing & Sequence | Stem selection, chopping, arrangement       |
| 4. Sound Source   | Timbre & Synthesis | Acoustic/analog vs. neural synthesis        |
| 5. Post-Prod      | Mix & Master       | Dynamics, EQ, spatial, final mastering      |
+-------------------+--------------------+--------------------+---------------------+
```

### 4.1 Permissible Axis States

Each axis MUST take exactly one of three values:

1. **`H` (Human)**: Executed exclusively by human agency or traditional non-generative digital processing.
2. **`H+A` (Human + AI Collaborative)**: Executed via interactive collaboration between human creators and generative AI tools.
3. **`A` (AI Autonomous)**: Executed by generative AI systems or neural models without active human performance or compositional modification.

### 4.2 Exemption Rule for Utility Processing
Standard non-generative utility DSP (e.g., static equalizers, dynamic range compressors, parametric reverb, surgical notch filtering, pitch correction used solely for intonation correction, corrective tools such as Melodyne and Auto-Tune) MUST be classified as **`H`**. A tool MUST NOT be classified as `A` or `H+A` merely because it incorporates internal optimization heuristics or machine learning for parameters, unless it generates novel musical content.

### 4.3 Axis Override Discipline
When a user manually sets an axis value that disagrees with auto-detected evidence, the implementing application MUST:
1. Present a non-blocking warning indicating the contradiction (e.g., "Evidence suggests A, you've set H. This override will be recorded in the manifest.").
2. Record the disagreement in the manifest's `overrides` array with the `autoValue`, `overriddenTo`, and `evidence` fields.
3. Never silently accept contradicting values without the audit trail. This is the primary trust mechanism making override events visible and auditable.

---

## 5. 243-State Classifier Evaluation Logic

Evaluating five axes across three possible states yields exactly $3^5 = 243$ distinct combinations. HPS-1.0 specifies an order-dependent classifier function.

### 5.1 Formal Evaluation Rules

Let $O, P, C, S, M \in \{\text{H}, \text{H+A}, \text{A}\}$ represent the five axis values.
Let $hCount = |\{x \in \{O, P, C, S, M\} \mid x = \text{H}\}|$.
Let $aCount = |\{x \in \{O, P, C, S, M\} \mid x = \text{A}\}|$.

The classifier MUST evaluate rules in strict numerical order. The first rule whose condition evaluates to `TRUE` assigns the tier code and terminates evaluation:

$$\begin{aligned}
\text{Rule 1 (Tier H)}:  & \quad \text{IF } O = \text{H} \land P = \text{H} \land hCount \ge 4 \implies \text{Tier } \mathbf{H} \\
\text{Rule 2 (Tier A)}:  & \quad \text{IF } O = \text{A} \land P = \text{A} \land C = \text{A} \land aCount \ge 4 \implies \text{Tier } \mathbf{A} \\
\text{Rule 3 (Tier X4)}: & \quad \text{IF } O = \text{A} \land P = \text{A} \land C \in \{\text{H}, \text{H+A}\} \implies \text{Tier } \mathbf{X4} \\
\text{Rule 4 (Tier X1)}: & \quad \text{IF } O = \text{H} \land P = \text{H} \implies \text{Tier } \mathbf{X1} \\
\text{Rule 5 (Tier X2)}: & \quad \text{IF } (O = \text{H+A} \lor P = \text{H+A}) \land aCount \le 2 \implies \text{Tier } \mathbf{X2} \\
\text{Rule 6 (Tier X3)}: & \quad \text{DEFAULT} \implies \text{Tier } \mathbf{X3}
\end{aligned}$$

> **Ordering rationale**: Rule 2 requires `C == A` as a hard gate for Tier A, and Rule 3 requires `C in {H, H+A}` for Tier X4. This resolves the collision on states like `O=A, P=A, S=A, M=A, C=H` (textbook X4) that would otherwise satisfy a simpler "4-of-5 A" Tier A rule. Any conforming implementation MUST pass exhaustive 243-state collision testing with zero unclaimed states and zero multi-tier collisions.

---

## 6. DAW Forensic Parser Subsystem

Conforming implementations MAY provide a DAW forensic parser subsystem that ingests raw project files to automatically suggest axis values and populate the tool chain.

### 6.1 Supported DAW Formats

| DAW Format | Extensions | Container / Encoding | Default Parse Confidence |
|---|---|---|---|
| Ableton Live | `.als` | GZIP-compressed XML DOM AST | `high (xml-ast)` |
| Logic Pro | `.logicx` / `.zip` | Package archive / binary project structure | `high (logic-native)` |
| FL Studio | `.flp` / `.zip` | Binary event stream | `high (binary-stream)` |
| REAPER | `.rpp` | Plain text nested AST | `high (plain-text-ast)` |
| Studio One | `.song` | ZIP archive (`song.xml`) DOM | `high (zip-xml-ast)` |
| DAWProject | `.dawproject` | Open standard ZIP (`project.xml`) | `high (dawproject-xml)` |
| Pro Tools | `.ptx` / `.pts` | Binary stream forensic scan | `medium (forensic-binary-scan)` |
| Cubase / Nuendo | `.cpr` | RIFF binary forensic scan | `medium (forensic-binary-scan)` |

### 6.2 Parse Confidence Tiers
Parsers MUST report an honest `parseConfidence` tier that is carried into all legal export deliverables:

1. **`high`**: Full AST decoded with track hierarchy, instruments, and clip bounds.
2. **`medium`**: Binary scraper that recovers valid track names and plugins without full AST deserialization.
3. **`partial`**: Audio sample assets discovered in project folders, but session container could not be parsed.
4. **`none`** (`scanFailed: true`): Unrecognized or corrupt file.

### 6.3 Complex Container & Binary Format Handling
Certain project formats (such as Logic Pro binary plists or proprietary binary chunks) require specialized deserialization. Parsers MUST detect unsupported or opaque binary structures and MUST NOT silently fall back to fabricated or assumed data. If a session container cannot be reliably parsed, the system MUST honestly report a reduced confidence tier (e.g., `parseConfidence: 'low'` or `'partial'`) and surface a visible notification to the user.

---

## 7. Forensic AI Fingerprint Engine

The Fingerprint Engine classifies detected plugins and samples via a three-tier hierarchy to suggest axis values with evidence labels.

### 7.1 Matching Tiers (Evaluated in Priority Order)

1. **ExactID**: Evaluates immutable binary plugin identifiers (such as VST3 GUIDs, AudioUnit component IDs, CLAP bundle IDs, or VST2 unique IDs). This guarantees reliable detection of tools even if their display names or bundle labels have been modified.
2. **NameMatch**: Evaluates contextual lexical matching against verified tool databases and plugin nomenclature, utilizing word-boundary constraints to prevent substring false positives.
3. **Behavioral Heuristics**: Evaluates routing topology, node capabilities, and stem generation markers to categorize tool behavior and identify potential generative systems.

### 7.2 Exemption Preservation
Tools classified as corrective utility DSP (Melodyne, Auto-Tune, Soothe2, Pro-Q 3) MUST be tagged `axis: null` — never counted as AI evidence. Do not classify a tool as AI merely because its marketing copy uses "AI-powered."

### 7.3 Sovereign Oracle Architecture
The Fingerprint Engine SHOULD support remote fingerprint database updates (a public reference table fetch, not user data) with local cache fallback and embedded defaults. Any such remote fetch MUST be disclosed in privacy copy. Implementations SHOULD provide a complete offline mode toggle.

---

## 8. HPS Manifest Schema & JSON-LD Linked Data

### 8.1 Canonicalization & JSON Format
HPS Manifests MUST be formatted as valid UTF-8 encoded JSON. Before digital signing or hash integrity checking, the JSON object MUST be canonicalized:
1. Object keys MUST be recursively sorted in lexicographical order (ascending ASCII bytes).
2. Insignificant whitespace MUST be removed.

### 8.2 Complete Field Specification

```json
{
  "$schema": "https://hps-standard.org/schema/hps-manifest-1.0.json",
  "@context": [
    "https://schema.org",
    "https://hps-standard.org/ns/1.0/context.jsonld"
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
  "overrides": [
    {
      "axis": "sound_source",
      "autoValue": "A",
      "overriddenTo": "H",
      "evidence": ["User flagged neural synth stem as re-tracked acoustic cello"]
    }
  ],
  "tool_chain": [
    {
      "stage": "origination",
      "tool": "AIVA Generative Seed Engine",
      "type": "ai_generative"
    }
  ],
  "industry_metadata": {
    "isrc": "US-XYZ-26-00001",
    "isni": "0000000121032310",
    "ipi": "00250165006",
    "upc": "012345678905",
    "label": "Indie Label Records",
    "catalog_number": "ILR-2026-001"
  },
  "creator": {
    "@id": "https://trustnodelogic.com/#creator",
    "name": "Justin Ray",
    "hps_id": "ed25519:7b3a9c8f4d2e1a0b3c5d7e9f1a2b4c6d8e0f2a4b6c8d0e2f4a6b8c0d2e4f6a8b0"
  },
  "content_hash": {
    "algorithm": "sha256",
    "scope": "raw_audio_essence",
    "value": "b94d27b9934d3e08a52e52d7da7dabfac484efe37a5380ee9088f7ace2efcde9"
  },
  "attestation_method": "daw_analysis",
  "daw_parse_confidence": "high (xml-ast)",
  "timestamp": "2026-08-05T12:00:00Z",
  "revision_history": [
    {
      "revision": 1,
      "timestamp": "2026-08-05T12:00:00Z",
      "hash": "b94d27b9934d3e08a52e52d7da7dabfac484efe37a5380ee9088f7ace2efcde9"
    }
  ],
  "merkle_root": "c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d4",
  "signature": {
    "algorithm": "ed25519",
    "public_key": "7b3a9c8f4d2e1a0b3c5d7e9f1a2b4c6d8e0f2a4b6c8d0e2f4a6b8c0d2e4f6a8b0",
    "value": "9a2f1c8d4e6b0f2a4c8e0b2d4f6a8c0e2b4d6f8a0c2e4b6d8f0a2c4e6b8d0f2a4"
  },
  "signatures": [
    {
      "signer": "Co-Producer Name",
      "role": "producer",
      "public_key": "d3f4a18b92e70c54109823f6e147ab95c80123456789abcdef0123456789abcd",
      "value": "1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d",
      "timestamp": "2026-08-05T13:00:00Z"
    }
  ]
}
```

### 8.3 Required vs. Optional Fields

**Required** (`$schema`, `hps_version`, `tier`, `axes`, `tool_chain`, `creator`, `content_hash`, `attestation_method`, `timestamp`, `signature`)

**Optional** (`@context`, `@id`, `@type`, `publisher`, `keywords`, `overrides`, `industry_metadata`, `daw_parse_confidence`, `revision_history`, `merkle_root`, `signatures`)

### 8.4 Industry Metadata Fields

| Field | Standard | Format | Manifest Path |
|---|---|---|---|
| ISRC | ISO 3901 | 12 chars (CC-XXX-YY-NNNNN) | `industry_metadata.isrc` |
| ISNI | ISO 27729 | 16 digits | `industry_metadata.isni` |
| IPI / CAE | CISAC | 9–11 digits | `industry_metadata.ipi` |
| UPC / EAN | GS1 | 12–13 digits | `industry_metadata.upc` |
| Record Label | — | Freeform string | `industry_metadata.label` |
| Catalog # | — | Freeform string | `industry_metadata.catalog_number` |

### 8.5 Linking to Publisher Authority Graph
To establish decentralized authority binding, every HPS Manifest SHOULD include a JSON-LD `publisher` block resolving to the publisher's organization graph:

```json
{
  "@id": "https://trustnodelogic.com/#organization",
  "@type": "Organization",
  "name": "TrustNodeLogic",
  "url": "https://trustnodelogic.com",
  "founder": {
    "@id": "https://trustnodelogic.com/#creator",
    "@type": "Person",
    "name": "Justin Ray"
  }
}
```

### 8.6 Cryptographic Signature Scheme
- **Algorithm**: Ed25519 (RFC 8032).
- **Signed Message**: The canonicalized JSON payload of the manifest excluding the `signature` key itself.
- **Audio Integrity**: The `content_hash.value` field MUST contain the SHA-256 digest of the raw PCM audio essence bytes (WAV `data` subchunk only, not the re-encoded or watermarked buffer unless that buffer is the final shipped artifact).
- **Multi-Signer**: Additional co-signers (producers, labels, writers) MAY be recorded in the `signatures[]` array using the same Ed25519 scheme. This is distinct from distributor attestation, which is a separate future `attestations[]` field.

### 8.7 Merkle Revision Audit Chain
The `revision_history` array and `merkle_root` field implement an append-only, hash-linked audit log:
- Each revision entry records a sequential revision index, timestamp, and content hash.
- The `merkle_root` is the root of a Merkle tree built over the revision hashes.
- Downstream verifiers can confirm no intermediate metadata edits occurred after initial attestation by recomputing the Merkle root from the revision history.

---

## 9. Identity Keystore Protocol (`.hpskey`)

The Ed25519 signing identity is generated locally in the browser (never transmitted) and may be exported and imported as a standardized `.hpskey` JSON vault:

```json
{
  "$schema": "https://hps-standard.org/schemas/v0.1/keystore.json",
  "keystore_version": "1.0",
  "created_at": "2026-08-22T16:25:00.000Z",
  "algorithm": "ed25519",
  "name": "Artist or Producer Name",
  "hps_id": "ed25519:e0a4f58c73b1842...",
  "public_key": "e0a4f58c73b1842...",
  "private_key": "4c3d2e1a9b8c7d6...",
  "client": "TrustNodeLogic HPS Attestation Engine v1.0.6"
}
```

On import, implementations MUST:
1. Verify 64 hex characters (32 bytes) for the private key.
2. Cryptographically derive the public key from the private key.
3. Assert `derived_public_key === provided_public_key` before writing to storage. Reject forged or corrupt keys.

---

## 10. Acoustic Watermark Principles & Interoperability

### 10.1 Functional Architecture
To ensure provenance survives physical playback, analog re-recording, and lossy audio transcoding, conforming implementations MAY incorporate an inaudible acoustic watermark directly within the audio essence.

- **Modulation Scheme**: Time-domain Direct-Sequence Spread-Spectrum (DSSS) or psychoacoustically shaped spread-spectrum phase modulation embedded into PCM samples.
- **Acoustic Transparency**: Watermark amplitude MUST be dynamically governed by psychoacoustic masking principles relative to local signal energy. Watermark injection MUST automatically attenuate to zero energy during passages of digital silence to prevent noise floor elevation.
- **Channel Support**: Watermarking SHOULD be embedded across individual audio channels to withstand mono downmixing or channel isolation.
- **Security Boundary**: The acoustic watermark provides a persistent binding to the manifest or creator signature material; primary cryptographic authenticity and non-repudiation are guaranteed by the Ed25519 digital signature, not by secrecy of the spread-spectrum sequence.

### 10.2 Blind Decoding & Extraction Requirements
Detection and recovery of embedded watermark payloads MUST support blind decoding:
- **Reference-Free Extraction**: Decoders MUST NOT require access to the unwatermarked original master audio.
- **Synchronization Resilience**: The extraction subsystem MUST accommodate temporal shifts, leading/trailing silence padding, and sample-rate conversions.
- **Interference Mitigation**: Extraction algorithms SHOULD employ differential filtering or correlation techniques to mitigate host-audio interference.

### 10.3 Embedding Integrity Check
After signing and watermark embedding, implementations SHOULD conduct an immediate loopback self-check verifying container chunk presence, audio hash verification, and signature validity before delivering the file to the user. Any failure MUST be surfaced as a visible user warning; silent failure is not acceptable.

---

## 11. Implementation & Container Embedding Guidance

### 11.1 RIFF WAV Container Embedding (`hps1` Chunk)
For WAVE audio files, the canonical manifest MUST be stored in a custom RIFF chunk:
- **Chunk ID**: `hps1` FourCC (`0x31737068` in little-endian ASCII). The manifest chunk also carries a `HPS1` magic marker inside the chunk data.
- **Chunk Data**: UTF-8 encoded canonical manifest JSON string.
- **Audio Essence Hash**: Computed over the raw PCM bytes of the `data` subchunk before RIFF re-assembly. Modifying ID3 tags, BWF metadata, or embedded XML MUST NOT affect the audio essence hash.

### 11.2 MP3 Container Embedding (ID3v2.4 `TXXX` Frame)
For MPEG-1 Audio Layer III files, the manifest SHOULD be stored in an ID3v2.4 frame:
- **Frame ID**: `TXXX`
- **Description**: `HPS_MANIFEST_1.0`
- **Value**: Canonical manifest JSON string.

> **Implementation status note**: MP3 watermark recovery on the verification path is implemented. MP3 as a signing input (attestation creation) should be confirmed against the reference implementation before claiming full MP3 support.

### 11.3 JSON Sidecar (`.hps.json`)
The manifest MAY be exported as a standalone `.hps.json` sidecar file for formats without native chunk embedding support.

### 11.4 DDEX ERN 4.3 XML Metadata Block
Distributor delivery XML messages MUST embed classification details under the namespace `xmlns:hps="https://trustnodelogic.com/hps/ns/1.0"`:

```xml
<hps:HPSClassificationBlock xmlns:hps="https://trustnodelogic.com/hps/ns/1.0">
  <hps:Tier>X2</hps:Tier>
  <hps:AttestationMethod>daw_analysis</hps:AttestationMethod>
  <hps:CreatorHpsId>ed25519:7b3a9c...8f12</hps:CreatorHpsId>
  <hps:ManifestPublicKey>7b3a9c...8f12</hps:ManifestPublicKey>
  <hps:ManifestSignatureValue>9a2f1c...4b8e</hps:ManifestSignatureValue>
  <hps:RawManifestPayload>eyIkc2NoZW1hI...==</hps:RawManifestPayload>
  <hps:ParseConfidence>high (xml-ast)</hps:ParseConfidence>
  <hps:CryptographicProof>
    <hps:ContentHash>b94d27b9...</hps:ContentHash>
    <hps:Ed25519Signature>9a2f1c...4b8e</hps:Ed25519Signature>
  </hps:CryptographicProof>
  <hps:IndustryMetadata>
    <hps:ISRC>US-XYZ-26-00001</hps:ISRC>
    <hps:ISNI>0000000121032310</hps:ISNI>
    <hps:IPI>00250165006</hps:IPI>
    <hps:UPC>012345678905</hps:UPC>
    <hps:Label>Indie Label Records</hps:Label>
    <hps:CatalogNumber>ILR-2026-001</hps:CatalogNumber>
  </hps:IndustryMetadata>
  <hps:Axes>
    <hps:Origination>H+A</hps:Origination>
    <hps:Performance>H+A</hps:Performance>
    <hps:Curation>H</hps:Curation>
    <hps:SoundSource>H</hps:SoundSource>
    <hps:PostProduction>H</hps:PostProduction>
  </hps:Axes>
</hps:HPSClassificationBlock>
```

---

## 12. Streaming Integration SDK

Conforming implementations MAY provide an embeddable client-side SDK for DSPs and streaming platforms to verify HPS manifest signatures, audio hashes, and Merkle logs on playback:

```javascript
// Attach HPS provenance badge to an HTML5 audio element
HpsStreamSDK.attachToAudioElement(player);
// Renders a floating [HPS-X1] badge; click reveals full axis breakdown
```

DSP badging format: `[HPS-H]`, `[HPS-X1]`, `[HPS-X2]`, `[HPS-X3]`, `[HPS-X4]`, `[HPS-A]` — badge click MUST reveal the full axis breakdown, not just the tier code.

---

## 13. Regulatory Compliance (EU AI Act Article 50)

HPS-1.0 is engineered to satisfy transparency mandates under **Article 50 of Regulation (EU) 2024/1689 (EU AI Act)**.

1. **Phased Compliance Timeline**: General Article 50 transparency obligations apply from **August 2, 2026**. The specific machine-readable marking requirement (Article 50(2)) is phased to **December 2, 2026** for AI systems already on market before August 2, 2026, under the May 2026 Digital Omnibus agreement. Implementations SHOULD use the framing *"ahead of the EU AI Act's phased transparency and machine-readable marking requirements (Article 50, Aug 2 – Dec 2, 2026)"* rather than citing a single date.

2. **Provider/Deployer Scope**: Article 50 obligations fall on **providers and deployers of AI systems**, not automatically on individual musicians using AI tools in production. Copy MUST NOT imply individual creators have personal hard legal deadlines.

3. **Article 50(2) Marking Requirement**: Mandates that deployers of AI systems generating audio mark the output in a machine-readable format. HPS-1.0 provides machine-readable JSON manifests embedded directly into RIFF container chunks and DDEX XML releases.

4. **Tamper-Evidence & Integrity Check**: The combination of Ed25519 signing, SHA-256 essence hashing, and Merkle audit chain prevents metadata falsification or post-export modification.

5. **Auditability of Overrides**: The `overrides` schema element records manual overrides of automated detection, providing transparent audit trails for legal and rights management review.

> **Legal caveat**: Nothing in this specification constitutes legal advice. "EU AI Act compliant" or "distributor-compliant" language MUST NOT appear in distributor-facing materials without independent legal review.

---

## 14. Security & Cryptographic Considerations

Conforming implementations MUST enforce the following cryptographic and operational security properties:

### 14.1 Canonicalization & Signature Malleability Resistance
To prevent signature malleability or bypass attacks via whitespace injection, attribute permutation, or numerical serialization variances:
- Manifest payloads MUST be canonicalized strictly adhering to **RFC 8785 (JSON Canonicalization Scheme - JCS)** prior to computing the Ed25519 signature or signature verification.
- The `signature` object itself MUST be omitted from the canonicalized byte stream during signature verification.
- Verifiers MUST reject any manifest containing duplicate keys or non-standard encodings.

### 14.2 Audio Essence Binding & Replay Resistance
To prevent transplantation attacks (where a signed manifest from an authentic track is re-attached to unauthenticated or synthetic audio):
- The `content_hash.value` MUST be computed exclusively across the uncompressed PCM audio bytes within the WAV `data` subchunk.
- Verifiers MUST recalculate the SHA-256 digest of the audio essence and verify an exact byte match before evaluating manifest validity. If the digest differs by even a single bit, verification MUST fail with `AudioEssenceMismatch`.
- Container metadata modifications (e.g., updating ID3 tags, Broadcast Wave BWF chunks, or RIFF info tags) MUST NOT alter the audio essence digest.

### 14.3 Private Key Security & Self-Sovereignty
- Conforming creator applications MUST generate Ed25519 keypairs client-side using cryptographically secure random number generators (e.g., `crypto.getRandomValues()`).
- Private keys MUST NEVER be transmitted across external networks, cloud synchronizers, or analytics endpoints.
- Stored identity vaults (`.hpskey`) MUST enforce public key re-derivation on import to prevent corrupted or malicious key injection (§9).

### 14.4 Tamper-Evidence of Audit Logs & Overrides
- The `overrides[]` array records intentional divergences between automated forensic detection and human user declarations.
- Because the `overrides[]` array is an integral component of the canonical signed payload, an adversary cannot delete, truncate, or alter auto-detected evidence records without invalidating the Ed25519 signature.
- Verifiers SHOULD surface any recorded overrides to downstream users or auditors to ensure complete disclosure transparency.

### 14.5 Limitation of Factual Truth Claims
Implementors, distributors, and verification tools MUST adhere to the self-attestation boundary (§1.2). A cryptographically valid signature proves non-repudiation and post-signing integrity; it does not constitute third-party proof of factual truthfulness. Verification interfaces MUST use precise status indicators (e.g., `"Signature: Valid"`, `"Audio Integrity: Intact"`, `"Process: Self-Attested"`) rather than misleading certification claims.

---

## 15. Governance & Versioning Policy

HPS-1.0 is governed by the HPS Technical Working Group under Semantic Versioning 2.0.0 rules:
- **Major Releases (`X.0.0`)**: Backwards-incompatible schema changes or rule redefinitions.
- **Minor Releases (`1.X.0`)**: Additive metadata fields, expanded tool classification types, or non-breaking XML/JSON extensions.
- **Patch Releases (`1.0.X`)**: Textual errata, specification clarifications, and documentation updates.

---

## 16. Technical Glossary

- **BPSK**: Binary Phase-Shift Keying. Modulation scheme used by the HPS acoustic watermark.
- **CLAP**: CLever Audio Plugin. Cross-platform plugin format with bundle ID.
- **DAWProject**: Open cross-DAW project interchange standard.
- **DSSS**: Direct-Sequence Spread-Spectrum. Technique spreading a signal over a wider bandwidth for noise resistance.
- **Ed25519**: Edwards-curve Digital Signature Algorithm using Curve25519 (RFC 8032).
- **ExactID**: Binary plugin identifier matching (VST3 GUID, AU ID, CLAP ID) used by the Fingerprint Engine.
- **ISNI**: International Standard Name Identifier (ISO 27729).
- **ISRC**: International Standard Recording Code (ISO 3901).
- **IPI/CAE**: Interested Parties Information — performing rights organization identifier (CISAC).
- **JCS**: JSON Canonicalization Scheme (RFC 8785).
- **JSON-LD**: JavaScript Object Notation for Linked Data (W3C Standard).
- **Merkle Root**: Cryptographic root hash of a Merkle tree constructed over a manifest's revision history.
- **PCM**: Pulse Code Modulation. Uncompressed digital audio sample representation.
- **RIFF**: Resource Interchange File Format. Container format used for WAV audio.
- **SHA-256**: Secure Hash Algorithm 256-bit cryptographic digest function.
- **Sovereign Oracle**: Remote fingerprint database update mechanism with local cache fallback and embedded defaults.
- **UPC/EAN**: Universal Product Code / European Article Number (GS1).

