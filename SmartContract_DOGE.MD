

### Trying to execute Ethereum smart contracts with DOGE

#### Aims to explore how to use ECDSA signature verification technology (or oracle)，Pay gas equivalently using DOGE in a public smart contract infrastructure (e.g. Ethereum) Execute smart contracts, And record virtual assets with Dogecoin addresses 。

Take common NFTs as an example, In NFT smart contracts，The unique tokenId of the virtual asset is mapped one-to-one or one-to-many with an Ethereum address to ensure ownership，In fact we might be able to map a Dogecoin address with a tokenId,theoretically possible，The problem is how to ensure that only the owner of the Dogecoin address can operate the corresponding virtual asset, And how to pay for gas with Dogecoin，For this I have considered solutions to these two problems.

#### 1.Utilize ECDSA Signature Verification Technology

In interactive functions such as mint, transfer, burn, etc.，Take Dogecoin address and corresponding private key signature (signature timestamp message) as function parameters，and record the recovered Ethereum address in the minting function（mid validator address），then every change operation after that is done by restoring the Ethereum address（mid validator addres）Compare with the verification address recorded for the first time，This ensures that the ownership of the tokenId belongs to the Dogecoin address，and cannot be changed by anyone other than the owner of Dogecoin。In addition to directly verifying signatures in smart contracts，The signature can also be verified by an oracle，This avoids the complexity of smart contracts（No need to record mid validator address）。

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

#### 2.Proxy Ethereum Account Equivalent/Pricing Pay Gas Fee

Take web app as an example，After the user enters the signature data for each operation，The server first estimates the required gas cost in advance，After converting to the number of Doge，Pay by user and get transaction Hash，After the transaction is confirmed，The transaction is executed by the server on behalf of the Ethereum account。If there is an error in the transaction, the cost of the loss will be borne by the user or the server itself.The following is a simple B/S architecture diagram.

![Demo](https://github.com/dpalwallet/DPalWallet/blob/main/image.png)

#### 3.Code and implementation example (simple NFT code)

3.1 Smart contract code

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

3.2 Interacting with the DPal Extension wallet (https://dpalwallet.github.io/)

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


#### 4.Remaining problem

The recoverid of the ecdsa signature allows four values of 0, 1, 2, and 3. The high probability will be 0 and 1. These two values are in line with the ecrecover function in the smart contract.But there is a very small probability that 2,3 may occur (I have not encountered this situation)，If you want to avoid a situation where 2,3 will make the user re-operate, you can deploy the signature query validator outside the smart contract by using an oracle.If you deploy the signature validator using an oracle, you actually avoid the need for recording mid validator address.
