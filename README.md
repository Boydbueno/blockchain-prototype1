# blockchain-prototype

> A simple blockchain prototype build with Web3 and VueJS

## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev
```

## Running your own node

By default the prototype uses rinkeby test network, where the contract is deployed.
For testing on own node, set `useLocalNode`to true and update the contract address accordingly.

``` bash
# Starting rinkeby node from command line
geth --syncmode "fast" --rinkeby --rpc --rpccorsdomain="*" --ws --wsorigins="*" console  --rpcapi="db,eth,net,web3,personal,web3"

# Running the project on testrpc

# Install testrpc
npm install -g ethereumjs-testrpc

# Run testrpc
testrpc

# Run testrpc with specific mnemonic (for metamask)
testrpc -m insert the mnemonic phrase from metamask here
# Import the given private keys in metamask

# Run ethereum wallet on own node (https://github.com/ethereum/mist/releases)
"/Applications/Ethereum Wallet.app/Contents/MacOS/Ethereum Wallet" --rpc http://localhost:8545
```