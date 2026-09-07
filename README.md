# Faster File Copy

Decompressed-files capture/delivery and memory mapping for Skyrim and Fallout 4.
The original Skyrim and Fallout 4 SKSE plugins live in [`Skyrim/`](Skyrim/) and
[`Fallout4/`](Fallout4/) and are unchanged from upstream.

> **OpenMW port:** this fork also carries an OpenMW-native version of the memory-mapped
> uncompressed-BSA path (serve BSA entries from a demand-paged read-only mapping instead of
> buffered reads, to cut asset-load latency). Morrowind BSAs are uncompressed, so the
> decompression-cache half doesn't apply and is intentionally not ported. See
> [`openmw/`](openmw/README.md).

Credit for the original: **1001Bits** (https://github.com/1001Bits/Faster-File-Copy).
