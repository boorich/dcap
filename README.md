# Dynamic Capability Acquisition Protocol (DCAP)

> **"The robots.txt for AI agents"**

DCAP is a decentralized protocol enabling autonomous agents to discover, evaluate, and acquire computational capabilities at runtime through semantic broadcasting.

## 🎯 Quick Start

**Latest Specification:** [DCAP.md](./DCAP.md) - **Version 2.4** (October 2025)

**For automation/parsing:** Always fetch `DCAP.md` - it's a stable URL that always points to the latest version.

---

## 📋 Current Version: 2.4

### Key Features

✅ **Format-agnostic agent identification** (`ctx.caller`)  
✅ **Independent payment tracking** (`ctx.payer`)  
✅ **Collision-tolerant session aggregation**  
✅ **Opt-in participation model** (robots.txt philosophy)  
✅ **No validation or constraints** on identifier formats  

### What's New in v2.4

**Added `ctx.caller` field:**
```json
{
  "ctx": {
    "caller": "any-string-identifier",  // Agent self-ID
    "payer": "0x123abc..."              // Payment address (independent)
  }
}
```

**Key Points:**
- `ctx.caller`: Any string chosen by agent (UUID, pseudonym, emoji, whatever!)
- `ctx.payer`: Payment address from transaction protocol
- Both optional and independent
- Session aggregation: prefer `caller`, fallback to `payer`
- Embraces emergent intelligence without constraining agents

**Full changelog:** See [archive/v2.4.md](./archive/v2.4.md)

---

## 📚 Version History

All historical versions are archived for reference:

- **[v2.4](./archive/v2.4.md)** (October 2025) - Format-agnostic agent identification
- **[v2.3](./archive/v2.3.md)** (October 2025) - Payment attribution with `ctx.payer`
- **[v2.2](./archive/v2.2.md)** (October 2025) - Cost tracking and economic efficiency
- **[v2.1](./archive/v2.1.md)** (October 2025) - Chain pattern detection with `ctx.args`
- **[v2.0](./archive/v2.0.md)** (September 2025) - Initial public specification

---

## 🚀 Implementation Guide

### For Tool Providers

**Recommended:** Implement [DCAP.md](./DCAP.md) (v2.4)

```javascript
// Minimal implementation
{
  "v": 2,
  "t": "perf_update",
  "sid": "your-tool-id",
  "tool": "read_file",
  "exec_ms": 45,
  "success": true
}

// With optional agent tracking (enables chain detection)
{
  "v": 2,
  "t": "perf_update",
  "sid": "your-tool-id",
  "tool": "read_file",
  "exec_ms": 45,
  "success": true,
  "ctx": {
    "caller": "agent-uuid-123"  // Any string works!
  }
}
```

### For Agent Consumers

**Recommended:** Implement [DCAP.md](./DCAP.md) (v2.4)

- Include `ctx.caller` in your tool invocations for better recommendations
- Use any identifier you want - format is unconstrained
- Opt-in: omitting `ctx.caller` still works, just no chain aggregation

### For Intelligence Systems

**Required:** Implement [DCAP.md](./DCAP.md) (v2.4)

```javascript
// Session aggregation logic
const sessionId = ctx?.caller || ctx?.payer;
if (sessionId) {
  // Group perf_updates by session
  // Build observed_sequence messages
}
```

---

## 🎓 Philosophy

DCAP follows the **robots.txt model**:

- **Opt-in participation**: All fields beyond basics are optional
- **No gatekeeping**: No validation, no registration, no approval
- **Format-agnostic**: Accept any string, embrace emergence
- **Collision-tolerant**: Let timestamps and context disambiguate
- **Evolution over perfection**: Watch what happens, adapt

**"We don't validate. We don't constrain. We observe and adapt."**

---

## 📦 Version Selection

| Use Case | Version |
|----------|---------|
| New implementations | [DCAP.md](./DCAP.md) (v2.4) |
| Agent identification | v2.4+ (required) |
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

**Built with the spirit of robots.txt - simple, voluntary, powerful.**
