# SPEC.md Examples

A growing collection of real SPEC.md files written for actual small projects, showing the format applied in practice.

This is separate from the versioned walkthrough example (the Judo Club Website, specified at each stage of the standard's own evolution), which stays in [`SPECmd-app/SPEC.md`](https://github.com/SPECmd-app/SPEC.md/tree/main/examples) alongside the standard itself. The examples here are independent, real specifications for their own projects — not tied to the standard's release cycle.

## Examples

- [`spec-md-website/SPEC.md`](spec-md-website/SPEC.md) — the SPEC.md for the SPEC.md project's own companion website (a static GitHub Pages site): what its homepage must link to, its machine-readable discovery files, and its "no outside dependency" requirement.
- [`specmd-cli/SPEC.md`](specmd-cli/SPEC.md) + [`specmd-cli/TRACE.md`](specmd-cli/TRACE.md) — the SPEC.md and paired TRACE.md for [`SPECmd-app/specmd-cli`](https://github.com/SPECmd-app/specmd-cli), a Core+Optional example with `optional_features.trace: true`. This is a snapshot copy; the four Normative Modules it declares live only in the `specmd-cli` repository, so links to them won't resolve here — see that repository for the complete, resolvable Specification Set.
- [`internal-it-ticketing-system/SPEC.md`](internal-it-ticketing-system/SPEC.md) — a Core-only example showing the Living Specification principle in practice: an internal IT service desk with only four decisions made so far (service-desk type, SSO scope for Agents, single-organization tenancy, intake channels), everything else explicitly left as `TBD` / Open Issues rather than invented.

## License

[Apache-2.0](LICENSE)
