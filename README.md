# KasVault — On-chain Buyback Vault v9 (Covenant Genesis Proof)

This package lets anyone verify, on-chain and independently, that the KasVault
buyback covenant holding the project's KAS liquidity is **exactly** the
published source code with **exactly** these genesis parameters.

- **Network:** Kaspa mainnet
- **Covenant ID:** `d1421c8a298f440250da5bbdea2a292401a5ea0264a9b0a3c60e6e5b2fba0a61`
- **Genesis TX:** `2395ab2f630f694b6fa045fd0943b535e2b37edd5de263f540581d55eef82f9d`
- **Genesis state:** 20 KAS seed, 20,261 tokens carried over from v8,
  3 KAS spent today (daily budget carried over honestly), daily buy limit
  30 KAS, min 33 tokens per KAS, rollover anchor 1791125242552 (ms),
  no recovery pending.

## What the vault does

`vault_v9.sil` is a Kaspa L1 covenant (SilverScript, consensus-enforced).
It can only **buy** the token on the bonding curve — never sell. Daily spend
is capped on-chain (`dayLimit`), each buy must return at least `minTokens`,
and a daily rollover entry resets the spent amount. The agent key can spend
within these limits; it cannot withdraw capital or change the rules.

**v9 adds emergency recovery, enforced by consensus:**
- `requestRecovery` — owner *or* backup key announces recovery (`recoverPending = 1`),
  which immediately **locks all buying**.
- `cancelRecovery` — only the primary owner can abort a pending recovery.
- `executeRecovery` — only the backup key, and only after a 48-hour DAA window
  (`ageDaa >= 172800`) measured on-chain, may spend the covenant. The covenant
  ends there.

If the agent key is ever lost, the vault is recoverable — and a compromised
backup key alone cannot move funds while the owner can still cancel.

## Verify it yourself

1. **Build the verifier** (Argent CLI, Sutton's genesis-proof branch):
   ```sh
   rustup default stable
   git clone https://github.com/argent-lang/argent && cd argent
   git checkout genesis-proof-tooling        # commit 9592dd996d2503eb166335d6da92548de193877f
   cargo build --release --bin argentc
   ```
2. **Verify the proof against the covenant ID** (from any Kaspa node):
   ```sh
   ./target/release/argentc genesis verify vault_v9_genesis_proof.json      --covenant-id d1421c8a298f440250da5bbdea2a292401a5ea0264a9b0a3c60e6e5b2fba0a61
   ```
   Expected: `covenant ID matches` — Silverscript ABIs, physical states and
   derived scripts checked.

3. **Get the covenant ID independently** (do not trust this README):
   The covenant ID is consensus data, committed to the covenant's genesis.
   Query the live vault UTXO from any mainnet node, e.g. via RPC
   `getUtxosByAddresses` for the current vault P2SH address, or ask the
   KasVault team for the current vault address and read the `covenantId`
   field of its UTXO. It must equal the ID above. The ID is stable for the
   covenant's entire lifetime — every continuation (buys, rollovers, reparams)
   keeps the same ID.

## History

The predecessor covenant `74c3061b…` (v8, proof files kept in this repo) was
migrated to v9 on 2026-10-04: same rules, plus emergency recovery. The v8
proof remains verifiable with the same procedure.

## Files

| File | Purpose |
|---|---|
| `vault_v9.sil` | The complete covenant source (SilverScript) |
| `vault_v9_artifact.json` | Compiler artifact / ABI (SilverScript compiler version embedded) |
| `vault_v9_bootstrap.json` | Genesis group: authorizing outpoint, output values, initial state |
| `vault_v9_genesis_proof.json` | Self-contained proof package for `argentc genesis verify` |
| `vault_v8.sil` … | v8 proof files (predecessor covenant, still verifiable) |

The proof ties the covenant ID to: the authorizing input's previous outpoint
(`fae20dce…:7`), the genesis output group (output 0, 2,000,000,000 sompi) and
the physical initial state encoded in the covenant script.

**Zero dev allocation. The vault can only buy. Verify, don't trust.**
