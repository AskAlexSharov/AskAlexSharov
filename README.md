<div align="center">

# Alex Sharov

**Go Tech Lead** — databases, storage engines, and the efficiency frontier

[![Erigon](https://img.shields.io/badge/Erigon-core%20contributor-2ea44f?style=flat-square&logo=ethereum&logoColor=white)](https://github.com/erigontech/erigon)
[![Blog](https://img.shields.io/badge/Blog-askalexsharov.github.io-1f6feb?style=flat-square&logo=hugo&logoColor=white)](https://askalexsharov.github.io/)
[![X](https://img.shields.io/badge/X-@AskAlexSharov-000000?style=flat-square&logo=x&logoColor=white)](https://twitter.com/AskAlexSharov)
![Location](https://img.shields.io/badge/Vietnam-lightgrey?style=flat-square)

</div>

---

I work on the storage layer of **[Erigon](https://github.com/erigontech/erigon)** — an Ethereum
implementation on the efficiency frontier. My commits mostly live below the application: B-tree page
geometry, minimal perfect hashing, dictionary compression, external sorting, and a long-running war
against the Go garbage collector.

The recurring question underneath all of it: **a multi-terabyte database should fit on an ordinary
disk and stay fast.** Most of my work is finding the layer where that quietly stops being true — a
page size, a freelist scan, a `[]any` cast — and fixing it there.

## 📈 Historical activity

Every PR I authored or reviewed across `erigontech` + `ledgerwatch`, 2019 → today:

```
2019  ▎                                              20
2020  ███████                                       412
2021  █████████████████▏                          1,008
2022  ██████████████████████████▋                 1,563
2023  ███████████████████▍                        1,133
2024  ███████████████████████████████▋            1,853
2025  ██████████████████████████████████████████  2,456
2026  ████████████████████████████▏               1,648  (to Jul)
```

|            |                                                |
| ---------: | :--------------------------------------------- |
| **10,093** | unique PRs                                     |
|  **5,651** | authored — **5,153 merged** (91%)              |
|  **4,555** | reviewed — mostly my team's work and my ideas   |

## 🧠 Where I go deep

| Area | Representative work |
| ---- | ------------------- |
| **Storage engines** — MDBX / LMDB internals | [Page size caps your database at ~8 TB](https://github.com/erigontech/erigon/pull/3323) · [skip freelist search for large page sequences](https://github.com/erigontech/erigon/pull/1213) · [commit prune batches so freed pages return to the freelist](https://github.com/erigontech/erigon/pull/9781) |
| **Memory & the Go GC** | [Off-heap Elias-Fano build for 3B keys](https://github.com/erigontech/erigon/pull/20640) · [killing the `[]any` cast behind a stop-the-world spike](https://github.com/erigontech/erigon/pull/21858) · [arena-allocated patricia `MatchFinder`](https://github.com/erigontech/erigon/pull/20136) · [selective `mmap` + `madv_will_need` edging out all-in-RAM](https://github.com/erigontech/erigon/pull/21792) |
| **Compression & encoding** | [Condensed Huffman pattern tables](https://github.com/erigontech/erigon/pull/5130) · [C SA-IS → Go's `index/suffixarray`](https://github.com/erigontech/erigon/pull/19191) · [page-level history compression](https://github.com/erigontech/erigon/pull/14780) · [key-prefix compression in `bolt`](https://github.com/ledgerwatch/bolt/pull/2) |
| **Indexing & perfect hashing** | [Parallel RecSplit](https://github.com/erigontech/erigon/pull/19538) · [fuse filters inside `.efi`/`.kvi`](https://github.com/erigontech/erigon/pull/15960) · [roaring bitmap log indices](https://github.com/erigontech/erigon/pull/1124) · [when a flat array beats a bitmap](https://github.com/erigontech/erigon/pull/19995) |
| **Sorting & pipelines** | [ETL "oldest appeared" correctness under buffer flush](https://github.com/erigontech/erigon/pull/2352) · [zero-copy `memDataProvider`](https://github.com/erigontech/erigon/pull/19780) · [`PagedWriter`: fixing the producer bottleneck](https://github.com/erigontech/erigon/pull/19944) |
| **Distribution at TB scale** | [`--webseeds`: BitTorrent + HTTP for multi-TB snapshots](https://github.com/erigontech/erigon/pull/8176) · [fetch `.torrent` from a provider instead of bundling](https://github.com/erigontech/erigon/pull/8346) |
| **State model & interfaces** | [txNum: indexing chain state by transaction number](https://github.com/erigontech/erigon/pull/5176) · [gRPC remote KV](https://github.com/erigontech/erigon/pull/788) · [array-based EVM interpreter stack](https://github.com/erigontech/erigon/pull/20351) |

## ⚡ Measured wins

Changes that shipped with before/after numbers:

| Change | Result |
| ------ | ------ |
| [Interpolation search over Elias-Fano `Seek`](https://github.com/erigontech/erigon/pull/19788) | `354 ns → 47 ns` — **7.5×** at n=100k |
| [4 KB MDBX page size](https://github.com/erigontech/erigon/pull/21553) | **1.7× faster flush** on Bloatnet |
| [Decompressor buffer reuse](https://github.com/erigontech/erigon/pull/17542) | `3,000 allocs/op → 0`, ~10% faster |
| [k-way merge in `HistoryRange`](https://github.com/erigontech/erigon/pull/19605) | **−19% allocations**; command runtime `4m58s → 4m5s` |
| [Torrent slot tuning](https://github.com/erigontech/erigon/pull/14903) | `sys` memory **20 GB → 8 GB**, throughput held at ~1 Gb/s |

## 🧭 The arc

Seven years, three storage engines, one database that kept outgrowing its disk:

```
2019  bolt          key-prefix compression in Erigon's original engine
2020  lmdb-go       freelist reuse, mmap ceilings, roaring bitmap log indices
2021  mdbx-go       the move to MDBX — page geometry, dirty-space control
2022  erigon-lib    parallel dictionary compressor, .seg format, the txNum state model
2023  snapshots     multi-TB distribution over BitTorrent + web seeds
2024  domains       commitment, .kvi indices, decompression caches
2025  filters       fuse filters baked into index files, page-level compression
2026  off-heap      Elias-Fano off the heap, arena allocation, array-based EVM stack
```

<div align="center">

<a href="https://commit-history.com/AskAlexSharov">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://commit-history.com/embed/AskAlexSharov?theme=dark" />
    <img alt="AskAlexSharov's commit history" src="https://commit-history.com/embed/AskAlexSharov" />
  </picture>
</a>

<br />
<br />

**Ask me about** storage engines · B-trees · perfect hashing · compression · Go performance

[Blog](https://askalexsharov.github.io/) · [X](https://twitter.com/AskAlexSharov) · [AskAlexSharov@gmail.com](mailto:AskAlexSharov@gmail.com)

</div>
