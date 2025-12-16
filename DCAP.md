# Dynamic Capability Acquisition Protocol (DCAP)

**Version**: 3.1 (Latest)  
**Date**: December 2025  
**Author**: M. Maurer  

> **Note:** This file always contains the latest version of the DCAP specification.  
> For specific versions, see the version history below.

---

**IMPORTANT:** Version 3.1 introduces **categorical compliance** based on the formal C_cap category theory foundation. Capabilities now form a mathematical category where tools are morphisms with typed signatures, enabling verified composition with provable correctness guarantees.

**Key Additions in v3.1:**
- **Typed signatures**: Tools declare explicit `input → output` types
- **Verified composition**: Agents can declare and execute type-checked tool chains
- **Cost enrichment**: Costs are provably additive under composition
- **Category laws**: Identity and associativity laws formally verified

**Reference Implementation:** [github.com/dcap-protocol/ccap-haskell](https://github.com/dcap-protocol/ccap-haskell)

---

For the complete specification, see [archive/DCAP_v3.1.md](./archive/v3.1.md).

## Quick Reference: Version 3.1 Changes

### Categorical Foundation (C_cap)

DCAP capabilities now form a category where:

```
Objects:     Capability types (Text, Image, JSON, URL, ...)
Morphisms:   Tools with typed signatures A → B
Composition: Tool chaining (g ∘ f)
Identity:    No-op transforms id_A for each type A
Enrichment:  Costs over (ℕ, +, 0)
```

**Category Laws (formally verified):**
```
Left Identity:   id_B ∘ f = f
Right Identity:  f ∘ id_A = f
Associativity:   h ∘ (g ∘ f) = (h ∘ g) ∘ f
Cost Identity:   cost(id_A) = 0
Cost Additivity: cost(g ∘ f) = cost(f) + cost(g)
```

### Typed Signatures

Tools now declare explicit type signatures:

```json
{
  "v": 2,
  "t": "semantic_discover",
  "sid": "summarizer-mcp",
  "tool": "summarize_text",
  "signature": {
    "input": "Text",
    "output": "Maybe<Text>",
    "cost": 5
  },
  "does": "Summarizes long text into key points",
  "when": ["summarization", "tldr"],
  "connector": { /* ... */ }
}
```

### Composite Capabilities (NEW)

Agents declare verified tool chains:

```json
{
  "v": 2,
  "t": "composite_capability",
  "agent_id": "agent-alice",
  "composite_id": "alice-url-to-summary",
  "chain": [
    {"tool_sid": "fetcher-mcp", "tool": "fetch_url", "signature": {"input": "URL", "output": "Maybe<HTML>", "cost": 2}},
    {"tool_sid": "extractor-mcp", "tool": "html_to_text", "signature": {"input": "HTML", "output": "Maybe<Text>", "cost": 1}},
    {"tool_sid": "summary-mcp", "tool": "summarize", "signature": {"input": "Text", "output": "Maybe<Text>", "cost": 5}}
  ],
  "signature": {"input": "URL", "output": "Maybe<Text>", "cost": 8}
}
```

**Validation Rules (enforced by agents and hubs):**
1. Type continuity: `chain[i].output == chain[i+1].input`
2. Endpoint agreement: `signature.input == chain[0].input`, `signature.output == chain[n-1].output`
3. Cost additivity: `signature.cost == sum(chain[i].cost)`

### Composite Receipt (NEW)

Agents report composition execution with per-step metrics:

```json
{
  "v": 2,
  "t": "composite_receipt",
  "agent_id": "agent-alice",
  "composite_id": "alice-url-to-summary",
  "success": true,
  "exec_ms": 523,
  "cost_paid": 8,
  "steps": [
    {"tool_sid": "fetcher-mcp", "tool": "fetch_url", "success": true, "exec_ms": 203, "cost_paid": 2},
    {"tool_sid": "extractor-mcp", "tool": "html_to_text", "success": true, "exec_ms": 94, "cost_paid": 1},
    {"tool_sid": "summary-mcp", "tool": "summarize", "success": true, "exec_ms": 226, "cost_paid": 5}
  ]
}
```

### Core Types

| Type | Description |
|------|-------------|
| `Text` | UTF-8 encoded text |
| `JSON` | Valid JSON document |
| `Image` | Binary image data |
| `Audio` | Binary audio data |
| `Video` | Binary video data |
| `Binary` | Arbitrary binary data |
| `URL` | Valid URL string |
| `HTML` | HTML document |
| `Markdown` | Markdown formatted text |
| `PDF` | PDF document |
| `Bool` | Boolean value |
| `Number` | Numeric value |
| `List<T>` | Ordered list of type T |
| `Maybe<T>` | Optional value (for fallible tools) |
| `Void` | Unit type (effects-only tools) |

### Compliance Levels

- **C_cap-compliant**: Has `signature` with registered types; can participate in verified composition
- **DCAP-basic**: No `signature`; semantic discovery only; cannot participate in typed composition

Both are valid DCAP participants. Full backward compatibility with v3.0.

---

## Version History

| Version | Date | Key Feature |
|---------|------|-------------|
| [3.1](./archive/v3.1.md) | December 2025 | Categorical compliance (C_cap), typed signatures, verified composition |
| [3.0](./archive/v3.0.md) | December 2025 | Agent verification (`usage_receipt`), corrected blockchain architecture |
| [2.7](./archive/v2.7.md) | November 2025 | Blockchain registration (architectural error, corrected in v3.0) |
| [2.6](./archive/v2.6.md) | November 2025 | Enhanced authentication (OAuth2, x402, API keys) |
| [2.5](./archive/v2.5.md) | October 2025 | Connector object for connection automation |
| [2.4](./archive/v2.4.md) | October 2025 | Enhanced security considerations |
| [2.3](./archive/v2.3.md) | October 2025 | Pattern detection capabilities |
| [2.2](./archive/v2.2.md) | October 2025 | Cost tracking (`cost_paid`, `currency`) |
| [2.1](./archive/v2.1.md) | October 2025 | Context tracking (`ctx` field) |
| [2.0](./archive/v2.0.md) | September 2025 | Initial public specification |

---

## Quick Reference: Message Types

| Message Type | Broadcaster | Purpose |
|--------------|-------------|---------|
| `semantic_discover` | Tools | Advertise capabilities with typed signatures |
| `perf_update` | Tools | Self-report execution metrics |
| `usage_receipt` | Agents | Report observed tool behavior |
| `composite_capability` | Agents | Declare verified tool composition |
| `composite_receipt` | Agents | Report composition execution |
| `error_pattern` | Tools | Report recurring error patterns |

---

## Implementation Paths

### For Tool Providers:
**Recommended: Add `signature` for C_cap compliance**
```json
"signature": {
  "input": "<input_type>",
  "output": "<output_type>",
  "cost": <cost_value>
}
```
- Use `Maybe<T>` for tools that can fail
- Report `cost_paid` matching declared `signature.cost`
- Existing v3.0 tools work unchanged (DCAP-basic mode)

### For Agent Developers:
**New capabilities in v3.1**
- Build type graphs from discovered `signature` fields
- Plan compositions via graph search (Dijkstra for cost optimization)
- Validate type continuity and cost additivity before execution
- Broadcast `composite_capability` before executing chains
- Broadcast `composite_receipt` with per-step metrics

### For Hub Operators:
**New validation responsibilities**
- Validate `composite_capability` for categorical correctness
- Reject compositions violating type continuity or cost additivity
- Optionally index tools by `signature.input`/`signature.output` for composition queries

---

## Roadmap

| Version | Feature |
|---------|---------|
| **v3.1** | C_cap — pure morphisms, category laws verified |
| **v3.2** | C_cap_K — Kleisli extension for effectful morphisms (`Maybe`, `List`, `IO`) |
| Future | Monoidal categories for parallel composition |
| Future | On-chain composition registry |

---

## References

- [C_cap Haskell Implementation](https://github.com/boorich/ccap-haskell) — Formal verification of category laws
- [Complete v3.1 Specification](./archive/v3.1.md) — Full protocol spec (1900 lines)
- [MCP Specification](https://modelcontextprotocol.io) — Model Context Protocol

---

## Authors

Martin Maurer  
Email: empeamtk@googlemail.com