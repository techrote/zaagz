# zaagz spec (working draft)

Status: draft, decisions made in conversation, not yet implemented.

## Purpose

zaagz is a companion to Zig. It exists to do what Zig forbids, through an explicit bridge, with every bypass traced.

## Bridge

- Primary target: C ABI, so zaagz works wherever Zig does.
- Second target: WebAssembly, so the same contract runs in browsers.
- Zig owns memory and the build. zaagz is a JIT layer that Zig loads.

## `nsafe`

- `nsafe` replaces Zig's `unsafe` keyword.
- Each `nsafe` is an explicit handoff: zaagz executes the operation.
- Zig does not check what zaagz returns. It trusts the result.
- Dangerous behaviour is never implicit. The source must say it.

## Corruption repair

Corruption is proven by one of:

1. A checksum mismatch.
2. A declared invariant failing.
3. A fault reported by zaagz's own JIT.

When corruption is proven and the `nsafe` form carries the repair opt-in marker, zaagz writes PRNG output into the affected addresses. Without the marker, nothing is written.

## PRNG sources

The PRNG source is pluggable. Examples, not all supported at first:

- Seeded PRNG (default)
- SHA-256 of the previously executed line, per millisecond awake
- ESP32 serial COM analog noise
- `/dev/random`
- CPU temperature jitter

Every repair logs the source ID and the bytes written. Deterministic mode pins the source to a seed.

## `xhaos`

`xhaos` deliberately corrupts bits in flows and outputs, as an opt-in per module.

- **Scope:** targeted. Each module declares its own `xhaos` setting in source. Nothing is chaotic by default.
- **Ratio:** bits corrupted per N bytes, e.g. 1/1, 1/2, 1/3. Default is subtle, 1/8.
- **Sampler:** pluggable pattern choosing which bits are corrupted. Fixed strides are the first built-in.
- **Testing allowlist:** during testing, `xhaos` may only target LUT colour values. Any other target is a compile error. Widening the allowlist is a deliberate code change.
- **Reproducibility:** each sampler's output is logged, so a trace replays the same corruption.

## Trace

Every `nsafe` call, repair, PRNG draw and `xhaos` selection is logged. A trace is enough to replay a run.

## Open questions

- Whether enabling `xhaos` needs a build-time gate as well as the module declaration.
- Licence.
