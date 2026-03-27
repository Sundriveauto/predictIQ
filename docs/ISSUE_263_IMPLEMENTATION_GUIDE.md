# Issue #263: Invariant Tests Implementation Guide

## Quick Reference

### Core Invariant
```rust
sum(outcome_stakes) == total_staked
```
Must hold after every market operation.

### Test Categories (15 total)

| Category | Tests | Key Scenarios |
|---|---|---|
| **Basic Lifecycle** | 4 | Creation, single/multiple bets |
| **Cancellation/Refunds** | 3 | Full/partial refunds, multiple cancels |
| **Resolution/Payouts** | 2 | Winner payouts, multiple winners |
| **Multi-Outcome** | 2 | 3+ outcomes, boundary values |
| **Edge Cases** | 2 | Valid operations, multiple refunds |

### Test Execution
```bash
# All invariant tests
cargo test --lib modules::invariants_test

# Specific test
cargo test --lib modules::invariants_test::test_stake_conservation_cancellation_refunds
```

## Test Structure

### Helper Function
```rust
fn assert_stake_conservation(env: &Env, client: &PredictIQClient, market_id: u64)
```
- Sums all outcome stakes
- Compares to `market.total_staked`
- Fails test if conservation violated

### Test Pattern
```rust
#[test]
fn test_stake_conservation_[scenario]() {
    let (env, client, admin) = setup();
    // Create market
    // Perform operations
    assert_stake_conservation(&env, &client, market_id);  // At each step
    // Verify final state
}
```

## Critical Flows Tested

### 1. Bet → Cancel → Refund
```rust
client.place_bet(...);
assert_stake_conservation(...);
client.cancel_market_admin(...);
assert_stake_conservation(...);
client.withdraw_refund(...);
assert_stake_conservation(...);
// Result: total_staked == 0
```

### 2. Bet → Resolve → Payout
```rust
client.place_bet(...);
assert_stake_conservation(...);
client.resolve_market(...);
assert_stake_conservation(...);
client.claim_payout(...);
assert_stake_conservation(...);
// Result: total_staked == 0
```

### 3. Multi-Outcome Scenarios
- Bets distributed across outcomes
- Partial refunds from specific outcomes
- Proportional payouts to winners

## Safety Checks

### What These Tests Prevent
- ❌ Stake funds disappearing during refunds
- ❌ Extra stake creation during resolution
- ❌ Incorrect payout calculations
- ❌ State corruption from complex flows

### What They Guarantee
- ✅ All stake funds accounted for
- ✅ Correct winner payout distribution
- ✅ State consistency across operations
- ✅ Financial integrity of market system

## Integration Points

### With Existing Tests
- Complements `markets_test.rs` (functional tests)
- Works with `events_test.rs` (event verification)
- Integrates with `oracles_test.rs` (oracle validation)

### CI/CD Integration
- Run on every commit: `cargo test --lib modules::invariants_test`
- Fail build if conservation violated
- Include in release verification

## Troubleshooting

### Test Failures
- **Conservation violation**: Check stake accounting in market operations
- **Outcome stake mismatch**: Verify `get_outcome_stake()` implementation
- **Total stake mismatch**: Check `total_staked` updates

### Common Issues
- **Missing assertions**: Add `assert_stake_conservation()` after state changes
- **Boundary values**: Test with `i128::MAX`, `i128::MIN`, `0`
- **Multi-outcome**: Test with 2, 3, 5+ outcomes

## Files

- **Test Implementation**: `contracts/predict-iq/src/modules/invariants_test.rs`
- **Full Documentation**: `docs/ISSUE_263_INVARIANT_TESTS.md`
- **PR Description**: `PULL_REQUEST_ISSUE_263.md`

---

**Quick Start**: `cargo test --lib modules::invariants_test`