# Le Petit Frigo - Blockchain Demo

This repository contains an **experimental** Dapp quickly created during the [Agricathon 2019](https://hack.opendata.ch/event/27#top) in response to the [Le Petit Frigo](https://hack.opendata.ch/project/356) challenge.

Built with [Truffle 4](https://github.com/trufflesuite/truffle/releases) and [eth-vue](https://github.com/DOkwufulueze/eth-vue), it consists of a [Vue.js](https://github.com/vuejs) frontend with smart contracts written in [Solidity](https://solidity.readthedocs.io/), which can be deployed to an Ethereum blockchain. In development, [Ganache](https://github.com/trufflesuite/ganache), [Remix IDE](http://remix.ethereum.org) and truffle-cli were used.

## Development

After cloning this project locally, install [Ganache](https://github.com/trufflesuite/ganache) or set up another blockchain for development purposes.

`truffle console --network ganache`

With a command like above you can start the Truffle console, then enter

`migrate --reset --compile-all`

If not using Ganache, you'll want to configure the network URLs in `app-users/src/libs/mixinViews.js` and `truffle-config.js`

`cd app-users`

Enter the app subfolder, where you'll also find a [README](app-users/README.md) with additional details. Install the frontend dependencies (see additional info):

`yarn`

Then run it:

`yarn run dev`

Open your browser and play with the Dapp!

## Resources

- [DOkwufulueze/eth-vue](https://github.com/DOkwufulueze/eth-vue)
- [Tutorial by Daniele Favi](https://www.danielefavi.com/create-your-blockchain-dapp-with-ethereum-and-vuejs/) ([GitHub](https://github.com/danielefavi/ethereum-vuejs-dapp))
- [Testing Solidity contracts in Remix](https://ambercontracts.wordpress.com/2019/02/13/testing-solidity-contracts-in-remix/)
- [vue-cli3-truffle-box](https://github.com/marcelobbfonseca/vue-cli3-truffle-box)
