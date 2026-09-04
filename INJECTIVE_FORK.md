# Injective fork

This is Choice's fork of a PancakeSwap Infinity repository, deployed to Injective EVM
(testnet `1439`, mainnet `1776`). Work branch: `injective`.

## The one rule

**`src/` is never edited.** Upstream's audits (Hexens, OtterSec, Zellic) describe the bytecode
we deploy, and that is only true while our `src/` is byte-identical to the pinned upstream
commit in [.injective-fork-base](.injective-fork-base). CI enforces it
([upstream-guard.yml](.github/workflows/upstream-guard.yml)) and will fail the PR otherwise.

Choice's own Solidity - fee controller, launchpad settler, aggregation router - lives in
`choice_v2_contracts`, never here.

## Diff vs upstream

| Path | Change |
| --- | --- |
| `script/config/injective-testnet.json` | New. Deploy config for chain 1439. `permit2` is the canonical `0x0000…78BA3` (already deployed on both Injective nets, nothing to deploy); `weth` is wINJ `0x0000000088827d2d103ee2d9A6b781773AE03FfB`. |
| `.injective-fork-base` | New. The upstream commit `src/` is pinned to. |
| `.github/workflows/upstream-guard.yml` | New. See below. |
| `.github/workflows/test.yml` | Trigger branch, path filters, profile - as in `infinity-core`. |

Nothing under `src/`.

## `factoryV2` / `factoryV3` / `factoryStable`

Set to `0x…dEaD`. No PancakeSwap v2, v3 or StableSwap deployment exists on Injective, and
`MixedQuoter` only stores these addresses - the legacy quote paths revert if anything ever
calls them, which is the intended outcome. Infinity CL and Bin quoting is unaffected.

## Moving the pin

Rebase `injective` onto the new upstream commit, update `.injective-fork-base` in the same
commit, run the full suite locally, and record the move in `choice_v2_contracts/deployments`.
The "Upstream drift" job reports how far behind the pin is on every PR, without failing it.

## Deploying

Injective RPCs can return `null` for a mined tx's receipt. Broadcast with `--slow`, **never
use `--resume`**, and confirm every contract with `eth_getCode` rather than with a receipt.
Full runbook: `choice_v2/CHOICE_V2_EVM_PLAN.md` §M1.

## Licence, and the change notice GPL-2.0 asks for

Upstream is PancakeSwap Infinity, Copyright (C) PancakeSwap, licensed GPL-2.0-or-later.
This fork is distributed under the same terms, and Choice claims no additional restriction
on any of it.

**Every file this fork changes is listed in "Diff vs upstream" above, and every one of those
changes was made on 2026-09-04.** That is the notice GPL-2.0 section 2(a) asks for: what was
changed, and when. Nothing under `src/` is touched at all - `upstream-guard.yml` fails the
build if it ever is - so the audited protocol source is byte-identical to the upstream commit
pinned in `.injective-fork-base`.
