#Medium 
The function `configureToken` can accept tokens which may have 18 decimals. This is problematic because function `getLockedWeightedValue` assumes that all tokens have 18 decimals as the protocol stated " // We are assuming all tokens have a maximum of 18 decimals and that USD Price is denoted in 1e18". 
```solidity
    function configureToken(
        address _tokenContract,
        ConfiguredToken memory _tokenData
    /* struct ConfiguredToken consists of (usdPrice,nftCost,decimals,active) */
    ) external onlyAdmin
    {
        if (_tokenData.nftCost == 0) revert NFTCostInvalidError();
        if (configuredTokens[_tokenContract].nftCost == 0) {
            // new token
            configuredTokenContracts.push(_tokenContract);
        }
        configuredTokens[_tokenContract] = _tokenData;
		// no check for more than 18 decimals
        emit TokenConfigured(_tokenContract, _tokenData);
    }
```

# Impact

Function `getLockedWeightedValue` can perform incorrect calculations.

# Remediation 

Add a require in the function `configureToken` , `require(_tokenData.decimals == 18)` you can specify how many you would like.