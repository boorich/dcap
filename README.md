# Dynamic Capability Acquisition Protocol (DCAP)

> **"Tool Discovery for AI agents - with the ability to call what you discover"**

DCAP is a decentralized protocol enabling autonomous agents to discover, evaluate, and acquire computational capabilities at runtime through semantic broadcasting. Version 2.5 introduces structured connector information, allowing agents to not just discover tools, but autonomously invoke them.

## 🎯 Quick Start

**Latest Specification:** [DCAP.md](./DCAP.md) - **Version 2.7** (December 2025)

**For automation/parsing:** Always fetch `DCAP.md` - it's a stable URL that always points to the latest version.

---

## 📋 Current Version: 2.7

### Key Features

✅ **ERC-8004 blockchain registration integration** - Link to on-chain trust layers  
✅ **Two-tier architecture** - Fast DCAP discovery + blockchain trust anchoring  
✅ **Comprehensive authentication support** (OAuth2, bearer, API key, x402, none)  
✅ **OAuth2 flow specification** with authorization_code, client_credentials, device_code  
✅ **Required HTTP headers** for proper content negotiation  
✅ **Session initialization** for stateful services  
✅ **Credential acquisition guidance** with `instructions_url` and `registration_url`  
✅ **Full autonomous tool invocation** with complete connection metadata  

### What's New in v2.7

**ERC-8004 Blockchain Registration:**
```json
{
  "blockchain_registrations": [
    {
      "protocol": "erc-8004",
      "namespace": "eip155",
      "chain_id": 1,
      "registry": "0x1234...",
      "agent_id": 42,
      "verification_url": "https://etherscan.io/nft/0x.../42"
    }
  ]
}
```

**Key Points:**
- **DCAP = Off-chain discovery layer** (fast, free, practical)
- **ERC-8004 = On-chain trust layer** (immutable, reputation, identity)
- **Together**: Fast discovery with blockchain trust anchoring
- **Optional**: Tools work without blockchain registration
- **Strategic**: DCAP complements ERC-8004, doesn't compete

**Full changelog:** See [archive/v2.7.md](./archive/v2.7.md)

### What's New in v2.6 (Previous)

**Enhanced `connector` object with authentication details:**
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
        "asset": "0x036CbD53842c5426634e7929541eC2318f3dCF7e",
        "currency": "USDC",
        "recipient": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb1",
        "price_per_call": 100000
      }
    },
    "headers": {
      "required": ["Accept", "Content-Type"],
      "optional": {
        "Accept": "application/json, text/event-stream",
        "Content-Type": "application/json"
      }
    },
    "protocol": {
      "type": "mcp",
      "version": "2024-11-05"
    },
    "session": {
      "required": false
    }
  }
}
```

**Key Points:**
- `auth.type`: Now includes `x402` (micropayments), `oauth2`, `bearer`, `api_key`, `none`
- **x402 micropayments**: Pay-per-call with crypto - the future of tool monetization
- `auth.details`: Includes `instructions_url`, `credential_source`, `registration_url`
- `headers`: Required and optional HTTP headers for content negotiation
- `session`: Initialization requirements for stateful services
- **The unlock**: Agents can autonomously handle x402 payments, OAuth, headers, and sessions

**Full changelog:** See [archive/v2.6.md](./archive/v2.6.md)

---

## 📚 Version History

All historical versions are archived for reference:

- **[v2.7](./archive/v2.7.md)** (December 2025) - ERC-8004 blockchain registration integration
- **[v2.6](./archive/v2.6.md)** (November 2025) - Enhanced authentication (OAuth2, headers, sessions)
- **[v2.5](./archive/v2.5.md)** (October 2025) - Dynamic tool acquisition with `connector` object
- **[v2.4](./archive/v2.4.md)** (October 2025) - Format-agnostic agent identification
- **[v2.3](./archive/v2.3.md)** (October 2025) - Payment attribution with `ctx.payer`
- **[v2.2](./archive/v2.2.md)** (October 2025) - Cost tracking and economic efficiency
- **[v2.1](./archive/v2.1.md)** (October 2025) - Chain pattern detection with `ctx.args`
- **[v2.0](./archive/v2.0.md)** (September 2025) - Initial public specification

---

## 🚀 Implementation Guide

### For Tool Providers

**Recommended:** Implement [DCAP.md](./DCAP.md) (v2.7)

```javascript
// Broadcast semantic_discover with full connector details (v2.6)
// Example: x402 micropayment tool
{
  "v": 2,
  "t": "semantic_discover",
  "sid": "your-tool-id",
  "tool": "get_strategy_code",
  "does": "Retrieves algorithmic trading strategy source code",
  "when": ["need trading strategy", "backtest algorithm"],
  "connector": {
    "transport": "http",
    "endpoint": "https://your-server.com:8080/mcp",
    "auth": {
      "type": "x402",
      "required": true,
      "details": {
        "network": "base-sepolia",
        "asset": "0x036CbD53842c5426634e7929541eC2318f3dCF7e",
        "currency": "USDC",
        "recipient": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb1",
        "price_per_call": 100000
      }
    },
    "headers": {
      "required": ["Accept", "Content-Type"],
      "optional": {
        "Accept": "application/json, text/event-stream",
        "Content-Type": "application/json"
      }
    },
    "protocol": {
      "type": "mcp",
      "version": "2024-11-05"
    },
    "session": {
      "required": false
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

**Recommended:** Implement [DCAP.md](./DCAP.md) (v2.7)

- Parse enhanced `connector` object with auth, headers, and session details
- Implement OAuth2 flows (authorization_code, client_credentials, device_code)
- Handle required HTTP headers for proper content negotiation
- Initialize sessions when `session.required: true`
- Use `instructions_url` and `registration_url` to guide credential acquisition
- Dynamically connect to discovered tools with full authentication support

### For Intelligence Systems

**Required:** Implement [DCAP.md](./DCAP.md) (v2.6)

```javascript
// Index tools by auth complexity and requirements
const tool = {
  ...semanticDiscoverMessage,
  transport: connector.transport,
  authType: connector.auth.type,
  authComplexity: calculateAuthComplexity(connector.auth),
  requiresHeaders: connector.headers?.required || [],
  requiresSession: connector.session?.required || false
};

// Include full connector info in recommendations
function recommendTool(query) {
  const match = findSemanticMatch(query);
  return {
    tool: match.tool,
    connector: match.connector,  // Full auth, headers, session info
    setupRequired: match.connector.auth.details.instructions_url,
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
| New implementations | [DCAP.md](./DCAP.md) (v2.7) |
| Blockchain trust integration | v2.7+ (optional) |
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
