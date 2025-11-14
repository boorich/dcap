# Dynamic Capability Acquisition Protocol (DCAP)

**Version**: 2.7 (Latest)  
**Date**: December 2025  
**Author**: M. Maurer  

> **Note:** This file always contains the latest version of the DCAP specification.  
> For specific versions, see [archive/v2.7.md](./archive/v2.7.md), [archive/v2.6.md](./archive/v2.6.md), [archive/v2.5.md](./archive/v2.5.md), [archive/v2.4.md](./archive/v2.4.md), [archive/v2.3.md](./archive/v2.3.md), [archive/v2.2.md](./archive/v2.2.md), [archive/v2.1.md](./archive/v2.1.md), [archive/v2.0.md](./archive/v2.0.md)

---

**IMPORTANT:** Version 2.7 introduces optional blockchain registration metadata, enabling integration with ERC-8004 and other on-chain trust layers. DCAP is now positioned as the off-chain discovery layer that complements blockchain-based agent trust systems.

**Two-Layer Architecture:**
- **DCAP (Off-Chain)**: Fast discovery, connection automation, performance telemetry, zero cost
- **ERC-8004 (On-Chain)**: Immutable identity, reputation aggregation, cryptographic validation

Together: Fast discovery with blockchain trust anchoring.

---

For the complete specification, see [archive/v2.7.md](./archive/v2.7.md).

## Quick Reference: Version 2.7 Changes

### ERC-8004 Blockchain Registration Integration

Version 2.7 adds optional `blockchain_registrations` array to `semantic_discover` messages, enabling tools to advertise their on-chain trust registrations:

```json
{
  "v": 2,
  "t": "semantic_discover",
  "tool": "financial_advisor",
  "does": "Provides SEC-compliant investment advice",
  "connector": { /* ... full connection details ... */ },
  "blockchain_registrations": [
    {
      "protocol": "erc-8004",
      "namespace": "eip155",
      "chain_id": 1,
      "registry": "0x1234...",
      "agent_id": 42,
      "verification_url": "https://etherscan.io/nft/0x.../42"
    }
  ],
  "proven_by": { "uses": 5420, "success_rate": 0.98 }
}
```

### Key Points:
- **`blockchain_registrations`**: Optional array linking to ERC-8004 or other blockchain registries
- **Two-tier trust**: DCAP provides fast discovery, ERC-8004 provides immutable trust
- **Complementary layers**: DCAP solves discovery/connection, ERC-8004 solves trust/identity
- **Backward compatible**: Tools without blockchain registration work unchanged
- **Strategic positioning**: DCAP is the off-chain component that makes blockchain trust practical

### Agent Workflow (v2.7):
```javascript
// 1. Discover tool via DCAP stream (milliseconds)
const tool = discoverTool('investment advice');

// 2. Check for blockchain registration
if (tool.blockchain_registrations && tool.blockchain_registrations.length > 0) {
  // 3. Verify on-chain (for high-value scenarios)
  const verification = await verifyERC8004(tool.blockchain_registrations[0]);
  if (!verification.verified) {
    throw new Error('Blockchain verification failed');
  }
}

// 4. Use DCAP connector to connect (fast, automated)
const { transport, endpoint, auth, headers } = tool.connector;
const client = await connectTo(transport, endpoint, auth, headers);

// 5. Invoke tool
const result = await client.callTool(tool.tool, args);

// 6. Post feedback to both DCAP (real-time) and blockchain (permanent)
broadcastPerfUpdate(tool, result);
if (tool.blockchain_registrations) {
  await submitBlockchainFeedback(tool.blockchain_registrations[0], result);
}
```

---

## Quick Reference: Version 2.6 Changes (Previous)

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

For full specification including ERC-8004 integration, OAuth2 flows, session management, and comprehensive examples, see [archive/v2.7.md](./archive/v2.7.md).
