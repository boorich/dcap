# Dynamic Capability Acquisition Protocol (DCAP)

**Version**: 2.5 (Latest)  
**Date**: October 2025  
**Author**: M. Maurer  

> **Note:** This file always contains the latest version of the DCAP specification.  
> For specific versions, see [archive/v2.5.md](./archive/v2.5.md), [archive/v2.4.md](./archive/v2.4.md), [archive/v2.3.md](./archive/v2.3.md), [archive/v2.2.md](./archive/v2.2.md), [archive/v2.1.md](./archive/v2.1.md), [archive/v2.0.md](./archive/v2.0.md)

---

**IMPORTANT:** Version 2.5 enables fully autonomous tool acquisition. Tools now broadcast structured `connector` information, allowing agents to discover AND invoke capabilities without manual configuration. This is the critical unlock for dynamic capability acquisition.

---

For the complete specification, see [archive/v2.5.md](./archive/v2.5.md).

## Quick Reference: Version 2.5 Changes

### New Field: `connector` Object in `semantic_discover`
```json
{
  "v": 2,
  "t": "semantic_discover",
  "tool": "get_strategy_code",
  "does": "Retrieves trading strategy source code",
  "when": ["need trading strategy", "backtest algorithm"],
  "connector": {
    "transport": "http",
    "endpoint": "https://robonet.example.com:8080/mcp",
    "auth": {
      "type": "x402",
      "required": true,
      "details": {
        "network": "base-sepolia",
        "asset": "0x036CbD53842c5426634e7929541eC2318f3dCF7e",
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

### Key Points:
- **`connector.transport`**: MCP transport type (`stdio`, `sse`, `http`)
- **`connector.endpoint`**: WHERE to connect (URL or command)
- **`connector.auth`**: Authentication requirements (`x402`, `bearer`, `api_key`, `none`)
- **`connector.protocol`**: Protocol specification (`mcp`, `rest`, `grpc`)
- **Enables autonomous acquisition**: Agents can now discover, connect, and invoke tools without pre-configuration

### Agent Workflow:
```javascript
// 1. Discover tool via DCAP stream
const tool = discoverTool('get trading strategy');

// 2. Parse connector details
const { transport, endpoint, auth } = tool.connector;

// 3. Establish connection (stdio, SSE, or HTTP streaming)
const client = await connectTo(transport, endpoint, auth);

// 4. Invoke tool
const result = await client.callTool(tool.tool, args);
```

---

For full specification including examples, security considerations, and implementation guidelines, see [archive/v2.5.md](./archive/v2.5.md).
