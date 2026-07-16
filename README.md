# NeuroRoute C

**A deterministic C11 routing and associative matching core for AI systems research.**

NeuroRoute C explores a simple question: how can neural-inspired execution graphs be implemented as transparent, reproducible systems code rather than probabilistic model behavior?

** ARM64 support

NeuroRoute C uses portable C11 and includes ARM64 static-library builds. The routing and Hamming-distance logic is unchanged across architectures. ARM64 library, test, and demo sources were cross-compiled with strict Clang warnings; native users can run `make test` and `make demo` on ARM64 hardware for full runtime validation.


The project combines two compact mechanisms:

1. **Weighted graph routing** that selects the strongest outbound path from a node, records route usage atomically, and preserves a deterministic fallback.
2. **256-bit spatial matching** that compares binary feature vectors and returns the configured entry with the smallest Hamming distance.

The result is a small, dependency-free research prototype that can be inspected, tested, embedded, and extended without a model server or external service.

## Why it is interesting

Modern AI systems often need more than model inference. They also need predictable routing, memory lookup, fallback behavior, execution counters, and components that can be verified independently.

NeuroRoute C provides a low-level experimental base for studying those ideas in ordinary C:

- deterministic decisions instead of sampled outputs;
- inspectable graph topology instead of hidden model state;
- binary fingerprint matching instead of text-only retrieval;
- explicit fallback behavior;
- reproducible tests and sanitizer validation;
- no third-party runtime dependencies.

It is not a neural network and it does not train or generate content. The design is **neural-inspired**, while the execution remains conventional, deterministic systems programming.

## Core mechanisms

### 1. Maximum-weight graph routing

Each node contains a local fallback identifier and zero or more outbound roads. A road contains a target identifier, a weight, a usage counter, and a link to the next road.

When `route_synaptic_signal()` runs:

1. Excitation below `NEURO_ACTIVATION_THRESHOLD` returns `0`.
2. The selected node's roads are scanned in insertion order.
3. The road with the greatest weight is selected.
4. The first road wins if multiple roads share the same maximum weight.
5. The selected road's usage counter is incremented atomically.
6. A node with no roads returns its configured fallback identifier.

This creates predictable route selection while retaining counters that can support later analysis or controlled adaptation.

### 2. Hamming-distance spatial matching

Each spatial entry stores four 64-bit coordinate words, forming a 256-bit binary feature vector.

`allocate_spatial_memory()` compares an input vector with every configured entry and returns the page identifier associated with the smallest Hamming distance. Equal-distance matches are resolved deterministically by selecting the first configured entry.

The historical function name is retained, but the function does not allocate operating-system memory. It performs nearest-match lookup over a fixed in-memory table.

## Potential uses

NeuroRoute C can serve as a starting point for experiments involving:

- deterministic agent or tool routing;
- policy-controlled handler selection;
- fallback routing in AI pipelines;
- binary request fingerprint matching;
- associative lookup prototypes;
- execution-graph and topology research;
- route-frequency instrumentation;
- embedded or edge-system dispatch;
- reproducible comparisons between deterministic and model-driven decisions;
- teaching low-level graph structures, atomics, Hamming distance, and C testing.

These are possible directions, not built-in claims. Integration logic is still required for a specific application.

## Repository structure

```text
.
|-- include/
|   |-- asm/
|   |   `-- primitives.h
|   |-- mm/
|   |   `-- spatial_pool.h
|   `-- neuro.h
|-- src/
|   |-- routing.c
|   `-- spatial_pool.c
|-- tests/
|   |-- test_routing.c
|   `-- test_spatial_pool.c
|-- examples/
|   `-- demo.c
|-- Makefile
|-- LICENSE
`-- README.md
```

## Requirements

- GCC or Clang with C11 support
- `make`
- A Unix-like shell only when using the optional builder script

No Python environment, external API, model provider, database, network connection, or third-party C library is required.

## Build

```bash
make
```

This creates:

```text
lib/libneuro_router.a
bin/test_routing
bin/test_spatial_pool
bin/neuro_demo
```

## Run the tests

```bash
make test
```

Expected output:

```text
routing tests: PASS
spatial pool tests: PASS
```

## Run the CLI demonstration

```bash
make demo
```

Expected output:

```text
Selected route target: 1002
Selected spatial page: 8192
```

## Run sanitizer validation

```bash
make sanitize
```

This rebuilds and runs the tests and demonstration with AddressSanitizer and UndefinedBehaviorSanitizer.

## Integrate the library

Include the library headers from another C project:

```c
#include <neuro.h>
#include <mm/spatial_pool.h>
```

Then link against:

```text
lib/libneuro_router.a
```

The header files expose functions for reset and initialization, node configuration, edge insertion, deterministic route selection, spatial-entry configuration, nearest-match lookup, and state inspection.

## Design guarantees

The current implementation preserves these deterministic rules:

- the greatest route weight wins;
- the first route wins a weight tie;
- the first spatial entry wins a distance tie;
- low excitation returns zero;
- an empty node returns its fallback identifier;
- selected-route counters are incremented atomically;
- node and spatial-entry indices are bounds-checked;
- the spatial density counter is applied to the entry that was actually selected.

The constants `ALPHA_NORMALIZER` and `BETA_EXPONENT` are retained from the original prototype, but the present routing function does not apply a scaling equation.

## Project scope

NeuroRoute C is a compact research and systems-programming prototype. It does not train a model, discover application functions automatically, execute stored numeric identifiers as machine code, replace a vector database, or modify itself recursively.

Its purpose is to make deterministic routing and binary spatial matching concrete, inspectable, and easy to experiment with.

## License

MIT License. See `LICENSE`.

