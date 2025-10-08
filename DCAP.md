# Dynamic Capability Acquisition Protocol (DCAP)

**Version**: 2.2 (Latest)  
**Date**: October 2025  
**Author**: M. Maurer  

> **Note:** This file always contains the latest version of the DCAP specification.  
> For specific versions, see [DCAP-v2.2.md](./DCAP-v2.2.md), [DCAP-v2.1.md](./DCAP-v2.1.md), [DCAP-v2.0.md](./DCAP-v2.0.md)

---

## Abstract

This document defines the Dynamic Capability Acquisition Protocol (DCAP), a decentralized protocol enabling autonomous agents to discover, evaluate, and acquire computational capabilities at runtime through semantic broadcasting. DCAP uses UDP multicast for capability advertisement and WebSocket streams for real-time intelligence distribution.

## 1. Introduction

### 1.1 Problem Statement
Current autonomous agent systems operate with fixed, pre-configured tool sets, limiting their ability to handle novel or specialized tasks. Existing tool discovery mechanisms rely on centralized registries, creating bottlenecks and single points of failure.

### 1.2 Solution Overview
DCAP enables tools to self-advertise their capabilities using semantic descriptions, allowing agents to discover and dynamically load appropriate tools based on task requirements rather than static configuration. Version 2.2 extends performance reporting to include cost tracking, enabling agents to optimize for both speed and economic efficiency.

## 2. Protocol Architecture

### 2.1 Network Topology
```
Tool Provider → UDP Broadcast → Intelligence Hub → WebSocket → Agent Consumer
     ^                              ^                   ^           ^
   MCP Tool                   Stream Aggregator    Real-time     Dynamic Agent
```

### 2.2 Transport Layers
- **Advertisement Layer**: UDP packets (port 10191)
- **Distribution Layer**: WebSocket streams 
- **Acquisition Layer**: MCP protocol over stdio/HTTP

## 3. Message Format Specification

### 3.1 Base Message Structure
All DCAP messages MUST conform to this JSON schema:

```json
{
  "v": <protocol_version>,
  "t": <message_type>, 
  "ts": <unix_timestamp>,
  "sid": <server_identifier>,
  [message_specific_fields]
}
```

#### Required Fields:
- `v` (number): Protocol version (current: 2)
- `t` (string): Message type identifier
- `ts` (number): Unix timestamp of message creation
- `sid` (string): Unique server identifier (8-12 characters)

### 3.2 Message Types

#### 3.2.1 Semantic Discovery Message (`t: "semantic_discover"`)
Advertises tool capabilities using natural language descriptions.

```json
{
  "v": 2,
  "t": "semantic_discover", 
  "ts": 1727100286,
  "sid": "abc123def456",
  "tool": <tool_name>,
  "does": <capability_description>,
  "when": [<trigger_contexts>],
  "good_at": [<specific_strengths>],
  "bad_at": [<known_limitations>], 
  "connects_to": <connection_endpoint>,
  "proven_by": {"uses": <count>, "success_rate": <float>}
}
```

##### Field Specifications:
- `tool` (string, required): Tool identifier, max 32 chars
- `does` (string, required): Natural language capability description, max 128 chars
- `when` (array, required): Trigger contexts, max 5 items, max 64 chars each
- `good_at` (array, optional): Specific strengths, max 5 items, max 32 chars each  
- `bad_at` (array, optional): Known limitations, max 3 items, max 32 chars each
- `connects_to` (string, required): Connection URI (mcp://, http://, etc.)
- `proven_by` (object, optional): Usage statistics

#### 3.2.2 Performance Update Message (`t: "perf_update"`)
Reports execution performance metrics, cost data, and optional invocation context.

**Version 2.1 Update**: Enhanced with optional execution context including sanitized arguments.

**Version 2.2 Update**: Added cost tracking for economic efficiency optimization.

```json
{
  "v": 2,
  "t": "perf_update",
  "ts": 1727100286, 
  "sid": "abc123def456",
  "tool": <tool_name>,
  "exec_ms": <execution_time>,
  "success": <boolean>,
  "cost_paid": <number>,
  "currency": <string>,
  "ctx": <context_object>
}
```

##### Field Specifications:
- `tool` (string, required): Tool identifier
- `exec_ms` (number, required): Execution time in milliseconds
- `success` (boolean, required): Whether execution succeeded
- `cost_paid` (number, optional): Actual cost charged in smallest currency units (v2.2)
- `currency` (string, optional): Currency identifier (e.g., "USDC", "ETH") (v2.2)
- `ctx` (object, optional): Execution context, MAY include:
  - `args` (object, optional): Sanitized input parameters (see 6.3.1)
  - `result_size` (number, optional): Response size in bytes
  - `cache_hit` (boolean, optional): Cache status
  - `error_code` (string, optional): Error identifier if `success: false`
  - Additional implementation-specific fields

##### Example with Cost Tracking (v2.2):
```json
{
  "v": 2,
  "t": "perf_update",
  "ts": 1759843829,
  "sid": "workbench-mcp",
  "tool": "get_strategy_code",
  "exec_ms": 450,
  "success": true,
  "cost_paid": 1000,
  "currency": "USDC",
  "ctx": {
    "args": {
      "strategy_name": "AIEnhancedPattern..."
    },
    "result_size": 15420,
    "cache_hit": false
  }
}
```

**Note on cost_paid**: Value is in smallest currency units (e.g., 1000 = 0.001 USDC with 6 decimals). Implementations should document their currency's decimal precision.

#### 3.2.3 Error Pattern Message (`t: "error_pattern"`)  
Describes failure modes and mitigation strategies.

```json
{
  "v": 2,
  "t": "error_pattern",
  "ts": 1727100286,
  "sid": "abc123def456", 
  "tool": <tool_name>,
  "error": <error_type>,
  "trigger": <failure_conditions>,
  "solution": <recommended_action>
}
```

## 4. Transport Protocol Requirements

### 4.1 UDP Advertisement Protocol
- **Port**: 10191
- **Packet Size**: MUST NOT exceed 1472 bytes (avoid fragmentation)
- **Encoding**: UTF-8 JSON
- **Reliability**: Best effort delivery (fire-and-forget)
- **Size Management**: If message with `ctx.args` exceeds 1400 bytes, implementations SHOULD:
  - Truncate string values (append "...")
  - Hash or omit large arguments
  - Prioritize: `tool` → `exec_ms` → `success` → `cost_paid` → `currency` → `ctx` (minimal)

### 4.2 WebSocket Distribution Protocol  
- **Upgrade Protocol**: HTTP/1.1 WebSocket upgrade
- **Subprotocol**: `dcap-v2`
- **Message Format**: JSON text frames
- **Heartbeat**: PING/PONG every 30 seconds

## 5. Discovery Algorithm

### 5.1 Agent Discovery Process
```
1. Agent receives task request
2. Parse semantic intent from request
3. Query accumulated tool knowledge base
4. Match intent against tool descriptions using:
   a. Direct keyword matching in 'when' array
   b. Semantic similarity scoring on 'does' field  
   c. Capability intersection with 'good_at' array
5. Rank candidates by:
   a. proven_by.success_rate (reliability)
   b. Average exec_ms (speed) (v2.1)
   c. Average cost_paid (economic efficiency) (v2.2)
6. Connect to selected tool via connects_to URI
7. Execute task with acquired capability
```

### 5.2 Semantic Matching Requirements
Implementations SHOULD support:
- Exact string matching for triggers
- Fuzzy string matching (edit distance < 3)
- Semantic similarity scoring (cosine similarity > 0.7)

### 5.3 Chain Pattern Detection (v2.1)
Intelligence consumers MAY track sequences of `perf_update` messages to identify tool usage patterns:
- Group messages by `sid` (server identifier)
- Track temporal proximity (typically 5-minute window)
- Build statistical models of successful tool sequences
- Use patterns to recommend optimized workflows

### 5.4 Economic Efficiency Optimization (v2.2)
When `cost_paid` and `currency` are present, intelligence consumers SHOULD:
- Track total cost alongside execution time
- Calculate efficiency score: `success_rate / (exec_ms + normalized_cost)`
- Identify cost-effective alternatives for equivalent capabilities
- Optimize tool chains for both speed and cost

## 6. Security Considerations

### 6.1 Denial of Service Protection
- Rate limiting: Max 100 messages per source IP per minute
- Message size validation: Reject packets > 1472 bytes
- Malformed message filtering: Strict JSON schema validation

### 6.2 Capability Validation
- Agents MUST validate tool endpoints before connection
- Sandbox unknown tools during initial execution
- Monitor tool behavior for anomalies

### 6.3 Privacy Considerations
- Server identifiers SHOULD be pseudonymous
- No personally identifiable information in broadcasts
- Tool usage patterns may reveal sensitive information
- Cost data is semi-public by nature (on-chain transactions)

#### 6.3.1 Argument Sanitization (v2.1)
When including `ctx.args` in `perf_update` messages, implementations MUST:
- **Truncate** string values exceeding 32 characters (append "...")
- **Hash** or **omit** values containing:
  - API keys, tokens, or credentials
  - Personally identifiable information (PII)
  - Sensitive business data
  - File paths revealing system structure
- **Consider** omitting `ctx.args` entirely if sanitization is complex

##### Example Sanitization:
```javascript
// Original args
{
  "api_key": "sk_live_abc123...",
  "user_email": "user@example.com",
  "query": "What are the top performing strategies?"
}

// Sanitized for broadcast
{
  "api_key": "[REDACTED]",
  "user_email": "[REDACTED]", 
  "query": "What are the top performin..."
}
```

#### 6.3.2 Cost Data Transparency (v2.2)
Cost information in `perf_update` messages:
- Reflects actual charged amount, not internal pricing structure
- Is already observable through blockchain/payment protocol
- Enables market efficiency through price transparency
- Tools MAY omit cost data if business model requires confidentiality

## 7. Implementation Guidelines  

### 7.1 Tool Provider Requirements
Tools implementing DCAP MUST:
- Generate unique, stable server identifiers
- Broadcast `semantic_discover` on first capability use
- Send `perf_update` messages for significant events
- Respect UDP packet size limits
- Sanitize arguments before including in `ctx.args` (v2.1)
- Report actual cost charged in `cost_paid` when applicable (v2.2)

### 7.2 Agent Consumer Requirements  
Agents implementing DCAP MUST:
- Maintain local capability knowledge base
- Handle UDP packet loss gracefully
- Implement connection retry logic for tool acquisition
- Validate tool responses before use
- Respect privacy of observed `ctx.args` data (v2.1)
- Consider cost alongside performance when selecting tools (v2.2)

### 7.3 Hub Implementation Requirements
Intelligence hubs MUST:
- Accept UDP packets on port 10191  
- Relay messages via WebSocket without modification
- Support multiple concurrent WebSocket connections
- Implement basic DoS protection

### 7.4 Intelligence Consumer Requirements (v2.1, updated v2.2)
Systems consuming DCAP streams for intelligence (e.g., Oracle agents) SHOULD:
- Track `perf_update` sequences by `sid` for pattern detection
- Maintain statistical models of tool performance AND cost (v2.2)
- Respect privacy constraints when analyzing `ctx.args`
- Only recommend patterns that were ACTUALLY OBSERVED
- Calculate efficiency metrics incorporating both time and cost (v2.2)

## 8. References

### 8.1 Normative References
- [RFC2119] Key words for use in RFCs
- [RFC6455] The WebSocket Protocol  
- [RFC768] User Datagram Protocol
- [MCP] Model Context Protocol Specification

### 8.2 Informative References
- [AutoGPT] Autonomous GPT-4 Experiment
- [LangChain] Framework for developing applications with LLMs
- [x402] x402 Micropayment Protocol

## 9. Changelog

### Version 2.2 (October 2025)
**Added:**
- `cost_paid` field to `perf_update` message for economic tracking
- `currency` field to `perf_update` message for multi-currency support
- Section 5.4: Economic efficiency optimization guidelines
- Section 6.3.2: Cost data transparency considerations
- Updated Section 5.1: Ranking now includes cost optimization

**Rationale:**
- Performance includes both speed and cost
- Oracle agents need cost data to recommend economically efficient tool chains
- Transparency enables market efficiency and price discovery
- Cost is already semi-public through payment protocols (e.g., x402, blockchain)
- Maintains backward compatibility (cost fields optional)

### Version 2.1 (October 2025)
**Added:**
- Enhanced `perf_update` message with optional `ctx` object
- Support for sanitized `ctx.args` in performance updates
- Section 5.3: Chain pattern detection guidelines
- Section 6.3.1: Argument sanitization requirements
- Section 7.4: Intelligence consumer requirements

**Rationale:**
- Enables pattern detection and chain optimization without introducing new message types
- Maintains backward compatibility (all new fields optional)
- Addresses privacy concerns through explicit sanitization requirements

### Version 2.0 (September 2025)
- Initial public specification

## Authors' Addresses

Martin Maurer  
Email: empeamtk@googlemail.com
