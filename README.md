# DPal Wallet
### A browser extension based wallet to interact with DogeCoin directly. Make it easier to tip and spend DOGE in secure way.

## UI DEMO 
![Demo](https://github.com/dpalwallet/DPalWallet/blob/3619481f94832cb6ecf968bf85d6256847a97f8c/DPal%20Wallet%20ui.gif)

## web apps intract with DogeCoin directly
[![Watch the video](https://img.youtube.com/vi/Fm1oTfiJZ58/hqdefault.jpg)](https://youtu.be/Fm1oTfiJZ58)

## Install (now beta)
[![Chrome Extension](https://www.google.com/chrome/static/images/chrome-logo.svg)](https://dpalwallet.github.io)

## Description

A crypto wallet only for Doge Coin.

A browser extension based wallet to interact with DogeCoin directly. Make it easier to tip and spend DOGE in secure way.

It support DEV a "web3" app for DogeCoin

DPal keep the private key locally and won't record the private key in the server.

So your Doge is your Doge, but keep the phrase(12 words)/key safty in your own way it's necessary. You can use phrase  
recovery the wallet at any time.

* dpal wallet is easy to spend doge
* Any website can intract with doge block chain easily and Directly widthout mid services throgh the injected API.
* [API and demo](./api.md)

## FAQ

- Why useable amount not eq balance of address ? 

  Because the UTXO of the change is not confirmed or the network delay

- Why notice error when i send ? 

  Network error or used unconfirmed utxo (it have some limit at Doge node when use unconfirmed utxo), and Because of wallet is 
  chrome based we dont tend to make a tx sender queue (maybe release later if complain too many).

- Can the mnemonic be imported into other wallets for recovery?

  Yes,You can import into other wallets that support BIP39

- Can the account imported with the private key be recovered through the mnemonic phrase?

  No,you can't
 
- Can I find a lost mnemonic?

  You will find the mnemonic in the show mnemonic page, But if you clear the computer data and no backup elsewhere, that means you can't find it back.

- I have a large amount of DOGE, it is recommended to transfer to a cold wallet?
   
  yES,Although this wallet can store private keys, it is recommended to only store amounts of DOGE for tips and spend 
   
## Contact Us

E-mails: mailto:dpalwallet@outlook.com

donate address : DEGzHPRaiFMhEip819w29iR6tTG8HajvAo

