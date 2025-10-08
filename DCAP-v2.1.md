# Dynamic Capability Acquisition Protocol (DCAP)

**Version**: 2.1  
**Date**: October 2025  
**Author**: M. Maurer  

## Abstract

This document defines the Dynamic Capability Acquisition Protocol (DCAP), a decentralized protocol enabling autonomous agents to discover, evaluate, and acquire computational capabilities at runtime through semantic broadcasting. DCAP uses UDP multicast for capability advertisement and WebSocket streams for real-time intelligence distribution.

## 1. Introduction

### 1.1 Problem Statement
Current autonomous agent systems operate with fixed, pre-configured tool sets, limiting their ability to handle novel or specialized tasks. Existing tool discovery mechanisms rely on centralized registries, creating bottlenecks and single points of failure.

### 1.2 Solution Overview
DCAP enables tools to self-advertise their capabilities using semantic descriptions, allowing agents to discover and dynamically load appropriate tools based on task requirements rather than static configuration.

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
Reports execution performance metrics and optional invocation context.

**Version 2.1 Update**: Enhanced with optional execution context including sanitized arguments.

```json
{
  "v": 2,
  "t": "perf_update",
  "ts": 1727100286, 
  "sid": "abc123def456",
  "tool": <tool_name>,
  "exec_ms": <execution_time>,
  "success": <boolean>,
  "ctx": <context_object>
}
```

##### Field Specifications:
- `tool` (string, required): Tool identifier
- `exec_ms` (number, required): Execution time in milliseconds
- `success` (boolean, required): Whether execution succeeded
- `ctx` (object, optional): Execution context, MAY include:
  - `args` (object, optional): Sanitized input parameters (see 6.3.1)
  - `result_size` (number, optional): Response size in bytes
  - `cache_hit` (boolean, optional): Cache status
  - `error_code` (string, optional): Error identifier if `success: false`
  - Additional implementation-specific fields

##### Example with Context:
```json
{
  "v": 2,
  "t": "perf_update",
  "ts": 1759843829,
  "sid": "workbench-mcp",
  "tool": "get_strategy_code",
  "exec_ms": 450,
  "success": true,
  "ctx": {
    "args": {
      "strategy_name": "AIEnhancedPattern..."
    },
    "result_size": 15420,
    "cache_hit": false
  }
}
```

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
  - Prioritize: `tool` → `exec_ms` → `success` → `ctx` (minimal)

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
5. Rank candidates by proven_by.success_rate
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

## 7. Implementation Guidelines  

### 7.1 Tool Provider Requirements
Tools implementing DCAP MUST:
- Generate unique, stable server identifiers
- Broadcast `semantic_discover` on first capability use
- Send `perf_update` messages for significant events
- Respect UDP packet size limits
- Sanitize arguments before including in `ctx.args` (v2.1)

### 7.2 Agent Consumer Requirements  
Agents implementing DCAP MUST:
- Maintain local capability knowledge base
- Handle UDP packet loss gracefully
- Implement connection retry logic for tool acquisition
- Validate tool responses before use
- Respect privacy of observed `ctx.args` data (v2.1)

### 7.3 Hub Implementation Requirements
Intelligence hubs MUST:
- Accept UDP packets on port 10191  
- Relay messages via WebSocket without modification
- Support multiple concurrent WebSocket connections
- Implement basic DoS protection

### 7.4 Intelligence Consumer Requirements (v2.1)
Systems consuming DCAP streams for intelligence (e.g., Oracle agents) SHOULD:
- Track `perf_update` sequences by `sid` for pattern detection
- Maintain statistical models of tool performance
- Respect privacy constraints when analyzing `ctx.args`
- Only recommend patterns that were ACTUALLY OBSERVED

## 8. References

### 8.1 Normative References
- [RFC2119] Key words for use in RFCs
- [RFC6455] The WebSocket Protocol  
- [RFC768] User Datagram Protocol
- [MCP] Model Context Protocol Specification

### 8.2 Informative References
- [AutoGPT] Autonomous GPT-4 Experiment
- [LangChain] Framework for developing applications with LLMs

## 9. Changelog

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

## Appendix A. Example Implementation

```javascript
// Tool Provider Example (v2.1 with context)
class DCAPTool {
  constructor(toolName, capabilities) {
    this.sid = generateServerId();
    this.tool = toolName;
    this.capabilities = capabilities;
  }
  
  broadcastDiscovery() {
    const message = {
      v: 2,
      t: "semantic_discover",
      ts: Math.floor(Date.now() / 1000),
      sid: this.sid,
      tool: this.tool,
      ...this.capabilities
    };
    
    udp.send(JSON.stringify(message), 10191, 'broadcast');
  }
  
  reportPerformance(toolName, execMs, success, args) {
    const message = {
      v: 2,
      t: "perf_update",
      ts: Math.floor(Date.now() / 1000),
      sid: this.sid,
      tool: toolName,
      exec_ms: execMs,
      success: success,
      ctx: {
        args: this.sanitizeArgs(args)
      }
    };
    
    // Ensure message fits in UDP packet
    const json = JSON.stringify(message);
    if (json.length > 1400) {
      // Truncate or omit ctx.args
      delete message.ctx.args;
    }
    
    udp.send(JSON.stringify(message), 10191, 'broadcast');
  }
  
  sanitizeArgs(args) {
    const sanitized = {};
    for (const [key, value] of Object.entries(args)) {
      if (typeof value === 'string') {
        // Redact sensitive keys
        if (/key|token|secret|password|credential/i.test(key)) {
          sanitized[key] = '[REDACTED]';
        }
        // Truncate long strings
        else if (value.length > 32) {
          sanitized[key] = value.substring(0, 29) + '...';
        }
        else {
          sanitized[key] = value;
        }
      } else {
        sanitized[key] = value;
      }
    }
    return sanitized;
  }
}

// Agent Consumer Example  
class DCAPAgent {
  constructor() {
    this.knownTools = new Map();
    this.ws = new WebSocket('ws://hub:10191', 'dcap-v2');
    this.ws.onmessage = this.handleMessage.bind(this);
  }
  
  handleMessage(event) {
    const msg = JSON.parse(event.data);
    if (msg.t === 'semantic_discover') {
      this.knownTools.set(msg.tool, msg);
    }
    else if (msg.t === 'perf_update') {
      this.updatePerformanceStats(msg);
    }
  }
  
  updatePerformanceStats(msg) {
    const tool = this.knownTools.get(msg.tool);
    if (!tool) return;
    
    // Track performance trends
    if (!tool.perf_history) tool.perf_history = [];
    tool.perf_history.push({
      ts: msg.ts,
      exec_ms: msg.exec_ms,
      success: msg.success
    });
    
    // Update proven_by statistics
    const recent = tool.perf_history.slice(-100);
    tool.proven_by = {
      uses: recent.length,
      success_rate: recent.filter(p => p.success).length / recent.length
    };
  }
}

// Intelligence Consumer Example (v2.1 - Chain Detection)
class ChainDetector {
  constructor() {
    this.sessions = new Map(); // sid -> tool sequence
    this.patterns = new Map(); // pattern hash -> statistics
  }
  
  handlePerformanceUpdate(msg) {
    const { sid, tool, success, ts } = msg;
    
    // Track sequence per session
    if (!this.sessions.has(sid)) {
      this.sessions.set(sid, []);
    }
    
    const sequence = this.sessions.get(sid);
    sequence.push({ tool, success, ts });
    
    // Detect chains (2+ tools within 5 minutes)
    if (sequence.length >= 2 && success) {
      const window = sequence.filter(s => ts - s.ts < 300); // 5 min
      if (window.length >= 2) {
        this.recordPattern(window.map(s => s.tool));
      }
    }
  }
  
  recordPattern(tools) {
    const key = tools.join('→');
    const stats = this.patterns.get(key) || { 
      count: 0, 
      first_seen: Date.now() 
    };
    stats.count++;
    this.patterns.set(key, stats);
  }
}
```

## Authors' Addresses

Martin Maurer  
Email: empeamtk@googlemail.com
