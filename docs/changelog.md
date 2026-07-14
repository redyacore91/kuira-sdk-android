# Changelog

All notable changes to the **Kuira Android SDK**. This page mirrors the SDK's
`CHANGELOG.md`; per-release notes are also on [GitHub Releases](https://github.com/kuiralabs/kuira-sdk-android/releases).

The SDK ships as a set of modules published **together on one version** to Maven Central
under `io.github.kuiralabs` (`dapp-ui`, `midnight-sdk`, `wallet-runtime`, `wallet-seed`,
`identity`, `auth`, `crypto`, `compact-engine`, `indexer`, `ledger`, `network`, `connector`,
`designsystem`, `testing`), plus the `io.github.kuiralabs.contract` and
`io.github.kuiralabs.localnet` Gradle plugins.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/); the SDK follows
[Semantic Versioning](https://semver.org/) (pre-1.0, so minor bumps may break API). Public-API
entries from `alpha03` onward are reconciled against each module's checked-in
binary-compatibility (`.api`) dump.

## [Unreleased]

_No changes yet._

## [0.1.0-alpha05] — 2026-07-14

Typed contract codegen and the unshielded money path.

### Added
- **Typed contract reads** — the `io.github.kuiralabs.contract` plugin generates typed accessors from `contract-info.json`: typed ledger fields (Counter, Cell) and `read` / view methods returning your circuit's real types (scalars, generated `data class`es for structs, `ByteArray`, `List<T>`, tuples), with `Uint` args range-checked to their declared width.
- **`readLocal` and `readMany`** — run a pure circuit against initial state with no deploy and no chain (`readLocal`), and batch several view reads on one state snapshot (`readMany`), with SDK-side indexer retry.
- **Contract constructor arguments** — deploy a contract passing constructor args, threaded through the circuit-call path (`compact-engine`).
- **Multi-contract projects** — a `contracts { }` container DSL: per-contract asset namespacing, a generated package per alias, and sync / validate / generate fanned out across every contract.
- **Unshielded money path** — the unshielded contract-call and withdrawal flows wired end-to-end (real UTXO movement).
- **Typed argument marshalling** — Compact enums marshalled as `CompactEnum(ordinal)`.
- **Dust-backup restore continuity** — restore-by-default instead of a genesis re-replay.
- **Version-coherence pre-flight** — chain-time sourcing and a client/node version check before value-bearing calls.
- **Live wallet sync indicator** on the wallet pill (`dapp-ui`).

### Changed
- **`MidnightSdk.close()` / `MidnightWallet.close()` are now `suspend`** — they suspend until an in-flight balance releases the Dust state. `MidnightSdkProvider.close()` is unchanged, so teardown through the provider needs no change.
- **BigInt marshalling** hardened across the contract-call path.

### Fixed
- **Dust-state races on a live chain** — teardown (network switch, logout, hard-lock) no longer frees the Dust state mid-balance (`DustLocalState has been closed`), and a balance-time failure now delta-re-syncs instead of wiping the checkpoint and replaying from genesis.
- **Sigil chip status color** — the floating sigil chip's status dot no longer reads "protected" green when the sigil is absent, initializing, or failed.

## [0.1.0-alpha04] — 2026-06-21

The frosted "Void" design system and the background-operation framework. ~70 new public types
across the modules (verified against the `.api` dumps).

### Added
- **Sovereign recovery phrase** — reveal a 24-word BIP-39 phrase behind biometrics and restore the exact wallet on any device; opt-in, one-way, on a `FLAG_SECURE` screen.
- **Session auto-lock** — idle, background, and screen-lock re-authentication, plus a manual "lock now".
- **Reactive contract state** — `MidnightContract.observeLedger()` (a `Flow` pushed by block subscriptions) and `MidnightSdk.observeBlocks()`.
- **Durable protocol orchestrator** — a ledger-anchored saga that resumes multi-step flows after process death.
- **NIGHT transfers** — `sendNight` with automatic change-UTXO consolidation, plus a 3-screen Send wizard with QR scan.
- **Per-transaction receive amounts** — real inbound value from UTXO-set provenance.
- **Background receive notifications** — background push on incoming NIGHT with per-transaction value alerts.
- **Foreground-operation framework** — live operation stage in a foreground-service notification and a wallet chip; operations can carry a return content intent.
- **Cloud-backup controls (true disable)** — fully disable Dust and app-state cloud backups; disabling deletes the remote blobs and resets digests.
- **Automatic app-state backup** — silent, hash-guarded capture on each sync.
- **Streamed cold sync** — shielded-state checkpoint + delta streamed to disk (no genesis re-replay, no first-sync GC-storm freeze).
- **Proactive Dust sync** and **automatic dust-proof recovery** (`Custom error: 170/171`) via delta re-sync.
- **Durable network preference** — exposed through `MidnightSdkProvider`.
- **x86_64 native ABI** — the crypto native library now ships `arm64-v8a` **and** `x86_64`, so Intel/Apple-silicon emulators run on-device proving.
- **Floating & resizable wallet/sigil chips** — opt-in draggable chips that dock to a screen edge.
- **Theme palettes** — seven built-in wallet themes (Kuira Monochrome, Paper, Catppuccin, Nord, Dracula, Tokyo Night, Rosé Pine), persisted.
- **Frosted "Void" design system** — GlassPanel v2, StarField, monochrome accent across the wallet, settings, recovery, and receive screens.
- **Redesigned Send flow** — amount presets, a prominent review step, honest in-flight copy.
- **Hardened identity errors** — typed exceptions for passkey, Sigil-overwrite, and Drive-consent failures.
- **Localnet Gradle plugin** — `io.github.kuiralabs.localnet`: auto `adb reverse` of localnet ports and wallet-key provisioning.

### Changed
- **`PanelBar(...)` signature** gained a `floating: Boolean` parameter; recompile against the new arity.

## [0.1.0-alpha03] — 2026-06-10

First release with checked-in `.api` dumps. No `alpha02` was published.

### Added
- **Contract Gradle plugin** — `io.github.kuiralabs.contract`: syncs compiled `.compact` artifacts into the app's assets and enforces the runtime-version pin.
- **`kuiraDoctor` preflight** — build-time environment checks (assetlinks reachability, Compact runtime pin, SDK-bundled-runtime layer, minSdk / cleartext) that fail fast instead of crashing at runtime.
- **One-call proving-key staging** — `ProvingKeyManager.installCircuitKeysFromAssets()`.
- **Cross-device Dust cloud backup** — encrypt-on-device Dust checkpoint to Google Drive `appDataFolder`, restored on a new device; consent UI in the wallet panel. Seed-derived AES-256-GCM (Drive is transport only).
- **Durable network selection** — the chosen network persists across launches.
- **`ContractCallProgressBar`** — a drop-in progress component for deploy/call stages.
- **`hilt-navigation-compose`** api-exposed for `hiltViewModel()` in consumer UI.

### Changed
- Bundled Compact runtime **0.15.0 → 0.16.0**.
- Proving mode surfaced as **"on-device"** (was "local").
- Throttled full wallet resync; balance now tracked via the reactive observer.

## [0.1.0-alpha01] — 2026-05-28

First public alpha of the Kuira Android SDK — build Midnight zero-knowledge dApps on Android
from a single Gradle dependency (`io.github.kuiralabs:dapp-ui`).

### Added
- **On-device ZK proving** — the native Rust `midnight-zkir` engine over JNI; `ProvingMode.LOCAL` by default, no proof server (`arm64-v8a`).
- **Passkey-derived Sigil identity** — one biometric mints a `did:key` identity + wallet seed; no seed phrase.
- **Embedded self-custodial wallet** — shielded + unshielded balances and Dust, in-process.
- **Compact contract runtime** — deploy and call `.compact` contracts (`MidnightContract`), with typed ledger reads.
- **dApp connector** — the standard Midnight `ConnectedAPI` over a local WebSocket, Android Binder, or a WebView bridge.
- **Drop-in Compose wallet UI** — `dapp-ui` (Sigil panel, balances, send/receive).
- Published to Maven Central under `io.github.kuiralabs`.
