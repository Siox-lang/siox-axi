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

## Components

| Entity | What it does |
|---|---|
| `axi::full::Interconnect<I, A, D, M, N, REGION>` | `M` Managers to `N` Subordinates. Writes and reads each carry one transaction at a time on independent paths, granted round-robin, with bursts of any length. IDs pass through unchanged. Unmapped addresses are answered by a built-in default subordinate: drained writes get `DECERR`, reads get every beat as `DECERR`. |
| `axi::lite::Interconnect<A, D, N, REGION>` | One Manager to `N` Subordinates. Subordinate `i` owns the `2^REGION` bytes at `i << REGION`; anything past the last one gets `DECERR`. Writes and reads route independently, one transaction each at a time. |
| `axi::stream::RegisterSlice<D>` | Full-throughput skid buffer: registers every output, `tready` included. |
| `axi::stream::Fifo<D, DEPTH>` | Synchronous first-word-fall-through FIFO. |

```siox
use axi::lite::{AxiLite, Subordinate, Interconnect};

entity Soc { clk: Bit in, cpu: AxiLite<32, 32> Subordinate }
impl Soc {
    let peripherals: AxiLite<32, 32>[2];
    let bus: Interconnect<32, 32, 2, 12> = { .clk = clk, .up = cpu, .down = peripherals };
    let uart: Uart = { .clk = clk, .bus = peripherals[0] };   // 0x0000
    let gpio: Gpio = { .clk = clk, .bus = peripherals[1] };   // 0x1000
}
```

## References

Behaviour follows the official sources:

- Arm, *AMBA AXI Protocol Specification* (IHI 0022): handshakes, bursts, and
  the default subordinate's `DECERR`, under which every transfer of a
  transaction still completes.
- AMD, *AXI Interconnect Product Guide* (PG059): single-issuing (Shared Access
  mode), separate write and read arbiters, and round-robin among
  equal-priority requesters.

## Test

Each `*_test.siox` drives an endpoint or component from a testbench:

```bash
sioxc --std <sioxc>/std --test lite_test.siox -o lite_test && ./lite_test
```

Needs a `sioxc` with Siox-lang/sioxc#69: arrays of view ports.
