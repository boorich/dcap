# Dynamic Capability Acquisition Protocol (DCAP)

**Version**: 3.0  
**Date**: December 2025  
**Author**: M. Maurer  

## Abstract

This document defines the Dynamic Capability Acquisition Protocol (DCAP), a decentralized protocol enabling autonomous agents to discover, evaluate, and acquire computational capabilities at runtime through semantic broadcasting. DCAP uses UDP multicast for capability advertisement and WebSocket streams for real-time intelligence distribution. Version 3.0 introduces agent verification through the `usage_receipt` message type, enabling third-party observation of tool performance alongside tool self-reports.

## 1. Introduction

### 1.1 Problem Statement
Current autonomous agent systems operate with fixed, pre-configured tool sets, limiting their ability to handle novel or specialized tasks. Existing tool discovery mechanisms rely on centralized registries, creating bottlenecks and single points of failure. Additionally, discovered tools often lack sufficient connection metadata, making autonomous acquisition impractical. Finally, tool-provided performance metrics (self-reports) may not reflect actual observed behavior, limiting trustworthy reputation systems.

### 1.2 Solution Overview
DCAP enables tools to self-advertise their capabilities using semantic descriptions, allowing agents to discover and dynamically load appropriate tools based on task requirements rather than static configuration. Version 3.0 introduces a distinction between tool self-reports (`perf_update`) and agent observations (`usage_receipt`), enabling intelligence systems to build reputation from third-party data.

## 2. Protocol Architecture

### 2.1 Network Topology
```
Tool Provider → UDP Broadcast → Intelligence Hub → WebSocket → Agent Consumer
     ^                              ^                   ^           ^
   MCP Tool                   Stream Aggregator    Real-time     Dynamic Agent
   (has sid)                                                    (has agent_id)
     ↓                                                                ↓
perf_update (self-report)                                   usage_receipt (observation)
```

### 2.2 Transport Layers
- **Advertisement Layer**: UDP packets (port 10191)
- **Distribution Layer**: WebSocket streams 
- **Acquisition Layer**: MCP protocol over stdio/HTTP/SSE (specified in connector)

### 2.3 Message Sources (v3.0)
DCAP distinguishes between two types of broadcasters:
- **Tools** (MCP servers): Identified by `sid`, broadcast capabilities and self-reports
- **Agents** (consumers): Identified by `agent_id`, broadcast observed tool behavior

This distinction enables intelligence systems to compare tool claims against agent observations.

## 3. Message Format Specification

### 3.1 Base Message Structure
All DCAP messages MUST conform to this JSON schema:

```json
{
  "v": <protocol_version>,
  "t": <message_type>, 
  "ts": <unix_timestamp>,
  "sid": <server_identifier>,     // for tool messages
  "agent_id": <agent_identifier>, // for agent messages
  [message_specific_fields]
}
```

#### Required Fields:
- `v` (number): Protocol version (current: 2)
- `t` (string): Message type identifier
- `ts` (number): Unix timestamp of message creation
- `sid` (string): Unique server identifier (8-12 characters) - for tool messages
- `agent_id` (string): Unique agent identifier (8-32 characters) - for agent messages

### 3.2 Message Types

#### 3.2.1 Semantic Discovery Message (`t: "semantic_discover"`)
Advertises tool capabilities using natural language descriptions with comprehensive connection information.

**Broadcast by:** Tools (MCP servers)

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
  "connector": {
    "transport": <transport_type>,
    "endpoint": <connection_url>,
    "auth": <auth_requirements>,
    "headers": <required_headers>,
    "protocol": <protocol_details>,
    "session": <session_requirements>
  },
  "proven_by": {"uses": <count>, "success_rate": <float>}
}
```

##### Field Specifications:
- `tool` (string, required): Tool identifier, max 32 chars
- `does` (string, required): Natural language capability description, max 128 chars
- `when` (array, required): Trigger contexts, max 5 items, max 64 chars each
- `good_at` (array, optional): Specific strengths, max 5 items, max 32 chars each  
- `bad_at` (array, optional): Known limitations, max 3 items, max 32 chars each
- `connector` (object, required): Connection specification
- `proven_by` (object, optional): Usage statistics (note: spec does not prescribe how this is calculated)

##### Connector Object Specification:
```json
"connector": {
  "transport": "stdio" | "sse" | "http",
  "endpoint": "<url_or_command>",
  "auth": {
    "type": "none" | "oauth2" | "bearer" | "x402" | "api_key",
    "required": <boolean>,
    "details": <auth_specific_object>
  },
  "headers": {
    "required": [<header_name>],
    "optional": {<header_name>: <default_value>}
  },
  "protocol": {
    "type": "mcp" | "rest" | "grpc",
    "version": "<version_string>",
    "methods": [<available_methods>]
  },
  "session": {
    "required": <boolean>,
    "initialization": <init_details>
  }
}
```

**Connector Field Details:**

- `transport` (string, required): Connection transport type
  - `"stdio"`: Standard input/output (local process)
  - `"sse"`: Server-Sent Events over HTTP
  - `"http"`: HTTP/HTTPS connection

- `endpoint` (string, required): Connection endpoint
  - For stdio: command to execute (e.g., `"npx @modelcontextprotocol/server-filesystem /path"`)
  - For sse/http: URL (e.g., `"https://example.com/mcp"`)

- `auth` (object, required): Authentication requirements
  - `type` (string, required): Authentication method
  - `required` (boolean, required): Whether authentication is mandatory
  - `details` (object, optional): Auth-specific configuration

**Authentication Type Details:**

**none:**
```json
{
  "type": "none",
  "required": false,
  "details": {
    "notes": "No authentication needed"
  }
}
```

**bearer:**
```json
{
  "type": "bearer",
  "required": true,
  "details": {
    "instructions_url": "https://example.com/docs/auth",
    "credential_source": "env:API_TOKEN",
    "header_format": "Bearer {token}"
  }
}
```

**api_key:**
```json
{
  "type": "api_key",
  "required": true,
  "details": {
    "location": "header" | "query",
    "param_name": "X-API-Key",
    "format": "{key}",
    "instructions_url": "https://example.com/docs/api-keys",
    "credential_source": "env:API_KEY",
    "registration_url": "https://example.com/signup"
  }
}
```

**oauth2:**
```json
{
  "type": "oauth2",
  "required": true,
  "details": {
    "flow": "authorization_code" | "client_credentials",
    "auth_url": "https://example.com/oauth/authorize",
    "token_url": "https://example.com/oauth/token",
    "scopes": ["read", "write"],
    "pkce_required": true,
    "instructions_url": "https://example.com/docs/oauth",
    "credential_source": "oauth_provider:example"
  }
}
```

**x402:**
```json
{
  "type": "x402",
  "required": true,
  "details": {
    "network": "base-sepolia",
    "asset": "0x036CbD53842c5426634e7929541eC2318f3dCF7e",
    "currency": "USDC",
    "recipient": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb1",
    "price_per_call": 100000
  }
}
```

- `headers` (object, optional): HTTP header requirements
  - `required` (array): List of required header names
  - `optional` (object): Optional headers with default values

- `protocol` (object, required): Protocol specification
  - `type` (string, required): Protocol type
  - `version` (string, optional): Protocol version
  - `methods` (array, optional): Available methods/endpoints

- `session` (object, optional): Session requirements
  - `required` (boolean): Whether session initialization is needed
  - `initialization` (object): Session initialization details

##### Example: Financial Tool
```json
{
  "v": 2,
  "t": "semantic_discover",
  "ts": 1735000000,
  "sid": "finadv-mcp",
  "tool": "financial_advisor",
  "does": "Provides SEC-compliant investment advice",
  "when": ["investment advice", "portfolio analysis", "financial planning"],
  "good_at": ["SEC compliance", "risk assessment", "tax optimization"],
  "bad_at": ["real-time trading", "crypto speculation"],
  "connector": {
    "transport": "http",
    "endpoint": "https://finadvice.ai/mcp",
    "auth": {
      "type": "oauth2",
      "required": true,
      "details": {
        "flow": "authorization_code",
        "auth_url": "https://finadvice.ai/oauth/authorize",
        "token_url": "https://finadvice.ai/oauth/token",
        "scopes": ["financial:read", "financial:advise"],
        "instructions_url": "https://finadvice.ai/docs/authentication",
        "pkce_required": true
      }
    },
    "headers": {
      "required": ["Accept", "Content-Type"],
      "optional": {
        "Accept": "application/json",
        "Content-Type": "application/json"
      }
    },
    "protocol": {
      "type": "mcp",
      "version": "2024-11-05",
      "methods": ["tools/list", "tools/call"]
    },
    "session": {
      "required": false
    }
  },
  "proven_by": {
    "uses": 5420,
    "success_rate": 0.98
  }
}
```

##### Example: Local Tool
```json
{
  "v": 2,
  "t": "semantic_discover",
  "ts": 1735000000,
  "sid": "filesystem-local",
  "tool": "read_file",
  "does": "Reads file contents from local filesystem",
  "when": ["need file contents", "read configuration"],
  "good_at": ["large files", "multiple encodings"],
  "bad_at": ["remote files", "binary files"],
  "connector": {
    "transport": "stdio",
    "endpoint": "npx @modelcontextprotocol/server-filesystem /workspace",
    "auth": {
      "type": "none",
      "required": false,
      "details": {
        "notes": "Local filesystem access, no authentication needed"
      }
    },
    "protocol": {
      "type": "mcp",
      "version": "2024-11-05",
      "methods": ["tools/list", "tools/call"]
    },
    "session": {
      "required": false
    }
  },
  "proven_by": {
    "uses": 8472,
    "success_rate": 0.99
  }
}
```

#### 3.2.2 Performance Update Message (`t: "perf_update"`)
Reports tool execution performance from the tool's perspective (self-report).

**Broadcast by:** Tools (MCP servers)

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
- `tool` (string, required): Tool identifier matching `semantic_discover`
- `exec_ms` (number, required): Execution time in milliseconds
- `success` (boolean, required): Whether execution succeeded
- `cost_paid` (number, optional): Cost incurred (if applicable)
- `currency` (string, optional): Currency denomination (e.g., "USDC", "USD")
- `ctx` (object, optional): Execution context

##### Context Object (`ctx`):
```json
"ctx": {
  "caller": <caller_identifier>,
  "payer": <payer_address>,
  "args": <sanitized_arguments>,
  "invocation_id": <uuid>
}
```

**Privacy Note:** The `ctx.args` field SHOULD be sanitized to remove sensitive information before broadcasting.

##### Example:
```json
{
  "v": 2,
  "t": "perf_update",
  "ts": 1735000000,
  "sid": "finadv-mcp",
  "tool": "financial_advisor",
  "exec_ms": 245,
  "success": true,
  "cost_paid": 100000,
  "currency": "USDC",
  "ctx": {
    "caller": "agent-alice",
    "payer": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb1",
    "args": {
      "portfolio_type": "aggressive",
      "risk_tolerance": "medium"
    },
    "invocation_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

#### 3.2.3 Usage Receipt Message (`t: "usage_receipt"`) - NEW in v3.0
Reports tool execution performance from the agent's perspective (third-party observation).

**Broadcast by:** Agents (tool consumers)

**Purpose:** Enables agents to share their observations of tool behavior, providing third-party data alongside tool self-reports. Intelligence systems MAY use this data for reputation calculation, comparison with tool claims, or other analytical purposes.

```json
{
  "v": 2,
  "t": "usage_receipt",
  "ts": <unix_timestamp>,
  "agent_id": "<agent_identifier>",
  "tool": "<tool_name>",
  "tool_sid": "<tool_server_id>",
  "success": <boolean>,
  "exec_ms": <number>,
  "cost_paid": <number>,
  "currency": "<string>",
  "payment_proof": "<string>",
  "invocation_id": "<uuid>",
  "error_observed": "<string>",
  "ctx": <object>,
  "blockchain_registrations": [<array>]
}
```

##### Required Fields:
- `v` (number): Protocol version (2)
- `t` (string): Message type ("usage_receipt")
- `ts` (number): Unix timestamp when observation was made
- `agent_id` (string): Agent identifier (8-32 chars, unique per agent)
- `tool` (string): Name of tool being reported on
- `tool_sid` (string): Server identifier of tool (from `semantic_discover`)
- `success` (boolean): Whether invocation succeeded (agent's observation)
- `exec_ms` (number): Execution time in milliseconds (agent's measurement)

##### Optional Fields:
- `cost_paid` (number): Actual cost paid by agent (if applicable)
- `currency` (string): Currency denomination (e.g., "USDC", "USD")
- `payment_proof` (string): Transaction hash or receipt identifier
- `invocation_id` (string): UUID linking to specific invocation
- `error_observed` (string): Error message if failure occurred
- `ctx` (object): Additional context about the observation
- `blockchain_registrations` (array): Agent's blockchain identity registrations

##### Blockchain Registration Object:
When agents include blockchain registration, they use this structure to link their DCAP identity to on-chain identity (e.g., ERC-8004):

```json
"blockchain_registrations": [
  {
    "agentId": <number>,
    "agentRegistry": "eip155:<chainId>:<registryAddress>",
    "tokenURI": "<ipfs_or_http_uri>",
    "verification_url": "<explorer_url>"
  }
]
```

**Blockchain Registration Field Details:**
- `agentId` (number, required): Agent identifier within the registry (ERC-721 tokenId per ERC-8004)
- `agentRegistry` (string, required): Combined format per ERC-8004: `"eip155:<chainId>:<registryAddress>"`
  - Example: `"eip155:1:0xabcd..."` for Ethereum mainnet
  - Example: `"eip155:8453:0x1234..."` for Base
- `tokenURI` (string, optional): URI to agent registration file (IPFS or HTTPS per ERC-8004)
- `verification_url` (string, optional): Human-readable verification link (e.g., block explorer NFT page)

**Note on ERC-8004 Compatibility:**
This format follows ERC-8004 (Trustless Agents) standard. The `agentRegistry` field uses ERC-8004's combined format `"eip155:<chainId>:<registryAddress>"` where the registry is an ERC-721 contract with URIStorage extension. The `agentId` corresponds to the ERC-721 tokenId.

Intelligence systems MAY use blockchain registrations to verify agent identity and weight observations accordingly. The protocol does not prescribe how this verification or weighting should be performed.

##### Example: Agent Report with Blockchain Identity
```json
{
  "v": 2,
  "t": "usage_receipt",
  "ts": 1735000000,
  "agent_id": "agent-alice-001",
  "tool": "financial_advisor",
  "tool_sid": "finadv-mcp",
  "success": false,
  "exec_ms": 5243,
  "cost_paid": 100000,
  "currency": "USDC",
  "payment_proof": "0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef",
  "invocation_id": "550e8400-e29b-41d4-a716-446655440000",
  "error_observed": "timeout after 5s waiting for response",
  "ctx": {
    "request_type": "portfolio_analysis",
    "retry_count": 2
  },
  "blockchain_registrations": [
    {
      "agentId": 789,
      "agentRegistry": "eip155:1:0xabcdefabcdefabcdefabcdefabcdefabcdefabcd",
      "tokenURI": "ipfs://QmXyZ123...",
      "verification_url": "https://etherscan.io/nft/0xabcdefabcdefabcdefabcdefabcdefabcdefabcd/789"
    }
  ]
}
```

##### Example: Simple Agent Report
```json
{
  "v": 2,
  "t": "usage_receipt",
  "ts": 1735000000,
  "agent_id": "agent-bob",
  "tool": "read_file",
  "tool_sid": "filesystem-local",
  "success": true,
  "exec_ms": 12
}
```

#### 3.2.4 Error Pattern Message (`t: "error_pattern"`)
Reports recurring error patterns for proactive error handling.

**Broadcast by:** Tools (MCP servers)

```json
{
  "v": 2,
  "t": "error_pattern",
  "ts": 1727100286,
  "sid": "abc123def456",
  "tool": <tool_name>,
  "error_type": <error_classification>,
  "frequency": <occurrence_count>,
  "sample_args": <sanitized_arguments>,
  "mitigation": <suggested_fix>
}
```

##### Field Specifications:
- `tool` (string, required): Tool identifier
- `error_type` (string, required): Error classification
- `frequency` (number, required): Occurrence count in recent window
- `sample_args` (object, optional): Sanitized sample of problematic arguments
- `mitigation` (string, optional): Suggested mitigation strategy

## 4. Transport Protocol Requirements

### 4.1 UDP Advertisement Protocol
- **Port**: 10191
- **Packet Size**: MUST NOT exceed 1472 bytes (avoid fragmentation)
- **Encoding**: UTF-8 JSON
- **Reliability**: Best effort delivery (fire-and-forget)

### 4.2 Size Management
If message exceeds 1400 bytes, implementations SHOULD truncate in this priority order:

1. Omit `ctx` object (for `perf_update`, `usage_receipt`)
2. Omit `blockchain_registrations` array (for `usage_receipt`)
3. Omit `connector.session` object
4. Omit `connector.headers.optional` object
5. Omit `connector.protocol.methods` array
6. Truncate `auth.details.instructions_url` to domain only
7. Omit `auth.details.registration_url`
8. Keep: `tool`, `connector.transport`, `connector.endpoint`, `connector.auth.type`, `connector.auth.required`

### 4.3 WebSocket Distribution Protocol  
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
   b. Average exec_ms (speed)
   c. Average cost_paid (economic efficiency)
   d. Auth complexity (prefer 'none' > 'api_key' > 'bearer' > 'oauth2')
6. Extract connector details from selected tool:
   a. Parse transport type and endpoint
   b. Check authentication requirements and available credentials
   c. Verify required headers can be provided
   d. Prepare session initialization if needed
   e. Establish connection according to protocol type
7. Execute task with acquired capability
8. Broadcast usage_receipt with observed performance (v3.0)
```

**Note:** The ranking criteria listed above are suggestions. Agents MAY implement any ranking algorithm appropriate for their use case.

### 5.2 Semantic Matching Requirements
Implementations SHOULD support:
- Exact string matching for triggers
- Fuzzy string matching (edit distance < 3)
- Semantic similarity scoring (cosine similarity > 0.7)

### 5.3 Chain Pattern Detection
Intelligence systems MAY:
- Track sequences of tool invocations
- Identify commonly chained tools
- Recommend tool chains for complex tasks

### 5.4 Economic Efficiency Optimization
Intelligence systems MAY:
- Track cost trends over time
- Identify cost-effective alternatives
- Recommend tools with best price/performance ratio

### 5.5 Dynamic Tool Acquisition
Agents SHOULD:
- Parse `connector` object to determine connection method
- Support multiple transport types or gracefully skip unsupported tools
- Validate endpoints before attempting connection
- Handle authentication flows or prompt users with `instructions_url`

## 6. Security Considerations

### 6.1 Denial of Service Protection
Intelligence hubs SHOULD implement:
- Rate limiting per `sid` and `agent_id`
- Message validation before distribution
- Duplicate message detection
- Malformed message rejection

### 6.2 Capability Validation
Agents MUST:
- Validate tool endpoints before connection
- Sandbox unknown tools during initial execution
- Monitor tool behavior for anomalies
- Verify TLS certificates for `https://` endpoints
- For `stdio` transport, validate commands are from trusted sources
- Validate OAuth redirect URIs match `auth_url` domain
- Never execute stdio commands with user-controlled input

### 6.3 Privacy Considerations

#### 6.3.1 Argument Sanitization
Tools broadcasting `perf_update` and agents broadcasting `usage_receipt` SHOULD:
- Sanitize `ctx.args` to remove sensitive information
- Replace sensitive values with type indicators (e.g., `"<email>"`, `"<token>"`)
- Never broadcast credentials, passwords, or API keys
- Omit `ctx` entirely if sanitization is uncertain

#### 6.3.2 Cost Data Transparency
- `cost_paid` and `currency` fields provide transparency
- Agents can compare claimed costs against actual charges
- Economic data enables market-driven tool selection

#### 6.3.3 Endpoint Security
Tools SHOULD:
- Use HTTPS for `http` transport endpoints
- Validate TLS certificates
- Provide clear authentication instructions
- Support secure credential acquisition methods
- Use environment variables or secure vaults for credential storage

#### 6.3.4 OAuth Security
OAuth implementations MUST:
- Validate redirect URIs
- Implement PKCE when `pkce_required: true`
- Never expose client secrets in broadcast messages
- Provide secure token storage recommendations
- Implement proper scope validation

#### 6.3.5 Blockchain Verification Security
When verifying agent blockchain registrations, intelligence systems SHOULD:
- Verify registry contract address matches known registries
- Confirm chain_id matches expected network
- Query on-chain data to validate agent_id exists
- Check for revocation or expiration status
- Validate metadata_uri content if used

**Warning:** Blockchain registration provides identity anchoring but does not guarantee agent honesty. Intelligence systems should still validate observation consistency and detect anomalous reporting patterns.

## 7. Implementation Guidelines  

### 7.1 Tool Provider Requirements
Tools implementing DCAP MUST:
- Generate unique, stable server identifiers (`sid`)
- Broadcast `semantic_discover` on first capability use with complete `connector` details
- Include comprehensive auth details per specification
- Specify all required HTTP headers in `connector.headers`
- Document session initialization requirements if applicable
- Respect UDP packet size limits (prioritize critical fields)

Tools implementing DCAP SHOULD:
- Send `perf_update` messages for significant events
- Sanitize arguments before including in `ctx.args`
- Report actual cost charged in `cost_paid` when applicable
- Ensure `connector.endpoint` is reachable from agent networks
- Keep authentication requirements current in `connector.auth`
- Provide valid `instructions_url` for credential acquisition

Tools implementing DCAP MAY:
- Broadcast `error_pattern` messages for recurring issues
- Include additional context in `ctx` fields
- Provide registration URLs for credential acquisition

### 7.2 Agent Consumer Requirements  
Agents implementing DCAP MUST:
- Maintain local capability knowledge base
- Handle UDP packet loss gracefully
- Implement connection retry logic for tool acquisition
- Validate tool responses before use
- Parse and validate `connector` information before attempting connection
- Implement secure credential storage for auth tokens
- Support multiple transport types or gracefully skip unsupported tools

Agents implementing DCAP SHOULD:
- Broadcast `usage_receipt` after tool invocations (v3.0)
- Report actual observed performance (what was measured)
- Include `cost_paid` if payment occurred
- Sanitize or omit sensitive information from `ctx`
- Handle OAuth flows or prompt users with `instructions_url`
- Respect `headers.required` when making HTTP requests
- Implement session initialization per `session` specification
- Check credential availability before attempting tool calls

Agents implementing DCAP MAY:
- Include `blockchain_registrations` for identity anchoring
- Choose which invocations to report (not required to report all)
- Include additional context in `ctx` field
- Include `invocation_id` for detailed tracking

### 7.3 Hub Implementation Requirements
Intelligence hubs implementing DCAP MUST:
- Listen on UDP port 10191 for broadcasts
- Validate message format before distribution
- Distribute messages via WebSocket with `dcap-v2` subprotocol
- Implement rate limiting per `sid` and `agent_id`
- Handle malformed messages gracefully

Intelligence hubs implementing DCAP SHOULD:
- Maintain message history for late-joining agents
- Implement duplicate message detection
- Track tool availability and broadcast status
- Provide query interfaces for capability search

Intelligence hubs implementing DCAP MAY:
- Calculate aggregate statistics (e.g., `proven_by` metrics)
- Compare `perf_update` (tool claims) vs `usage_receipt` (agent observations)
- Weight observations by agent blockchain verification status
- Detect and flag discrepancies between tool self-reports and agent observations
- Implement any reputation or trust algorithms appropriate for their use case

**Note:** The protocol does not prescribe how hubs calculate reputation metrics. Hubs are free to implement any algorithm using the available message data.

### 7.4 Intelligence Consumer Requirements
Systems consuming DCAP streams (e.g., Oracle agents) SHOULD:
- Track message sequences by `sid` and `agent_id` for pattern detection
- Maintain statistical models of tool performance
- Respect privacy constraints when analyzing `ctx` fields
- Index tools by `connector.transport` for transport-specific recommendations
- Track authentication requirements to inform agents of access needs
- Classify tools by `auth.type` complexity for smart recommendations

Systems consuming DCAP streams MAY:
- Implement any reputation or ranking algorithm using message data
- Correlate tool self-reports with agent observations
- Weight agent observations by any criteria (blockchain verification, history, etc.)
- Detect and report anomalous patterns
- Calculate efficiency metrics incorporating time and cost

## 8. References

### 8.1 Normative References
- [RFC2119] Key words for use in RFCs
- [RFC6455] The WebSocket Protocol  
- [RFC768] User Datagram Protocol
- [RFC6749] OAuth 2.0 Authorization Framework
- [RFC7636] Proof Key for Code Exchange (PKCE)
- [MCP] Model Context Protocol Specification

### 8.2 Informative References
- [AutoGPT] Autonomous GPT-4 Experiment
- [LangChain] Framework for developing applications with LLMs
- [x402] x402 Micropayment Protocol
- [EIP-155] Ethereum Chain ID Specification
- [ERC-8004] ERC-8004: Trustless Agents - Discover agents and establish trust through reputation and validation (draft)

## 9. Changelog

### Version 3.0 (December 2025)
**Added:**
- New message type: `usage_receipt` for agent-reported observations
- `agent_id` field for agent message identification
- `blockchain_registrations` array for agent identity anchoring (in `usage_receipt`)
- Distinction between tool self-reports and agent observations
- Guidelines for agent verification broadcasting
- Security considerations for blockchain verification

**Changed:**
- Clarified that `perf_update` is a tool self-report
- Specified that `proven_by` calculation is implementation-specific (not prescribed by protocol)
- Updated security section to address agent identity verification

**Removed:**
- `blockchain_registrations` from `semantic_discover` (corrected architectural error from v2.7)

**Rationale:**
- Version 2.7 incorrectly placed `blockchain_registrations` in tool messages (`semantic_discover`)
- Tools (MCP servers) are not agents - they are identified by `sid`
- Blockchain registration (ERC-8004) is for agent identity, not tool identity
- v3.0 corrects this by moving blockchain registration to agent messages (`usage_receipt`)
- Adding `usage_receipt` enables third-party verification: agents report what they observe, tools report what they claim
- Intelligence systems can now compare tool claims against agent observations
- The protocol remains pure: it defines message formats, not trust algorithms

**Migration from v2.7:**
- Tools: No changes required. Continue broadcasting `semantic_discover` and `perf_update`
- Agents: New capability to broadcast `usage_receipt` with optional blockchain identity
- Hubs: New message type to distribute; can optionally use for reputation calculations
- Backward compatible: v2.7 tools work unchanged; v3.0 adds new capability

**Breaking Change:**
- `blockchain_registrations` removed from `semantic_discover` message
- Tools broadcasting v2.7 format with blockchain registrations will have that field ignored
- Since blockchain registrations were for agent identity (not tool identity), this correction has no practical impact on existing tools

### Version 2.7 (November 2025)
- Added `blockchain_registrations` to `semantic_discover` (architectural error, corrected in v3.0)
- Introduced two-tier architecture concept (DCAP + blockchain)

### Version 2.6 (November 2025)
- Enhanced `connector` object with comprehensive authentication details
- Added OAuth2, bearer, api_key, and x402 authentication types
- Added `headers` specification for HTTP requests
- Added `session` initialization requirements
- Enabled truly autonomous tool acquisition

### Version 2.5 (October 2025)
- Introduced `connector` object for connection automation
- Added transport types: stdio, sse, http
- Added basic authentication specification

### Version 2.4 (October 2025)
- Enhanced security considerations
- Added credential management guidelines

### Version 2.3 (October 2025)
- Added pattern detection capabilities
- Enhanced discovery algorithm

### Version 2.2 (October 2025)
- Added cost tracking: `cost_paid` and `currency` fields
- Enabled economic efficiency optimization

### Version 2.1 (October 2025)
- Added context tracking: `ctx` field in `perf_update`
- Added `invocation_id` for message correlation
- Enhanced privacy considerations

### Version 2.0 (September 2025)
- Initial public specification
- Core message types: `semantic_discover`, `perf_update`, `error_pattern`
- UDP and WebSocket transport protocols
- Basic discovery algorithm

## Appendix A. Example Agent Implementation

```javascript
// Agent with Usage Receipt Broadcasting (v3.0)
class DCAPAgentV3 {
  constructor(agentId, blockchainRegistrations = null) {
    this.agentId = agentId;
    this.blockchainRegistrations = blockchainRegistrations;
    this.knownTools = new Map();
    
    // Connect to DCAP stream
    this.ws = new WebSocket('ws://hub:10191', 'dcap-v2');
    this.ws.onmessage = this.handleMessage.bind(this);
  }
  
  handleMessage(event) {
    const msg = JSON.parse(event.data);
    
    if (msg.t === 'semantic_discover') {
      this.knownTools.set(msg.tool, msg);
    }
  }
  
  async invokeTool(toolName, args) {
    const tool = this.knownTools.get(toolName);
    if (!tool) throw new Error(`Tool ${toolName} not found`);
    
    // Measure actual execution time
    const startTime = Date.now();
    
    try {
      // Connect and invoke tool
      const client = await this.connectToTool(tool);
      const result = await client.callTool(toolName, args);
      const execMs = Date.now() - startTime;
      
      // Broadcast usage receipt with observation
      this.broadcastUsageReceipt({
        tool: toolName,
        tool_sid: tool.sid,
        success: true,
        exec_ms: execMs,
        cost_paid: result.cost,
        currency: result.currency,
        invocation_id: result.invocation_id
      });
      
      return result;
      
    } catch (error) {
      const execMs = Date.now() - startTime;
      
      // Report failures too (critical for accurate observation!)
      this.broadcastUsageReceipt({
        tool: toolName,
        tool_sid: tool.sid,
        success: false,
        exec_ms: execMs,
        error_observed: error.message
      });
      
      throw error;
    }
  }
  
  broadcastUsageReceipt(receipt) {
    const message = {
      v: 2,
      t: 'usage_receipt',
      ts: Math.floor(Date.now() / 1000),
      agent_id: this.agentId,
      ...receipt
    };
    
    // Include blockchain identity if available
    if (this.blockchainRegistrations) {
      message.blockchain_registrations = this.blockchainRegistrations;
    }
    
    // Broadcast via UDP
    this.udpBroadcast(message);
  }
  
  async connectToTool(tool) {
    const { transport, endpoint, auth, headers } = tool.connector;
    
    // Handle different transports
    if (transport === 'stdio') {
      return await this.connectStdio(endpoint);
    } else if (transport === 'sse' || transport === 'http') {
      return await this.connectHttp(endpoint, auth, headers);
    }
    
    throw new Error(`Unsupported transport: ${transport}`);
  }
  
  async connectHttp(endpoint, auth, headers) {
    // Prepare authentication
    const authHeaders = await this.prepareAuth(auth);
    
    // Prepare required headers
    const allHeaders = {
      ...headers?.optional,
      ...authHeaders
    };
    
    // Return HTTP client
    return new MCPHttpClient(endpoint, allHeaders);
  }
  
  async prepareAuth(auth) {
    if (!auth.required || auth.type === 'none') {
      return {};
    }
    
    if (auth.type === 'bearer') {
      const token = await this.getCredential(auth.details.credential_source);
      return {
        'Authorization': auth.details.header_format.replace('{token}', token)
      };
    }
    
    if (auth.type === 'api_key') {
      const key = await this.getCredential(auth.details.credential_source);
      if (auth.details.location === 'header') {
        return {
          [auth.details.param_name]: auth.details.format.replace('{key}', key)
        };
      }
    }
    
    if (auth.type === 'oauth2') {
      // Handle OAuth2 flow
      const token = await this.handleOAuth2(auth.details);
      return {
        'Authorization': `Bearer ${token}`
      };
    }
    
    if (auth.type === 'x402') {
      // Handle x402 payment
      const payment = await this.handleX402Payment(auth.details);
      return {
        'X-402-Payment': payment
      };
    }
    
    throw new Error(`Unsupported auth type: ${auth.type}`);
  }
  
  async getCredential(source) {
    if (source.startsWith('env:')) {
      const envVar = source.substring(4);
      return process.env[envVar];
    }
    // Handle other credential sources...
    throw new Error(`Unsupported credential source: ${source}`);
  }
  
  udpBroadcast(message) {
    const json = JSON.stringify(message);
    const buffer = Buffer.from(json, 'utf8');
    
    if (buffer.length > 1472) {
      console.warn('Message exceeds UDP packet size, truncating...');
      // Handle truncation...
    }
    
    // Broadcast via UDP to port 10191
    // Implementation details...
  }
}

// Usage Example
const agent = new DCAPAgentV3('agent-alice', [
  {
    agentId: 789,
    agentRegistry: 'eip155:1:0xabcd...',
    verification_url: 'https://etherscan.io/nft/0xabcd.../789'
  }
]);

// Wait for discovery
await new Promise(resolve => setTimeout(resolve, 2000));

// Invoke tool - automatically broadcasts usage_receipt
try {
  const result = await agent.invokeTool('financial_advisor', {
    portfolio: { stocks: 0.6, bonds: 0.4 },
    risk_tolerance: 'medium'
  });
  console.log('Result:', result);
} catch (error) {
  console.error('Tool invocation failed:', error);
}
```

## Appendix B. Message Size Optimization

Example of handling UDP size constraints:

```javascript
function truncateMessage(message) {
  let json = JSON.stringify(message);
  
  if (json.length <= 1400) {
    return json;
  }
  
  // Priority 1: Remove ctx
  if (message.ctx) {
    delete message.ctx;
    json = JSON.stringify(message);
    if (json.length <= 1400) return json;
  }
  
  // Priority 2: Remove blockchain_registrations
  if (message.blockchain_registrations) {
    delete message.blockchain_registrations;
    json = JSON.stringify(message);
    if (json.length <= 1400) return json;
  }
  
  // Priority 3: Remove connector.session
  if (message.connector?.session) {
    delete message.connector.session;
    json = JSON.stringify(message);
    if (json.length <= 1400) return json;
  }
  
  // Continue with other optimizations...
  
  return json;
}
```

## Authors' Addresses

Martin Maurer  
Email: empeamtk@googlemail.com

