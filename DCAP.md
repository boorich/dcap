# Dynamic Capability Acquisition Protocol (DCAP)

**Version**: 3.0 (Latest)  
**Date**: December 2025  
**Author**: M. Maurer  

> **Note:** This file always contains the latest version of the DCAP specification.  
> For specific versions, see the version history below.

---

**IMPORTANT:** Version 3.0 introduces agent verification through the `usage_receipt` message type. This enables third-party observation of tool performance alongside tool self-reports, providing the foundation for reputation systems based on agent consensus rather than tool claims.

**Key Architectural Distinction:**
- **Tools** (MCP servers): Identified by `sid`, broadcast capabilities and self-reports
- **Agents** (consumers): Identified by `agent_id`, broadcast observed tool behavior

Together: Tools claim performance, agents verify reality.

---

For the complete specification, see [archive/DCAP_v3.md](./archive/DCAP_v3.md).

## Quick Reference: Version 3.0 Changes

### Agent Verification via Usage Receipt

Version 3.0 introduces a new message type for agents to report their observations:

```json
{
  "v": 2,
  "t": "usage_receipt",
  "ts": 1735000000,
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

### Key Points:
- **`usage_receipt`**: NEW message type for agent observations
- **`agent_id`**: Agents identify themselves separately from tools
- **Third-party verification**: Agents report what they actually observed, not what tools claimed
- **Blockchain identity**: Agents can optionally anchor their identity on-chain (ERC-8004, etc.)
- **Protocol purity**: Spec defines message formats; intelligence systems decide how to use them
- **No breaking changes for tools**: Tools continue working unchanged

### Agent Workflow (v3.0):
```javascript
// 1. Discover tool via DCAP stream
const tool = discoverTool('financial advice');

// 2. Invoke and measure
const start = Date.now();
const result = await invokeTool(tool, args);
const execMs = Date.now() - start;

// 3. Broadcast what you actually observed
broadcastUsageReceipt({
  agent_id: 'agent-alice',
  tool: tool.tool,
  tool_sid: tool.sid,
  success: true,
  exec_ms: execMs,
  cost_paid: result.cost,
  blockchain_registrations: myBlockchainId  // optional
});
```

### What Changed from v2.7:
- **Corrected:** Moved `blockchain_registrations` from tool messages to agent messages
- **Added:** `usage_receipt` message type for agent observations
- **Clarified:** Tools report via `perf_update` (self-report), agents report via `usage_receipt` (observation)
- **Architectural fix:** MCP servers aren't agents; blockchain registration is for agent identity

---

## Version History

| Version | Date | Key Feature |
|---------|------|-------------|
| [3.0](./archive/DCAP_v3.md) | December 2025 | Agent verification (`usage_receipt`), corrected blockchain architecture |
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

### 1. Semantic Discovery (`t: "semantic_discover"`)
**Broadcast by:** Tools

Tools advertise their capabilities with connection details:
```json
{
  "v": 2,
  "t": "semantic_discover",
  "sid": "finadv-mcp",
  "tool": "financial_advisor",
  "does": "Provides SEC-compliant investment advice",
  "when": ["investment advice", "portfolio analysis"],
  "connector": {
    "transport": "http",
    "endpoint": "https://finadvice.ai/mcp",
    "auth": {
      "type": "oauth2",
      "required": true,
      "details": { /* OAuth2 configuration */ }
    }
  },
  "proven_by": {"uses": 5420, "success_rate": 0.98}
}
```

### 2. Performance Update (`t: "perf_update"`)
**Broadcast by:** Tools

Tools report their own execution (self-report):
```json
{
  "v": 2,
  "t": "perf_update",
  "sid": "finadv-mcp",
  "tool": "financial_advisor",
  "exec_ms": 245,
  "success": true,
  "cost_paid": 100000,
  "currency": "USDC"
}
```

### 3. Usage Receipt (`t: "usage_receipt"`) - NEW in v3.0
**Broadcast by:** Agents

Agents report what they observed (third-party verification):
```json
{
  "v": 2,
  "t": "usage_receipt",
  "agent_id": "agent-alice",
  "tool": "financial_advisor",
  "tool_sid": "finadv-mcp",
  "success": true,
  "exec_ms": 250,
  "cost_paid": 100000,
  "currency": "USDC",
  "blockchain_registrations": [/* optional identity */]
}
```

---

## Implementation Paths

### For Tool Providers:
**No changes required from v2.6/v2.7**
- Continue broadcasting `semantic_discover` with connector details
- Continue broadcasting `perf_update` with execution data
- Your tools work unchanged in v3.0

### For Agent Developers:
**New capability in v3.0**
- Broadcast `usage_receipt` after tool invocations
- Report what you actually observed (exec_ms, success, cost_paid)
- Optionally include blockchain identity for reputation weighting
- Help build trustworthy reputation systems

### For Hub Operators:
**New message type to handle**
- Distribute `usage_receipt` messages like other message types
- Optionally compare tool claims vs agent observations
- Optionally calculate reputation from agent consensus
- Protocol doesn't prescribe HOW - use data as needed

---

For the complete v3.0 specification including all message formats, transport protocols, security considerations, and implementation examples, see [archive/DCAP_v3.md](./archive/DCAP_v3.md).
