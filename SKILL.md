---
name: web3-security-suite
description: >
  All-in-one Web3 security toolkit — unified skill combining methodologies from pashov/skills,
  ZealynxSecurity/krait, JoranHonig/grimoire, CDSecurity, HackenProof, OpenZeppelin, Trail of Bits,
  and foundry-poc-mainnet-fork. Routes to the right mode based on your intent.
  Triggers: "x-ray", "recon", "pre-audit", "audit-prep", "readiness check", "audit", "scan contracts",
  "quick audit", "bug bounty", "triage report", "draft finding", "write finding", "foundry poc",
  "write poc", "reproduce bug", "hunt summary", "checkpoint", "secure dev", "develop contracts".
---

# Web3 Security Suite — Unified Skill

Combines methodologies from pashov/skills, ZealynxSecurity/krait, JoranHonig/grimoire,
CDSecurity/cdsecurity-skills, hackenproof-public/skills, cholakovvv/foundry-poc, OpenZeppelin/openzeppelin-skills,
and Trail of Bits/skills into one routing skill.

---

## Mode Detection

Read the user's request and route to exactly one mode:

| Trigger phrases | Mode |
|-----------------|------|
| "x-ray", "recon", "pre-audit report", "readiness report", "summarize this protocol" | → **X-RAY** |
| "audit-prep", "audit readiness", "prepare for audit", "pre-audit check", "NatSpec check" | → **AUDIT-PREP** |
| "audit", "scan", "check this contract", "review for security", "full audit" | → **AUDIT** (full 12-agent) |
| "quick audit", "quick scan", "fast audit", "krait" | → **QUICK-AUDIT** (4-phase krait) |
| "bug bounty", "triage", "triage report", "hackenproof", "scope check" | → **BOUNTY-TRIAGE** |
| "draft finding", "write finding", "document vulnerability", "write up this bug" | → **FINDING-DRAFT** |
| "foundry poc", "write poc", "reproduce bug", "fork mainnet", "exploit poc", "validate finding" | → **FOUNDRY-POC** |
| "hunt summary", "checkpoint", "session summary", "save progress", "what did we find" | → **HUNT-SUMMARY** |
| "develop contracts", "secure dev", "openzeppelin", "add pausability", "make upgradeable" | → **SECURE-DEV** |

If the mode is ambiguous, ask the user a one-question AskUserQuestion with all 9 modes as options.

---

## MODE: X-RAY (Pre-Audit Recon)

*Source: pashov/skills x-ray + krait recon*

Print banner:
```
██╗  ██╗      ██████╗  █████╗ ██╗   ██╗
╚██╗██╔╝      ██╔══██╗██╔══██╗╚██╗ ██╔╝
 ╚███╔╝ █████╗██████╔╝███████║ ╚████╔╝
 ██╔██╗ ╚════╝██╔══██╗██╔══██║  ╚██╔╝
██╔╝ ██╗      ██║  ██║██║  ██║   ██║
╚═╝  ╚═╝      ╚═╝  ╚═╝╚═╝  ╚═╝   ╚═╝
Pre-Audit Intelligence Report
```

### Phase 1 — Enumerate (parallel)

Run these in one message:
- `find . -name "*.sol" ! -path "*/test/*" ! -path "*/lib/*" ! -path "*/node_modules/*" ! -path "*/mocks/*" | sort`
- `wc -l $(find . -name "*.sol" ! -path "*/test/*" ! -path "*/lib/*") 2>/dev/null | tail -1`
- `cat foundry.toml 2>/dev/null || cat hardhat.config.* 2>/dev/null || echo "no config found"`
- `git log --oneline -20 2>/dev/null || echo "no git"`
- `git log --format="%ae" | sort | uniq -c | sort -rn | head -5 2>/dev/null`

### Phase 2 — Read & Classify (parallel agents)

Spawn 2 background agents:

**Agent A — Architecture & Invariants:**
```
You are a senior Web3 security researcher doing pre-audit recon.
Read all in-scope .sol files. For each contract identify:
1. Contract type (ERC20/721/1155, AMM, lending, bridge, governance, vault, staking, proxy)
2. External dependencies and integrations (oracles, other protocols, cross-chain)
3. Access control roles and trust model
4. State-changing entry points with value flow
5. Core invariants that MUST hold (e.g. "total shares <= total assets", "sum balances == totalSupply")
6. Economic attack surfaces (flash loans, price manipulation, MEV)
Output: structured markdown per contract.
```

**Agent B — Attack Surface & Risk Scoring:**
```
You are a smart contract hacker doing pre-audit recon.
Read all in-scope .sol files. For each file produce:
1. Risk score 1-10 (10=highest) with justification
2. Top 3 most suspicious code patterns or logic sections
3. Missing validations, unchecked returns, dangerous patterns
4. Composability risks with external protocols
5. Temporal risks (time-locks, epochs, cooldowns that could be abused)
6. Git-blame high-churn areas if git history available
Output: risk-ranked file table + detailed findings per high-risk area.
```

### Phase 3 — Synthesize Report

Write `x-ray/x-ray-report.md` with:
```markdown
# X-Ray Pre-Audit Report — [Project] — [Date]

## Overview
[Protocol type, purpose, 3-sentence summary]

## Codebase Stats
| Metric | Value |
|--------|-------|
| In-scope files | N |
| Total lines | N |
| Contracts | N |
| External deps | list |

## Threat Model
### Protocol Type Profile
[AMM / lending / bridge / staking / etc — typical attack vectors for this type]

### Attack Surface (Risk-Ranked)
| File | Risk | Key Concern |
|------|------|-------------|
| ... | 8/10 | ... |

### Trust Model
[Who can do what, admin keys, upgradeability]

## Invariants to Test
- [ ] invariant 1
- [ ] invariant 2
...

## Composability Dependencies
[External protocols, oracles, bridges this protocol trusts]

## Red Flags
[Top 5 most suspicious things found — specific file:line references]

## Recommended Audit Focus
[Where to spend most time]

## Test Coverage Assessment
[What tests exist, what's missing]
```

---

## MODE: AUDIT-PREP (Readiness Check)

*Source: CDSecurity/cdsecurity-skills audit-prep*

Print banner:
```
 █████╗ ██╗   ██╗██████╗ ██╗████████╗    ██████╗ ██████╗ ███████╗██████╗
██╔══██╗██║   ██║██╔══██╗██║╚══██╔══╝    ██╔══██╗██╔══██╗██╔════╝██╔══██╗
███████║██║   ██║██║  ██║██║   ██║       ██████╔╝██████╔╝█████╗  ██████╔╝
██╔══██║██║   ██║██║  ██║██║   ██║       ██╔═══╝ ██╔══██╗██╔══╝  ██╔═══╝
██║  ██║╚██████╔╝██████╔╝██║   ██║       ██║     ██║  ██║███████╗██║
╚═╝  ╚═╝ ╚═════╝ ╚═════╝ ╚═╝   ╚═╝       ╚═╝     ╚═╝  ╚═╝╚══════╝╚═╝
```

### Parallel Discovery (Turn 1)

Run in one message:
- Detect framework: `foundry.toml` / `hardhat.config.*`
- Find in-scope `.sol` (exclude `test/`, `script/`, `lib/`, `node_modules/`, `mocks/`)
- Find test files
- `which slither 2>/dev/null && echo SLITHER=yes || echo SLITHER=no`
- `which aderyn 2>/dev/null && echo ADERYN=yes || echo ADERYN=no`
- `forge coverage 2>/dev/null | tail -20 || echo "coverage-unavailable"`

### Spawn 3 Parallel Agents

**Agent A — Test Coverage & Quality (Phase 1+2):**
Check: coverage %, test file ratio, assertion density, fuzz tests present, invariant tests present, edge cases for critical paths. Score /100.

**Agent B — Documentation & Code Hygiene (Phase 3+4+6):**
Check: NatSpec on all public functions, SPDX headers, locked pragmas, no `console.log`, no TODO/FIXME, SafeERC20 used, reentrancy guards, proper error messages. Score /100.

**Agent C — Infrastructure & Deployment (Phase 5+7+8):**
Check: dependency versions pinned, no high-risk deps, deployment scripts exist, SECURITY.md present, scope.md/KNOWN_ISSUES.md, CI config, upgrade patterns documented. Score /100.

### Score & Report

| Phase | Weight |
|-------|--------|
| 1. Test Coverage | 15% |
| 2. Test Quality | 15% |
| 3. NatSpec Docs | 10% |
| 4. Code Hygiene | 10% |
| 5. Dependencies | 10% |
| 6. Best Practices | 15% |
| 7. Deployment | 10% |
| 8. Project Docs | 15% |

Verdict: 90-100 = Audit Ready | 75-89 = Almost Ready | 50-74 = Needs Work | <50 = Not Ready

Output a scored markdown table per phase, Quick Wins table (top 5 fixes), and overall verdict.

---

## MODE: AUDIT (Full 12-Agent Security Audit)

*Source: pashov/skills solidity-auditor*

Print banner:
```
██████╗  █████╗ ███████╗██╗  ██╗ ██████╗ ██╗   ██╗    ███████╗██╗  ██╗██╗██╗     ██╗     ███████╗
██╔══██╗██╔══██╗██╔════╝██║  ██║██╔═══██╗██║   ██║    ██╔════╝██║ ██╔╝██║██║     ██║     ██╔════╝
██████╔╝███████║███████╗███████║██║   ██║██║   ██║    ███████╗█████╔╝ ██║██║     ██║     ███████╗
██╔═══╝ ██╔══██║╚════██║██╔══██║██║   ██║╚██╗ ██╔╝    ╚════██║██╔═██╗ ██║██║     ██║     ╚════██║
██║     ██║  ██║███████║██║  ██║╚██████╔╝ ╚████╔╝     ███████║██║  ██╗██║███████╗███████╗███████║
╚═╝     ╚═╝  ╚═╝╚══════╝╚═╝  ╚═╝ ╚═════╝   ╚═══╝      ╚══════╝╚═╝  ╚═╝╚═╝╚══════╝╚══════╝╚══════╝
```

### Turn 1 — Discover

In one message (parallel):
- `find . -name "*.sol" ! -path "*/interfaces/*" ! -path "*/lib/*" ! -path "*/mocks/*" ! -path "*/test/*" ! -name "*.t.sol" | sort`
- `mktemp -d ./.audit-XXXXXX` → `{bundle_dir}`
- ToolSearch `select:Agent`

### Turn 2 — Build Bundles

Build `{bundle_dir}/source.md`: ALL in-scope .sol files with `### path` headers + fenced code blocks.

Then build 12 agent bundles = `source.md` + specialty file (inline the specialty description below):

| Bundle | Specialty |
|--------|-----------|
| agent-1 | **Math & Precision**: Overflow/underflow, rounding direction (always round against user in lending/vault), precision loss in division, mulDiv ordering, basis point math errors, uint128/uint256 casting truncation. |
| agent-2 | **Access Control**: Missing `onlyOwner`/role checks, broken role hierarchy, privilege escalation, front-running initialization, `tx.origin` misuse, signature replay, public functions that should be internal. |
| agent-3 | **Economic Security**: Sandwich attacks, price manipulation via spot price, flash loan attacks, fee-on-transfer tokens, rebasing tokens, donate attacks on vaults (ERC-4626 inflation), incentive misalignment, MEV extraction. |
| agent-4 | **Execution Trace**: Reentrancy (cross-function, cross-contract, read-only), callback hijacking, ERC777 hooks, ERC1155 callbacks, unsafe external calls, unchecked return values, gas griefing. |
| agent-5 | **Invariants**: Protocol-level invariants that must always hold (totalShares ≤ totalAssets, sum(balances) == totalSupply, etc.). Find code paths that violate them. State machine transitions that skip required states. |
| agent-6 | **Periphery & Integration**: Oracle manipulation (Chainlink stale data, TWAP too short, single-source), cross-chain bridge assumptions, protocol composability failures, token standard edge cases (zero address, self-transfer, zero amount). |
| agent-7 | **First Principles**: From-scratch fresh-eye review. Ignore the code's apparent intent — what does it actually do? Where does the code's behavior diverge from what a normal user would expect? Trust assumptions that aren't enforced. |
| agent-8 | **Asymmetry**: Deposit vs withdraw asymmetry, mint vs burn asymmetry, lock vs unlock asymmetry. Does every action have a symmetric reversal? Can a user lock funds irreversibly? One-way doors. |
| agent-9 | **Boundary & Edge Cases**: Block 0, zero amounts, zero address, max uint, single user (total supply = user balance), empty pool, first depositor, last withdrawer, exact equal values in comparisons. |
| agent-10 | **Numerical Gap Hunter**: Gaps between integer representations — values that are valid on input but invalid on output. e.g. a 1-wei remainder that's permanently locked, shares that round to 0, dust accumulation that overflows. |
| agent-11 | **Trust Gap Hunter**: Gaps between what the code assumes about actors vs what actors can actually do. Protocol assumes user is honest but doesn't enforce it. Admin assumed to be trusted but can rug. |
| agent-12 | **Flow Gap Hunter**: Gaps between the intended happy path and what the code actually allows. Missing steps in multi-step flows. Steps that can be skipped. Steps that can be replayed. Order dependencies not enforced. |

Print line counts for all bundles.

### Turn 3 — Spawn All 12 Agents (Background, Parallel)

For agents 1-9 (specialty prompt):
```
You are an attacker. Your specialty: [SPECIALTY FROM TABLE ABOVE].
Read {bundle_dir}/agent-N-bundle.md fully before producing findings.

A FINDING requires: file, function, root cause (one sentence), minimal fix (diff), proof (numbers/trace/quoted code).
A LEAD is a promising hypothesis you couldn't fully verify — still emit it, marked [LEAD].

Output for each issue:
**[FINDING/LEAD] #N — [Title]**
- File: path:line
- Function: name
- Root cause: [one sentence]
- Impact: [severity: Critical/High/Medium/Low] — [one sentence impact]
- Proof: [concrete trace or quoted code showing the bug]
- Fix: [minimal fix as diff or description]
- Confidence: [0-100]

Don't skim. Trust your discomfort.
```

For agents 10-12 (gap-hunter prompt):
```
You are an attacker specialized in finding gaps between system layers.
Your gap-hunter specialty: [SPECIALTY FROM TABLE ABOVE].
Read {bundle_dir}/agent-N-bundle.md fully.

A gap finding requires: the TWO layers where the gap exists, root cause at the seam, proof the gap is exploitable.
Still emit LEADs for unverified gaps.

Output same format as above, plus:
- Gap Seam: [Layer A] ↔ [Layer B]
```

Wait for all 12 to complete before Turn 4.

### Turn 4 — Deduplicate, Gate & Report

1. **Dedup**: Group by (Contract, function, bug-class). Merge synonymous findings. Keep best per group, annotate `[agents: N]`. NEVER merge across different functions.

2. **Multi-fix preservation**: If 2+ agents suggest different fixes for same issue, present all as Option A / Option B.

3. **Completeness gate**: Every (Contract, function) from any raw finding MUST appear in final output. Print: `Completeness: N unique (Contract, function) in raw, N covered.`

4. **Severity gate per finding**:
   - Is there a plausible execution path to exploit? (BLOCKS if no)
   - Can an unprivileged attacker trigger it? Or what privilege is needed?
   - What is the realistic impact? (funds at risk, protocol broken, DoS)
   - Demote to LEAD if no concrete proof.

5. **Output format**:

```markdown
## Audit Report — [Project] — [Date]

### Summary
| Severity | Count |
|----------|-------|
| Critical | N |
| High | N |
| Medium | N |
| Low | N |
| Informational | N |

### Findings

#### [C-01] Title
- **Severity**: Critical
- **Contract**: path:line
- **Function**: name()
- **Root Cause**: [one sentence]
- **Impact**: [concrete impact]
- **Proof**:
```solidity
// trace or coded example
```
- **Fix**:
```diff
+ // fix
- // vulnerable code
```
- **Agents**: [1, 3, 7]
```

Auto-clean: `rm -rf {bundle_dir}` after report.

---

## MODE: QUICK-AUDIT (4-Phase Krait Methodology)

*Source: ZealynxSecurity/krait*

Faster single-agent audit using Krait's structured 4-phase pipeline. No parallel agents.

### Phase 0 — Recon (5 min)

Run `find . -name "*.sol" ! -path "*/test/*" ! -path "*/lib/*" | sort`. For each contract:
- Type classification (DeFi primitive, vault, token, bridge, governance, etc.)
- Risk score 1-10
- Entry points count
- External calls count

Produce risk-ranked file list. Focus remaining phases on top 3 highest-risk files.

### Phase 1 — Detection (3-pass, 4 lenses)

For each high-risk file, run 3 passes:
1. **Structural pass**: function signatures, visibility, modifiers, state variables
2. **Flow pass**: trace value flows, identify all paths to state changes
3. **Adversarial pass**: how would an attacker abuse each entry point?

Apply 4 mindsets per pass:
- **Optimizer**: find economic value to extract
- **Insider**: what can a privileged actor steal?
- **User**: what can a normal user do that breaks invariants?
- **Protocol**: what external protocols can be manipulated against this?

### Phase 2 — State Analysis

For each pair of state variables that should be coupled (e.g. `totalShares` + `totalAssets`):
- Can they desync? Under what conditions?
- Are mutations always atomic?
- Is there masking code that hides a desync?

### Phase 3 — Kill Gates (8 gates, must pass all for HIGH/CRITICAL finding)

1. Code path exists to reach vulnerable function ✓/✗
2. Prerequisites are achievable by attacker class ✓/✗
3. State can be set up in a single transaction ✓/✗ (or multi if flash loan)
4. Impact is concrete and quantifiable ✓/✗
5. No revert guards block the attack before impact ✓/✗
6. Fix is non-trivial (not already there, just missed) ✓/✗
7. Impact persists after transaction (or value is extracted) ✓/✗
8. Not already a known/acknowledged issue — check whitepaper, auditor briefs, CLAUDE.md, KNOWN_ISSUES files in the repo (`git log --all --oneline` first to find any context docs) ✓/✗
9. Unprivileged attacker triggers it (not maker/admin/owner/the one who misconfigured it) ✓/✗
10. A third party loses real funds — not the person who set the parameters, not just event spam ✓/✗

Finding that fails any gate → demoted to MEDIUM/LOW/INFO.

### Phase 4 — Report

Same format as AUDIT mode but no parallel agents. Include kill gate results per finding.

---

## MODE: BOUNTY-TRIAGE (Bug Bounty & HackenProof)

*Source: hackenproof-public/skills + shuvonsec/claude-bug-bounty*

When the user describes or pastes a vulnerability finding:

### Step 0 — 3-Point Hard Pre-Filter (MANDATORY — run BEFORE writing any PoC or analysis)

Fail **any one** → discard immediately, do not proceed.

1. **Unprivileged attacker triggers it** — the attack must be executable by an external address with no special role (not maker, not admin, not owner, not the one who set the parameters). If the only way to trigger the bug is for the victim to misconfigure their own state → DISCARD.

2. **A third party loses real funds** — tokens stolen, drained, or permanently locked. The victim must be someone *other than* the person who triggered or configured the vulnerable state. Event spam, UI confusion, or "misleading events" without fund loss → DISCARD.

3. **Not a documented trade-off** — check ALL of these before analyzing code:
   - Project whitepaper / docs
   - Any auditor brief, KNOWN_ISSUES, or CLAUDE.md in the repo (`git log --all --oneline`, `find . -name "*.md" | xargs grep -l "known\|acknowledged\|by design"`)
   - Previous audit reports linked in the README
   - The program's HackenProof scope page exclusions
   If the behavior is acknowledged → DISCARD.

**What to hunt for instead:**
- Unprivileged attacker steals / drains / permanently locks a *third party's* funds
- State corruption that persists and affects real balances for other users
- Chain-level impact: consensus failures, node crashes, transaction processing halted
- Protocol assumption violations exploitable by anyone regardless of who set the parameters

**Lesson learned (Aqua audit):** `ship()` multi-call and `dock([])` empty-array were BOTH explicitly documented in the project's auditor brief as "by design / maker awareness required." Both submissions were wasted. Always read the repo docs first.

### Step 1 — Validate Before Triaging

Run these gates in order (fail = stop + tell user what's missing):
1. **Scope Gate**: Is the target contract/function in scope? Is the impact category in scope?
2. **Commit/Version Gate**: Does the finding reference a specific deployed address or commit?
3. **PoC Gate**: Is there a reproducing PoC (tx hash, test, or step-by-step trace)?
4. **Duplicate Gate**: Is this root cause + impact combination already known?

### Step 2 — Technical Validation

Read the relevant code. Confirm:
- Vulnerable code path exists
- Attack is executable by the specified actor class (permissionless vs restricted)
- Impact is as claimed (check for mitigating factors)
- Loss/impact is quantifiable

### Step 3 — Severity Assessment (CVSS-inspired for Web3)

| Factor | Options |
|--------|---------|
| Attack vector | Permissionless / Whitelisted / Admin-only |
| Preconditions | None / Specific state required / Unlikely state |
| Impact | Funds lost / Frozen / Protocol broken / Data leak / DOS |
| Scope | All users / Subset of users / Single user |
| Reversibility | Irreversible / Reversible by admin / Self-reversible |

Map to: Critical (funds lost, permissionless) / High / Medium / Low / Informational

### Step 4 — PoC Guidance

If PoC is missing or weak, generate skeleton:
```
For HackenProof submission, provide:
1. Affected contract: [address on chain]
2. Vulnerable function: [name + selector]
3. Attack steps:
   a. [Setup state]
   b. [Call function with params]
   c. [Observe impact]
4. Testnet reproduction: [Hardhat/Foundry test or tx hash]
5. Expected loss calculation: [formula]
```

### Step 5 — Submission Checklist

```markdown
## Bug Bounty Submission Checklist
- [ ] Title: [Protocol] — [Vulnerability Type] — [Impact]
- [ ] Severity: [Critical/High/Medium/Low]
- [ ] Affected contract: [address]
- [ ] Vulnerable function: [name]
- [ ] Root cause: [one sentence]
- [ ] Attack scenario: [numbered steps]
- [ ] Impact: [concrete loss/effect]
- [ ] PoC: [test file or tx hash]
- [ ] Fix suggestion: [optional]
- [ ] Scope confirmation: [yes, in scope per program rules]
```

---

## MODE: FINDING-DRAFT (Document a Vulnerability)

*Source: JoranHonig/grimoire finding-draft*

When the user describes a vulnerability they want to document:

### Step 1 — Gather Context (ask user if missing)

- What contract/function is affected?
- What is the root cause (one sentence)?
- What is the impact (who loses what)?
- What attacker class is required (permissionless / whitelisted / admin)?
- Does a PoC exist?

### Step 2 — Construct Title

Format: `[Where] — [How] — [What]`
Examples:
- `LendingPool.liquidate() — Missing slippage check — Liquidator can receive less collateral than expected`
- `Vault.deposit() — ERC4626 inflation attack — First depositor can steal subsequent depositors' funds`

Present to user for confirmation.

### Step 3 — Severity Classification

| Severity | Criteria |
|----------|----------|
| Critical | Direct fund loss, permissionless, >1% of TVL |
| High | Fund loss with preconditions, or >minor but <critical |
| Medium | Temporary freeze, partial loss, or requires specific conditions |
| Low | Best-practice violation, unlikely scenario, minimal impact |
| Informational | Code quality, gas optimization, documentation |

### Step 4 — Draft Finding File

Write to `findings/[severity]-[slug].md`:

```markdown
---
title: "[Title]"
severity: [Critical/High/Medium/Low/Informational]
type: [arithmetic | access-control | reentrancy | logic | oracle | flash-loan | other]
status: [draft | confirmed | disputed | acknowledged | fixed]
contracts:
  - path/to/Contract.sol:line
---

## Description

[2-4 paragraphs covering: what component, what flaw, what preconditions, what impact]
[State the minimum attacker class required]

## Proof of Concept

```solidity
// Foundry test or step-by-step trace
```

## Impact

[Concrete impact — who loses what, how much, how irreversible]

## Recommendation

[Specific minimal fix — what to add/change/remove]

## References

- [Link to similar findings, EIPs, audit reports]
```

### Step 5 — Suggest Follow-ups

- No PoC exists? → Suggest switching to FOUNDRY-POC mode
- Similar contracts? → Check for same pattern in sibling contracts
- Pattern generalizable? → Note for future audit checklist

---

## MODE: FOUNDRY-POC (Mainnet Fork Proof of Concept)

*Source: cholakovvv/foundry-poc-mainnet-fork*

### Required Inputs (ask if missing)

1. Vulnerability description (root cause + attack path + impact)
2. Chain (ethereum, base, arbitrum, optimism, polygon, bnb, avalanche)
3. Fork target (block number or `latest`)
4. Deployed contract addresses for every contract in the attack path

### Classify Before Writing (MANDATORY)

State out loud which category this finding is:
- **(a) Frozen historical**: Bad state already on-chain; fork at post-vulnerable block and assert the broken state
- **(b) Forward-looking**: Pattern will repeat; start from an actor in safe state and reproduce the full attack chain
- **(a+b) Both**: Primary PoC is (b); include (a) assertion as additional evidence

### Hard Rules

1. **Foundry only** — even if target repo uses Hardhat
2. **Real contracts only** — no mocks, no stubs; bind all protocol addresses as `constant`
3. `setUp()` MUST start with `vm.createSelectFork(RPC_URL, blockNumber)`
4. Test must execute the full path from trigger to realized impact
5. Attacker/recipient addresses use `makeAddr()` — protocol addresses must be real deployed addresses

### PoC Template

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/Test.sol";

// [Protocol name] PoC — [Vulnerability title]
// Finding: [one-line description]
// Chain: [chain name] | Block: [block number]

interface IVulnerableContract {
    // only the functions actually called in the attack
}

contract [Protocol]PoC is Test {
    // Protocol addresses (mainnet)
    address constant VULNERABLE_CONTRACT = 0x...;
    address constant TOKEN = 0x...;

    // Test actors
    address attacker;

    string constant RPC_URL = "[chain rpc]";
    uint256 constant FORK_BLOCK = [block];

    function setUp() public {
        vm.createSelectFork(RPC_URL, FORK_BLOCK);
        attacker = makeAddr("attacker");
    }

    function test_exploit() public {
        // 1. Setup: state the starting conditions
        // 2. Attack: execute each step with vm.prank where needed
        // 3. Assert: prove the impact
        
        uint256 attackerBalanceBefore = IERC20(TOKEN).balanceOf(attacker);
        
        vm.startPrank(attacker);
        // [attack steps]
        vm.stopPrank();
        
        uint256 attackerBalanceAfter = IERC20(TOKEN).balanceOf(attacker);
        assertTrue(attackerBalanceAfter > attackerBalanceBefore, "Exploit failed");
        
        console.log("Profit:", attackerBalanceAfter - attackerBalanceBefore);
    }
}
```

Run with: `forge test --match-test test_exploit -vvvv --fork-url [RPC] --fork-block-number [BLOCK]`

---

## MODE: HUNT-SUMMARY (Session Checkpoint)

*Source: JoranHonig/grimoire hunt-summary*

Capture a timestamped checkpoint of everything found so far in this audit session.

### Step 1 — Get timestamp
`date +%Y-%m-%d-%H%M%S`

### Step 2 — Mine the conversation

Review the full session transcript. Extract:
- **Confirmed findings**: issues with proof, severity, location
- **Interesting leads**: promising but unverified hypotheses
- **Dead ends**: paths ruled out (with WHY — so next session doesn't re-walk them)
- **Open questions**: blocked on external info, user clarification, or time
- **Next actions**: prioritized concrete steps for the next session

### Step 3 — Write checkpoint

Write to `findings/hunt-summary-[timestamp].md`:

```markdown
# Hunt Summary — [Target] — [YYYY-MM-DD HH:MM]

> Session focus: [what this session set out to do]

## Confirmed Findings
[For each: title, severity, location, one-line impact, proof artifact path]

## Interesting Leads
[For each: hypothesis, where to look, what would confirm/refute it, confidence: low/medium/high]

## Dead Ends
[For each: what was checked, why it's NOT a bug — essential to avoid re-walking]

## Open Questions
[What needs external info, user clarification, or scope confirmation]

## Artifacts Created
[All files written this session]

## Next Actions (prioritized)
1. [Most impactful next step]
2. ...
```

### Step 4 — Present summary

Show: output path + section counts (e.g., "2 confirmed, 4 leads, 3 dead ends, 5 next actions")

---

## MODE: SECURE-DEV (OpenZeppelin Secure Development)

*Source: OpenZeppelin/openzeppelin-skills develop-secure-contracts*

### Core Rule

**Never write custom code for something OpenZeppelin already provides.** Before writing any logic:
1. Search project for existing OZ imports
2. Check `lib/openzeppelin-contracts/` or `node_modules/@openzeppelin/contracts/`
3. Use the library component. Override only virtual functions.

### Common Patterns

| Need | OZ Component | Never write |
|------|-------------|-------------|
| Ownership | `Ownable` | `require(msg.sender == owner)` |
| Roles | `AccessControl` | Custom role mapping |
| Pause | `Pausable` or `ERC20Pausable` | Custom paused modifier |
| Reentrancy guard | `ReentrancyGuard` | Custom mutex |
| Safe ERC20 | `SafeERC20` | Direct `.transfer()` |
| ERC20 | `ERC20` | Custom transfer logic |
| Upgradeable | `UUPSUpgradeable` / `TransparentUpgradeableProxy` | Custom proxy |
| Timelock | `TimelockController` | Custom delay |

### Workflow

1. Read user's existing contracts before suggesting anything
2. Identify what pattern they need
3. Find it in the installed OZ library (read the actual source)
4. Show the minimal change: import + inherit + override only what's needed
5. Never copy-paste library source into user's contract — always import

### Generator Reference (for Solidity + Foundry/Hardhat)

```bash
# Generate reference implementation
npx @openzeppelin/contracts-cli generate --name MyToken --features pausable,mintable
# Diff against their current contract to see what's needed
# Apply only the missing pieces
```

---

## Usage Examples

```
/web3-security-suite x-ray              # pre-audit recon report
/web3-security-suite audit-prep         # readiness scoring
/web3-security-suite audit              # full 12-agent parallel audit
/web3-security-suite quick-audit        # fast 4-phase krait audit
/web3-security-suite bug-bounty         # triage + submission checklist
/web3-security-suite draft finding      # document a vulnerability
/web3-security-suite foundry poc        # mainnet fork PoC
/web3-security-suite hunt summary       # session checkpoint
/web3-security-suite secure dev         # OZ-based secure development
```

Or just describe what you need and the skill routes automatically.

---

*Combined from: pashov/skills · ZealynxSecurity/krait · JoranHonig/grimoire · CDSecurity/cdsecurity-skills · hackenproof-public/skills · cholakovvv/foundry-poc-mainnet-fork · OpenZeppelin/openzeppelin-skills · shuvonsec/claude-bug-bounty · Trail of Bits/skills*
