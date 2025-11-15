# Dynamic Capability Acquisition Protocol (DCAP)

> **"Tool Discovery for AI agents - with the ability to call what you discover"**

DCAP is a decentralized protocol enabling autonomous agents to discover, evaluate, and acquire computational capabilities at runtime through semantic broadcasting. Version 3.0 introduces agent verification, enabling third-party observation of tool performance alongside tool self-reports.

## 🎯 Quick Start

**Latest Specification:** [DCAP.md](./DCAP.md) - **Version 3.0** (December 2025)

**For automation/parsing:** Always fetch `DCAP.md` - it's a stable URL that always points to the latest version.

---

## 📋 Current Version: 3.0

### Key Features

✅ **Agent verification** via `usage_receipt` message type - Third-party tool observation  
✅ **Tools vs Agents distinction** - Separate identifiers (`sid` vs `agent_id`)  
✅ **Blockchain identity anchoring** - Agents can link to ERC-8004 or other registries  
✅ **Protocol purity** - Spec defines messages, systems decide how to use them  
✅ **Comprehensive authentication support** (OAuth2, bearer, API key, x402, none)  
✅ **OAuth2 flow specification** with authorization_code, client_credentials, device_code  
✅ **Required HTTP headers** for proper content negotiation  
✅ **Session initialization** for stateful services  
✅ **Full autonomous tool invocation** with complete connection metadata  

### What's New in v3.0

**Agent Verification:**
```json
{
  "v": 2,
  "t": "usage_receipt",
  "agent_id": "agent-alice",
  "tool": "financial_advisor",
  "tool_sid": "finadv-mcp",
  "success": false,
  "exec_ms": 5243,
  "cost_paid": 100000,
  "currency": "USDC",
  "error_observed": "timeout after 5s",
  "blockchain_registrations": [
    {
      "agentId": 789,
      "agentRegistry": "eip155:1:0xabcd...",
      "verification_url": "https://etherscan.io/nft/0xabcd.../789"
    }
  ]
}
```

**Key Points:**
- **Tools report via `perf_update`** (self-report: what they claim)
- **Agents report via `usage_receipt`** (observation: what they saw)
- **Third-party verification**: Intelligence systems can compare claims vs reality
- **Blockchain identity**: Agents (not tools) can anchor identity on-chain
- **Architectural fix**: Corrects v2.7's placement of blockchain registrations
- **No breaking changes for tools**: Existing tools work unchanged

**Full changelog:** See [archive/DCAP_v3.md](./archive/DCAP_v3.md)

---

## 📚 Version History

All historical versions are archived for reference:

- **[v3.0](./archive/DCAP_v3.md)** (December 2025) - Agent verification, corrected blockchain architecture
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
- Broadcast `semantic_discover` (your capabilities + connection info)
- Broadcast `perf_update` (your execution results)

**Agents (consumers):**
- Listen to stream, discover tools
- Connect using `connector` details (OAuth2, x402, etc.)
- **NEW in v3.0:** Broadcast `usage_receipt` (what you observed)

**Intelligence systems (hubs):**
- Aggregate `perf_update` (tool claims) and `usage_receipt` (agent observations)
- Compare claims vs reality
- Build reputation however you want

### For Tool Providers

Broadcast your capabilities:
```json
{
  "v": 2,
  "t": "semantic_discover",
  "sid": "your-tool-id",
  "tool": "your_tool",
  "does": "What your tool does",
  "when": ["trigger phrases"],
  "connector": { /* how to connect */ }
}
```

Report your execution:
```json
{
  "v": 2,
  "t": "perf_update",
  "sid": "your-tool-id",
  "tool": "your_tool",
  "exec_ms": 45,
  "success": true
}
```

**See [DCAP.md](./DCAP.md) for complete connector specification (OAuth2, x402, etc.)**

### For Agent Consumers

Discover and invoke tools, then broadcast what you observed:
```json
{
  "v": 2,
  "t": "usage_receipt",
  "agent_id": "your-agent-id",
  "tool": "tool_you_used",
  "tool_sid": "tool-server-id",
  "success": true,
  "exec_ms": 52,
  "blockchain_registrations": [/* optional ERC-8004 identity */]
}
```

**See [DCAP.md](./DCAP.md) for complete specification and examples**

### For Intelligence Systems

Listen to both message types:
- `perf_update`: What tools claim
- `usage_receipt`: What agents observed

**The protocol doesn't prescribe how to use this data.** You decide:
- How to calculate reputation
- How to weight agent observations
- How to handle discrepancies
- How to verify blockchain identities

**See [archive/DCAP_v3.md](./archive/DCAP_v3.md) for detailed implementation examples**

---

## 🎓 Philosophy

DCAP is **DNS for AI agents** - but with the ability to call any discovered capability:

- **Discovery + Invocation**: Not just "what exists" but "how to reach it"
- **Autonomous acquisition**: Agents extend their capabilities without human configuration
- **Open participation**: Broadcast your tools, accept any transport, embrace any auth
- **Network effects**: More tools = more intelligence = more value
- **Evolution over perfection**: Watch what emerges, adapt the protocol

**"Discovery is powerful. Discovery + autonomous invocation is transformative."**

---

## 📦 Version Selection

| Use Case | Version |
|----------|---------|
| New implementations | [DCAP.md](./DCAP.md) (v3.0) |
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

**Philosophy:** Propose boldly. Validate through use.

---

## 📜 License

This specification is provided as-is for implementation by any party.

## 📧 Contact

**Martin Maurer**  
Email: empeamtk@googlemail.com  
GitHub: [@boorich](https://github.com/boorich)

---

**Built for the agent economy - discover, connect, execute. No gatekeepers.**
