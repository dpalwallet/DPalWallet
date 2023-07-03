
# Api & demo

## use api in typescript

```javascript
const doge = window?.DogeApi;
```

## enable to interact

```javascript
const { status } = await doge.enable();
if ( status === 'success') {
  const { userAddress } = await doge.userAddress();
  // const { network } = await doge.network();
}

// or check isEnabled
if (await doge.isEnabled()) {
  const { userAddress } = await doge.userAddress();
  // const { network } = await doge.network();
}

// full demo
if(doge){
  if (await doge.isEnabled()) {
    const { userAddress } = await doge.userAddress();
    // const { network } = await doge.network();
  } else {
    const { status } = await doge.enable();
    if ( status === 'success') {
      const { userAddress } = await doge.userAddress();
      // const { network } = await doge.network();
    }
  }
}else{
  console.log(`please install dpal wallet`)
}
```

## request to pay

```javascript
if (await doge.isEnabled()) {
  const rs = await doge.useDoge(cost, toAddress, 'Buy Things Info');
  if (rs?.txid) {
    // successed
    // you can track the transaction is confirmed by txid in doge chain
    // for example use chain.so or orther doge api serveices
    // curl "https://chain.so/api/v3/transaction/DOGE/597bafa25fcbb081467bdeb030a42bf441dbfcc054bdcfad31a829d7db5d931f" -H "API-KEY: {{api_key}}"
    // https://chain.so/api/ 
  }
}
```

## request to sign message

```javascript
// use this method to login server can send token by decoded sig
if (await doge.isEnabled()) {
  const rs = await doge.sign();
  if (rs?.message && rs.sig) {
    // rs.sig example : H5DYFib9KhCRnOpb63/qNTROn6mrvXPuNw5aoogwYNaEBF2QP4uKo5CDPbJmZNiO7HBJIETaLLtSPpU9dVtkSzE=
    // you can use bitcore-lib-doge recover this signature and get r,s,v
  }
}
```

## transfer d20(^v1.2.2)

```javascript
if (await doge.isEnabled()) {
  const inscriptionid = "inscriptionid"
  const receiver = "receiver address"
  const rs = await doge.transferd20(inscriptionid,receiver);
  if (rs?.txid) {
    // successed
    // you can track the transaction is confirmed by txid in doge chain
    // for example use chain.so or orther doge api serveices
    // curl "https://chain.so/api/v3/transaction/DOGE/597bafa25fcbb081467bdeb030a42bf441dbfcc054bdcfad31a829d7db5d931f" -H "API-KEY: {{api_key}}"
    // https://chain.so/api/ 
  }
}
```

## timeout

```javascript
// enable useDoge api have 3 mins timeout
```
