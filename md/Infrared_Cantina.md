# Introduction

A time-boxed security review of the **Infrared** protocol was done by **0xodus**, with a focus on the security aspects of the application's smart contracts implementation.

# Disclaimer

A smart contract security review can never verify the complete absence of vulnerabilities. This is a time, resource and expertise bound effort where I try to find as many vulnerabilities as possible. I can not guarantee 100% security after the review or even if the review will find any problems with your smart contracts. Subsequent security reviews, bug bounty programs and on-chain monitoring are strongly recommended.

# Severity classification

| Severity               | Impact: High | Impact: Medium | Impact: Low |
| ---------------------- | ------------ | -------------- | ----------- |
| **Likelihood: High**   | Critical     | High           | Medium      |
| **Likelihood: Medium** | High         | Medium         | Low         |
| **Likelihood: Low**    | Medium       | Low            | Low         |

**Impact** - the technical, economic and reputation damage of a successful attack

**Likelihood** - the chance that a particular vulnerability gets discovered and exploited

**Severity** - the overall criticality of the risk

# Findings Summary

| ID     | Title                                                                                        | Severity | Status    |
| ------ | -------------------------------------------------------------------------------------------- | -------- | --------- |
| [L-01] | Lack of Access Control on setIBGT and setRed could potentially lead to a DoS on the Protocol | Low      | Confirmed |

# Detailed Findings

# [L-01] Lack of Access Control on `setIBGT` and `setRed` could potentially lead to a DoS on the Protocol

## Description

The `setIBGT` and `setRed` functions set the addresses of the ibgt and red tokens that are used in the protocol. Lack of access control for the functions allows the attacker to effectively take contol of the two tokens.

The two functions should be called during deployment. However, since the tokens inherit the `ERC20PresetMinterPauser` contract and the functions check:

1. Whether the address provided is a zero address
2. If the address have already been set before
3. The address given has granted the Infrared contract the MINTER_ROLE.

In order to execute the attack, the attacker only needs to create their own ERC20Token that inherits the `ERC20PresetMinterPauser` contract and grant the infrared contract the `MINTER_ROLE` and then call the two functions and setting their token in both functions. Since the addresses can't be changed once set, the only way to solve this would be to upgrade the protocol with suitable logic. Although this is meant to be called immediately after deployment, an attacker could still frontrun the transaction to commit the attack.

This allows the attacker to effectively take control of the Minting and pausing of transfers, minting and Burning of the tokens. as they can later transfer the `MINTER_ROLE` to themselves as they are the admins.

## Impact Explanation

If the attacker is successful, their actions will greatly impact the protocol since they effectively control the Minting and can also pause the transfer, minting and burning of the ibgt and red tokens used in the protocol, this in turn can lead to a Denial of Service attack on the protocol. Therefore has High Impact

## Likelihood Explanation

The likelihood of this happening is low since there is no immediate financial benefit for the attacker and would also require the attacker to have a valid ERC20Token that is ERC20PresetMinterPauser and have given the MINTER_ROLE to the infrared contract.

## Recommedation

The following steps can be taken to mitigate the issue:

Add the onlyGovernor modifier to the two functions since the Governor address is a trusted entity

```diff
     /// @inheritdoc IInfrared
-   function setIBGT(address _ibgt) external {
+  function setIBGT(address _ibgt) external onlyGovernor{
    ...
      /// @inheritdoc IInfrared
-    function setRed(address _red) external {
+   function setRed(address _red) external onlyGovernor {
```

Or the addresses(ibgt and red ) can be set in the constructor and then modify their respective contracts to allow the `MINTER_ROLE` to be initially given to a trusted entity during deployment and then transfered to the infrared contract after its deployment.
