# HPS-1.0 Specification: Hybrid Production Standard

**Normative Specification for Multi-Axis AI Attestation, Process Classification, and Cryptographic Provenance in Music Production**

- **Standard Identifier**: HPS-1.0
- **Document Version**: 1.0.0 (Final Specification)
- **Date**: August 2026
- **Status**: Published Specification
- **Publisher**: HPS Standards Working Group / TrustNodeLogic (`https://trustnodelogic.com`)
- **Normative Schema**: `https://hps-standard.org/schema/hps-manifest-1.0.json`
- **JSON-LD Context**: `https://trustnodelogic.com/hps/ns/1.0/context.jsonld`

---

## 1. Introduction & Scope

### 1.1 Purpose
The Hybrid Production Standard (HPS-1.0) defines an open, implementation-agnostic framework for classifying and attesting to human and artificial intelligence (AI) contributions across the music creation process.

HPS-1.0 addresses the limitation of binary AI disclosure flags ("AI-generated: Yes/No") by establishing a multi-axis evaluation model. The standard measures process, not tool presence, allowing precise classification across five discrete creative stages. It provides a formal JSON Manifest schema, deterministic tier classification rules, JSON-LD Linked Data context bindings, cryptographic signature requirements, and multi-container embedding specifications.

### 1.2 Normative References
The following standards contain provisions that, through reference in this text, constitute normative provisions of HPS-1.0:

- **RFC 8032**: *Edwards-Curve Digital Signature Algorithm (Ed25519)*, Internet Engineering Task Force (IETF).
- **FIPS PUB 180-4**: *Secure Hash Standard (SHA-256)*, National Institute of Standards and Technology (NIST).
- **RFC 8259**: *The JavaScript Object Notation (JSON) Data Interchange Format*, IETF.
- **W3C JSON-LD 1.1**: *A JSON-based Serialization for Linked Data*, World Wide Web Consortium (W3C).
- **Schema.org**: *CreativeWork & Organization Vocabularies*, W3C Community Group.
- **DDEX ERN 4.3**: *Electronic Release Notification Message Suite Standard*, Digital Data Exchange (DDEX).
- **ID3v2.4.0**: *ID3 Informal Standard for Informal Metadata Container*, ID3.org.
- **RIFF WAV**: *Multimedia Programming Interface and Data Specifications 1.0 (WAVE Form Audio File Format)*, IBM/Microsoft.

---

## 2. Definitions & Terminology

For the purposes of this specification, the following terms apply:

- **Attestation**: A signed, self-declared statement describing the production process used to create a specific audio recording.
- **Audio Essence**: The raw, uncompressed Pulse Code Modulation (PCM) audio sample data of a recording, excluding container headers or metadata chunks.
- **Axis**: A discrete stage of the music production workflow evaluated independently (`Origination`, `Performance`, `Curation`, `Sound Source`, `Post-Production`).
- **Classifier**: The deterministic function that maps a 5-tuple of axis states into exactly one production tier (`H`, `X1`, `X2`, `X3`, `X4`, `A`).
- **Generative AI**: Algorithmic or neural network systems capable of producing novel musical structures, stems, lyrics, vocal performances, or synthesized sound sources based on training data or prompt conditioning.
- **HPS Manifest**: The canonical JSON / JSON-LD structure containing the declared axis states, derived tier, creator identity attributes, audio essence hash, and digital signature.
- **JSON-LD Linked Data**: Semantic Web metadata structure utilizing `@context`, `@id`, and `@type` parameters to bind manifestations to authoritative web graph entities (`https://trustnodelogic.com/#organization`).
- **Tamper-Evidence**: The cryptographic property ensuring that any alteration to the audio essence or manifest payload invalidates the digital signature.

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
- **Definition**: The composition, performance, curation, and sound sources are entirely executed by human creators.
- **AI Allowance**: Generative AI tools are strictly prohibited for content generation. Non-generative utility DSP tools (e.g., utility pitch correction, noise reduction, static EQ) are permitted provided they do not generate novel structural or timbral material.
- **Normative Condition**: $O = \text{H}$, $P = \text{H}$, and at least four axes evaluated as $\text{H}$.

#### Tier X1: Human-Led Hybrid
- **Definition**: Primary composition and performance are human-driven. AI assistance is limited to secondary elements, atmospheric textures, or automated mixing/mastering processing.
- **AI Allowance**: Secondary generative tools permitted for texture or post-production. Core melody, chords, lyrics, and lead performance remain strictly human.
- **Normative Condition**: $O = \text{H}$, $P = \text{H}$, and fewer than four axes evaluated as $\text{H}$.

#### Tier X2: Co-Creative Hybrid
- **Definition**: Human creators and AI systems actively collaborate during origination or performance stages. The human producer maintains real-time editorial and directional control.
- **AI Allowance**: Generative assistance on composition or performance, provided AI does not dominate more than two axes ($\text{count}(\text{A}) \le 2$).
- **Normative Condition**: $(O = \text{H+A} \lor P = \text{H+A}) \land \text{count}(\text{A}) \le 2$.

#### Tier X3: AI-Led Hybrid
- **Definition**: Generative AI creates the primary compositional or structural seed, which is subsequently edited, re-arranged, re-sampled, or substantially re-worked by a human producer.
- **AI Allowance**: Primary composition or stem generation via AI, subject to substantial human curation or post-processing.
- **Normative Condition**: Default classification for hybrid configurations not matching Tiers X1, X2, or X4.

#### Tier X4: Curated Hybrid
- **Definition**: Generative AI executes composition, performance, and sound synthesis; human involvement is confined to selection, structural arrangement, stem mixing, and final curation.
- **AI Allowance**: AI generates all raw musical assets. Human agency is restricted to editorial curation ($C \in \{\text{H}, \text{H+A}\}$).
- **Normative Condition**: $O = \text{A}$, $P = \text{A}$, and $C \in \{\text{H}, \text{H+A}\}$.

#### Tier A: AI Production
- **Definition**: Generative AI executes composition, performance, curation, and synthesis. Human input is limited to text prompts, initial seed parameters, or unedited selection.
- **AI Allowance**: Full synthetic generation. No manual human performance, composition, or structural editing.
- **Normative Condition**: $O = \text{A}$, $P = \text{A}$, $C = \text{A}$, and at least four axes evaluated as $\text{A}$.

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
Standard non-generative utility DSP (e.g., static equalizers, dynamic range compressors, parametric reverb, surgical notch filtering, and utility pitch correction used solely for intonation correction) MUST be classified as **`H`**. A tool MUST NOT be classified as `A` or `H+A` merely because it incorporates internal optimization heuristics or machine learning for parameters, unless it generates novel musical content.

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

---

## 6. HPS Manifest Schema & JSON-LD Linked Data

### 6.1 Canonicalization & JSON Format
HPS Manifests MUST be formatted as valid UTF-8 encoded JSON adhering to `schema/hps-manifest-1.0.json`.

Before digital signing or hash verification, the JSON object MUST be canonicalized:
1. Object keys MUST be recursively sorted in lexicographical order (ascending ASCII bytes).
2. Insignificant whitespace (spaces, tabs, newlines outside string literals) MUST be removed.

### 6.2 Fields & JSON-LD Specification

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
  "creator": {
    "@id": "https://trustnodelogic.com/#justin-ray",
    "name": "Justin Ray",
    "hps_id": "ed25519:7b3a9c...8f12"
  },
  "content_hash": {
    "algorithm": "sha256",
    "scope": "raw_audio_essence",
    "value": "b94d27b9934d3e08a52e52d7da7dabfac484efe37a5380ee9088f7ace2efcde9"
  },
  "attestation_method": "daw_analysis",
  "timestamp": "2026-08-05T12:00:00Z",
  "signature": {
    "algorithm": "ed25519",
    "public_key": "7b3a9c...8f12",
    "value": "9a2f1c...4b8e"
  }
}
```

### 6.3 Linking to TrustNodeLogic Authority Graph (`@organization`)
To establish decentralized authority binding, every HPS Manifest SHOULD include a JSON-LD `publisher` block resolving to the TrustNodeLogic organization graph:

```json
{
  "@id": "https://trustnodelogic.com/#organization",
  "@type": "Organization",
  "name": "TrustNodeLogic",
  "url": "https://trustnodelogic.com",
  "founder": {
    "@id": "https://trustnodelogic.com/#justin-ray",
    "@type": "Person",
    "name": "Justin Ray"
  }
}
```

This ensures semantic web indexers, legal auditors, and DSP ingestion engines can trace attestation provenance back to the authoritative standards body without relying on proprietary centralized registries.

### 6.4 Cryptographic Signature Scheme
- **Algorithm**: Ed25519 (Edwards-curve Digital Signature Algorithm per RFC 8032).
- **Signed Message**: The canonicalized JSON payload of the manifest excluding the `signature` key itself.
- **Audio Integrity**: The `content_hash.value` field MUST contain the SHA-256 digest of the raw PCM audio essence.

---

## 7. Regulatory Compliance (EU AI Act Article 50)

HPS-1.0 is engineered to satisfy transparency mandates under **Article 50 of Regulation (EU) 2024/1689 (EU AI Act)**.

1. **Article 50(2) Marking Requirement**: Mandates that deployers of AI systems generating audio mark the output in a machine-readable format. HPS-1.0 provides machine-readable JSON manifests embedded directly into RIFF container chunks and DDEX XML releases.
2. **Tamper-Evidence & Verification**: The combination of Ed25519 signing and SHA-256 essence hashing prevents metadata falsification or post-export modification.
3. **Auditability of Overrides**: The optional `overrides` schema element records manual overrides of automated detection heuristics, providing transparent audit trails for legal and rights management review.

---

## 8. Implementation & Container Embedding Guidance

### 8.1 RIFF WAV Container Embedding (`hps1` Chunk)
For WAVE audio files, the canonical manifest MUST be stored in a custom RIFF chunk:
- **Chunk ID**: `hps1` FourCC (`0x31737068` in little-endian ASCII).
- **Chunk Data**: UTF-8 encoded canonical manifest JSON string.

### 8.2 MP3 Container Embedding (ID3v2.4 `TXXX` Frame)
For MPEG-1 Audio Layer III files, the manifest MUST be stored in an ID3v2.4 frame:
- **Frame ID**: `TXXX` (User defined text information frame).
- **Description**: `HPS_MANIFEST_1.0`
- **Value**: Canonical manifest JSON string.

### 8.3 DDEX ERN 4.3 XML Metadata Block
Distributor delivery XML messages MUST embed classification details inside a dedicated XML namespace block:

```xml
<hps:HPSClassificationBlock xmlns:hps="https://trustnodelogic.com/hps/ns/1.0">
  <hps:Tier>X2</hps:Tier>
  <hps:AttestationMethod>daw_analysis</hps:AttestationMethod>
  <hps:CreatorHpsId>ed25519:7b3a9c...8f12</hps:CreatorHpsId>
  <hps:ManifestPublicKey>7b3a9c...8f12</hps:ManifestPublicKey>
  <hps:ManifestSignatureValue>9a2f1c...4b8e</hps:ManifestSignatureValue>
  <hps:RawManifestPayload>eyIkc2NoZW1hI...==</hps:RawManifestPayload>
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

## 9. Governance & Versioning Policy

HPS-1.0 is governed by the HPS Technical Working Group under Semantic Versioning 2.0.0 rules:
- **Major Releases (`X.0.0`)**: Backwards-incompatible schema changes or rule redefinitions.
- **Minor Releases (`1.X.0`)**: Additive metadata fields, expanded tool classification types, or non-breaking XML/JSON extensions.
- **Patch Releases (`1.0.X`)**: Textual errata, specification clarifications, and documentation updates.

---

## 10. Technical Glossary

- **Ed25519**: Edwards-curve Digital Signature Algorithm using Curve25519 (RFC 8032).
- **JSON-LD**: JavaScript Object Notation for Linked Data (W3C Standard).
- **PCM**: Pulse Code Modulation. Uncompressed digital audio sample representation.
- **RIFF**: Resource Interchange File Format. Container format used for WAV audio.
- **SHA-256**: Secure Hash Algorithm 256-bit cryptographic digest function.
