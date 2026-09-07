# Faster File Copy for OpenMW

This is the **OpenMW port** of Faster File Copy.

The original is a Skyrim SKSE plugin with two optimizations for asset-load
latency:

1. **Uncompressed BSA entries** copied from a read-only, demand-paged archive
   memory map instead of the engine's buffered `ReadFile` chain.
2. A **decompression cache** for compressed (zlib/LZ4) BSA entries, warmed into
   RAM so later requests skip decompression and storage I/O.

OpenMW is a separate engine, so the SKSE hook doesn't apply. This port
implements the **memory-mapped uncompressed path (#1) natively** inside
OpenMW's BSA reader.

## Scope: why only the memory-map half

Morrowind's BSA format is **uncompressed** — entries are stored raw. So the
decompression-cache half (#2) has nothing to decompress for the Morrowind
content this engine targets, and is intentionally not ported (it would be dead
code). The memory-map half is the applicable, useful optimization here.

## What it does

`Bsa::BSAFile::getFile` normally opens a buffered constrained file stream for
each requested asset. When the port is enabled, `BSAFile::open` memory-maps the
whole archive read-only (`boost::iostreams::mapped_file_source`) and `getFile`
returns a stream **over the demand-paged mapping** — no per-read syscall, no
intermediate copy. The OS pages the archive in on demand and keeps hot pages
resident, cutting per-asset read latency. This matters most for the many small
mesh/texture/sound reads during cell transitions and save loads.

Loose files are left alone (already served by the OS file cache), matching the
original.

## Setting (`[General]` in `settings.cfg`)

- **`bsa memory mapping`** — `true` to serve BSA entries from a memory map;
  `false` (default) uses buffered reads.

Falls back to buffered reads automatically if a mapping can't be created (e.g. a
path not representable in the native narrow encoding), so it is safe to leave on.
Standalone tools (bsatool) never map. 64-bit builds are the meaningful target.

## Results

Verified in-game on a large Morrowind + Tamriel Rebuilt modlist: all 11 BSA
archives (Morrowind/Tribunal/Bloodmoon/TR_Data + mod archives) memory-mapped
with 0 failures; cells load with no asset errors.

## Where the code lives

Integrated into the OpenMW engine. Full source on the fork:

- **Repo:** https://github.com/sunhawken/openmw
- **Branch:** `official-jolt-ragdoll`
- **Commit:** `d41f8b911b` — "Add memory-mapped BSA reads to cut asset-load latency (Faster-File-Copy port)"

Touched files:

- `components/bsa/bsafile.{hpp,cpp}` — the memory map + mapped-stream read path
- `apps/openmw/engine.cpp` — sets the flag from the setting before archives open
- `components/settings/categories/general.hpp` + `files/settings-default.cfg` — the setting

## Applying the patch

[`openmw-bsa-memory-mapping.patch`](openmw-bsa-memory-mapping.patch) is the
change, exported from the OpenMW fork. From an OpenMW source checkout:

```sh
git apply openmw/openmw-bsa-memory-mapping.patch
```

Then build OpenMW and set `bsa memory mapping = true` in `settings.cfg`.

## Credit

Concept and original Skyrim/Fallout 4 implementation: **1001Bits** —
https://github.com/1001Bits/Faster-File-Copy
