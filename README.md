# siox-axi

The AMBA AXI4 family interfaces for [siox](https://github.com/Siox-lang/sioxc). Each
module has one width-generic struct with every signal and one view per
endpoint role.

| Module | Interface | Struct | Views |
|---|---|---|---|
| `axi::lite` | AXI4-Lite | `AxiLite<A, D>` | `Manager` / `Subordinate` |
| `axi::full` | AXI4 | `Axi4<I, A, D>` | `Manager` / `Subordinate` |
| `axi::stream` | AXI4-Stream | `AxiStream<D>` | `Transmitter` / `Receiver` |

The clock and reset are not part of the structs; they stay ordinary ports of
the endpoint entities. Each module's header says what is left out.

## Use

Until siox has a package manager, put the `axi/` directory at your project
root (copy, symlink or submodule) and import from it:

```siox
use axi::lite::{AxiLite, Subordinate};

entity Peripheral { clk: Bit in, bus: AxiLite<32, 32> Subordinate }
```

A parent connects a `Manager` and a `Subordinate` through one `let wire: AxiLite<32, 32>;`.

## Test

Each `<module>_test.siox` drives a small endpoint from a testbench:

```bash
sioxc --std <sioxc>/std --test lite_test.siox -o lite_test && ./lite_test
```

Needs a `sioxc` with Siox-lang/sioxc#68: width-parameterised struct ports.
