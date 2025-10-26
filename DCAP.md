# Dynamic Capability Acquisition Protocol (DCAP)

**Version**: 2.4 (Latest)  
**Date**: October 2025  
**Author**: M. Maurer  

> **Note:** This file always contains the latest version of the DCAP specification.  
> For specific versions, see [DCAP-v2.4.md](./DCAP-v2.4.md), [DCAP-v2.3.md](./DCAP-v2.3.md), [DCAP-v2.2.md](./DCAP-v2.2.md), [DCAP-v2.1.md](./DCAP-v2.1.md), [DCAP-v2.0.md](./DCAP-v2.0.md)

---

**IMPORTANT:** This specification follows the robots.txt philosophy - all optional fields enable additional features through voluntary participation. Session identifiers (`ctx.caller`, `ctx.payer`) are format-agnostic: any string works. The protocol does not validate or constrain identifier formats, embracing emergent intelligence patterns.

---

For the complete specification, see [DCAP-v2.4.md](./DCAP-v2.4.md).

## Quick Reference: Version 2.4 Changes

### New Field: `ctx.caller`
```json
{
  "ctx": {
    "caller": "any-string-identifier",  // v2.4: Agent self-identification
    "payer": "0x123abc..."              // v2.3: Payment address (independent)
  }
}
```

### Key Points:
- **`ctx.caller`**: Any string chosen by agent (UUID, pseudonym, etc.) - enables chain aggregation
- **`ctx.payer`**: Payment address from transaction protocol - tracks economic activity
- **Both are optional and independent** - serve different purposes
- **Format-agnostic**: Relay accepts any string, no validation
- **Opt-in model**: Omitting both fields = no aggregation (but tool still works)

### Session Aggregation Logic:
```javascript
const sessionId = ctx?.caller || ctx?.payer;  // Prefer caller, fallback to payer
if (sessionId) {
  // Group this perf_update with others from same session
}
```

---

For full specification including examples, security considerations, and implementation guidelines, see [DCAP-v2.4.md](./DCAP-v2.4.md).
