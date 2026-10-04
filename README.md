# KasVault — On-chain Buyback Vault (Covenant Genesis Proof)

This package lets anyone verify, on-chain and independently, that the KasVault
buyback covenant holding the project's KAS liquidity is **exactly** the
published source code with **exactly** these genesis parameters.

- **Network:** Kaspa mainnet
- **Covenant ID:** `74c3061bd18a5bb8b16bc7557bcb0e95b46c40fa548aa30f991720284cfb67ed`
- **Genesis TX:** `93c1bc0ad8b876d66a1e31f9a5c8016267ed342208f21c15ac939b9d18a30de8`
- **Genesis state:** 20 KAS seed, 0 tokens, daily buy limit 30 KAS,
  min 33 tokens per KAS, rollover anchor 1791125242552 (ms)

## What the vault does

`vault_v8.sil` is a Kaspa L1 covenant (SilverScript, consensus-enforced).
It can only **buy** the token on the bonding curve — never sell. Daily spend
is capped on-chain (`dayLimit`), each buy must return at least `minTokens`,
and a daily rollover entry resets the spent amount. The agent key can spend
within these limits; it cannot withdraw capital or change the rules.

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
   ./target/release/argentc genesis verify vault_v8_genesis_proof.json \
     --covenant-id 74c3061bd18a5bb8b16bc7557bcb0e95b46c40fa548aa30f991720284cfb67ed
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

## Files

| File | Purpose |
|---|---|
| `vault_v8.sil` | The complete covenant source (SilverScript) |
| `vault_v8_artifact.json` | Compiler artifact / ABI (SilverScript compiler version embedded) |
| `vault_v8_bootstrap.json` | Genesis group: authorizing outpoint, output values, initial state |
| `vault_v8_genesis_proof.json` | Self-contained proof package for `argentc genesis verify` |

The proof ties the covenant ID to: the authorizing input's previous outpoint
(`35d9d44b…:7`), the genesis output group (output 0, 2,000,000,000 sompi) and
the physical initial state encoded in the covenant script.

**Zero dev allocation. The vault can only buy. Verify, don't trust.**
