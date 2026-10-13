# Audit report: tip-jar (Pinocchio, SPL Token)

Scope: `programs/tip-jar/src/{lib,instructions,state}.rs`, focused on withdraw access control, arithmetic, and PDA validation.

Method: manual review, plus `cargo build-sbf --offline` and `cargo test --offline` (6/6 existing tests pass). I also wrote a temporary PoC test for one hypothesis (see "Refuted hypotheses") and removed it afterwards.

**Verdict: accept. No exploitable findings. Two informational notes.**

## Access control (withdraw)

- `owner.is_signer()` is required before anything else (`instructions.rs`, withdraw).
- `Jar::load` requires the jar to be owned by the program, to be exactly `JAR_LEN` long, and to carry the discriminator. Only `initialize` can create such an account, because the jar is a PDA that only this program can sign for.
- `owner.address() == jar_state.owner` is enforced and returns `Unauthorized`. The stored owner is bound into the jar's seeds, so a different owner cannot be substituted.
- `vault == jar_state.vault` is enforced. The vault token account's authority is the jar PDA, set by `InitializeAccount3`. The transfer is signed with the jar's own seeds (`jar`, owner, mint, bump), and the bump comes from stored state.
- The destination must be a token account of the jar's mint. It is rejected if it equals the vault. It may be any such account, which is acceptable because only the owner can sign.
- The token program ID is checked, so there is no arbitrary-CPI risk.
- Existing tests cover both a stranger signing and the owner key supplied without a signature.

Result: no authorization bypass found.

## Arithmetic

- The program does no arithmetic itself. Amounts are parsed as exactly 8 LE bytes, since `try_into` to `[u8; 8]` rejects any other length. Zero is rejected.
- Withdraw checks `amount > vault.amount()` and returns `InsufficientFunds`. Balance changes happen inside the SPL Token program, which uses checked math, so vault overflow on tip is not reachable.
- Offsets (`1`, `33`, `65`, `97`, length `98`) are consistent. `read_address` uses `get(..)`, so there are no panics from slicing.

Result: no overflow, underflow, or truncation issues.

## PDA validation

- `initialize` derives the jar from `[jar, owner, mint]` and the vault from `[vault, jar]` via `find_program_address`. It compares both against the supplied accounts, so canonical bumps are used and stored.
- `tip` and `withdraw` do not re-derive the PDAs. They rely on program ownership of the jar plus the stored `vault` address, which is equivalent because only `initialize` writes a jar.
- The jar and vault are created with `create_account_with_minimum_balance_signed`, with the program ID and the token program as owners. The system program ID and the token program ID are both checked.
- The mint must be owned by the token program. A non-mint account owned by the token program would make `InitializeAccount3` fail, so initialization is rejected.
- Re-initialization fails (`initialize_twice_fails`).

Result: PDA validation is sound.

## Refuted hypotheses

- **Pre-funding the jar or vault PDA to block `initialize` (lamport-griefing).** I tested this by transferring 1,000,000 lamports to each PDA before calling `initialize`. Initialize still succeeded in both cases, because the pinocchio-system helper handles pre-funded accounts. Not exploitable.

## Informational notes

1. **Open initialization (info).** `initialize` is permissionless, and the owner is taken from instruction data. Anyone can create a jar for any `(owner, mint)` pair, paying the rent themselves. This is harmless because the owner is fixed by the seeds and the creator gains no rights. The cost is that a jar may exist for an owner who did not ask for it.
2. **Fixed token program, no Token-2022 and no close path (info).** Only the legacy SPL Token program is supported. Token-2022 mints are rejected by the owner check on the mint. There is no instruction to close the jar or vault and reclaim rent. Both are design limitations, not vulnerabilities.

## Test gaps (suggestions, not defects)

- No test for a `destination` account of a different mint, or for `destination == vault`.
- No test for a wrong token program ID or a wrong `jar` account (not program-owned).
- No test for malformed instruction data length.
