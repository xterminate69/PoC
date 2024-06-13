#HIGH
If a user wants to lock a token he can either lock it using the function of `lock` or `lockonbehalf`. When a user specifies what he wants to lock the function `_lock` processes the amounts given by the user and does some checks. Then it imports the users info by declaring a storage `lockedToken` which is set to `lockedtokens` , one is an array of all users and the other one just a struct.

```solidity
@>      LockedToken storage lockedToken = lockedTokens[_lockRecipient][
            _tokenContract
        ];
```
As we can see it updates `lockedToken` instead of `lockedTokens`, which is problematic because every time a user locks their tokens this will be updated. Because `lockedToken` is set each time a user enters this will cause users to lose funds.
```solidity
@>      lockedToken.remainder = remainder;
@>      lockedToken.quantity += _quantity;
@>      lockedToken.lastLockTime = uint32(block.timestamp);
@>      lockedToken.unlockTime =
            uint32(block.timestamp) +
            uint32(_lockDuration);


```
There is a second exploit that can be cause if this issue is fixed that is the `unlock` function, it has the same mechanism. A user can unlock as many tokens as he wants because it does not update the correct way. The correct way to update is to use `lockedTokens` since this `lockedToken.quantity -= _quantity;` uses `lockedToken`. 
```solidity
@>      LockedToken storage lockedToken = lockedTokens[msg.sender][
            _tokenContract
        ];
        ...
@>      lockedToken.quantity -= _quantity;

```
# impact 
The first issue can cause users to lose funds but will never leave the protocol. The second is theft of funds because a malicious actor can withdraw as many tokens as he wishes.

# PoC
1. A user wants to lock his contract using the `lock` function ,he calls `lock` and specifies a contract and an amount of 10000
2. The function `_lock` is called and after some checks takes the money of the user and "updates" his position.
3. When the user wants to unlock he calls the function `unlock` and specifies an amount of 5000, it reverts.
4. It reverted because according to `lockedTokens` which correctly assumes that a user does not have enough funds.
```solidity 
// the withdraw
@>      LockedToken storage lockedToken = lockedTokens[msg.sender][
            _tokenContract
        ];
// it looks if a user has any lockedTokens
```

## second exploit
For the sake of the argument we assume the above issue is fixed.

1. A user has locked 10k with `lock` which calls `_lock`
2. The user(malicious actor) can withdraw as much as 10k each entry because of the check
```solidity
@>      LockedToken storage lockedToken = lockedTokens[msg.sender][ // A user can withdraw 10k at a time
            _tokenContract 
        ];
```
3. Once the function reaches the update quantity it updates it wrongly
```solidity
@>      lockedToken.quantity -= _quantity; // it updates lockedToken istead of lockedTokens
```
4. A user can reenter until all funds are withdrawn

# Remediation

Instead of updating `lockedToken` update `lockedTokens` and if you want to get the current state of the array `lockedtokens` to do calculations do not use storage use memory. 
