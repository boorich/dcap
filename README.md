# DCAP Specification Repository

This repository contains all versions of the Dynamic Capability Acquisition Protocol (DCAP) specification.

## Current Version

**Latest:** [DCAP.md](./DCAP.md) - Version 2.2 (October 2025)

## Version History

### [Version 2.2](./DCAP-v2.2.md) - October 2025
**Status:** Current

**Key Changes:**
- Added `cost_paid` field to `perf_update` for economic tracking
- Added `currency` field to `perf_update` for multi-currency support
- Economic efficiency optimization guidelines (Section 5.4)
- Cost data transparency considerations (Section 6.3.2)
- Updated tool ranking to include cost optimization

**Migration from v2.1:**
- Fully backward compatible
- Cost fields are optional
- Existing v2.1 implementations continue to work

**Rationale:**
- Performance optimization includes both speed AND cost
- Oracle agents need cost data to recommend economically efficient tool chains
- Transparency enables market efficiency and price discovery

---

### [Version 2.1](./DCAP-v2.1.md) - October 2025
**Status:** Superseded

**Key Changes:**
- Enhanced `perf_update` message with optional `ctx` object
- Support for sanitized `ctx.args` in performance updates
- Chain pattern detection guidelines (Section 5.3)
- Argument sanitization requirements (Section 6.3.1)
- Intelligence consumer requirements (Section 7.4)

**Migration from v2.0:**
- Fully backward compatible
- All new fields are optional
- Existing v2.0 implementations continue to work

---

### [Version 2.0](./DCAP-v2.0.md) - September 2025
**Status:** Superseded

**Initial Features:**
- UDP broadcast for capability advertisement
- WebSocket distribution layer
- Three message types: `semantic_discover`, `perf_update`, `error_pattern`
- Semantic matching algorithms
- Security and privacy considerations

---

## Implementation Guide

### For Tool Providers

**New implementations:** Use [Version 2.1](./DCAP-v2.1.md)
- Implement `perf_update` with `ctx.args` for better observability
- Follow argument sanitization guidelines (Section 6.3.1)

**Existing v2.0 implementations:** 
- Continue working without changes
- Upgrade recommended for chain detection support

### For Agent Consumers

**New implementations:** Use [Version 2.1](./DCAP-v2.1.md)
- Parse `ctx.args` from `perf_update` messages
- Implement local performance tracking

**Existing v2.0 implementations:**
- Continue working without changes
- Gracefully ignore unknown fields in `ctx`

### For Intelligence Systems (Oracle Agents)

**Use [Version 2.1](./DCAP-v2.1.md):**
- Track `perf_update` sequences by `sid` (Section 5.3)
- Build statistical models of tool chains
- Respect privacy of observed `ctx.args`

---

## Version Selection Guide

| Use Case | Recommended Version |
|----------|---------------------|
| New tool provider | v2.2 |
| New agent consumer | v2.2 |
| Intelligence/Oracle system | v2.2 (required for cost optimization) |
| Cost-aware applications | v2.2 (required) |
| Existing v2.1 system | v2.1 (works fine, upgrade for cost tracking) |
| Existing stable system | v2.0 (works fine, missing features) |
| Maximum compatibility | v2.0 |

---

## Contributing

To propose changes to the specification:
1. Create a new version file (e.g., `DCAP-v2.2.md`)
2. Document all changes in the Changelog section
3. Update this README with version details
4. Submit for review

---

## License

This specification is provided as-is for implementation by any party.

## Contact

Martin Maurer  
Email: empeamtk@googlemail.com

