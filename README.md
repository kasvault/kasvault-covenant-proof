# KasVault — Covenant Genesis Proofs (Buyback Vault v10 + StakePool v3)

This repository lets anyone verify, on-chain and independently, that the
KasVault covenants holding the project's KAS are **exactly** the published
source code with **exactly** these genesis parameters.

- **Network:** Kaspa mainnet
- **Tooling:** Argent CLI, genesis-proof branch (Sutton covenant standard)

## Live covenants

### StakePool v3 — staking (live since 2026-10-17)

- **Covenant ID:** `598b4b8ebbc51b13b90b338770ce68f8a51df3301ea1320169f69d6b649171eb`
- **Genesis TX:** [`b91f882b…`](https://kaspa.stream/txs/b91f882b4941562914488aa1d0812d5cf1e9c655d6388c9e6aaa3f5c3a2cad10)
- **Source:** [`stake_pool_v3.sil`](stake_pool_v3.sil)
- **Genesis state:** `{kas: 2,308,439,450, totalTok: 1,550,000, rewardPerTok: 2,291,083,468, deposits: 4}`,
  funded from the retired v2 pool, so the distribution index carried over exactly.

`stake_pool_v3.sil` is a Kaspa L1 covenant (SilverScript, consensus-enforced).
Stakers deposit kvault and earn the curve's creator fees pro-rata via an
on-chain distribution index. Reward capital can only leave through the claim
formula or the owner's drain entry, and the pool always keeps a 0.5 KAS
carrier. A 0.1 KAS stake fee goes into the pool and is distributed to all
stakers immediately; a 0.05 KAS claim fee is taken out of the payout, so claims
fund themselves. Unstake returns the tokens to the wallet that owns them.

**No permissionless entries.** Every entry requires the agent signature. This
is a deliberate v3 change: on 2026-10-17 we measured on mainnet that a KCC20
token cell held by a covenant can be moved by *any* valid transition of that
covenant family — presence alone is not a lock. A staking covenant holding
KCC20 tokens must therefore never expose an unsigned entry.

Stake at the portal: **https://lighthearted-swan-ef22f8.netlify.app/**

### Buyback Vault v10 — buybacks (live since 2026-10-04)

- **Covenant ID:** `7a1791e82a1d17d9ddab81fb48ace6688b09d7f6dbe6782dd569426a2b5f455b`
- **Genesis TX:** [`54642c27…`](https://kaspa.stream/txs/54642c273a7e38792cc1647f189fe83830d98a088d41a163e2c7172e9a1df1e6)
- **Source:** [`vault_v10.sil`](vault_v10.sil) — 111 lines, 8 entries
- **Genesis state:** 20 KAS seed, 20,261 tokens, daily buy limit 30 KAS,
  min 33 tokens per KAS, rollover anchor 1791199462605 (ms)

`vault_v10.sil` can only **buy** kvault on the bonding curve — never sell.
Daily spend is capped on-chain, each buy must return at least `minTokens`,
and the daily rollover is agent-signed (v10 fix: closes a 0.18 KAS/day
griefing window). The agent key can spend within these limits; it cannot
withdraw capital or change the rules. Emergency recovery (owner or backup
key, 48-hour DAA window, backup-key execution) is enforced by consensus.

Read the whole rule set before trusting anything: 8 entries, and what each one
is allowed to do —
`buy` (agent-signed, daily-capped, minimum tokens out), `rollover`
(agent-signed, once per 24 h), `reparam` (owner-signed, minTokens only),
`topup` (owner-signed, money in), `drain` (owner-signed exit), and the three
recovery entries `requestRecovery` / `cancelRecovery` / `executeRecovery`.

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
   ./target/release/argentc genesis verify vault_v10_genesis_proof.json \
       --covenant-id 7a1791e82a1d17d9ddab81fb48ace6688b09d7f6dbe6782dd569426a2b5f455b
   ./target/release/argentc genesis verify stake_pool_v3_genesis_proof.json \
       --covenant-id 598b4b8ebbc51b13b90b338770ce68f8a51df3301ea1320169f69d6b649171eb
   ```
   Expected: `covenant ID matches` — Silverscript ABIs, physical states and
   derived scripts checked. `stake_pool_v2_genesis_proof.json` still verifies
   against the retired v2 covenant ID `85f1d6c0…` and is kept for history.
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
| StakePool v2 | `85f1d6c0…` | retired 2026-10-17 (v3), proof kept |
| StakePool v3 | `598b4b8e…` | **live** |

The v1→v2 pool migration (2026-10-07) moved every staked position through
on-chain unstake and re-stake. No funds were carried by trust — every UTXO
is a receipt on kaspa.stream.

The v2→v3 replacement (2026-10-17) happened the same way: every staked
position was unstaked on-chain, the pool was rebuilt with entries that all
require a signature, and positions were staked again. v2 holds no KAS and no
tokens today.

## Files

| File | Purpose |
|---|---|
| `stake_pool_v3.sil` | Staking covenant source, **live** |
| `stake_pool_v3_artifact.json` | Compiler artifact / Silverscript ABI |
| `stake_pool_v3_bootstrap.json` | Genesis group: authorizing outpoint, output, initial state |
| `stake_pool_v3_genesis_proof.json` | Self-contained proof package |
| `stake_pool_v2.sil` | Retired staking covenant source (SilverScript) |
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
