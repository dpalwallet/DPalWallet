
# Api & demo

## use api in typescript

```javascript
const doge = window?.DogeApi;
```

## enable to interact

```javascript
const { status } = (await doge.enable()) || {};
if (result?.status === 'success') {
  const { userAddress } = await doge.userAddress();
  const { network } = await doge.network();
}

// or check isEnabled
if (doge && (await doge.isEnabled())) {
  const { userAddress } = await doge.userAddress();
  const { network } = await doge.network();
}
```

## request to pay(Test)

```javascript
if (await doge.isEnabled()) {
  const rs = await doge.useDoge(cost, toAddress, 'Buy Things');
  if (rs?.txid) {
    // successed
    // you can track the transaction is confirmed by txid in doge chain
  } else {
    // failed
  }
}
```

## timeout

```javascript
// enable useDoge api have 3 mins timeout
```

## event

```javascript
doge.on("networkchange", async (curNetwork) => {
  const data = await doge.userAddress()
  console.log(curNetwork, data.userAddress)
})
```

