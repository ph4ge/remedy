## Bug Description

The bug lies within the `DegenSubaccount.sol` contract. The following three functions do not enforce a check on the amount that can be retrieved:
- `requestWithdrawSecondaryAsset(uint256 secondaryAmount)`
- `executeWithdrawSecondaryAsset()`
- `function fastWithdrawSecondaryAsset(address to, uint256 secondaryAmount)`



## Impact

## Risk Breakdown
Difficulty to Exploit: medium
Weakness: underflow
Remedy Vulnerability Scoring System 1.0 Score: 4.6
RVSS:1.0/CR:X/IR:L/AR:X/MAV:N/MAC:L/MPR:H/MUI:N/MS:C/MC:N/MI:H/MA:N

It is acknowledged that the exploitability surface is reduced because of the `onlyGlobalOperator` modifier.

## Recommendation

Harden the check before withdrawal:

In the `DegenSubaccount.sol` contract, modify the concerned functions in this vein:

```solidity
function requestWithdrawSecondaryAsset(uint256 secondaryAmount) external onlyGlobalOperator {

// only Global operator can request JUSD

(uint256 maxWithdrawValue,) = getMaxWithdrawAmount(address(dealer));

require(secondaryAmount &lt;= maxWithdrawValue, "withdraw amount is too big");

IDealer(dealer).requestWithdraw(address(this), 0, secondaryAmount);

}

  

function executeWithdrawSecondaryAsset() external onlyGlobalOperator {

// only Global operator can withdraw JUSD

IDealer(dealer).executeWithdraw(address(this), jojoOperator, false, "");

}

  

function fastWithdrawSecondaryAsset(address to, uint256 secondaryAmount) external onlyGlobalOperator {

(uint256 maxWithdrawValue,) = getMaxWithdrawAmount(address(dealer));

require(secondaryAmount &lt;= maxWithdrawValue, "withdraw amount is too big");

IDealer(dealer).fastWithdraw(address(this), to, 0, secondaryAmount, false, "");

}
```

## References

N/A

## Proof Of Concept

In the `test/` the `Subaccount.t.sol` test file can be modified in the following way:

```solidity
vm.startPrank(address(this));

jusd.mint(50e6);

jusd.approve(address(jojoDealer), 50e6);

jojoDealer.deposit(0, 50e6, degenAccount);

DegenSubaccount(degenAccount).requestWithdrawSecondaryAsset(10e6);

DegenSubaccount(degenAccount).executeWithdrawSecondaryAsset();

DegenSubaccount(degenAccount).fastWithdrawSecondaryAsset(address(this), 10e6);

//////////////////////////////////////////////////////////////////////////
// additional instructions
//////////////////////////////////////////////////////////////////////////

// checking that the balances are ok for the secondary withdraw functions
assertEq(jusd.balanceOf(address(this)), 20e6);

assertEq(jusd.balanceOf(address(jojoDealer)), 30e6);

  

// trying to withdraw more than availabe triggers an underflow

//console.log("BALANCE OF dealer: %s", jusd.balanceOf(address(jojoDealer)));

DegenSubaccount(degenAccount).requestWithdrawSecondaryAsset(60e6);

DegenSubaccount(degenAccount).executeWithdrawSecondaryAsset();

//DegenSubaccount(degenAccount).fastWithdrawSecondaryAsset(address(this), 60e6);

```
