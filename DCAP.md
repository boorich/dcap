# Dynamic Capability Acquisition Protocol (DCAP)

**Version**: 2.6 (Latest)  
**Date**: November 2025  
**Author**: M. Maurer  

> **Note:** This file always contains the latest version of the DCAP specification.  
> For specific versions, see [archive/v2.6.md](./archive/v2.6.md), [archive/v2.5.md](./archive/v2.5.md), [archive/v2.4.md](./archive/v2.4.md), [archive/v2.3.md](./archive/v2.3.md), [archive/v2.2.md](./archive/v2.2.md), [archive/v2.1.md](./archive/v2.1.md), [archive/v2.0.md](./archive/v2.0.md)

---

**IMPORTANT:** Version 2.6 enhances autonomous tool acquisition with comprehensive authentication flows (OAuth2, bearer, API keys, x402), required HTTP headers, and session initialization details. Agents can now discover tools AND autonomously connect to them with full auth support.

---

For the complete specification, see [archive/v2.6.md](./archive/v2.6.md).

## Quick Reference: Version 2.6 Changes

### Enhanced `connector` Object with Authentication Details
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

### Key Points:
- **`connector.auth.type`**: Now includes `oauth2`, `bearer`, `api_key`, and `x402` (micropayments)
- **`auth.details`**: Comprehensive auth specs including `instructions_url`, `credential_source`
- **`connector.headers`**: Required and optional HTTP headers for proper content negotiation
- **`connector.session`**: Session initialization requirements for stateful services
- **x402 micropayments**: Pay-per-call with crypto (the future of tool monetization)
- **Full OAuth2 support**: `flow`, `auth_url`, `token_url`, `scopes`, `pkce_required`
- **Credential guidance**: Agents know WHERE and HOW to obtain required credentials

### Agent Workflow (v2.6):
```javascript
// 1. Discover tool via DCAP stream
const tool = discoverTool('get trading strategy');

// 2. Parse connector details
const { transport, endpoint, auth, headers, session } = tool.connector;

// 3. Prepare x402 payment (or other auth)
if (auth.type === 'x402') {
  const payment = await initializeX402Payment(
    auth.details.network,
    auth.details.asset,
    auth.details.price_per_call
  );
  credentials = payment;
} else if (auth.required && !hasCredential(auth.details.credential_source)) {
  throw new Error(`Setup required: ${auth.details.instructions_url}`);
}

// 4. Prepare authentication and headers
const credentials = await getCredentials(auth);
const requestHeaders = prepareHeaders(headers, credentials, auth);

// 5. Establish connection with auth
const client = await connectTo(transport, endpoint, requestHeaders);

// 6. Initialize session if required
if (session?.required) {
  await initializeSession(client, session);
}

// 7. Invoke tool (payment happens automatically via x402 header)
const result = await client.callTool(tool.tool, args);
```

---

For full specification including OAuth2 flows, session management, and comprehensive examples, see [archive/v2.6.md](./archive/v2.6.md).
