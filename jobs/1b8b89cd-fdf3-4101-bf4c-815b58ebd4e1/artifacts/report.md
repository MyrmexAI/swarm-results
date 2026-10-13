# jitoSOL, mSOL, and bSOL: liquid staking on Solana

Short comparison of three Solana liquid staking tokens (LSTs). Figures are protocol parameters and design choices, not live APY. Staking yield moves with Solana inflation, validator performance, and (for jitoSOL) MEV. This is not investment advice.

## How liquid staking works on these tokens

A holder deposits SOL (or, in some flows, an existing stake account) and receives an SPL token that represents a share of a stake pool. The pool delegates that SOL to many validators. Inflation rewards (and, for jitoSOL, MEV and related tips) increase the SOL backing each LST, so the token’s SOL exchange rate rises. Holders do not claim staking rewards manually. [1][2][3][7]

Solana epochs are 432,000 slots, with a target slot time of 400 ms (about two days), but actual epoch length varies. [4] Direct unstake therefore waits through deactivation, typically described as about one epoch or 2–3 days. [2][3][5]

## Comparison

| | **jitoSOL** | **mSOL** | **bSOL** |
| --- | --- | --- | --- |
| Issuer | Jito (JitoSOL stake pool; Jito DAO holds the manager role) [1][6] | Marinade Finance (DAO-controlled liquid staking program) [3][7] | BlazeStake / SolBlaze (SPL stake-pool token) [8][9] |
| Mint | `J1toso1uCk3RLmjorhTtrVwY9HJ7X8V9yYac6Y7kGCPn` [5] | `mSoLzYCxHdYgdzU16g5QSh3i5K3z3KZK7ytfqcJm7So` [8] | `bSo13r4TkiE4KumL71LsHTPpL2euBYLFx6h9HP3piy1` [8] |
| Pool / program shape | SPL Stake Pool `Jito4APyf642JPZPx3hGc6WWJ8zPKtRbRs4P815Awbb` [5] | Marinade’s own Anchor liquid-staking program, not a plain SPL stake pool [7] | SPL stake-pool token; app at stake.solblaze.org [8] |
| How yield is earned | Solana inflation rewards **plus** MEV auction proceeds passed to the pool. Docs also describe priority-fee distributions. Rewards compound into the JitoSOL/SOL rate. Jito’s rewards UI has cited roughly 5–7% staking APY and 1–2% MEV APY as illustrative ranges, and says those figures are approximate. [1][2][5] | Solana inflation rewards on delegated stake, auto-compounded into the mSOL price versus SOL. No protocol MEV share is part of the design in the sources used here. Extra yield only if the holder uses mSOL in DeFi. [3][4] | Solana inflation rewards on delegated stake, accruing into bSOL. Governance texts treat pool APY as staking yield after validator and pool commission, not as a separate MEV product. [9][10] |
| Protocol fee on rewards | Annual management fee equal to **4% of total rewards** (staking and MEV, after validator commission). Jito says that is about **0.3% of deposited SOL per year**. [5] | On-chain state has a `reward_fee` applied to rewards, with a hard cap of **10%** of rewards (`MAX_REWARD_FEE`). The live basis-point value is an account field, not a constant in the source reviewed. [7] | Epoch (pool) commission has been governed in a **0–5%** range. SBGP-0009 described a then-current **5%** epoch commission split 2.5% / 2.5% between SolBlaze and the DAO, and proposed a temporary **0%** until 1 March (extendable). SBGP-0011 lets SolBlaze set commission inside 0–5% without a full proposal. Confirm the live rate in the app. [9][11] |
| Deposit / withdrawal fees | **0.1%** only on direct unstake via the site. Selling on a DEX (Jito points to Jupiter) avoids that fee but takes market spread. Stake-account deposits can sit in the Interceptor for **10 hours** (early claim fee falls from about 10% to 0%). [5] | Instant “liquid unstake” uses an internal SOL/mSOL pool. Source comments set the curve at **3% max to 0.3% min** initially, with **25%** of that fee to the treasury. Delayed unstake is the zero-fee path in user docs (2–3 days). The program also has deposit and delayed-unstake fee fields, each capped at **0.2%**. [3][7] | Not specified as a fixed public schedule in the governance proposals reviewed. SPL stake-pool withdrawals follow the usual epoch delay or a DEX sale. [8][9] |
| Validator set | **StakeNet / Steward**: top **400** by on-chain score, equal target stake. Binary filters include inflation commission ≤ **5%** over 30 epochs, MEV commission ≤ **10%**, no historical commission above **50%**, vote participation ≥ **0.97**, not in the superminority (top ~33.3% of stake), BAM connectivity, and acceptable tip/priority-fee merkle authorities. Ranking prefers lower inflation commission, then lower MEV commission, then age, then vote credits. Governance can blacklist. [5][6][12] | Open-source scoring bot. 2022 user docs: **400+** validators scored on performance, commission, and decentralization. Glossary: commission allowed up to **10%**, with a score bonus for lower commission; manual blacklist for end-of-epoch commission hikes and vote-credit cheating. Separate from Marinade Select / Marinade Native. [3][4] | Mix of algorithmic stake, **veBLZE gauges**, a high-performance slice, and BlazeBoost matching for custom liquid staking. SBGP-0009 proposed 50% / 20% / 20% / 10% across those buckets and a preference for **0% inflation and 0% MEV commission**. Later proposals (0013, 0017, 0018) shifted the algorithmic slice toward a **Verified Validators** set (veBLZE gate, anti-sybil review) with roughly equal stake, **2×** for 0% commission validators. A rebalancer bot applies the policy. Max validator commission for the pool has been governed in **0–10%**. [9][10][11][13] |
| Who picks validators | On-chain Steward; parameters set by Jito DAO; keepers are permissionless. [6] | Off-chain delegation-strategy pipeline plus on-chain validator manager. Marinade can blacklist. [4] | Off-chain rebalancer plus DAO votes. SBGP-0011 also gives the team discretion inside stated ranges. [9][11] |
| Governance token | JTO (Jito DAO) [6] | MNDE [3] | BLZE / veBLZE (lock up to 5 years for full vote weight) [14] |

## Yield in more detail

**jitoSOL.** Inflation rewards are the usual Solana staking yield. On top of that, searchers bid in Jito’s auction for transaction sequences; auction proceeds are shared with validators and stakers after commission. Jito says the pool is paid from auction amounts, so holders are not themselves taking MEV trading risk. MEV is volatile: it depends on volume, volatility, and searcher competition. A separate technical FAQ describes a TipRouter split of **97% of MEV to validators/stakers and 3% protocol fee**, and a **1.5%** fee on priority-fee distributions. That MEV split is not the same line item as the **4% of total rewards** management fee in the general FAQ; both are Jito docs and should be read as distinct charges unless Jito consolidates them. [1][2][5]

**mSOL.** Yield is staking rewards only, reflected as a rising mSOL/SOL price each epoch. Marinade’s own APR/APY formulas are estimates from inflation, credits, stake, and commission, and the glossary calls them optimistic. The 2022 “about 6.2% APY” figure in the mSOL 101 post is historical, not a current quote. [3][4]

**bSOL.** Yield is likewise staking rewards left in the pool. Governance has treated validator commission and pool epoch commission as the main drags, and has also noted that frequent rebalancing can cost APY (SBGP-0018 estimated about **11 bps**, from a **2%** drag on rewards under a rotation model). [9][10]

Holding the LST in a wallet does not pay a second yield. Lending, LPing, or “looping” can add return and adds smart-contract, liquidation, and impermanent-loss risk. Marinade’s own guide says so explicitly. [3]

## Fees that actually hit the staker

Validator commission is taken before the LST holder’s share. Jito excludes high-commission validators and then charges 4% of what remains. [5] Marinade scores validators down as commission rises toward 10%. [4] BlazeStake has pushed the set toward 0% commission and 0% MEV commission and has run its own epoch commission between 0% and 5%. [9][11]

Exit costs differ:

- jitoSOL: 0.1% if you unstake in the app; a DEX sale is instant and fee-free from Jito’s side, but you take spread and slippage. [5]
- mSOL: delayed unstake at the protocol rate with no unstake fee in user docs; instant unstake pays a liquidity-curve fee (comments: 0.3–3%). [3][7]
- bSOL: epoch-delayed redemption or a market sale; a published withdrawal percentage was not in the sources used here. [8][9]

## Validator set and decentralization

All three spread stake instead of sending it to one validator, but the policies diverge.

Jito’s set is the most mechanical: 400 validators, equal delegation, hard exclusion of the superminority, and a requirement to run Jito’s MEV stack (and, in current parameters, BAM). That improves MEV capture and blocks the largest validators, but it also means stake cannot go to non-Jito clients. [1][5][12]

Marinade’s published goal is a large set scored for performance, commission, and geographic decentralization, with a public blacklist for commission rugs and credit cheating. Scoring runs off-chain and is open source. [3][4]

SolBlaze started as a broad “staker of last resort” and then narrowed part of the stake toward verified, contribution-screened validators because the open set attracted suspected sybils. A material share is still directed by veBLZE gauges and by custom-staking boosts, so token holders and validators who promote the pool influence who gets stake. [9][10][13][14]

## Main risks

Shared risks:

- **Smart-contract and admin risk.** A bug or a compromised manager key can lose or freeze stake. Jito relies on the Solana Labs stake-pool program and its audits, and says the pool is non-custodial. [6] Marinade publishes audits (Kudelski, Ackee, Neodyme, Sec3, and a later Neodyme review) and caps some fees in code, but upgrade authority and the validator-manager key still matter. The program limits how much stake can be moved per epoch if that key is stolen. [7] bSOL governance shows the operating team can change commission and delegation inside voted ranges. [11]
- **Validator performance and commission.** Delinquency, skipped slots, or a commission increase reduce yield. Jito and Marinade try to drop bad validators; a lag still costs an epoch of rewards. [4][12]
- **Liquidity and depeg versus the exchange rate.** The LST’s market price can trade below the SOL it can be redeemed for. Large exits then wait an epoch or pay a DEX discount. mSOL’s instant path can also widen toward the 3% side of its fee curve when pool SOL is thin. [3][5][7]
- **Solana staking risks.** Rewards vary with inflation and epoch length. Jito’s FAQ states that slashing does not currently exist on Solana, and treats critical contract bugs as the severe loss case. [5]
- **DeFi risk is optional but common.** Using the LST as collateral can be liquidated even while the staking position is healthy. [3][5]

Token-specific risks:

- **jitoSOL.** MEV yield is unstable and can be near zero in quiet markets. The design depends on validators running Jito software and on tip distribution; a validator that fails merkle-authority or priority-fee checks is dropped, but a distribution bug would hit rewards. The Interceptor exists because stake-account deposits were used as “toxic flow” against the pool’s liquidity. Direct unstake costs 0.1%. [1][5]
- **mSOL.** Custom program rather than the shared SPL stake pool, so the audited Marinade program is its own trust surface. Liquid-unstake fees rise when the internal pool is unbalanced. Delegation is off-chain; a bad bot or a blacklist error mis-allocates stake, though the per-epoch move cap is meant to limit damage. [4][7]
- **bSOL.** Delegation policy has changed repeatedly by DAO proposal, and SBGP-0011 delegates day-to-day fee and commission-cap changes to the team (epoch commission 0–5%, validator commission cap 0–10%). Gauge and verified-validator slices can concentrate stake relative to a pure score. Suspected sybils were a stated reason for the 0013 redesign. Pool commission is not fixed at zero. [9][10][11][13]

## Bottom line

- Choose **jitoSOL** if the goal is staking yield plus a share of Solana MEV, with a fixed 400-validator StakeNet set and a published 4% cut of rewards. Accept MEV variance and the Jito-client requirement. [1][5][12]
- Choose **mSOL** if the goal is a long-running Marinade LST with a large scored set and a documented instant-versus-delayed exit. Yield is inflation rewards after validator commission and Marinade’s reward fee, not MEV. [3][4][7]
- Choose **bSOL** if the goal is BlazeStake’s decentralization-and-gauges design and BLZE governance over who is staked to. Read the current epoch commission in the app; the voted band is 0–5%, not a single permanent rate. [9][11]

## Sources

1. Jito, “Introduction to Jito.” https://github.com/jito-foundation/jito-omnidocs/blob/master/jitosol/introduction-to-jito/index.md
2. Jito, “Maximum Extractable Value.” https://github.com/jito-foundation/jito-omnidocs/blob/master/jitosol/jitosol-liquid-staking/maximum-extractible-value/index.md
3. Marinade, “mSOL 101: How to get started with liquid staking on Solana” (28 Feb 2022), site export. https://github.com/marinade-finance/marinade-staging.webflow/blob/main/blog/msol-101-how-to-get-started-with-liquid-staking-on-solana.html
4. Marinade delegation-strategy glossary (commission cap, blacklist, APR/APY, epoch length). https://github.com/marinade-finance/delegation-strategy-2/blob/master/glossary.md
5. Jito, “General FAQs” (fees, StakeNet top 400, mint and pool addresses, Interceptor, unstake fee, reward ranges). https://github.com/jito-foundation/jito-omnidocs/blob/master/jitosol/faqs/general-faqs/index.md
6. Jito, security overview and StakeNet introduction (non-custodial pool, DAO manager, Steward). https://github.com/jito-foundation/jito-omnidocs/blob/master/jitosol/jitosol-liquid-staking/security/overview/index.md and https://github.com/jito-foundation/jito-omnidocs/blob/master/stakenet/introduction/index.md
7. Marinade liquid-staking program: `state/mod.rs` and `state/liq_pool.rs` (reward fee cap, liquidity-pool fee curve, deposit/unstake fee caps, stake-move limit) and README audits. https://github.com/marinade-finance/liquid-staking-program
8. Solana token list entries for mSOL and bSOL (names, mints, websites). https://github.com/solana-labs/token-list/blob/main/src/tokens/solana.tokenlist.json
9. SolBlaze DAO, SBGP-0009 “Increase Pool APY” (delegation mix, 5% epoch commission, temporary 0% commission, 0% MEV-commission slice). https://github.com/SolBlazeOrg/governance-proposals/blob/master/proposals/0009-increase-pool-apy.md
10. SolBlaze DAO, SBGP-0018 “Optimize Delegation and Incentives” (equal stake across verified validators, 2× for 0% commission, rebalancing drag). https://github.com/SolBlazeOrg/governance-proposals/blob/master/proposals/0018-optimize-delegation-and-incentives.md
11. SolBlaze DAO, SBGP-0011 “Dynamically Adjust Protocol Parameters” (epoch commission 0–5%, validator commission cap 0–10%). https://github.com/SolBlazeOrg/governance-proposals/blob/master/proposals/0011-dynamically-adjust-protocol-parameters.md
12. Jito StakeNet parameters and scoring (400 validators, commission and delinquency thresholds, superminority). https://github.com/jito-foundation/jito-omnidocs/blob/master/stakenet/jito-steward/parameters/index.md and https://github.com/jito-foundation/jito-omnidocs/blob/master/stakenet/jito-steward/validators/scoring-system/index.md
13. SolBlaze DAO, SBGP-0013 “Revamp Delegation Strategy” (verified validators, sybil concern, veBLZE requirement). https://github.com/SolBlazeOrg/governance-proposals/blob/master/proposals/0013-revamp-delegation-strategy.md
14. SolBlaze DAO, SBGP-0001 “Stake Gauges” (veBLZE gauge share of delegation). https://github.com/SolBlazeOrg/governance-proposals/blob/master/proposals/0001-stake-gauges.md

Jito technical FAQ (TipRouter 97/3 MEV split and priority-fee fee): https://github.com/jito-foundation/jito-omnidocs/blob/master/jitosol/faqs/technical-faqs/index.md

Live rates (APY, TVL, current bSOL commission, Marinade `reward_fee` basis points) should be read from the protocol apps or on-chain state. Docs and proposals above do not freeze those numbers.
