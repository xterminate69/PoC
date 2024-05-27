#Medium 
The function `setUSDThresholds` is used to create thresholds for a vote. It can set `APPROVE_THRESHOLD` and `DISAPPROVE_THRESHOLD` to a value that the protocol thinks it should be.
```solidity

    function setUSDThresholds(
        uint8 _approve,
        uint8 _disapprove
    ) external onlyAdmin {
        if (usdUpdateProposal.proposer != address(0))
            revert ProposalInProgressError();
          /* @audit-HIGH centralization bug, the problem is that a malicious admin can set 
          APPROVE_THRESHOLD to 6 for example and proposals will never be passed causing votes to be obsolete */
        APPROVE_THRESHOLD = _approve;
        DISAPPROVE_THRESHOLD = _disapprove;
		
        emit USDThresholdUpdated(_approve, _disapprove);
    }
```
```
As we can see function `approveUSDPrice` heavily relies on `APPROVE_THRESHOLD` wich will execute the price update

```solidity
    function approveUSDPrice(
	...
    {
	...
@>      if (usdUpdateProposal.approvalsCount >= APPROVE_THRESHOLD) {
            _execUSDPriceUpdate();
        }

        emit ApprovedUSDPrice(msg.sender);
    }
```
```
The problem is that a malicious admin can set `APPROVE_THRESHOLD` to 6 and a proposal would be impossible to pass. This report is submitted per that the admin is not full trusted as stated by the protocol.
# PoC
1. A proposal is passed that the admin may not agree with.
2. The malicious admin sets `APPROVE_THRESHOLD` to 6 or 600000 and the proposal will never pass.

# Impact
Since a malicious admin can set `APPROVE_THRESHOLD` to whatever value he wants, the admin can make votes obsolete

# Remediation

Remove this function entirely it is not really needed for votes. It is recommended to set the `APPROVE_THRESHOLD` to 3 and `DISAPPROVE_THRESHOLD` to 3 for a majority vote.
