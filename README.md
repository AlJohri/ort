<div align=center>
<img src="https://parcel.pyke.io/v2/cdn/assetdelivery/ortrsv2/docs/trend-banner.png" width="350px">
</div>
<div align=center>
<a href="https://app.codecov.io/gh/pykeio/ort" target="_blank"><img alt="Coverage Results" src="https://img.shields.io/codecov/c/gh/pykeio/ort?style=for-the-badge"></a> <img alt="MSRV" src="https://img.shields.io/crates/msrv/ort?style=for-the-badge"> <img alt="ONNX Runtime" src="https://img.shields.io/badge/onnxruntime-v1.26.0-blue?style=for-the-badge&logo=cplusplus">
</div>
<hr /><br />

> **AlJohri fork note.** This fork relaxes `Session::run` / `run_with_options` / `run_binding` / `run_binding_with_options` / `run_async` from `&mut self` to `&self`. Internally these methods already delegate to private `&self` impls (`run_inner` etc.), so the public `&mut self` is only a signature-level constraint. The change lets multiple threads call `Run` concurrently on an `Arc<Session>` without fabricating `&mut` from `&` (which is UB under Rust's aliasing model).
>
> Microsoft's high-level intent for ONNX Runtime is stated at <https://onnxruntime.ai/docs/reference/high-level-design.html>:
>
> > "Multiple threads can invoke the Run() method on the same inference session object. See API doc for more details."
>
> An explicit Microsoft contributor statement that this contract is load-bearing across EPs appears in microsoft/onnxruntime#19301: *"InferenceSession::Run() is guaranteed to be thread-safe meaning multiple threads can call this function concurrently."*
>
> **This fork is intended for the CPU EP only** — that's where Microsoft's thread-safety contract is best-documented and best-tested. Other EPs have varying and EP-specific caveats: DirectML explicitly forbids concurrent `Run` and requires `enable_mem_pattern = false`; CUDA Graphs forbids concurrent `Run` when enabled; plain CUDA / TensorRT aren't addressed in the official docs, so concurrent use is at your own risk.
>
> The `&mut self` signature was introduced in upstream commit [`bd2aff7`](https://github.com/pykeio/ort/commit/bd2aff711ec364db8d41be6a3c4690e5feaf36b4) ("refactor!: make `Session::run` take `&mut self`", 2025-02-08), first released in `v2.0.0-rc.10`. Through `v2.0.0-rc.9` the method took `&self` and the share-via-`Arc` pattern was directly expressible; this fork restores that. Upstream's canonical pinned discussion is pykeio/ort#402 ("Thread safety and `Send`/`Sync`"); the explicit refusals to expose a `&self` variant since rc.10 are pykeio/ort#511 and pykeio/ort#415.
>
> **Pulling new upstream:**
>
> ```sh
> OLD_BASE=$(git rev-parse --short HEAD~1)          # upstream commit our patch is currently on top of
> git fetch upstream
> git rebase upstream/main                          # one commit replays on top of new upstream; resolve conflicts if any
> git tag aljohri-$OLD_BASE HEAD@{1}                # anchor pre-rebase HEAD so its hash stays GC-safe
> git push origin aljohri-$OLD_BASE
> git push --force-with-lease origin main
> ```
>
> Downstream consumers pin to the resulting commit hash via `rev = "..."` in `Cargo.toml` (not the tag — tags exist solely to keep old hashes reachable on GitHub after the force-push). Tag names mirror the upstream base short-sha so it's obvious which upstream `main` each fork build is rebased on.

`ort` is a Rust interface for performing hardware-accelerated inference & training on machine learning models in the [Open Neural Network Exchange](https://onnx.ai/) (ONNX) format.

Based on the now-inactive [`onnxruntime-rs`](https://github.com/nbigaouette/onnxruntime-rs) crate, `ort` is primarily a wrapper for Microsoft's [ONNX Runtime](https://onnxruntime.ai/) library, but offers support for [other pure-Rust runtimes](https://ort.pyke.io/backends).

`ort` with ONNX Runtime is super quick - and it supports almost [any hardware accelerator](https://ort.pyke.io/perf/execution-providers) you can think of. Even still, it's light enough to run on your users' devices.

When you need to deploy a PyTorch/TensorFlow/Keras/scikit-learn/PaddlePaddle model either on-device or in the datacenter, `ort` has you covered.

## 📖 Documentation
- [Guide](https://ort.pyke.io/)
- [API reference](https://docs.rs/ort/2.0.0-rc.12/ort/)
- [Examples](https://github.com/pykeio/ort/tree/main/examples)
- [Migrating from v1.x to v2.0](https://ort.pyke.io/migrating/v2)

## 🤔 Support
- [Discord: `#🦀｜ort-general`](https://discord.gg/uQtsNu2xMa)
- [GitHub Discussions](https://github.com/pykeio/ort/discussions)

## 🌠 Backers
<a href="https://opencollective.com/pyke-osai">
<img src="https://opencollective.com/pyke-osai/backers.svg" />
</a>

## 💖 FOSS projects using `ort`
<sub>[Open a PR](https://github.com/pykeio/ort/pulls) to add your project here 🌟</sub>

<!--
    This section only showcases projects with OSI-approved licenses: https://opensource.org/licenses
    Businesses that sponsor pyke will appear at the top of the README instead!
-->

- **[Text Embeddings Inference (TEI)](https://github.com/huggingface/text-embeddings-inference)** uses `ort` to deliver high-performance ONNX Runtime inference for text embedding models.
- **[Magika](https://github.com/google/magika)** uses `ort` for neural network-based file type detection.
- **[retto](https://github.com/NekoImageLand/retto)** uses `ort` for reliable, fast ONNX inference of PaddleOCR models on Desktop and WASM platforms.
- **[edge-transformers](https://github.com/npc-engine/edge-transformers)** uses `ort` for accelerated transformer model inference at the edge.
- **[`sbv2-api`](https://github.com/neodyland/sbv2-api)** is a fast implementation of Style-BERT-VITS2 text-to-speech using `ort`.
- **[BoquilaHUB](https://github.com/boquila/boquilahub/)** uses `ort` for local AI deployment in biodiversity conservation efforts.
- **[CamTrap Detector](https://github.com/bencevans/camtrap-detector)** uses `ort` to detect animals, humans and vehicles in trail camera imagery.
- **[Ortex](https://github.com/relaypro-open/ortex)** uses `ort` for safe ONNX Runtime bindings in Elixir.
- **[oar-ocr](https://github.com/GreatV/oar-ocr)** A comprehensive OCR library, built in Rust with `ort` for efficient inference.
- **[`FastEmbed-rs`](https://github.com/Anush008/fastembed-rs)** uses `ort` for generating vector embeddings, reranking locally.
- **[Ahnlich](https://github.com/deven96/ahnlich)** uses `ort` to power their AI proxy for semantic search applications.
- **[Murmure](https://github.com/Kieirra/murmure)** uses `ort` as its core engine, leveraging NVIDIA Parakeet to deliver fully local, free, private and cross‑platform Speech‑to‑Text enhanced with LLM post‑processing.
- **[Valentinus](https://github.com/kn0sys/valentinus)** uses `ort` to provide embedding model inference inside LMDB.
- **[SilentKeys](https://github.com/gptguy/silentkeys)** uses `ort` for fast, on-device real-time dictation with NVIDIA Parakeet and Silero VAD.
- **[Xybrid](https://github.com/xybrid-ai/xybrid)** uses `ort` to run LLMs, ASR, and TTS natively on-device across iOS, Android, Flutter, and Unity apps and games.
