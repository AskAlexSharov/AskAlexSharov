<div align="center">

# Alex Sharov

[X](https://twitter.com/AskAlexSharov) · [AskAlexSharov@gmail.com](mailto:AskAlexSharov@gmail.com)

</div>

---

I work on the storage layer of **[Erigon](https://github.com/erigontech/erigon)**, an Ethereum
implementation. My commits mostly live below the application: B-tree page geometry, minimal perfect
hashing, dictionary compression, external sorting, and a long-running war against the Go garbage
collector.

The recurring question underneath all of it: **a multi-terabyte database should fit on an ordinary
disk and stay fast.** Most of my work is finding the layer where that quietly stops being true — a
page size, a freelist scan, a `[]any` cast — and fixing it there.

<div align="center">

<a href="https://commit-history.com/AskAlexSharov">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://commit-history.com/embed/AskAlexSharov?theme=dark" />
    <img alt="AskAlexSharov's commit history" src="https://commit-history.com/embed/AskAlexSharov" />
  </picture>
</a>

</div>

|            |                                               |
| ---------: | :-------------------------------------------- |
| **10,093** | unique PRs                                    |
|  **5,651** | authored — **5,153 merged** (91%)             |
|  **4,555** | reviewed — mostly my team's work and my ideas  |

## 🗜️ We wrote our own compression algorithm

Erigon doesn't compress its data files with gzip or zstd. We built our own codec — the **`.seg`
format** — because off-the-shelf stream compressors can't do the one thing a node actually needs:
**random access**. You have to jump straight to word *N* of a 40 GB file without touching the 39 GB
in front of it.

- **Dictionary build** — words are concatenated into *superstrings*; a [SA-IS suffix array](https://github.com/erigontech/erigon/tree/main/db/seg/sais) finds every repeated substring, patterns are scored, and the winners are reduced into a bounded dictionary.
- **Optimal parse** — a [patricia trie](https://github.com/erigontech/erigon/tree/main/db/seg/patricia) finds every pattern match inside a word, then a DP picks the cover that compresses best.
- **Huffman** — pattern IDs and positions each get their own condensed Huffman tables.

Dictionary size is a three-way trade between file size, RAM, and read speed. On a 74 GB BSC
transactions file: 1M patterns → `35,871 MB` needing 70 MB of RAM to open; 32K patterns →
`39,626 MB` needing 5 MB, and decompressing faster (`3m00s` vs `4m06s`).

My work on it: [parallel compressor](https://github.com/ledgerwatch/erigon-lib/pull/223) ·
[`ParallelCompressor` / `DecompressedFile`](https://github.com/ledgerwatch/erigon-lib/pull/234) ·
[condensed Huffman pattern tables](https://github.com/erigontech/erigon/pull/5130) ·
[C SA-IS → Go's `index/suffixarray`](https://github.com/erigontech/erigon/pull/19191) ·
[arena-based patricia `MatchFinder`](https://github.com/erigontech/erigon/pull/20136) ·
[page-level compression for history](https://github.com/erigontech/erigon/pull/14780) ·
and, back in 2019, [key-prefix compression in `bolt`](https://github.com/ledgerwatch/bolt/pull/2) —
Erigon's original storage engine.

## 🧠 Where I go deep

| Area | Representative work |
| ---- | ------------------- |
| **Storage engines** — MDBX / LMDB internals | [Page size caps your database at ~8 TB](https://github.com/erigontech/erigon/pull/3323) · [skip freelist search for large page sequences](https://github.com/erigontech/erigon/pull/1213) · [commit prune batches so freed pages return to the freelist](https://github.com/erigontech/erigon/pull/9781) |
| **Memory & the Go GC** | [Off-heap Elias-Fano build for 3B keys](https://github.com/erigontech/erigon/pull/20640) · [killing the `[]any` cast behind a stop-the-world spike](https://github.com/erigontech/erigon/pull/21858) · [arena-allocated patricia `MatchFinder`](https://github.com/erigontech/erigon/pull/20136) · [selective `mmap` + `madv_will_need` edging out all-in-RAM](https://github.com/erigontech/erigon/pull/21792) |
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
| [k-way merge in `HistoryRange`](https://github.com/erigontech/erigon/pull/19605) | **−19% allocations**; command runtime `4m58s → 4m5s` |
| [Torrent slot tuning](https://github.com/erigontech/erigon/pull/14903) | `sys` memory **20 GB → 8 GB**, throughput held at ~1 Gb/s |

<div align="center">

<br />

**Ask me about** storage engines · B-trees · perfect hashing · compression · Go performance

[X](https://twitter.com/AskAlexSharov) · [AskAlexSharov@gmail.com](mailto:AskAlexSharov@gmail.com)

</div>
