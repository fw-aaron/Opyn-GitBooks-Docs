# OToken: Create + Manage Options in existing markets

## Introduction

Opyn uses options to provide insurance. Every option supported by the Convexity Protocol is integrated through an oToken smart contract which is an [EIP-20](https://eips.ethereum.org/EIPS/eip-20) compliant representation of options issued by the protocol. Options sellers create options by locking up collateral for some period of time and minting oTokens. Each oToken protects a unit of the specified underlying ERC20 asset. The Options seller can sell these oTokens on an exchange to earn [premiums](./#glossary-of-terms). The oToken marketplaces deployed for the purpose of insurance are ocDai and ocUSDC. 

The main functionality offered by this section is as below:

1. Create Options \(oTokens\)
2. Keep the oToken vaults sufficiently collateralized
3. Liquidate undercollateralized vaults
4. Exercise oTokens during the expiry window

## State Changing Functions 

### Create Options

_**The collateral for the ocDai and ocUSDC options markets are all ETH**_. If integrating with those markets, look at the [ETH Collateralized Options](otoken.md#eth-collateralized-options) section.

Create is a specific action which opens a vault, adds collateral, and issue oTokens. oTokens can only be issued once a vault exists and has sufficient collateral. Create does all these steps in 1 function call. \([See here for difference between create, mint and issue](protocol-overview/glossary-of-terms.md)\)

#### ETH Collateralized Options

This function[ opens a new vault](otoken.md#open-vault), [adds ETH collateral](otoken.md#add-eth-collateral-options) to it, and [issues oTokens ](otoken.md#issue-otokens)from the vault. 

```javascript
function createETHCollateralOption(uint256 amtToCreate, address receiver) payable
```

> `amtToCreate` : The number of oTokens to create
>
> `receiver` : The account to send the newly minted oTokens to
>
> `msg.value` : The collateral to be added to the newly created vault
>
> `msg.sender` : The account that will be the owner of the vault

{% tabs %}
{% tab title="Solidity" %}
```javascript
oToken ocDai = oToken(0x3BA...);

// Specify the amount of ETH collateral you want to put down in wei
uint256 collateral = 1000000000000000000;

// This function tells you the maximum number of options you can safely issue at the minimum collateralization ratio (currently 160%)/ 
// Note: It is reccomended that you create less than this amount of options. 
uint256 maxNumOptions = ocDai.maxOTokensIssuable(collateral);

// Assuming you want to be 200% collateralized
const collateralizationRatio = 200;
const numOptions = maxNumOptions * 160 / collateralizationRatio;

// This contract creates and receives 200% collateralized options against 1 ETH as collateral
ocDai.createETHCollateralOption.value(collateral)(numOptions, address(this));
```
{% endtab %}

{% tab title="Web3 1.0" %}
```javascript
const ocDai = oToken.at(0x3FDB...);
const myAccount = 'Set Your Address';

// Specify the amount of ETH collateral you want to put down in wei
const collateral = '1000000000000000000';

// This function tells you the maximum number of options you can safely issue at 160% collateralization. 
// Note: It is reccomended that you create less than this amount of options. 
const maxNumOptions = await ocDai.methods.maxOTokensIssuable(collateral).call();

// Assuming you want to be 200% collateralized
const collateralizationRatio = 200;
const numOptionsToIssue = maxNumOptions * 160 / collateralizationRatio;

// myAccount creates 200% collateralized options against 1 ETH as collateral
await ocDai.methods.createETHCollateralOption(
    numOptionsToIssue, 
    myAccount)
    .send({
        from: myAccount, 
        value: collateral,
        gasPrice: 7000000000
    });
```
{% endtab %}
{% endtabs %}

#### ERC20 Collateralized Options

This function[ opens a new vault](otoken.md#open-vault), [adds ERC20 collatera](otoken.md#erc20-collateralized-options)[l](otoken.md#erc20-collateralized-options) to it, and [issues oTokens ](otoken.md#issue-otokens)from the vault. 

```javascript
function createERC20CollateralOption(uint256 amtToCreate, uint256 amtCollateral, address receiver) 
```

> `amtToCreate` : The number of oTokens to create
>
> `amtCollateral` : The collateral to be added to the newly created vault
>
> `receiver` : The account to send the newly minted oTokens to
>
> `msg.sender` : The account that will be the owner of the vault

{% tabs %}
{% tab title="Solidity" %}
```javascript
oToken ocDai = oToken(0x3BA...);

// Specify the amount of collateral you want to put down in smallest decimal units (wei)
uint256 collateral = 1000000000000000000;

// This function tells you the maximum number of options you can safely issue at the minimum collateralization ratio (currently 160%)/ 
// Note: It is reccomended that you create less than this amount of options. 
uint256 maxNumOptions = ocDai.maxOTokensIssuable(collateral);

// Assuming you want to be 200% collateralized
const collateralizationRatio = 200;
const numOptions = maxNumOptions * collateralizationRatio / 160;

// This contract creates and receives 200% collateralized options against 1 ERC20 as collateral
ERC20Collateral.approve(address(ocDai), 1000000000000000000000000000000);
ocDai.createERC20CollateralOption(numOptions, collateral, address(this));
```
{% endtab %}

{% tab title="Web3 1.0" %}
```javascript
const ocDai = oToken.at(0x3FDB...);
const myAccount = 'Set Your Address';

// Specify the amount of ERC20 collateral you want to put down in wei
const collateral = '1000000000000000000';

// This function tells you the maximum number of options you can safely issue at 160% collateralization. 
// Note: It is reccomended that you create less than this amount of options. 
const maxNumOptions = await ocDai.methods.maxOTokensIssuable(collateral).call();

// Assuming you want to be 200% collateralized
const collateralizationRatio = 200;
const numOptions = maxNumOptions * 160 / collateralizationRatio;

// Need to first approve the oToken contract to spend the ERC20 tokens
await collateral.methods.approve(
    ocDai.options.address,
    '1000000000000000000000000000000'
    )
     .send({
        from: myAccount, 
        gasPrice: 7000000000
    });
    
    
// myAccount creates 200% collateralized options against 1 ERC20 as collateral
await ocDai.methods.createERC20CollateralOption(
    numOptions, 
    collateral,
    myAccount)
    .send({
        from: myAccount, 
        value: 0,
        gasPrice: 7000000000
    });


```
{% endtab %}
{% endtabs %}

### Add Options

_**The collateral for the ocDai and ocUSDC options markets are all ETH**_. If integrating with those markets, look at the [ETH Collateralized Options](otoken.md#eth-collateralized-options-1) section.

#### ETH Collateralized Options

This function[ ](otoken.md#open-vault)[adds ETH collateral](otoken.md#add-eth-collateral-options) to an existing vault and [issues oTokens ](otoken.md#eth-collateralized-options-1)from the vault. 

```javascript
function addETHCollateralOption(uint256 amtToCreate, address receiver) payable
```

> `amtToCreate` : The number of oTokens to create

> `receiver` : The account to send the newly minted oTokens to
>
> `msg.value` : The collateral to be added to the existing vault
>
> `msg.sender` : The account that is the owner of the vault

{% tabs %}
{% tab title="Solidity" %}
```javascript
oToken ocDai = oToken(0x3BA...);

// Specify the amount of ETH collateral you want to put down in wei
uint256 collateral = 1000000000000000000;

// This function tells you the maximum number of options you can safely issue at the minimum collateralization ratio (currently 160%)/ 
// Note: It is reccomended that you create less than this amount of options. 
uint256 maxNumOptions = ocDai.maxOTokensIssuable(collateral);

// Assuming you want to be 200% collateralized
const collateralizationRatio = 200;
const numOptionsToIssue = maxNumOptions * collateralizationRatio / 160;

// This contract creates and receives 200% collateralized options against 1 ETH as collateral
ocDai.addETHCollateralOption.value(collateral)(numOptions, address(this));
```
{% endtab %}

{% tab title="Web3 1.0" %}
```javascript
const ocDai = oToken.at(0x3FDB...);
const myAccount = 'Set Your Address';

// Specify the amount of ETH collateral you want to put down in wei
const collateral = '1000000000000000000';

// This function tells you the maximum number of options you can safely issue at 160% collateralization. 
// Note: It is reccomended that you create less than this amount of options. 
const maxNumOptions = await ocDai.methods.maxOTokensIssuable(collateral).call();

// Assuming you want to be 200% collateralized
const collateralizationRatio = 200;
const numOptionsToIssue = maxNumOptions * 160 /collateralizationRatio ;

// myAccount creates 200% collateralized options against 1 ETH as collateral
await ocDai.methods.addETHCollateralOption(
    numOptionsToIssue, 
    myAccount)
    .send({
    from: myAccount, 
    value: collateral,
    gasPrice: 7000000000
    });

```
{% endtab %}
{% endtabs %}

#### ERC20 Collateralized Options

This function [adds ERC20 collatera](otoken.md#erc20-collateralized-options)[l](otoken.md#erc20-collateralized-options) to an existing vault and [issues oTokens ](otoken.md#issue-otokens)from the vault. 

```javascript
function addERC20CollateralOption(uint256 amtToCreate, uint256 amtCollateral, address receiver)
```

> `amtToCreate` : The number of oTokens to create
>
> `amtCollateral` : The collateral to be added to the existing vault
>
> `receiver` : The account to send the newly minted oTokens to

> `msg.sender` : The account that is the owner of the vault

{% tabs %}
{% tab title="Solidity" %}
```javascript
oToken ocDai = oToken(0x3BA...);

// Specify the amount of collateral you want to put down in smallest decimal units (wei)
uint256 collateral = 1000000000000000000;

// This function tells you the maximum number of options you can safely issue at the minimum collateralization ratio (currently 160%)/ 
// Note: It is reccomended that you create less than this amount of options. 
uint256 maxNumOptions = ocDai.maxOTokensIssuable(collateral);

// Assuming you want to be 200% collateralized
const collateralizationRatio = 200;
const numOptions = maxNumOptions * collateralizationRatio / 160;

// This contract creates and receives 200% collateralized options against 1 ERC20 as collateral
ERC20Collateral.approve(address(ocDai), 1000000000000000000000000000000);
ocDai.addERC20CollateralOption(numOptions, collateral, address(this));
```
{% endtab %}

{% tab title="Web3 1.0" %}
```javascript
const ocDai = oToken.at(0x3FDB...);
const myAccount = 'Set Your Address';

// Specify the amount of ERC20 collateral you want to put down in wei
const collateral = '1000000000000000000';

// This function tells you the maximum number of options you can safely issue at 160% collateralization. 
// Note: It is reccomended that you create less than this amount of options. 
const maxNumOptions = await ocDai.methods.maxOTokensIssuable(collateral).call();

// Assuming you want to be 200% collateralized
const collateralizationRatio = 200;
const numOptions = maxNumOptions * 160 / collateralizationRatio;

// Need to first approve the oToken contract to spend the ERC20 tokens
await collateral.methods.approve(
    ocDai.options.address,
    '1000000000000000000000000000000'
    )
    .send({
        from: myAccount, 
        gasPrice: 7000000000
    });
    
    
// myAccount creates 200% collateralized options against 1 ERC20 as collateral
await ocDai.methods.addERC20CollateralOption(
    numOptions, 
    collateral,
    myAccount)
    .send({
        from: myAccount, 
        value: 0,
        gasPrice: 7000000000
    });
```
{% endtab %}
{% endtabs %}

### Create and Sell Options

_**The collateral for the ocDai and ocUSDC options markets are all ETH**_. If integrating with those markets, look at the [ETH Collateralized Options](otoken.md#eth-collateralized-options-2) section.

Create is a specific action which opens a vault, adds collateral, and issue oTokens. oTokens can only be issued once a vault exists and has sufficient collateral. Create does all these steps in 1 function call. \([See here for difference between create, mint and issue](protocol-overview/glossary-of-terms.md)\)

#### ETH Collateralized Options

This function[ opens a new vault](otoken.md#open-vault), [adds ETH collateral](otoken.md#add-eth-collateral-options) to it, [issues oTokens ](otoken.md#issue-otokens)from the vault, and [sells the oTokens](optionsexchange-buy-and-sell-otokens.md#sell-otokens) for ETH on Uniswap. This function is sufficient for a new option seller to go be fully integrated and start earning premiums. 

```javascript
function createAndSellETHCollateralOption(uint256 amtToCreate, address payable receiver) payable
```

> `amtToCreate` : The number of oTokens to create
>
> `receiver` : The account to send the newly minted oTokens to
>
> `msg.value` : The collateral to be added to the newly created vault
>
> `msg.sender` : The account that will be the owner of the vault

{% tabs %}
{% tab title="Solidity" %}
```javascript
oToken ocDai = oToken(0x3BA...);

// Specify the amount of ETH collateral you want to put down in wei
uint256 collateral = 1000000000000000000;

// This function tells you the maximum number of options you can safely issue at the minimum collateralization ratio (currently 160%)/ 
// Note: It is reccomended that you create less than this amount of options. 
uint256 maxNumOptions = ocDai.maxOTokensIssuable(collateral);

// Assuming you want to be 200% collateralized
const collateralizationRatio = 200;
const numOptions = maxNumOptions * 160 / collateralizationRatio;

// This contract creates and receives 200% collateralized options against 1 ETH as collateral
ocDai.createAndSellETHCollateralOption.value(collateral)(numOptions, address(this));
```
{% endtab %}

{% tab title="Web3" %}
```javascript
const ocDai = oToken.at(0x3FDB...);
const myAccount = 'Set Your Address';

// Specify the amount of ETH collateral you want to put down in wei
const collateral = '1000000000000000000';

// This function tells you the maximum number of options you can safely issue at 160% collateralization. 
// Note: It is reccomended that you create less than this amount of options. 
const maxNumOptions = await ocDai.methods.maxOTokensIssuable(collateral).call();

// Assuming you want to be 200% collateralized
const collateralizationRatio = 200;
const numOptionsToIssue = maxNumOptions * 160 / collateralizationRatio;

// myAccount creates 200% collateralized options against 1 ETH as collateral
await ocDai.methods.createAndSellETHCollateralOption(
    numOptionsToIssue, 
    myAccount)
    .send({
    from: myAccount, 
    value: collateral,
    gasPrice: 7000000000
    });
```
{% endtab %}
{% endtabs %}

#### ERC20 Collateralized Options

This function[ opens a new vault](otoken.md#open-vault), [adds ERC20 collateral](otoken.md#erc20-collateralized-options) to it, [issues oTokens ](otoken.md#issue-otokens)from the vault, and [sells the oTokens](optionsexchange-buy-and-sell-otokens.md#sell-otokens) for ETH on Uniswap.

```javascript
function createAndSellERC20CollateralOption(uint256 amtToCreate, uint256 amtCollateral, address payable receiver)
```

> `amtToCreate` : The number of oTokens to create
>
> `amtCollateral` : The collateral to be added to the newly created vault
>
> `receiver` : The account to send the newly minted oTokens to
>
> `msg.sender` : The account that will be the owner of the vault

{% tabs %}
{% tab title="Solidity" %}
```javascript
oToken ocDai = oToken(0x3BA...);

// Specify the amount of collateral you want to put down in smallest decimal units (wei)
uint256 collateral = 1000000000000000000;

// This function tells you the maximum number of options you can safely issue at the minimum collateralization ratio (currently 160%)/ 
// Note: It is reccomended that you create less than this amount of options. 
uint256 maxNumOptions = ocDai.maxOTokensIssuable(collateral);

// Assuming you want to be 200% collateralized
const collateralizationRatio = 200;
const numOptions = maxNumOptions * collateralizationRatio / 160;

// This contract creates and receives 200% collateralized options against 1 ERC20 as collateral
ERC20Collateral.approve(address(ocDai), 1000000000000000000000000000000);
ocDai.createAndSellERC20CollateralOption(numOptions, collateral, address(this));
```
{% endtab %}

{% tab title="Web3 1.0" %}
```javascript
const ocDai = oToken.at(0x3FDB...);
const myAccount = 'Set Your Address';

// Specify the amount of ERC20 collateral you want to put down in wei
const collateral = '1000000000000000000';

// This function tells you the maximum number of options you can safely issue at 160% collateralization. 
// Note: It is reccomended that you create less than this amount of options. 
const maxNumOptions = await ocDai.methods.maxOTokensIssuable(collateral).call();

// Assuming you want to be 200% collateralized
const collateralizationRatio = 200;
const numOptions = maxNumOptions * 160 / collateralizationRatio;

// Need to first approve the oToken contract to spend the ERC20 tokens
await collateral.methods.approve(
    ocDai.options.address,
    '1000000000000000000000000000000'
    )
     .send({
        from: myAccount, 
        gasPrice: 7000000000
    });
    
    
// myAccount creates 200% collateralized options against 1 ERC20 as collateral
await ocDai.methods.createAndSellERC20CollateralOption(
    numOptions, 
    collateral,
    myAccount)
    .send({
        from: myAccount, 
        value: 0,
        gasPrice: 7000000000
    });
```
{% endtab %}
{% endtabs %}

### Add and Sell Options

_**The collateral for the ocDai and ocUSDC options markets are all ETH**_. If integrating with those markets, look at the [ETH Collateralized Options](otoken.md#eth-collateralized-options-3) section.

#### ETH Collateralized Options

This function[ ](otoken.md#open-vault)[adds ETH collateral](otoken.md#add-eth-collateral-options) to an existing vault, [issues oTokens ](otoken.md#issue-otokens)from the vault, and [sells the oTokens](optionsexchange-buy-and-sell-otokens.md#sell-otokens) on Uniswap for ETH.

```javascript
function addAndSellETHCollateralOption(uint256 amtToCreate, uint256 vaultIndex, address payable receiver) payable
```

> `amtToCreate` : The number of oTokens to create
>
> `vaultIndex` : The index of the vault to add oTokens to
>
> `receiver` : The account to send the newly minted oTokens to
>
> `msg.value` : The collateral to be added to the existing vault
>
> `msg.sender` : The account that is the owner of the vault

{% tabs %}
{% tab title="Solidity" %}
```javascript
oToken ocDai = oToken(0x3BA...);
ocDai.addAndSellETHCollateralOption.value(100000)(10, 1, 0xFDA...);
```
{% endtab %}

{% tab title="Web3 1.0" %}
```javascript
const ocDai = oToken.at(0x3FDB...);
await ocDai.methods.addAndSellETHCollateralOption(
    10, 
    1,
    0xFDA...
    ).send({from: myAccount, value: 100000});
```
{% endtab %}
{% endtabs %}

#### ERC20 Collateralized Options

This function [adds ERC20 collateral](otoken.md#erc20-collateralized-options) to an existing vault, [issues oTokens ](otoken.md#issue-otokens)from the vault, and [sells the oTokens](optionsexchange-buy-and-sell-otokens.md#sell-otokens) for ETH on Uniswap.

```javascript
function addAndSellERC20CollateralOption(uint256 amtToCreate, uint256 amtCollateral, uint256 vaultIndex, address payable receiver)
```

> `amtToCreate` : The number of oTokens to create
>
> `amtCollateral` : The collateral to be added to the existing vault
>
> `vaultIndex` : The index of the vault to add oTokens to
>
> `receiver` : The account to send the newly minted oTokens to
>
> `msg.sender` : The account that is the owner of the vault

{% tabs %}
{% tab title="Solidity" %}
```javascript
oToken ocDai = oToken(0x3BA...);
ERC20Collateral.approve(address(ocDai), 1000000000000000000000000000000);
ocDai.addAndSellERC20CollateralOption(10, 1000000, 1, 0xFDA...);
```
{% endtab %}

{% tab title="Web3 1.0" %}
```javascript
const ocDai = oToken.at(0x3FDB...);
await collateral.methods.approve(
    ocDai.options.address,
    '1000000000000000000000000000000'
    ).send();
    
await ocDai.methods.addAndSellERC20CollateralOption(
    10,
    100000,
    1,
    0xFDA...
    ).send();
```
{% endtab %}
{% endtabs %}

### **Open Vault**

The open vault function creates an empty vault and sets the owner of the vault to be the msg.sender. The collateral and the outstanding puts issued of the vault are set to 0. 

A vault can only be opened before expiry. You can check if a contract has expired by calling [`hasExpired()`](otoken.md#has-the-contract-expired).  

Each owner can also only own one vault. You can check if the owner already owns a vault by calling the [`hasVault()`](otoken.md#has-vault)`.`

```javascript
function openVault() public returns (bool)
```

> `msg.sender`: The account that shall be the owner of the vault
>
> `RETURN`: Bool, if a new vault was successfully opened

{% tabs %}
{% tab title="Solidity" %}
```javascript
oToken ocDai = oToken(0x3BA...);


require(ocDai.hasExpired() == false, "Can only open vault before expiry");
require(ocDai.hasVault(address(this)) == false, "Each owner can only have 1 vault");
uint256 vaultIndex = ocDai.openVault();
```
{% endtab %}

{% tab title="Web3 1.0" %}
```javascript
const ocDai = oToken.at(0x3FDB...);
const myAccount = 'Set Your Address';

const hasExpired = await ocDai.methods.hasExpired().call();
const hasVault = await ocDai.methods.hasVault(myAccount).call();

if(!hasExpired && !hasVault) {
    await ocDai.methods.openVault().send({
        from: myAccount,
        gasPrice: 7000000000
    });
}
```
{% endtab %}
{% endtabs %}

### Add Collateral

The add collateral functions serve two purposes. The first is to add collateral to a new vault. A vault needs to meet the minimum collateral requirements before it is[ safe](./#glossary-of-terms) for the owner to mint oTokens.The second purpose is to top up i.e. turn a vault that is unsafe because it doesn't meet the minimum collateral requirements into a vault that is [safe](./#glossary-of-terms). 

The add collateral functions can be called at any point before the oToken's expiry. You can check if a contract has expired by calling `hasExpired()`. 

**The collateral for the ocDai and ocUSDC options markets are all ETH.** 

#### Add ETH Collateral

The `addETHCollateral()` function is called if the specified collateral for the oToken contract is ETH. This function call will fail if the collateral type is not ETH. 

```javascript
function addETHCollateral(address payable vaultOwner) public payable returns (uint256) 
```

> `vaultIndex` : The address of the vault owner, whose vault the protocol will add collateral to.
>
> `msg.sender` : The account from which ETH collateral will be transferred into the oToken contract
>
> `msg.value` : The amount of ETH to add

{% tabs %}
{% tab title="Solidity" %}
```javascript
oToken ocDai = oToken(0x3BA...);
address payable myVault = 'Enter the vault address';
require(ocDai.hasExpired() == false, "Can only add collateral before expiry");
uint256 vaultCollateralBalance = ocDai.addETHCollateral.value(1000000)(myVault);
```
{% endtab %}

{% tab title="Web3 1.0" %}
```javascript
const ocDai = oToken.at(0x3FDB...);
const myVault = 'Enter the vault address';

const hasExpired = await ocDai.methods.hasExpired().call();
if(!hasExpired) {
    await ocDai.methods.addETHCollateral(myVault)
    .send({
        from: myAccount, 
        value:1000000,
        gasPrice: 7000000000
        });
}
```
{% endtab %}
{% endtabs %}

#### Add ERC20 Collateral

The `addERC20Collateral()` function is called if the specified collateral for the oToken contract is any ERC20 asset. The function will only add the specified collateral asset. It will fail if called on any other asset other than the pre specified collateral asset. 

```javascript
function addERC20Collateral(uint256 vaultIndex, uint256 amt) returns (uint256)
```

> `vaultIndex` : The address of the vault owner, whose vault the protocol will add collateral to.

> `amt` : The amount of collateral tokens to add
>
> `msg.sender` : The account from which the ERC20 collateral asset will be transferred into the oToken contract

{% tabs %}
{% tab title="Solidity" %}
```javascript
oToken ocDai = oToken(0x3BA...);
/** 
 * the spender needs to approve the oToken contract to spend 
 * their ERC20Collateral because they are transferring collateral 
 * tokens to the oToken contract. 
 */
ERC20Collateral.approve(address(ocDai), 1000000000000000000000000000000);
require(ocDai.hasExpired() == false, "Can only add collateral before expiry");
uint256 vaultCollateralBalance = ocDai.addERC20Collateral(1, 100000000);
```
{% endtab %}

{% tab title="Web3 1.0" %}
```javascript
const ocDai = oToken.at(0x3FDB...);
const hasExpired = await ocDai.methods.hasExpired().call();

if(!hasExpired) {

    await collateral.approve(
    ocDai.options.address,
    '1000000000000000000000000000000'
    ).send();
    
    await ocDai.methods.addERC20Collateral(1, 1000000).send();
}
```
{% endtab %}
{% endtabs %}

### Issue oTokens

The issue oTokens functionality allows the vault owner to mint new options in the form of oTokens before expiry of the oToken contract. These oTokens are ERC-20 tokens and are fungible within a specific series. Different series of oTokens are not fungible with each other. 

To mint oTokens, the vault that the tokens are being minted from must meet the [`minCollateralizationRatio`](./#glossary-of-terms) requirements. Further, only the owner of a vault can mint oTokens. 

Once oTokens have been minted, they can be sold on an exchange like Uniswap.

```javascript
function issueOTokens (uint256 oTokensToIssue, address receiver) public
```

> `oTokensToIssue`: The amount of oTokens to issue
>
> `receiver`: The address that the newly minted oTokens will be sent to
>
> `msg.sender` : The account that is the owner of the vault that is issuing the oTokens. Only the owner can issue oTokens

{% tabs %}
{% tab title="Solidity" %}
```javascript
oToken ocDai = oToken(0x3BA...);
require(ocDai.hasExpired() == false, "Can only issue oTokens before expiry");
ocDai.issueOTokens(1000, 0xFB3...);
```
{% endtab %}

{% tab title="Web3 1.0" %}
```javascript
const ocDai = oToken.at(0x3BA...);
const hasExpired = await ocDai.methods.hasExpired().call();
if(!hasExpired) {
    await ocDai.methods.issueOTokens(1000, 0xFB3...).send();
}
```
{% endtab %}
{% endtabs %}

### Remove Collateral

The remove collateral function allows a vault owner to remove excess collateral in their vault. By removing collateral, a vault owner reduces the ratio of collateral to oTokens issued i.e. the vault's [collateralization ratio](./#glossary-of-terms), thus making the vault less safe. 

A vault owner can remove collateral before expiry of the oToken contract. The vault needs to remain safe after the collateral has been removed. 

```javascript
function removeCollateral(uint256 amtToRemove) 
```

> `amtToRemove` : The amount of collateral to remove
>
> `msg.sender` : The account of the owner of the vault. The collateral will be sent to this account.

{% tabs %}
{% tab title="Solidity" %}
```javascript
oToken ocDai = oToken(0x3BA...);
require(ocDai.hasExpired() == false, "Can only remove collateral before expiry");
ocDai.removeCollateral(1000000);
```
{% endtab %}

{% tab title="Web3 1.0" %}
```javascript
const ocDai = oToken.at(0x3BA...);
const hasExpired = await ocDai.methods.hasExpired().call();
if(!hasExpired) {
    await ocDai.methods.removeCollateral(1000000).send();
}
```
{% endtab %}
{% endtabs %}

### Burn oTokens

The burn oTokens functionality allows a vault owner to reduce the amount of insurance that they have provided by bringing back oTokens to the oTokens smart contract. By burning oTokens, a vault owner increases the ratio of collateral to oTokens Issued i.e. the vault's [collateralization ratio](./#glossary-of-terms), thus making the vault [safer. ](./#glossary-of-terms)

A vault owner can burn oTokens any time before expiry of the oToken contract. You can check if a contract has expired by calling `hasExpired()`. 

```javascript
function burnOTokens(uint256 amtToBurn)
```

> `amtToBurn` : The amount of oTokens to burn

> `msg.sender` : The owner of the vault and the account from which oTokens will be burned

{% tabs %}
{% tab title="Solidity" %}
```javascript
oToken ocDai = oToken(0x3BA...);
/** 
 * the spender needs to approve the oToken contract to spend 
 * their oTokens because they are transferring oTokens 
 * to the oToken contract. 
 */
ocDai.approve(address(ocDai), 1000000000000000000000000000000);
require(ocDai.hasExpired() == false, "Can only burn oTokens before expiry");
ocDai.burnOTokens(100);
```
{% endtab %}

{% tab title="Web3 1.0" %}
```javascript
const ocDai = oToken.at('0x3BA...');
const hasExpired = await ocDai.methods.hasExpired().call();
if(!hasExpired) {
    await ocDai.methods.approve(
    ocDai.options.address,
    '1000000000000000000000000000000'
    ).send();
    
    await ocDai.methods.burnOTokens('1000000').send();
}
```
{% endtab %}
{% endtabs %}

### Liquidate

A vault that fails to meet the [minimum collateralization requirement](protocol-overview/glossary-of-terms.md) is subject to liquidation by other users of the protocol to return the vault back to a safe state. The liquidate function can only be called before expiry. 

Only an [unsafe](./#glossary-of-terms) vault can be liquidated. You can check if a vault is unsafe by calling the function `isUnsafe(uint256 vaultIndex).` 

When a liquidation occurs, a liquidator may return some or all of the outstanding oTokens issued on behalf of a vault owner and in return receive a discounted amount of collateral held by the vault owner; this discount is defined as the [liquidation incentive](./#glossary-of-terms). 

A liquidator may close up to a certain fixed percentage \([i.e. liquidation factor\)](./#glossary-of-terms) of the outstanding oTokens issued by the [unsafe](./#glossary-of-terms) vault. 

```javascript
function liquidate(address vaultOwner, uint256 oTokensToLiquidate)
```

> `vaultOwner` : The owner of the vault whose vault is unsafe and is to be liquidated. \(To get the all the owners of vaults in the oToken contract, check out Opyn's subgraph for the events\)
>
> `oTokensToLiqudate` : The amount of oTokens that the liquidator brings back to the oToken contract
>
> `msg.sender` : The account from which oTokens will be transferred into the oToken contract. The collateral along with a liquidation incentive will also be paid out to this account.

{% tabs %}
{% tab title="Solidity" %}
```javascript
oToken ocDai = oToken(0x3BA...);

/** 
 * the spender needs to approve the oToken contract to spend 
 * their oTokens because they are transferring oTokens 
 * to the oToken contract. 
 */
ocDai.approve(address(ocDai), 1000000000000000000000000000000);

uint256 vaultIndex = '0xFD..'; 
require(ocDai.hasExpired() == false, "Can only liquidate before expiry");
require(ocDai.isUnsafe(vaultIndex), "Vault is safe");
ocDai.liquidate(vaultIndex, 10);
```
{% endtab %}

{% tab title="Web3 1.0" %}
```javascript
const ocDai = oToken.at('0x3BA...');

const vaultIndex = '0xFD..'; 
const hasExpired = await ocDai.methods.hasExpired().call();
const isUnsafe = await ocDai.methods.isUnsafe(vaultIndex).call();

if((!hasExpired) && isUnsafe) {
    await ocDai.methods.approve(
    ocDai.options.address,
    '1000000000000000000000000000000'
    ).send();
    
    await ocDai.methods.liquidate(vaultIndex, '10').send();
}
```
{% endtab %}
{% endtabs %}

### Exercise

During the [exercise window](./#glossary-of-terms), any oToken holders can transfer their oTokens and corresponding amount of underlying tokens and in return get out strike price amount of the collateral asset per unit of oToken exercised. 

To determine if it is the exercise window, call [`isExerciseWindow()`](otoken.md#is-it-the-exercise-window). The [exercise](./#glossary-of-terms) function can only be called during the exercise window. 

The amount of underlying tokens to be transferred can be calculated by calling the [`underlyingRequiredToExercise(..)`](otoken.md#underlying-required-to-exercise) function. Users first need to [`approve(..)`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol) the oToken contract to spend their underlying and oTokens before they can call `exercise(..)` on a vault. This is because the exercise function transfers in the ERC20 oTokens and underlying tokens into the oToken contract \([Sec 5.1 Physical Settlement](https://drive.google.com/file/d/1YsrGBUpZoPvFLtcwkEYkxNhogWCU772D/view)\). 

While exercise can be called at anytime during the exercise window, it may be unprofitable to exercise unless there was an actual crash in the price of the underlying asset relative to the price of the strike asset. 

```javascript
function exercise(uint256 oTokensToExercise, address payable[] memory vaultsToExerciseFrom) payable
```

> `oTokensToExercise` : The amount of oTokens being exercised. \(See here to get the [oToken balance of the msg.sender](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol#L50)\)
>
> `vaultsToExerciseFrom` : The addresses of vault owners to exercise from. Collateral will be taken from the vaults in the order passed in. See  Opyn's subgraph to get the all the owners of vaults in the oToken contract\)

> `msg.sender` : The account from which oTokens and underlying assets will be transferred into the oToken contract. This account will also get paid out the claims made. 
>
> `msg.value` : If the underlying protected is ETH, then the msg.value is the amount of ETH transferred. If not, it should be set to 0.

{% tabs %}
{% tab title="Solidity" %}
```javascript
oToken ocDai = oToken(0x3BA...);

/** 
 * the spender needs to approve the oToken contract to spend 
 * their oTokens and underlying because they are transferring 
 * oTokens and underlying tokens to the oToken contract. 
 */
ocDai.approve(address(ocDai), 1000000000000000000000000000000);
cDai.approve(address(ocDai), 1000000000000000000000000000000);

require(ocDai.isExerciseWindow() == true, "Can only exercise during the exericse window");

uint256 amtToExercise = 1000;

// need to have sufficient underlying to make a claim
uint256 underlyingToTransfer = ocDai.underlyingRequiredToExercise(amtToExercise);
require(cDai.balanceOf(address(this) >= underlyingToTransfer, "Insufficient underlying to exercise");

ocDai.exercise(amtToExercise);
```
{% endtab %}

{% tab title="Web3 1.0" %}
```javascript
const ocDai = oToken.at('0x3BA...');

const vaultIndex = '1'; 
const isExerciseWindow = await ocDai.methods.isExerciseWindow().call();

if(isExerciseWindow) {
    // Approve the underlying and oTokens to be transferred
    await ocDai.methods.approve(
    ocDai.options.address,
    '1000000000000000000000000000000'
    ).send();
    
    await cDai.methods.approve(
    ocDai.options.address,
    '1000000000000000000000000000000'
    ).send();   
    
    const amtToExercise = '1000';
    
    // need to have sufficient underlying to make a claim
    const underlyingToTransfer = await ocDai.methods.underlyingRequiredToExercise(amtToExercise).call();
    
    if ((await cDai.methods.balanceOf(myAccount).call()) >= underlyingToTransfer) {
        await ocDai.methods.exercise(amtToExercise).send();
    }
}
```
{% endtab %}
{% endtabs %}

### Redeem Vault Balance

Once the oToken contract has expired, any vault owner can redeem their share of collateral and any underlying if an exercise call was made on their vault. The amount of collateral that the owner gets back is the balance left after any exercise calls made which affected their vault. If no exercise calls were made which affected their vault, they get all their collateral back. 

You can call the `hasExpired()` function to check if the oToken contract has expired. 

```javascript
function redeemVaultBalance()
```

> `msg.sender` : The address that owns the vault. The collateral and underlying balance from the vault will be sent to the owner of this account.

{% tabs %}
{% tab title="Solidity" %}
```javascript
oToken ocDai = oToken(0x3BA...);
require(ocDai.hasExpired() == true, "Can only claim collateral back after the exericse window");
ocDai.redeemVaultBalance()
```
{% endtab %}

{% tab title="Web3 1.0" %}
```javascript
const ocDai = oToken.at('0x3BA...');

const hasExpired = await ocDai.methods.hasExpired().call();

if(hasExpired) {    
    await ocDai.methods.redeemVaultBalance().send();
}
```
{% endtab %}
{% endtabs %}

### Remove Underlying

The vault owner function allows the owner of the vault to withdraw underlying from their vault if an exercise call affected their vault. This function can be called at any time. 

```javascript
function removeUnderlying() public
```

> `msg.sender` : The owner of the vault. Only the vault owner can call this function.

{% tabs %}
{% tab title="Solidity" %}
```javascript
oToken ocDai = oToken(0x3BA...);
ocDai.removeUnderlying();
```
{% endtab %}

{% tab title="Web3 1.0" %}
```javascript
const ocDai = oToken.at('0x3BA...');

await ocDai.methods.removeUnderlying().send();
```
{% endtab %}
{% endtabs %}

## Key Events 

<table>
  <thead>
    <tr>
      <th style="text-align:left">Event</th>
      <th style="text-align:left">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">
        <p><code>VaultOpened(</code>
        </p>
        <p><code>uint256 vaultIndex,</code>
        </p>
        <p><code>address vaultOwner)</code>
        </p>
      </td>
      <td style="text-align:left">Emitted upon a successful <a href="otoken.md#open-vault">Open Vault</a>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><code>ETHCollateralAdded(</code>
        </p>
        <p><code>uint256 vaultIndex,</code>
        </p>
        <p><code>uint256 amount, </code>
        </p>
        <p><code>address payer)</code>
        </p>
      </td>
      <td style="text-align:left">Emitted upon a successful <a href="otoken.md#add-eth-collateral">Add ETH Collateral</a>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><code>ERC20CollateralAdded(</code>
        </p>
        <p><code>uint256 vaultIndex,</code>
        </p>
        <p><code>uint256 amount,</code>
        </p>
        <p><code>address payer)</code>
        </p>
      </td>
      <td style="text-align:left">Emitted upon a successful <a href="otoken.md#add-erc20-collateral">Add ERC20 Collateral</a>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><code>IssuedOTokens(</code>
        </p>
        <p><code>address issuedTo, </code>
        </p>
        <p><code>uint256 oTokensIssued, </code>
        </p>
        <p><code>uint256 vaultIndex)</code>
        </p>
      </td>
      <td style="text-align:left">Emitted upon a successful <a href="otoken.md#issue-otokens">Issue oTokens</a>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><code>Liquidate (</code>
        </p>
        <p><code>uint256 amtCollateralToPay,</code>
        </p>
        <p><code>uint256 vaultIndex,</code>
        </p>
        <p><code>address liquidator)</code>
        </p>
      </td>
      <td style="text-align:left">Emitted upon a successful <a href="otoken.md#liquidate">Liquidate</a>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><code>Exercise (</code>
        </p>
        <p><code>uint256 amtUnderlyingToPay, </code>
        </p>
        <p><code>uint256 amtCollateralToPay,</code>
        </p>
        <p><code>address exerciser)</code>
        </p>
      </td>
      <td style="text-align:left">Emitted upon a successful <a href="otoken.md#exercise">Exercise</a>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><code>ClaimedCollateral(</code>
        </p>
        <p><code>uint256 amtCollateralClaimed,</code>
        </p>
        <p><code>uint256 amtUnderlyingClaimed, uint256 vaultIndex, </code>
        </p>
        <p><code>address vaultOwner)</code>
        </p>
      </td>
      <td style="text-align:left">Emitted upon a successful <a href="otoken.md#claim-collateral">Claim Collateral</a>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><code>BurnOTokens (</code>
        </p>
        <p><code>uint256 vaultIndex, </code>
        </p>
        <p><code>uint256 oTokensBurned)</code>
        </p>
      </td>
      <td style="text-align:left">Emitted upon a successful <a href="otoken.md#burn-otokens">Burn oTokens</a>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><code>RemoveCollateral (</code>
        </p>
        <p><code>uint256 vaultIndex, </code>
        </p>
        <p><code>uint256 amtRemoved, </code>
        </p>
        <p><code>address vaultOwner)</code>
        </p>
      </td>
      <td style="text-align:left">Emitted upon a successful <a href="otoken.md#remove-collateral">Remove Collateral</a>
      </td>
    </tr>
  </tbody>
</table>

## Read Only Functions 

### Strike Price 

The strike price is the specified price that the buyer/owner of 1 oToken gets if they make a claim and give up their underlying asset. A return value of \(100, -15\) corresponds to 100 \* 10^-15. 

```javascript
function strikePrice() returns (uint256, int32)
```

> `RETURN` : returns a Number which is a tuple - \(value, exponent\)

{% tabs %}
{% tab title="Solidity" %}
```javascript
oToken ocDai = oToken(0x3BA...);
(uint256 value, int32 exponent) = ocDai.strikePrice();
```
{% endtab %}

{% tab title="Web3 1.0" %}
```javascript
const ocDai = oToken.at(0x3BA...);
let val, exp = ocDai.methods.strikePrice().call();
```
{% endtab %}
{% endtabs %}

### oToken Exchange Rate

The oTokenExchangeRate is the amount of underlying that the smallest unit of oToken protects. 

| oToken | Amount Underlying Protected | Smallest Unit of cDai protected |
| :--- | :--- | :--- |
| 1 ocDai | 1 cDai | 10^-8 cDai |
| 1 ocUSDC | 1 cUSDC | 10^-8 cUSDC |

```javascript
function oTokenExchangeRate() returns (uint256, int32)
```

> `RETURN` : returns a Number which is a tuple \(value, exponent\)

{% tabs %}
{% tab title="Solidity" %}
```javascript
oToken ocDai = oToken(0x3BA...);
(uint256 value, int32 exponent) = ocDai.oTokenExchangeRate();
```
{% endtab %}

{% tab title="Web3 1.0" %}
```javascript
const ocDai = oToken.at(0x3BA...);
let val, exp = ocDai.methods.oTokenExchangeRate().call();
```
{% endtab %}
{% endtabs %}

### Expiry Date

The date on which the oToken contract expires is the day when the options stop having value. After this date, the option buyer/ owner can't exercise their options. 

```javascript
function expiry() returns (uint256)
```

> `RETURN` : returns a UNIX timestamp

{% tabs %}
{% tab title="Solidity" %}
```javascript
oToken ocDai = oToken(0x3BA...);
uint256 expiryTimestamp = ocDai.expiry();
```
{% endtab %}

{% tab title="Web3 1.0" %}
```javascript
const ocDai = oToken.at(0x3BA...);
let expiryTime = ocDai.methods.expiry().call();
```
{% endtab %}
{% endtabs %}

### Underlying Required to Exercise

This function calculates the amount of underlying tokens to be transferred based on how many oTokens are being exercised. 

```javascript
function  underlyingRequiredToExercise(uint256 oTokensToExercise) view returns (uint256)
```

> `RETURN` : The number of underlying tokens to transfer

{% tabs %}
{% tab title="Solidity" %}
```javascript
oToken ocDai = oToken(0x3BA...);
expiryTimestamp = ocDai.underlyingRequiredToExercise(10);
```
{% endtab %}

{% tab title="Web3 1.0" %}
```javascript
const ocDai = oToken.at(0x3BA...);
let underlyingToTransfer = ocDai.methods. underlyingRequiredToExercise(10).call();
```
{% endtab %}
{% endtabs %}

### Has the contract Expired

This function checks if an existing oToken contract has expired. 

```javascript
function hasExpired() view returns (bool)
```

> `RETURN` : If the existing oToken contract has expired

{% tabs %}
{% tab title="Solidity" %}
```javascript
oToken ocDai = oToken(0x3BA...);
bool hasExpired = ocDai.hasExpired();
```
{% endtab %}

{% tab title="Web3 1.0" %}
```javascript
const ocDai = oToken.at(0x3BA...);
let hasExpired = ocDai.methods.hasExpired().call();
```
{% endtab %}
{% endtabs %}

### Is it the Exercise Window 

This function checks if it is the[ exercise window](./#glossary-of-terms) for an existing oToken contract. 

```javascript
function isExerciseWindow() view returns (bool)
```

> `RETURN` : If the existing oToken contract has expired

{% tabs %}
{% tab title="Solidity" %}
```javascript
oToken ocDai = oToken(0x3BA...);
bool isExerciseWindow = ocDai.isExerciseWindow();
```
{% endtab %}

{% tab title="Web3 1.0" %}
```javascript
const ocDai = oToken.at(0x3BA...);
let isExerciseWindow = ocDai.methods.isExerciseWindow().call();
```
{% endtab %}
{% endtabs %}

### Has Vault

This function return true if the owner already has a vault

```javascript
function hasVault(address payable owner) public view returns (bool)
```

> `owner` : The function returns true if the address passed in is already the owner of a vault in the given oToken contract.
>
> `RETURN` : \(Boolean\) True if the owner has a vault.

{% tabs %}
{% tab title="Solidity" %}
```javascript
oToken ocDai = oToken(0x3BA...);
bool alreadyHasVault = ocDai.hasVault(0xFBA...);
```
{% endtab %}

{% tab title="Web3 1.0" %}
```javascript
const ocDai = oToken.at(0x3BA...);
let alreadyHasVault = ocDai.methods.hasVault(0xFBA...).call();
```
{% endtab %}
{% endtabs %}

### Get Vault by Index

This function gets the vault at the passed in index. 

```javascript
function getVaultByIndex(uint256 vaultIndex) public view returns (uint256, uint256, address)
```

> vaultIndex : The index of the vault to get

> `RETURN` : tuple of values \(collateral, oTokensIssued, owner\)

{% tabs %}
{% tab title="Solidity" %}
```javascript
oToken ocDai = oToken(0x3BA...);
(uint256 collateral, uint256 oTokensIssued, address owner)  = ocDai.getVaultsByIndex(1);
```
{% endtab %}

{% tab title="Web3 1.0" %}
```javascript
const ocDai = oToken.at(0x3BA...);
let result = ocDai.methods.getVaultByIndex(1).call();
```
{% endtab %}
{% endtabs %}

### Is the given vault unsafe

This function checks if the vault at the given index is safe.

```javascript
function isUnsafe(uint256 vaultIndex) view returns (bool)
```

> vaultIndex : The index of the vault to get

> `RETURN` : true if the vault is unsafe

{% tabs %}
{% tab title="Solidity" %}
```javascript
oToken ocDai = oToken(0x3BA...);
bool isUnsafe  = ocDai.isUnsafe(1);
```
{% endtab %}

{% tab title="Web3 1.0" %}
```javascript
const ocDai = oToken.at(0x3BA...);
let isUnsafe = ocDai.methods.isUnsafe(1).call();
```
{% endtab %}
{% endtabs %}

### Get Number of Vaults 

This function returns the number of vaults in the oToken contract.

```javascript
function numVaults() public view returns (uint256)
```

> `RETURN` : The total number of vaults that exist in the oToken contract



{% tabs %}
{% tab title="Solidity" %}
```javascript
oToken ocDai = oToken(0x3BA...);
unit256 totalVaults = ocDai.numVaults();
```
{% endtab %}

{% tab title="Web3 1.0" %}
```javascript
const ocDai = oToken.at(0x3BA...);
let totalVaults = ocDai.methods.numVaults().call();
```
{% endtab %}
{% endtabs %}

### Maximum number of oTokens issuable

This function returns the maximum number of oTokens that can safely be issued at the given minimum collateralization ratio. We recommend issuing less than this amount, since a slight change in price can make the vault unsafe. 

```javascript
function maxOTokensIssuable(uint256 collateralAmt) public view returns (uint256)
```

> `CollateralAmt`The amount of collateral \(in wei\) that is to be supplied.

> `RETURN` : The maximum number of oTokens that can safely be issued at the given minimum collateralization ratio \(160%\).



{% tabs %}
{% tab title="Solidity" %}
```javascript
oToken ocDai = oToken(0x3BA...);

// Specify the amount of ETH collateral you want to put down in wei
uint256 collateral = 1000000000000000000;

// This function tells you the maximum number of options you can safely issue at the minimum collateralization ratio (currently 160%)/ 
// Note: It is reccomended that you create less than this amount of options. 
uint256 maxNumOptions = ocDai.maxOTokensIssuable(collateral);

// Assuming you want to be 200% collateralized
const collateralizationRatio = 200;
const numOptions = maxNumOptions * 160 / collateralizationRatio;
```
{% endtab %}

{% tab title="Web3 1.0" %}
```javascript
const ocDai = oToken.at(0x3FDB...);

// Specify the amount of ETH collateral you want to put down in wei
const collateral = '1000000000000000000';

// This function tells you the maximum number of options you can safely issue at 160% collateralization. 
// Note: It is reccomended that you create less than this amount of options. 
const maxNumOptions = await ocDai.methods.maxOTokensIssuable(collateral).call();

// Assuming you want to be 200% collateralized
const collateralizationRatio = 200;
const numOptionsToIssue = maxNumOptions * 160 / collateralizationRatio;
```
{% endtab %}
{% endtabs %}

