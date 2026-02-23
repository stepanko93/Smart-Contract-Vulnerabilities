# Layer3 Staking: Permanent DoS via SafeCast Overflow

## 📝 Overview
This repository contains a Proof of Concept (PoC) demonstrating a critical mathematical vulnerability in the Layer3 Staking contract. The flaw allows any user to permanently brick the contract, locking all staked funds and unclaimed rewards.

## 🔬 Vulnerability Details
The vulnerability stems from an unchecked inflation of the global `rewardPerShare` variable combined with a strict `uint128` downcast using OpenZeppelin's `SafeCast`.

1. **Dust Deposit Inflation:** If a user stakes a minimal amount (e.g., `4 wei`), the global `totalWeights` becomes extremely small (e.g., `1 wei`).
2. **Reward Calculation:** As time progresses, the `_calculatedRewardPerShare()` function divides the standard accumulated rewards by this dust `totalWeights`, causing the `uint256` result to inflate to an astronomically high value.
3. **SafeCast Revert (The DoS):** The `_updateReward` function, which is a prerequisite for almost all state-changing functions (`stake`, `withdraw`, `getReward`), attempts to downcast this inflated value into a `uint128` state variable:
   ```solidity
   _staker.rewardPerShareSnapshot = rewardPerShare.toUint128();
 4. Impact: The inflated value exceeds type(uint128).max. SafeCast catches this and reverts the transaction (SafeCastOverflowedUintDowncast). Because _updateReward reverts, the entire contract is permanently paralyzed.

      🛠️ Proof of Concept (Foundry)
    
The included StakingExploit.t.sol file simulates the attack on a local Foundry environment.

Steps to Reproduce:
1. Initialize a Foundry project.
2. Place StakingExploit.t.sol in your test/ directory.
3. Run the following command to execute the test and witness the forced revert:
 
   Bash
   
forge test --mt test_brickPool_safeCastOverflow -vvvv

The test intentionally expects a revert to PASS, proving the Denial of Service is executable.
