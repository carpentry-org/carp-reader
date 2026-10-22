# Reader benchmarks

Build with the reference Carp compiler and pass a source file:

```sh
carp --optimize -b benchmark/reader.carp
./out/Untitled carp-reader.carp
./out/Untitled carp-reader.carp syntax
```

The program reads the file before timing, then performs 100 reads, including
destruction of their results. The default mode uses `Reader.parse`; `syntax`
uses `Reader.read-syntax`. It prints total nanoseconds and the
accumulated top-level form or syntax-node count. Divide the elapsed time by 100 for the
mean parse time. Run fresh processes several times and compare medians on
identical input with the same compiler and optimization settings.

Use real source as well as flat arrays and nested groups or reader prefixes.
A small flat input alone does not expose repeated subtree copying.

`test/reference-ownership.carp` supplies deterministic copy-count assertions
for the reference compiler. Its observers preserve ordinary copy behavior
and include positive controls. Metacarp emits intrinsic recursive copies
and does not call those observer hooks. The portable lifetime and borrowed
lowering checks live in `test/ownership.carp`.

## Local measurements

Optimized reference Carp on macOS arm64, seven fresh processes with 100 reads
per process, using the same Metacarp REPL engine source at `b3d6cee`. Values
are median milliseconds per read, including result destruction.

| Path | Strict-only parent `b173317` | Initial 0.5.0 implementation `97c8bc1` | 0.5.0 with shared builders |
|---|---:|---:|---:|
| Strict parsing | 1.812 | 9.103 | 1.137 |
| Lossless reading | | 8.074 | 1.529 |

The strict builder was also faster than the strict-only parent on qualified
symbols, a 3,000-element numeric array, 128 nested groups, and 128 nested reader
prefixes. `test/builders.carp` checks forms, all nested source positions, and
errors against lowering an editing document, including incomplete buffers.
