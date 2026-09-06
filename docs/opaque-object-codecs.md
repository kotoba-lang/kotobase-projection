# Opaque object CID migration

New concatenated projection packs and encrypted stored blocks use raw CIDv1
(`0x55`). Independently encoded logical row blocks and query bundles remain
DAG-CBOR (`0x71`). The pack is a concatenation with bundle-provided offsets and
lengths; it is not a CAR archive. CAR framing belongs to `io-ipld-car`.

This changes newly produced pack CIDs, encrypted stored CIDs, and bundle CIDs
that link to them. Logical plaintext row-block identities remain unchanged.
The query-bundle field grammar stays at version 1: links already carry their
codec, and readers locate the named object and verify logical blocks. Existing
objects and bundles are not rewritten. The in-memory reader remains compatible
with historical DAG-CBOR-labelled pack addresses. External stores must treat
pack links as opaque object addresses rather than decode them as metadata.
The inspected `kotobase-peer` object-store paths use CID as an opaque key; its
materialized-view CLJS suite passed against this implementation (17 tests / 77
assertions). Other deployable consumers must verify their resolved pins and
object handling during rollout. Merge is not evidence of live deployment.

`stored-cid` identifies ciphertext, while `cid` identifies plaintext. This patch
corrects writing their identities; it does not add ciphertext-CID verification
to decryption callbacks. Plaintext verification remains in `decode-range`.

Arrow IPC integration is follow-up work: define a versioned physical-layout
descriptor and decoder negotiation before writing Arrow blocks. Keep IPC bytes
raw and preserve column buffers through execution. Existing Bloom, min/max,
offset/length, coalescing, and delta-chain semantics must retain their behavior.
Current range upper bounds are inclusive; an OrderedMap half-open API needs an
explicit adapter.

See [the accepted cross-repository ADR](https://github.com/com-junkawasaki/root/blob/main/90-docs/adr/2609060000-ipld-adl-selector-car-boundaries.edn).
