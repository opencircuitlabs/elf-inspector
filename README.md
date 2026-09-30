# ELF Inspector

A dependency-free ELF32/ELF64 parser and firmware memory inspector for embedded build pipelines.

It parses headers, load segments, sections, string tables, static/dynamic symbols, bindings, visibility, and architecture metadata in little- or big-endian files. Higher-level APIs map runtime addresses, extract sections, identify symbols, calculate flash/RAM/BSS/debug usage, detect segment overlap, and compare builds.

## Quick start

```bash
npm install
npm test
node src/cli.js info build/firmware.elf
node src/cli.js sections build/firmware.elf
node src/cli.js symbols build/firmware.elf
node src/cli.js extract build/firmware.elf .text text.bin
node src/cli.js json build/firmware.elf > firmware.json
```

```js
import {
  parseElf, analyzeElf, findSymbol,
  addressToFileOffset, extractSection, compareElf
} from '@opencircuitlabs/elf-inspector';

const elf = parseElf(fileBytes);
const usage = analyzeElf(elf);
const symbol = findSymbol(elf, programCounter);
const offset = addressToFileOffset(elf, programCounter);
```

## Supported structures

- ELF32 and ELF64;
- little- and big-endian encoding;
- 32-bit addresses and safe-range 64-bit addresses;
- program headers and load-segment permissions;
- common section types and allocation/write/execute flags;
- section-name and symbol string tables;
- `SYMTAB` and `DYNSYM` entries for 32- and 64-bit files;
- symbol binding, type, visibility, size, address, and section association.

Every offset and size is checked against file bounds before slicing. Unsafe 64-bit offsets are rejected rather than rounded. Extended ELF section numbering is reported as unsupported instead of being misinterpreted.

## Memory accounting

Analysis reports file-backed allocated bytes as flash, writable allocated bytes as runtime RAM, `NOBITS` writable bytes as BSS, and `.debug*` sections separately. Initialized writable sections therefore contribute to both flash and RAM, matching embedded deployment.

Load segment ranges are checked for virtual-memory overlap. Runtime addresses map to file offsets through `PT_LOAD` segments first and allocated sections second; BSS correctly has no file offset.

## Build comparison

`compareElf` reports added, removed, and resized sections plus added or changed symbols, including size and address deltas. It also flags entry-point changes. This supports firmware-size reviews without requiring a platform toolchain on the analysis host.

## License

MIT
