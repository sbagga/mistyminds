# Mobile Rust core libraries

<!-- toc -->
### List of the libaries
1. [Kernel](#Kernel)
2. [Multimedia](#Multimedia) 
3. [Graphics](#Graphics)
4. [UIKit](#UIKit)
5. [Webview](#Webview)
6. [Application](#Application)
7. [Database](#Database)
8. [Security](#Security)
9. [Math](#Math)
10. [AI](#AI)



### Kernel
* <span style="color:red">[Rust for Linux](https://github.com/Rust-for-Linux/linux) - Rust development for the Linux Kernel </span>


### Multimedia

* [enginesound](https://github.com/DasEtwas/enginesound) — A GUI and command line application used to procedurally generate semi-realistic engine sounds. Featuring in-depth configuration, variable sample rate and a frequency analysis window.
* [Glicol](https://github.com/chaosprint/glicol) — Graph-oriented live coding language written in Rust for collaborative musicking in browsers.
* [ncspot](https://github.com/hrkfdn/ncspot) - Cross-platform ncurses Spotify client, inspired by ncmpc and the likes. (https://github.com/hrkfdn/ncspot/workflows/Build/badge.svg)](https://github.com/hrkfdn/ncspot/actions?query=workflow%3ABuild)
* [Polaris](https://github.com/agersant/polaris) — A music streaming application.  (https://api.travis-ci.org/agersant/polaris.svg?branch=master)](https://travis-ci.org/agersant/polaris)
* [Spotify TUI](https://github.com/Rigellute/spotify-tui) — A Spotify client for the terminal written in Rust. (https://github.com/Rigellute/spotify-tui/workflows/Continuous%20Integration/badge.svg?branch=master)
* [Spotifyd](https://github.com/Spotifyd/spotifyd) — An open source Spotify client running as a UNIX daemon. (https://github.com/Spotifyd/spotifyd/workflows/Continuous%20Integration/badge.svg?branch=master)


### Graphics
* [WGPU](https://github.com/gfx-rs/wgpu) - wgpu is a cross-platform, safe, pure-rust graphics api. It runs natively on Vulkan, Metal, D3D12, D3D11, and OpenGLES; and on top of WebGPU on wasm.



### UIKit
* [Makepad](https://github.com/makepad/makepad) - a GPU native 2D/3D GUI kit implemented in pure Rust with high level declarative UI DSL and component libraries
* [Druid](https://github.com/linebender/druid) - A data-first Rust-native UI toolkit

### Webview
* [Servo](https://github.com/servo) - Rust implemented browser kernel

### Application
* [Tauri](https://github.com/tauri-apps/tauri) - Tauri is a framework for building tiny, blazingly fast binaries for all major desktop platforms. Developers can integrate any front-end framework that compiles to HTML, JS and CSS for building their user interface. The backend of the application is a rust-sourced binary with an API that the front-end can interact with
* [Yew](https://github.com/yewstack/yew) - Yew is a modern Rust framework for creating multi-threaded front-end web apps with WebAssembly
* [Rust web framework comparison](https://github.com/flosse/rust-web-framework-comparison) - A comparison of some web frameworks written in Rust. This overview only contains frameworks that work on stable Rust.




### Database

* [Databend](https://github.com/datafuselabs/databend) - A Modern Real-Time Data Processing & Analytics DBMS with Cloud-Native Architecture (https://github.com/datafuselabs/databend/actions/workflows/databend-release.yml/badge.svg)](https://github.com/datafuselabs/databend/actions/workflows/databend-release.yml)
* [indradb](https://crates.io/crates/indradb) — Rust based graph database (https://api.travis-ci.org/indradb/indradb.svg?branch=master)](https://travis-ci.org/indradb/indradb)
* [Lucid](https://github.com/lucid-kv/lucid) — High performance and distributed KV store accessible through a HTTP API. (https://github.com/lucid-kv/lucid/workflows/Lucid/badge.svg?branch=master)](https://github.com/lucid-kv/lucid/actions?workflow=Lucid)
* [Materialize](https://github.com/MaterializeInc/materialize) - Streaming SQL database powered by Timely Dataflow (https://badge.buildkite.com/97d6604e015bf633d1c2a12d166bb46f3b43a927d3952c999a.svg?branch=main)](https://buildkite.com/materialize/tests)
* [noria](https://github.com/mit-pdos/noria) [[noria](https://crates.io/crates/noria)] — Dynamically changing, partially-stateful data-flow for web application backends (https://api.travis-ci.org/mit-pdos/noria.svg?branch=master)](https://travis-ci.org/mit-pdos/noria)
* [ParityDB](https://github.com/paritytech/parity-db) — Fast and reliable database, optimised for read operation
* [PumpkinDB](https://github.com/PumpkinDB/PumpkinDB) — an event sourcing database engine
* [Qdrant](https://github.com/qdrant/qdrant) - An open source vector similarity search engine with extended filtering support (https://github.com/qdrant/qdrant/workflows/Tests/badge.svg)](https://github.com/qdrant/qdrant/actions)
* [seppo0010/rsedis](https://github.com/seppo0010/rsedis) — A Redis reimplementation in Rust (https://api.travis-ci.org/seppo0010/rsedis.svg?branch=master)](https://travis-ci.org/seppo0010/rsedis)
* [Singularity-Data/RisingWave](https://github.com/singularity-data/risingwave) - the next-generation streaming database in the cloud (https://github.com/singularity-data/risingwave/actions/workflows/main.yml/badge.svg)](https://github.com/singularity-data/risingwave/actions/workflows/main.yml/badge.svg?branch=main)
* [Skytable](https://github.com/skytable/skytable) — A multi-model NoSQL database (https://img.shields.io/github/workflow/status/skytable/skytable/Tests?style=flat-square)
* [sled](https://crates.io/crates/sled) — A (beta) modern embedded database (https://github.com/spacejam/sled/workflows/Rust/badge.svg?branch=master)](https://github.com/spacejam/sled/actions?workflow=Rust)
* [TerminusDB](https://github.com/terminusdb/terminusdb-store) - open source graph database and document store (https://github.com/terminusdb/terminusdb-store/workflows/Build/badge.svg?branch=master)](https://github.com/terminusdb/terminusdb-store/actions)
* [tikv](https://github.com/tikv/tikv) — A distributed KV database in Rust (https://ci.pingcap.net/job/tikv_ghpr_test/badge/icon)](https://ci.pingcap.net/job/tikv_ghpr_test/)
* [vorot93/libmdbx-rs](https://github.com/vorot93/libmdbx-rs) [[mdbx-sys](https://crates.io/crates/mdbx-sys)] — Rust bindings for MDBX, a "fast, compact, powerful, embedded, transactional key-value database, with permissive license". This is a fork of mozilla/lmdb-rs with patches to make it work with libmdbx.
* [WooriDB](https://github.com/naomijub/wooridb) - General purpose time serial database inspired by Crux and Datomic.


### Security
* [Rust Cryptography Interest Group (RCIG)](https://cryptography.rs/) - list of actively maintained, high-quality cryptography libraries independently developed by members of the Rust Community.
* [rustls](https://github.com/rustls/rustls) - Rust implementation of the TLS library supported by ISRG project (https://www.memorysafety.org/initiative/rustls/)
* [Rust Crypto](https://github.com/RustCrypto) - Cryptographic algorithms written in pure Rust
* [TabbySSL](https://github.com/ymjing/tabbyssl) - OpenSSL compatibility layer for the Rust SSL/TLS stack

### Math
* [nalgebra](https://github.com/dimforge/nalgebra) - Linear algebra library for the Rust programming language.
* [ndarray](https://github.com/rust-ndarray/ndarray) - provides an n-dimensional container for general elements and for numerics.
* [Rust Math](https://arewegameyet.rs/ecosystem/math/) - Linear algebra libraries, quaternions, color conversion and more

### AI
* [Rust AI and ML](https://vaaaaanquish.github.io/Awesome-Rust-MachineLearning/#gpu) - This repository is a list of machine learning libraries written in Rust. It's a compilation of GitHub repositories, blogs, books, movies, discussions, papers. This repository is targeted at people who are thinking of migrating from Python
