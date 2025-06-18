# Introducing for new version Eth-Bundle-Launch v1.2 release
**Ethereum, BSC Token Bundle Launch Project Ver 1.2 - 300+ wallets supported!**

The new version is here! Eth-Bundle-Launch v1.2 brings improved stability, cleaner transaction handling, and optimized support for bundle buys and dispersals. Designed for speed and control, this release makes launching and managing your ETH-based bundles more efficient than ever

***Feel free to reach out to our team on [Telegram](https://t.me/trustaibotsdevteam) or [devteam@trustaibots.com](mailto:devteam@trustaibots.com) if you have any questions or want to launch***

In version 1.1, the number of buyer wallets was limited to 48. However, in version 1.2, there is no longer limitation available, and new features have been added—such as automatic tax updates at the end of the bundle buy. This means you no longer need to worry about setting tax percentages manually during the bundle process.

This project facilitates a bundle token launch for an ERC-20 token involving at most 300 wallets for sniping. The process is divided into 4 steps:
1. Token Contract deployment
2. Estimate the required ETHs for each buyer wallets
3. Acquire Authorization Key from bloXroute
4. Configure and launch script

## 1. Token Contract deployment

The project has 2 main components, one is the script and another is the smart contract. 

### Steps to Customize the Token Contract:

Navigate to either ```contracts/contract-mainnet.sol``` or ```contracts/contract-bsc.sol```, depending on your target network:

- ```contract-mainnet.sol```: Contract for Ethereum Mainnet
- ```contract-bsc.sol```: Contract for BSC Mainnet

The only difference between these contracts is the Uniswap V2, Pancakeswap V2 router address specified in the code.
Customize the contract as needed to fit your requirements.

To ensure that the initial buy/sell tax doesn't affect bundle purchases, you must set the following variables to zero:

```
uint256 private _buyTax = 0;
uint256 private _sellTax = 0;
```

After the bundle buy is complete, the final transaction should be a tax update that sets the desired tax rates (for example, 30% buy tax and 35% sell tax). This ensures the new tax rates apply to users' regular buys after the bundle purchase.

Make sure your contract includes a function named setTaxes(buy, sell) to handle tax changes, like this:

```
function setTaxes(
        uint256 newFeeOnBuy,
        uint256 newFeeOnSell
    ) public onlyOwner {
        _buyTax = newFeeOnBuy;
        _sellTax = newFeeOnSell;
    }
```

### Deploy the Token Contract

- Use [Remix](https://remix.ethereum.org/) to deploy the contract

Once deployed, ensure that:
- You have transferred ETH to the contract. This ETH will serve as the amount for initial liquidity.
- You have transferred tokens to the contract. These tokens will also be used for initial liquidity.

### Updating the ABI

If any function or variable names in the contract are modified:

- Download the updated ABI JSON file using Etherscan or Remix.
- Replace the file located at ```src/abi/token-abi.json``` with the new ABI JSON file.

## 2. Estimate the required ETHs for each buyer wallets

Use python script ```./calculate_eth_for_wallets.py``` on Ubuntu

- Install python3
```
apt install python3
```

- Modify parameters

```python
// Parameters in calculate_eth_for_wallets.py
...
initial_eth = 2.0	// ETHs in initial liquidity 
initial_tokens = 100    // Tokens in initial liquidity (100 is fine, no need to change this parameter)
num_wallets = 95 	// Wallet count to be used as buyers
target_percentage = 95  // Token hold percentage by sniping
...
```

- Execute the python script
```
python3 calculate_eth_for_wallets.py
```
Execution results are
```bash
Wallet 1: Needs 0.0325 ETH
Wallet 2: Needs 0.0336 ETH
Wallet 3: Needs 0.0347 ETH

...

Wallet 49: Needs 0.6386 ETH
Wallet 50: Needs 0.7407 ETH
Total ETH required by all wallets: 8.0000 ETH
```

## 3. Acquire Authorization Key from bloXroute

Sign up for a bloXroute account and visit [https://portal.bloxroute.com/details](https://portal.bloxroute.com/details) to obtain your Authorization Header. This is your API key used to authenticate with bloXroute’s bundle services.More actions

Make sure your account has an active API key. To simulate or submit a high volume of bundles, you may need to upgrade to a premium plan. You can explore available plans at [https://portal.bloxroute.com/plans](https://portal.bloxroute.com/plans).

## 4. Configure and launch script

### Configure 

Navigate ```src/config.ts``` and configure the script

```ts
export const MAINNET_RPC= "https://boldest-bold-uranium.quiknode.pro/..../"
// Replace this with your eth mainnet rpc
export const BSC_MAINNET_RPC="https://rpc.ankr.com/bsc/..."
// Replace this with your bsc mainnet rpc
export const SEPOLIA_RPC="https://old-green-glade.ethereum-sepolia.quiknode.pro/.../"
// Replace this with your sepolia testnet rpc

export const VERSION = 1.2

export const API_KEY = "" // bloXroute Authorization Key

// Ethereum Mainnet = 1
// Binance Smart Chain Mainnet = 56
// Sepolia Testnet = 11155111
export const NET_MODE = 1  // Set this to the desired chain ID: Ethereum or BSC Mainnet. 
// Note: bloXroute only supports Ethereum and BSC mainnets. Testnets like Sepolia are not supported—you can try them for initial testing, but bloXroute will return a failure response.

export const ZOMBIE_PRIVATE_KEY = "0x....."  
// Replace this zombiw wallet key with your own private key.
// The zombie wallet is a dedicated ETH wallet used to manage and distribute funds for bundle buys, sells, and transaction fees.

export const OWNER_PRIVATE_KEY = "0xa...."; 
// Replace this deployer wallet key with your own private key.

export const BRIBE_PAYER_KEY = "0x..."; 
// Replace this briber wallet key with the one of the bribe payer you have.

export const BUYER_PRIVATE_KEY = [
    "0x....", // Replace with buyer wallet private keys.
    "0xc....", // ...
    "0x....", // ...
    // Add more keys as needed...
    "0x....1", // ...
    "0x...", // ...
];

export const BUY_AMOUNT = [
    "0.0325", // Replace with the calculated ETH amount per wallet from result of calculate_eth_for_wallets.py.
    "0.0336", // ...
    "0.0347", // ...
    // Add more amounts as needed...
    "0.6386", // ...
    "0.7407", // ...
];

// Each private key in the BUYER_PRIVATE_KEY array must correspond to the same index in the BUY_AMOUNT array. 
// In other words, the size of both arrays should be the same, and each private key should match the respective buy amount

export const TOKEN_ADDRESS = "0x..."; 
// Replace with the address of your deployed token contract.

export const BRIBE_AMOUNT = 0.1; 
// Set the bribe amount to be used with the flashbots / bloXroute provider.

export const BUY_TAX = 30       // buy tax percent
export const SELL_TAX = 35      // sell tax percent

export const USE_MAESTRO_ROUTER = 1      // 1 : use maestro router, 0: use uniswap router

export const SWAP_FEE_PER_BUYER = 0.012; // Swap fee in ETH reserved with each buyer for bundle buy or future sells (not for the token purchase itself)
```

### Launch preparation

- Ensure that all the wallets (Deployer, Briber, and Buyer) have enough balance to cover the transaction fees for sniping the tokens.

    Deployer Wallet: Must have enough balance to cover the transaction fee for executing the ```openTrading``` function in the token contract.

    Briber Wallet: Must have enough ETH to transfer the bribe and cover the associated transaction fee.

    Buyer Wallets: Each buyer wallet must have enough ETH to purchase tokens, along with the transaction fee required for the token swap.

- Confirm that no liquidity pool already exists for the token contract.
- Make sure that ETH and tokens have already been transferred to the token contract address.
- Verify that the amount of tokens each buyer intends to purchase does not exceed the max-transaction limit. If necessary, adjust the ETH balance in the buyer wallets to accommodate the swap.

### Launch the script 

#### Run the following commands
```bash
npm install
```
Or
```bash
npm install -g yarn
yarn install
```

#### To start the script:
```bash
npm start
```
Or
```bash
yarn start
```

### Note 

- If you have renamed the 'openTrading' or 'setTaxes' function in the contract, make sure to update the name in the ```snipe.ts``` file as well.

```ts
...
let estimatedGas
const openTradingData = token_contract.methods.openTrading()
// Update the function name here to match the changes in your contract. Ensure the ABI is also updated as mentioned earlier.

try {
    estimatedGas = await openTradingData.estimateGas({
...

const setTaxData = token_contract.methods.setTaxes(BUY_TAX, SELL_TAX);
// Update the function name here to match the changes in your contract. Ensure the ABI is also updated as mentioned earlier.
try {
    estimatedGas = await setTaxData.estimateGas({
```

- Always double-check that the ABI (```abi/token-abi.json```) is updated to reflect any changes in the contract before proceeding.

- If you keep seeing the error message 'Miner did not approve your transaction,' try increasing the bribe ETH amount by setting the ```BRIBE_AMOUNT``` value in ```src/config.ts``` to a higher number.

# Contact us for your launch

Feel free to reach out to our team on [Telegram](https://t.me/trustaibotsdevteam) or [devteam@trustaibots.com](mailto:devteam@trustaibots.com) if you have any questions or want to launch bundle

# Author

- Github: [Dev Team](https://github.com/trustaibots)
- Email: [devteam@trustaibots.com](mailto:devteam@trustaibots.com)
- Telegram: https://t.me/trustaibotsdevteam
- Website: https://trustaibots.com
