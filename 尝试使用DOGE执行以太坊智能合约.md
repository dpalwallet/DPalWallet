
### 尝试使用 DOGE 执行以太坊智能合约

#### 旨在探讨如何使用ECDSA签名验证技术，在公共的智能合约基础设施（例如以太坊）中使用 DOGE 等价支付 gas 费用 执行智能合约, 并用狗狗币地址记录虚拟资产 。

以常见的NFT 为例, 在NFT智能合约中，虚拟资产的唯一 tokenId 一对一或者一对多 映射了以太坊地址来保证所属权，事实上我们或许可以用tokenId 映射 狗狗币地址。理论上是可行的，问题在于如何确保只有狗狗币地址的拥有者才能操作对应的虚拟资产,以及如何单纯使用狗狗币支付gas费用，为此我考虑了这两个问题的解决方案。

#### 1.利用 ECDSA 签名验证技术

原理如下: 在铸造、转移、销毁等交互函数中，传入该地址对应的私钥签名（签名时间戳消息），并在铸造函数中记录恢复的以太坊地址（中间验证地址），之后每次更改操作都通过恢复以太坊地址（中间验证地址）与第一次记录的验证地址进行比对，即可验证该签名，这样可以保证 tokenId 的所属权归该狗狗币地址所有，并且他人无法更改。

```solidity

    // Mapping from token ID to owner doge address
    mapping(uint256 => string) public _owners;
    
    // Mapping from owner doge address to validate address
    mapping(string => address) public validator;

    // mint a nft
    function _mint(
        string memory dogeAddress,
        string memory timestamp_msg,
        uint256 tokenId, 
        bytes32 r,
        bytes32 s,
        uint8 i
    ) public {
        
        require(!_exists(tokenId), "ERC721: token already minted");

        uint256 msgTime;
        bool err;
        (msgTime,err) = strToUint(timestamp_msg);

        require(err, "ERC721: string convert to uint246 error");
        require(now()*1000 - msgTime <= 5*60*1000, "ERC721: sign allowed in 5min");

        address validatorAddress = recoverEcdsa(strTosha256(timestamp_msg),i,r,s); // compute validate eth address

        _owners[tokenId] = dogeAddress; // doge coin address
        validator[dogeAddress] = validatorAddress;
        
    }
    
    // transfer a nft
    function _transfer(
        uint256 tokenId,
        string memory from, // dogeaddress 
        string memory to, // doge address
        string memory timestamp_msg,
        uint8 i,
        bytes32 r,
        bytes32 s
    ) public {

        uint256 msgTime;
        bool err;
        (msgTime,err) = strToUint(timestamp_msg);

        require(err, "ERC721: string convert to uint246 error");
        require(now()*1000 - msgTime <= 5*60*1000, "ERC721: sign allowed in 5min");

        require(validator[from] != address(0));
        
        address adr0 = recoverEcdsa(strTosha256(timestamp_msg),i,r,s);
        require(adr0 == validator[from]);

        _owners[tokenId] = to;
        validator[to] = adr0;
    }
    
```

#### 2.代理以太坊账户 等价/定价 支付 gas 费用

以 web app 为例，用户每次操作传入签名数据后，服务端先预先估算所需 gas 费用，换算成 Doge 数量后，由用户支付并获取交易Hash，交易确认后，由服务端代理以太坊账户执行交易。如果交易出错，则将损耗的费用由用户或者服务端自己承担，实际上由于 doge 的 网络费用极低，这样频繁的退还交易操作，不会影响用户体验。以下为简单的 B/S 架构图。
![Demo](https://github.com/dpalwallet/DPalWallet/blob/main/image.png)

#### 3.代码及实现示例（简单的 NFT 代码）

3.1 智能合约代码

```solidity

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.2;

import "@openzeppelin/contracts/utils/cryptography/ECDSA.sol";
import "@openzeppelin/contracts/utils/cryptography/SignatureChecker.sol";

contract DogeNFT {

    // Mapping from token ID to owner doge address
    mapping(uint256 => string) public _owners;
    mapping(string => address) public validator;

    function now() public view returns (uint256){
        return block.timestamp;
    }

    function strToUint(string memory _str) public pure returns(uint256 res, bool err) {
        for (uint256 i = 0; i < bytes(_str).length; i++) {
            if ((uint8(bytes(_str)[i]) - 48) < 0 || (uint8(bytes(_str)[i]) - 48) > 9) {
                return (0, false);
            }
            res += (uint8(bytes(_str)[i]) - 48) * 10**(bytes(_str).length - i - 1);
        }
        
        return (res, true);
    }

    function varintBufNum(uint n) public pure returns(bytes memory){
        if(n < 253){
            return abi.encodePacked(uint8(n));
        }else if (n < 0x10000){
            return abi.encodePacked(uint8(253),uint16(n));
        }else if (n < 0x100000000){
            return abi.encodePacked(uint8(254),uint32(n));
        }else{
            return abi.encodePacked(
                uint8(255),
                int(int(n) & int(-1)),
                uint(n) / 0x100000000
            );
        }
    }

    function strTosha256(string memory msg) public pure returns(bytes32){
        string memory MAGIC_BYTES = "Dogecoin Signed Message:\n";
        bytes32 m = sha256(
            abi.encodePacked(
                varintBufNum(abi.encodePacked(MAGIC_BYTES).length),
                MAGIC_BYTES,
                varintBufNum(abi.encodePacked(msg).length),
                msg
            )
        );
        return sha256(abi.encodePacked(m));
    }


    function recoverEcdsa(       
        bytes32 hash,
        uint8 recoveryid,
        bytes32 r,
        bytes32 s) public pure returns(address){
        uint8 v = recoveryid + 27;
        return ecrecover(hash, v, r, s);
    }
    
    /**
     * @dev Returns whether `tokenId` exists.
     *
     * Tokens can be managed by their owner or approved accounts via {approve} or {setApprovalForAll}.
     *
     * Tokens start existing when they are minted (`_mint`),
     * and stop existing when they are burned (`_burn`).
     */
    function _exists(uint256 tokenId) internal view virtual returns (bool) {
        bytes memory dogeAddress = bytes(_owners[tokenId]); // Uses memory
        return dogeAddress.length != 0;
    }

    // mint a nft
    function _mint(
        string memory dogeAddress,
        string memory timestamp_msg,
        uint256 tokenId, 
        bytes32 r,
        bytes32 s,
        uint8 i
    ) public {
        
        require(!_exists(tokenId), "ERC721: token already minted");

        uint256 msgTime;
        bool err;
        (msgTime,err) = strToUint(timestamp_msg);

        require(err, "ERC721: string convert to uint246 error");
        require(now()*1000 - msgTime <= 5*60*1000, "ERC721: sign allowed in 5min");

        address validatorAddress = recoverEcdsa(strTosha256(timestamp_msg),i,r,s);

        _owners[tokenId] = dogeAddress;
        validator[dogeAddress] = validatorAddress;
    }

    function _transfer(
        uint256 tokenId,
        string memory from, // dogeaddress 
        string memory to, // doge address
        string memory timestamp_msg,
        uint8 i,
        bytes32 r,
        bytes32 s
    ) public {

        uint256 msgTime;
        bool err;
        (msgTime,err) = strToUint(timestamp_msg);

        require(err, "ERC721: string convert to uint246 error");
        require(now()*1000 - msgTime <= 5*60*1000, "ERC721: sign allowed in 5min");

        require(validator[from] != address(0));
        
        address adr0 = recoverEcdsa(strTosha256(timestamp_msg),i,r,s);
        require(adr0 == validator[from]);

        _owners[tokenId] = to;
        validator[to] = adr0;
    }

}

```

3.2 与 DPal 插件钱包交互 

request to sign message(^v1.0.19)

```javascript

const doge = window?.DogeApi;

if (await doge.isEnabled()) {
  const rs = await doge.sign();
  if (rs?.message && rs.sig) {
    // rs.sig example : H5DYFib9KhCRnOpb63/qNTROn6mrvXPuNw5aoogwYNaEBF2QP4uKo5CDPbJmZNiO7HBJIETaLLtSPpU9dVtkSzE=
    // you can use npm install bitcore-lib-doge recover this signature and get r,s,v
  }
}
```

request to pay

```javascript
if (await doge.isEnabled()) {
  const rs = await doge.useDoge(cost, toAddress, 'Pay gas fees');
  if (rs?.txid) {
    // successed
    // you can track the transaction is confirmed by txid in doge chain
    // map the transaction id with your webapps orderid.
    // you don't need to build a complex address allocator for your system anymore.
  }
}
```


#### 4.遗留问题

ecdsa 签名的 recoverid 允许 0,1,2,3 四个值，大概率会是0，1 这两个值是符合 智能合约中的 ecrecover 函数的，小概率可能会出现 3，4 的情况（我并没有遇到这种情况），如果你避免出现3，4 以至于会令用户重新操作的情况，可以采用预言机的办法在智能合约之外部署签名查询验证器。
