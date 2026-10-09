# KasVault — Covenant Genesis Proofs (Buyback Vault v10 + StakePool v2)

This repository lets anyone verify, on-chain and independently, that the
KasVault covenants holding the project's KAS are **exactly** the published
source code with **exactly** these genesis parameters.

- **Network:** Kaspa mainnet
- **Tooling:** Argent CLI, genesis-proof branch (Sutton covenant standard)

## Live covenants

### StakePool v2 — staking (live since 2026-10-07)

- **Covenant ID:** `85f1d6c05391582bc724cfdca36ee9192eb123c3c25acd12620f91d964956381`
- **Genesis TX:** [`01097ddb…`](https://kaspa.stream/txs/01097ddbac052f36c1a59da3f545d8b9e994fcefb593ad45a57bf268e0f57d43)
- **First stake:** [`e8486aee…`](https://kaspa.stream/txs/e8486aee07760fc096f609e23a404cb36af8aedaf82455b9a745a38b1086844e) — 1,000,000 kvault
- **Genesis state:** `{kas: 0, totalTok: 0, rewardPerTok: 0, deposits: 0}`, funded by a 1 KAS carrier UTXO

`stake_pool_v2.sil` is a Kaspa L1 covenant (SilverScript, consensus-enforced).
Stakers deposit kvault and earn curve trading fees pro-rata via an on-chain
distribution index. Reward capital can only leave through the claim formula —
there is no drain or sweep entry, not even for KasVault. Unstake returns the
staked tokens to the wallet that owns them.

Stake at the portal: **https://lighthearted-swan-ef22f8.netlify.app/**

### Buyback Vault v10 (live since 2026-10-04)

- **Covenant ID:** `7a1791e82a1d17d9ddab81fb48ace6688b09d7f6dbe6782dd569426a2b5f455b`
- **Genesis TX:** [`54642c27…`](https://kaspa.stream/txs/54642c273a7e38792cc1647f189fe83830d98a088d41a163e2c7172e9a1df1e6)
- **Genesis state:** 20 KAS seed, 20,261 tokens, daily buy limit 30 KAS,
  min 33 tokens per KAS, rollover anchor 1791199462605 (ms)

`vault_v10.sil` can only **buy** kvault on the bonding curve — never sell.
Daily spend is capped on-chain, each buy must return at least `minTokens`,
and the daily rollover is agent-signed (v10 fix: closes a 0.18 KAS/day
griefing window). The agent key can spend within these limits; it cannot
withdraw capital or change the rules. Emergency recovery (owner or backup
key, 48-hour DAA window, backup-key execution) is enforced by consensus.

## Verify it yourself

1. **Build the verifier** (Argent CLI, Sutton's genesis-proof branch):
   ```sh
   rustup default stable
   git clone https://github.com/argent-lang/argent && cd argent
   git checkout genesis-proof-tooling        # commit 9592dd996d2503eb166335d6da92548de193877f
   cargo build --release --bin argentc
   ```
2. **Verify each proof against the covenant ID** (from any Kaspa node):
   ```sh
   ./target/release/argentc genesis verify stake_pool_v2_genesis_proof.json \
       --covenant-id 85f1d6c05391582bc724cfdca36ee9192eb123c3c25acd12620f91d964956381
   ./target/release/argentc genesis verify vault_v10_genesis_proof.json \
       --covenant-id 7a1791e82a1d17d9ddab81fb48ace6688b09d7f6dbe6782dd569426a2b5f455b
   ```
   Expected: `covenant ID matches` — Silverscript ABIs, physical states and
   derived scripts checked.
3. **Get the covenant ID independently** (do not trust this README): query
   the live UTXO from any mainnet node (`getUtxosByAddresses` on the
   covenant P2SH address) and read its `covenantId` field. The ID is stable
   for the covenant's entire lifetime.

## History

| Covenant | ID (prefix) | Status |
|---|---|---|
| Buyback Vault v8 | `74c3061b…` | superseded 2026-10-04, proof kept |
| Buyback Vault v9 | `d1421c8a…` | superseded 2026-10-04 (v10 rollover-sig fix), proof kept |
| Buyback Vault v10 | `7a1791e8…` | **live** |
| StakePool v1 | `2dcc2fd2…` | superseded 2026-10-07 (v2 migration), all holdings unstaked |
| StakePool v2 | `85f1d6c0…` | **live** |

The v1→v2 pool migration (2026-10-07) moved every staked position through
on-chain unstake and re-stake. No funds were carried by trust — every UTXO
is a receipt on kaspa.stream.

## Files

| File | Purpose |
|---|---|
| `stake_pool_v2.sil` | Staking covenant source (SilverScript) |
| `stake_pool_v2_artifact.json` | Compiler artifact / ABI |
| `stake_pool_v2_bootstrap.json` | Genesis group: authorizing outpoint, output, initial state |
| `stake_pool_v2_genesis_proof.json` | Self-contained proof package |
| `vault_v10.sil` | Buyback vault source (SilverScript) |
| `vault_v10_artifact.json` | Compiler artifact / ABI |
| `vault_v10_bootstrap.json` | Genesis group (temporal encoded as int, per covenant standard) |
| `vault_v10_genesis_proof.json` | Self-contained proof package |
| `vault_v8.sil`, `vault_v9.sil` … | History proof files (still verifiable) |

**Token:** kvault (KCC20 on KRON) — https://kron.technology/token/kvault
**Staking portal:** https://lighthearted-swan-ef22f8.netlify.app/

**Zero dev allocation. The vault can only buy. Verify, don't trust.**
