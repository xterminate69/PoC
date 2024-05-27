#QA
There is no need to declare `tokenowner`, put `msg.sender` directly in the parameters to save gas and have cleaner code.
```solidity
--   address tokenOwner = msg.sender; // @audit-QA no need to declare tokenOwner just put th msg.sender in the lock function
--   _lock(_tokenContract, _quantity, tokenOwner, lockRecipient);
++   _lock(_tokenContract, _quantity, tokenOwner, lockRecipient);
```
https://github.com/code-423n4/2024-05-munchables/blob/57dff486c3cd905f21b330c2157fe23da2a4807d/src/managers/LockManager.sol#L287-L293