# Dynamic Capability Acquisition Protocol (DCAP)

> **"Tool Discovery for AI agents - with the ability to call what you discover"**

DCAP is a decentralized protocol enabling autonomous agents to discover, evaluate, and acquire computational capabilities at runtime through semantic broadcasting. Version 3.1 introduces **categorical compliance** — capabilities now form a mathematical category with typed signatures, enabling verified composition with provable correctness guarantees.

## 🎯 Quick Start

**Latest Specification:** [DCAP.md](./DCAP.md) - **Version 3.1** (December 2025)

**For automation/parsing:** Always fetch `DCAP.md` - it's a stable URL that always points to the latest version.

**Formal Verification:** [github.com/dcap-protocol/ccap-haskell](https://github.com/boorich/ccap-haskell) - Category laws verified in Haskell

---

## 📋 Current Version: 3.1

### Key Features

✅ **Categorical foundation (C_cap)** - Formal category theory basis with verified laws  
✅ **Typed signatures** - Tools declare explicit `input → output` types  
✅ **Verified composition** - Type-checked tool chains with provable correctness  
✅ **Cost enrichment** - Costs are provably additive under composition  
✅ **Agent verification** via `usage_receipt` - Third-party tool observation  
✅ **Composite capabilities** - Declare and execute verified pipelines  
✅ **Tools vs Agents distinction** - Separate identifiers (`sid` vs `agent_id`)  
✅ **Blockchain identity anchoring** - Agents can link to ERC-8004 or other registries  
✅ **Protocol purity** - Spec defines messages, systems decide how to use them  
✅ **Comprehensive authentication support** (OAuth2, bearer, API key, x402, none)  
✅ **Full autonomous tool invocation** with complete connection metadata  

### What's New in v3.1

**Typed Signatures:**
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
  "connector": { /* how to connect */ }
}
```

**Composite Capabilities:**
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

**Category Laws (formally verified in Haskell):**
```
Left Identity:   id_B ∘ f = f
Right Identity:  f ∘ id_A = f  
Associativity:   h ∘ (g ∘ f) = (h ∘ g) ∘ f
Cost Identity:   cost(id_A) = 0
Cost Additivity: cost(g ∘ f) = cost(f) + cost(g)
```

**Key Points:**
- **Typed morphisms**: Tools declare `signature` with input/output types and cost
- **Verified composition**: Agents validate type continuity and cost additivity
- **Effect types**: Use `Maybe<T>` for fallible tools, `List<T>` for non-deterministic
- **Two compliance levels**: C_cap-compliant (typed) or DCAP-basic (semantic only)
- **Full backward compatibility**: v3.0 tools work unchanged

**Full changelog:** See [archive/v3.1.md](./archive/v3.1.md)

---

## 📚 Version History

All historical versions are archived for reference:

- **[v3.1](./archive/v3.1.md)** (December 2025) - Categorical compliance (C_cap), typed signatures, verified composition
- **[v3.0](./archive/v3.0.md)** (December 2025) - Agent verification, corrected blockchain architecture
- **[v2.7](./archive/v2.7.md)** (November 2025) - Blockchain registration (architectural error, corrected in v3.0)
- **[v2.6](./archive/v2.6.md)** (November 2025) - Enhanced authentication (OAuth2, headers, sessions)
- **[v2.5](./archive/v2.5.md)** (October 2025) - Dynamic tool acquisition with `connector` object
- **[v2.4](./archive/v2.4.md)** (October 2025) - Format-agnostic agent identification
- **[v2.3](./archive/v2.3.md)** (October 2025) - Payment attribution with `ctx.payer`
- **[v2.2](./archive/v2.2.md)** (October 2025) - Cost tracking and economic efficiency
- **[v2.1](./archive/v2.1.md)** (October 2025) - Chain pattern detection with `ctx.args`
- **[v2.0](./archive/v2.0.md)** (September 2025) - Initial public specification

---

## 🚀 Implementation Guide

### Quick Summary

**Tools (MCP servers):**
- Broadcast `semantic_discover` (capabilities + connection info + **typed signature**)
- Broadcast `perf_update` (execution results)

**Agents (consumers):**
- Listen to stream, discover tools
- **NEW in v3.1:** Build type graph from `signature` fields
- **NEW in v3.1:** Plan compositions via graph search
- **NEW in v3.1:** Broadcast `composite_capability` (verified chains)
- **NEW in v3.1:** Broadcast `composite_receipt` (chain execution results)
- Broadcast `usage_receipt` (what you observed)

**Intelligence systems (hubs):**
- **NEW in v3.1:** Validate `composite_capability` for categorical correctness
- Aggregate `perf_update` (tool claims) and `usage_receipt` (agent observations)
- Compare claims vs reality
- Build reputation however you want

### For Tool Providers

Broadcast your capabilities with typed signature:
```json
{
  "v": 2,
  "t": "semantic_discover",
  "sid": "your-tool-id",
  "tool": "your_tool",
  "signature": {
    "input": "Text",
    "output": "Maybe<JSON>",
    "cost": 3
  },
  "does": "What your tool does",
  "when": ["trigger phrases"],
  "connector": { /* how to connect */ }
}
```

**Signature Guidelines:**
- Use `Maybe<T>` if your tool can fail (most network/API tools)
- Use `List<T>` if your tool returns multiple results
- Set `cost` to actual charge — agents will compare against `cost_paid`
- Tools without `signature` work but can't participate in verified composition

Report your execution:
```json
{
  "v": 2,
  "t": "perf_update",
  "sid": "your-tool-id",
  "tool": "your_tool",
  "exec_ms": 45,
  "success": true,
  "cost_paid": 3
}
```

**See [DCAP.md](./DCAP.md) for complete connector specification (OAuth2, x402, etc.)**

### For Agent Consumers

Discover tools, plan compositions, execute and report:

**1. Build type graph from discovered tools:**
```javascript
// Tools with signatures become edges in a type graph
// Nodes: types (Text, JSON, Image, ...)
// Edges: tools with cost as weight
```

**2. Plan composition (Dijkstra for cost optimization):**
```javascript
const path = findPath('URL', 'Text'); // Returns cheapest tool chain
```

**3. Validate and declare composition:**
```json
{
  "v": 2,
  "t": "composite_capability",
  "agent_id": "your-agent-id",
  "composite_id": "your-composition-id",
  "chain": [ /* tools with signatures */ ],
  "signature": { "input": "URL", "output": "Text", "cost": 8 }
}
```

**4. Execute and report:**
```json
{
  "v": 2,
  "t": "composite_receipt",
  "agent_id": "your-agent-id",
  "composite_id": "your-composition-id",
  "success": true,
  "exec_ms": 523,
  "cost_paid": 8,
  "steps": [ /* per-step metrics */ ]
}
```

**See [DCAP.md](./DCAP.md) for complete specification and examples**

### For Intelligence Systems

Listen to all message types:
- `semantic_discover`: Tool capabilities with typed signatures
- `perf_update`: What tools claim
- `usage_receipt`: What agents observed (single tools)
- `composite_capability`: Declared compositions
- `composite_receipt`: Composition execution results

**New in v3.1 — Validation responsibilities:**
- Validate `composite_capability` for type continuity
- Validate cost additivity (`signature.cost == sum(chain[i].cost)`)
- Reject invalid compositions
- Optionally index by `signature.input`/`signature.output` for composition queries

**The protocol doesn't prescribe how to use this data.** You decide:
- How to calculate reputation
- How to weight agent observations
- How to handle discrepancies
- How to verify blockchain identities

**See [archive/v3.1.md](./archive/v3.1.md) for detailed implementation examples**

---

## 🔬 Formal Foundation

DCAP v3.1 is grounded in category theory. The **C_cap** category provides:

- **Provable composition**: If types align, composition is valid
- **Predictable costs**: `cost(g ∘ f) = cost(f) + cost(g)` — always
- **Identity guarantees**: No-op transforms exist and behave correctly
- **Associativity**: Restructure pipelines without changing semantics

**Reference implementation:** [ccap-haskell](https://github.com/dcap-protocol/ccap-haskell)

```haskell
-- Category laws verified via QuickCheck
prop_leftIdentity  f = checkLeftIdentity f   -- id_B ∘ f = f
prop_rightIdentity f = checkRightIdentity f  -- f ∘ id_A = f  
prop_associativity f g h = checkAssociativity f g h  -- h ∘ (g ∘ f) = (h ∘ g) ∘ f
prop_costIdentity t = cost (capId t) == 0
prop_costComposition f g = cost (g ∘ f) == cost f + cost g
```

---

## 🗺️ Roadmap

| Version | Feature | Status |
|---------|---------|--------|
| **v3.1** | C_cap — pure morphisms, category laws | ✅ Released |
| **v3.2** | C_cap_K — Kleisli composition for `Maybe`, `List`, `IO` | Planned |
| Future | Monoidal categories — parallel composition | Exploring |
| Future | On-chain composition registry | Exploring |

---

## 🎓 Philosophy

DCAP is **DNS for AI agents** - but with the ability to call any discovered capability:

- **Discovery + Invocation**: Not just "what exists" but "how to reach it"
- **Autonomous acquisition**: Agents extend their capabilities without human configuration
- **Verified composition**: Mathematical guarantees, not just convention
- **Open participation**: Broadcast your tools, accept any transport, embrace any auth
- **Network effects**: More tools = more intelligence = more value
- **Evolution over perfection**: Watch what emerges, adapt the protocol

**"Discovery is powerful. Discovery + autonomous invocation is transformative. Discovery + verified composition is infrastructure."**

---

## 📦 Version Selection

| Use Case | Version |
|----------|---------|
| New implementations | [DCAP.md](./DCAP.md) (v3.1) |
| Verified composition | v3.1+ (required) |
| Typed signatures | v3.1+ (recommended) |
| Agent verification | v3.0+ (optional) |
| Third-party observation | v3.0+ (recommended) |
| Blockchain agent identity | v3.0+ (optional) |
| OAuth2 / Session support | v2.6+ (required) |
| Dynamic tool acquisition | v2.5+ (required) |
| Agent identification | v2.4+ |
| Payment tracking | v2.3+ |
| Cost optimization | v2.2+ |
| Chain detection | v2.1+ |
| Basic discovery | v2.0 (works but limited) |

**Migration:** All versions are backward compatible. Upgrade anytime.

---

## 🤝 Contributing

To propose changes:

1. Draft your changes to `DCAP.md`
2. Create a new version in `archive/vX.Y.md`
3. Update this README
4. Submit PR with rationale

**Philosophy:** Propose boldly. Validate through use. Verify formally.

---

## 📜 License

This specification is provided as-is for implementation by any party.

## 📧 Contact

**Martin Maurer**  
Email: empeamtk@googlemail.com  
GitHub: [@boorich](https://github.com/boorich)

---

**Built for the agent economy - discover, connect, compose, execute. No gatekeepers. Provably correct.**