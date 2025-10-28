# Dynamic Capability Acquisition Protocol (DCAP)

> **"Tool Discovery for AI agents - with the ability to call what you discover"**

DCAP is a decentralized protocol enabling autonomous agents to discover, evaluate, and acquire computational capabilities at runtime through semantic broadcasting. Version 2.5 introduces structured connector information, allowing agents to not just discover tools, but autonomously invoke them.

## 🎯 Quick Start

**Latest Specification:** [DCAP.md](./DCAP.md) - **Version 2.5** (October 2025)

**For automation/parsing:** Always fetch `DCAP.md` - it's a stable URL that always points to the latest version.

---

## 📋 Current Version: 2.5

### Key Features

✅ **Dynamic tool acquisition** with structured `connector` object  
✅ **MCP transport support** (stdio, SSE, HTTP streaming)  
✅ **Structured authentication** (x402, bearer, API key, none)  
✅ **Protocol specification** (MCP, REST, gRPC)  
✅ **Autonomous capability invocation** without manual configuration  

### What's New in v2.5

**Added `connector` object to `semantic_discover`:**
```json
{
  "connector": {
    "transport": "http",
    "endpoint": "https://robonet.example.com:8080/mcp",
    "auth": {
      "type": "x402",
      "required": true,
      "details": {
        "network": "base-sepolia",
        "asset": "0x036CbD...",
        "currency": "USDC"
      }
    },
    "protocol": {
      "type": "mcp",
      "version": "2024-11-05"
    }
  }
}
```

**Key Points:**
- `connector.transport`: MCP transport type (stdio/sse/http)
- `connector.endpoint`: WHERE to connect (URL or command)
- `connector.auth`: Authentication requirements with structured details
- `connector.protocol`: Protocol type, version, and methods
- **The unlock**: Agents can now discover AND invoke tools without pre-configuration

**Full changelog:** See [archive/v2.5.md](./archive/v2.5.md)

---

## 📚 Version History

All historical versions are archived for reference:

- **[v2.5](./archive/v2.5.md)** (October 2025) - Dynamic tool acquisition with `connector` object
- **[v2.4](./archive/v2.4.md)** (October 2025) - Format-agnostic agent identification
- **[v2.3](./archive/v2.3.md)** (October 2025) - Payment attribution with `ctx.payer`
- **[v2.2](./archive/v2.2.md)** (October 2025) - Cost tracking and economic efficiency
- **[v2.1](./archive/v2.1.md)** (October 2025) - Chain pattern detection with `ctx.args`
- **[v2.0](./archive/v2.0.md)** (September 2025) - Initial public specification

---

## 🚀 Implementation Guide

### For Tool Providers

**Recommended:** Implement [DCAP.md](./DCAP.md) (v2.5)

```javascript
// Broadcast semantic_discover with connector details
{
  "v": 2,
  "t": "semantic_discover",
  "sid": "your-tool-id",
  "tool": "read_file",
  "does": "Reads file contents from filesystem",
  "when": ["need file contents", "read configuration"],
  "connector": {
    "transport": "http",
    "endpoint": "https://your-server.com:8080/mcp",
    "auth": {
      "type": "x402",
      "required": true,
      "details": {
        "network": "base-sepolia",
        "asset": "0x036CbD...",
        "currency": "USDC"
      }
    },
    "protocol": {
      "type": "mcp",
      "version": "2024-11-05"
    }
  }
}

// Continue sending perf_update as usual
{
  "v": 2,
  "t": "perf_update",
  "sid": "your-tool-id",
  "tool": "read_file",
  "exec_ms": 45,
  "success": true
}
```

### For Agent Consumers

**Recommended:** Implement [DCAP.md](./DCAP.md) (v2.5)

- Parse `connector` object from `semantic_discover` messages
- Implement MCP transport support (stdio, SSE, HTTP streaming)
- Handle authentication requirements (x402, bearer, API key)
- Dynamically connect to discovered tools without pre-configuration

### For Intelligence Systems

**Required:** Implement [DCAP.md](./DCAP.md) (v2.5)

```javascript
// Index tools by transport and auth requirements
const tool = {
  ...semanticDiscoverMessage,
  transport: connector.transport,
  authType: connector.auth.type
};

// Include connector info in recommendations
function recommendTool(query) {
  const match = findSemanticMatch(query);
  return {
    tool: match.tool,
    connector: match.connector,  // Agent can now invoke directly
    reasoning: "..."
  };
}
```

---

## 🎓 Philosophy

DCAP is **Google for AI agents** - but with the ability to call any discovered capability:

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
| New implementations | [DCAP.md](./DCAP.md) (v2.5) |
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
