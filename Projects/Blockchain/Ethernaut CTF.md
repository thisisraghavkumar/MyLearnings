# Learnings from OpenZepplin [Ethernaut CTF](https://ethernaut.openzeppelin.com/) 

## Level 1 - Sending tranasactions to a  contract

To send transactions to a contract at address `addr` use the `sendTransaction` function. This function takes a json object or a dictionary which contains fields such as *from*, *to*, *value* and *gas*. Transaction can be sent as follows -

```javascript
web3.eth.sendTransaction({'from': 'your_address', 'to': 'address', 'value' : 1})
```

## Level 2 - Constructor in contract

To add a constructor in the contract use the `constructor` keyword instead of creating a function with the name of the contract.

```solidity
contract MyContract {

    address payable private _owner;

    constructor() {
        _owner = msg.sender;
    }
}
```

## Level 3 - Source of randomness

Values such as *blockhash* and *timestamp* are often used as pseduo random values in smart contracts, but this is not secure. These values are not secure and other contracts can mimic them to attack the smart contract developed by us.

To reference a contract that has been deployed in the blockchain network, we need to know two things -
1. The definition of the contract
2. The address of the contract

It can be done as follows -
```solidity
// deployed at 0xabcdef0123456789
contract Callee {
    function calleeFunction() public returns (bool) {
        return true;
    }
}

contract Caller {
    Callee private calleeInstance;

    constructor() {
        calleeInstance = Callee(0xabcdef0123456789);
    }

    function callCallee() {
        calleeInstance.calleeFunction();
    }
}
``` 

## Level 4 - Difference between txn.origin and msg.sender

A transaction is a message between two ethereum addresses. When a contract receives a transaction the value of `msg.sender` is the address of the account that sent the transaction. However, it is not necessary that the transaction was started in the first place by an action of `msg.sender`. It night happen that *msg.sender* started the transaction in response to some transaction that it has received from another address. 

Consider this - Alice calls a contract *A*, this contract then does some processing and calls another contract *B*. When *B* receives this transaction from *A*, it sees *msg.sender* as *A* and *txn.origin* as Alice. `txn.origin` is often an account address that has triggered a chain of transactions. Even though each contract sees *msg.sender* as the last contract in chain that called this one, all of them will see the origin of chain in *txn.origin*.

## Level 5 - Beware of overflows

Solidity is a typed language, thus every variable has a data type and hence it has a fixed size. If the content in the variable exceeds its capacity, then the variable overflows and wraps to 0 as all the bits are set to 0 and increase from there. 

Therefore always take care of overflows or use SafeMath.sol  from OpenZepplin library.

To import OpenZepplin library when it is not installed, one can use the github link to the document to be imported. Like this - 
```solidity
import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/math/SafeMath.sol";
```

## Level 6 - Delegation

There is a mechanism in solidity, whereby one can execute a code written in one contract in the context of another contract. What that means is that A.sol will execute it's logic and affect the memory and storage of B.sol. This is done by using the `delegatecall` method in solidity. 

```solidity
// deployed at 0xabcdef0123456789
contract A {
    address private _owner;
    function setOwner(address addr) returns (address){
        address memory oldAddress;
        oldAddress = _owner;
        _owner = addr;
       return oldAddress;
    }
}

contract B {
    address private _owner;
    address A libraryCode;

    constructor() {
        _owner = msg.sender;
        libraryCode = 0xabcdef0123456789
    }

    function delegateReset() external {
        bytes memory payload = abi.encodeWithSignature("setOwner(address)",0x00000000000000000);
        (bool success, bytes memory result) = libraryCode.delegateCall(payload); // this will set the owner of this contract to 0x0
    }
}
```

[Addresses](https://docs.soliditylang.org/en/v0.8.0/types.html#address) in solidity have two other methods for interfacing with other contracts when their abi is not known. These functions are `call` and `staticcall`. `call` simply calls the function in other contract whereas `staticcall` behaves like `delegatecall` but unlike the latter it reverts if the code tries to modify the state of the calling contract.

## Level 7 - You can always send ether to an address

Currently there are three ways to send ether to a contract address -
1. The contract implements a `receive` method so that it can receive ether from other addresses.
2. The contract receives mining reward as it is set as the address to which mining rewards should be sent.
3. Some other contract self destructs and sends any remaining balance to this contract's address.

`selfdestruct(address)` method will cause a contract to self destruct i.e. freeing up the space its code and state is taking and send any remaining balance to the address given in the function call. Contracts should call `selfdesctruct` because it is an operation with negative gas i.e. it pays to self destroy a contract s it promotes network well being by freeing up space.  

## Level 8 - Private is not always secret

Usually we expect that a private variable cannot be read outside of an object, however in ethereum private values can be read by examining the storage state, since everything is on the chain i.e. data is in a LevelDB store. State of a contract is stored in 2^256 slots of 32 bytes each. Each slots can be accessed by using its 0 based index.

To access the content of a slot in javascript call `web3.eth.getStorageAt(addr,slotNo)`. This will return the data in that slot in hex format, which can be converted into ascii by using the function `web3.utils.hexToAscii(hexData)`.

Therefore we must always be careful to not store secrets as plaintext in contracts because nothing is secret, not even private members. If secrets has to be stored it is better to hash and then store them as private and on receiving a plaintext value, hash t and compare to the secret hash. This is also not very advisable yet it is better than storing plaintext in contracts.

## Level 9 - King

If a contract does not have a payable `receive` or `fallback` function, then any attempts to send transaction to it's address will result in a failure, except if `selfdestruct` is used as shown in ***Level 7***.

One can create a transaction to call a function in a smart contract, we need to know the address of the contract, the signature of function and the arguments to be passed to the function. The signature of a function includes the name of the function and the type of its formal arguments. E.g. the signature of a function `function myFunction(string str, uint n, bytes32 b) public returns (bool)` is "myFunction(str,uint,bytes32)". Notice that only the name and the type of arguments of the function is included. Suppose we have a function that allocates a number of tokens *(n)* to an address *(addr)* and it is declared as `allocateToken(address addr, unit n)` and we wish to call it with *n* = 101 and *addr* = 0xabcdef0123456789. If we do not have access to the code of the contract and we only know this much then we an call this function in following ways.

First - get the abi encoded value of the function signature along with the arguments, like this
```javascript
var encodedPayload = web3.eth.abi.encodeWithSignature('myFunction(address,uint)')+web3.eth.abi.encodeParameter('address','0xabcdef123456789').substring(2)+web3.eth.abi.encodeParameter('uint',101).substring(2)
``` 
We can also use an object describing the function to get the encoded data, usually truffle compiled contracts give you such definition in json format but they can be written by hand as well, e.g. -
```javascript
func = {name: 'myFunction', type: 'function', inputs: [{type: 'address', 'name': 'addr'}, {type: 'uint', name: 'n'}]}
var encodedFunctionCall = web3.eth.abi.encodeFunctionCall(func, ['0xabcdef0123456789', 101])
```

Both *encodedPayload* and *encodedFunctionCall* have the same string value and an equal to operator between them will result in **true**.

Now there are two ways to create the transaction
1. Using web3 API
2. Using metamask

To make a transaction call using web3 API, call
```javascript
var txn = {from: 'accountAddress', to: 'contractAddress', data: encodedPayload};
web3.eth.sendTransaction(txn);
```

To make a call using metamask you need to specify the contract address in to field and then paste the *encodedPayload* value in the hex data field. One needs to switch on this feature from advance setting otherwise the hex data fields is not available.

It is also possible to call a function in solidiy this way. This is requred only when we do not know the abi but only the function signature. It can be done as follows.

```solidity
address(contractAddress).call(abi.encodeWithSignature('myFunction(address,uint)',0xabcdef0123456789,101))
```

We can specify the value in the transaction and the gas using `address(adr).call{value: val, gas: gas}("...")`.

## Level 10 - Why prefer high level functions

In solidity we can do a lot using low level functions such as `call`, including sending ether to other addresses. Almost all the things can be done using this function but this is not advisable, because if the call stack fails at any stage of call stack the execution is not reverted i.e. the stage is not restored to original but the function returns false, while any change made to the state persist. Therefore it is always advisable to use high level alternatives such as `transfer` and if there is a payable function named *func* then make calls using `func{value: 1 ether, gas: 2000000 wei}(argument)` calls. 

Whenever you send ether to any address, beware that the address can be of a contract and sending ether can start a chain of transactions if the receiving contract is a malicious, therefore always take great care while sending ether out of contracts.

## Level 11 - State modifiers

State modifiers in solidity can be used to prescribe what a function can or cannot do with storage. These modifiers are -
1. [VIew](https://docs.soliditylang.org/en/develop/contracts.html#view-functions) - View function promise to not modify the state but there is no run time check of this. Therefore be careful to use a view function for which the implementation is unknown.
2. [Pure](https://docs.soliditylang.org/en/develop/contracts.html#pure-functions) - Pure functions promise to neither modify nor read from state. While checks preventing state writes are enforced, there are no guarantees that the function will not read from state.

In this level we need to create a function that returns different values when called different number of times.

## Level 12 - Privacy

All the data that is stored on the ethereum chain can be read, even if it is marked private in a contract. Therefore it is never a goof idea to store secrets in plaintext on a public chain like ethereum.

The storage of a contract is organized as 2^256 slots of 32 bytes each. The slots are 0 indexed. To read a slot at index 'p' we can use the *[getStorageAt()](https://web3js.readthedocs.io/en/v1.3.4/web3-eth.html#getstorageat)* function in javscript 
```javascript
await web3.eth.getStorageAt(contract_address, p);
```
The state variables are stored in the storage with [this layout](https://docs.soliditylang.org/en/v0.8.5/internals/layout_in_storage.html#layout-of-state-variables-in-storage).

## Level 13 - Gatekeeper One

`txn.origin` is always an externally owned account's address that triggers a chain of transactions, whereas `msg.sender` is the address of the last address in this chain.

While typcasting between types explicitly, the lower order bytes are preserved and the higher order bytes are cut. Thus if a byte32 value is typecasted or **compared** to a bytes16 value, then only the lower 16 bytes of the 32 bytes will be taken into account.

Masking is an alternative to explicit type casting between data types, especially bytes. Masking should be preferred as it is faster than type casting. Masking can be done using the `&` operator.

`web3.utils.hexToBytes()` can be used to convert hexadecimal strings to bytes. Also note that `bytes` when printed in javascript are shown as an array. Once can use a *toString()* function on them to display their value as a number.

## Level 14 - Gatekeeper Two

The negate operator `~` can be used to perform a bitwise negation of data in solidity. The *xor* operation is carried using `^` in solidity.

It is possible to write inline assembly in solidity code using a language called [Yul](https://docs.soliditylang.org/en/v0.8.5/assembly.html). Although the language itself is not exactly assembly, yet it is powerful enough to give greater control to developer as well as provide more familiar coding constructs. Inine assembly in solidity looks like following 
```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.4.16 <0.9.0;

library GetCode {
    function at(address _addr) public view returns (bytes memory o_code) {
        assembly {
            // retrieve the size of the code, this needs assembly
            let size := extcodesize(_addr)
            // allocate output byte array - this could also be done without assembly
            // by using o_code = new bytes(size)
            o_code := mload(0x40)
            // new "memory end" including padding
            mstore(0x40, add(o_code, and(add(add(size, 0x20), 0x1f), not(0x1f))))
            // store length in memory
            mstore(o_code, size)
            // actually retrieve the code, this needs assembly
            extcodecopy(_addr, add(o_code, 0x20), 0, size)
        }
    }
}
```
Notice the keyword ***assembly*** and the following code block.

`extcodesize` is a solidity assembly instruction which returns the size of the code at an address. This operation has to be carried out using inline assembly as there is no compiler supported solidity statement for this operation, yet.

There are *two* situations when this operation will return 0


1. When it is called for an externally owned address or a user's address
2. When it is called for a contract whose constructor has not yet finished execution.

More perspective can be given for case b above with this example - the constructor of a contract calls a library function, the library function in turn calls the `extcodesize` function with the address `msg.sender`. Now since the constructor of the contract at address `msg.sender` is not yet finished, therefore the opcode will return 0.

## Level 15 - Naught Coin

ERC20 is a standard which defines the interface or functions that a contract must have to issue fungible tokens to its users. All the tokens that follow this standard are called ERC20 tokens. The standard lays down the functions and the events a contract must execute to manage the tokens owned by users. While it is only a standard, it has been materialized as an interface as well as classes that implement the interface. Many organizations have published their implementation of the interface and the token code e.g. [OpenZepplin](https://docs.openzeppelin.com/contracts/2.x/api/token/erc20) has an ERC20 [interface](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/IERC20.sol) as well as a [contract](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol).

An ERC token defines many functions that a user can execute for their coins. The holding are managed using a mapping from address to unsigned integer to store the balances of the user. There is another mapping of the form *address* => *address* => *uint* called allowance which stores the information about how much a user can spend on behalf of another users. Users can approve other addresses to spend some coins on their behalf, this quantity is called an allowance. There are functions to approve allowances as well as function to spend coins on behalf of other addresses as long as the spending address has some allowance with the desired owner address.

## Level 16 - Preservation

`delegatecall` is a solidity operation which allows executing the logic defined in another contract on the context of one's own contract. By context we mean the storage and memory of the contract. Thus libraries can define the logic in their functions and then they are called in customer contracts using *delegatecall*. Let's take an example 
```solidity
contract Library {
    uint private a;
    uint private b;
    function setValue(uint _v) {
        a = _v;
    }
}

contract Business {
    uint private a;
    function setValueUsingLibrary(uint _v){
          address(0xaabbccddeeff0011223344556677889912345678).delegatecall(abi.encodeWithSignature("setValue(uint256)", _v));
    }
}
```

Then on calling the *setVaueUsingLibrary* function, the value of *Business.a* integer will be set to *_v*. Do note that the *library* function will execute as if it is executing in the *business* contract. Therefore it will assume the **same** memory layout. Thus if the library function is updating the third state variable declared in the Library contract, it will try to update the third state variable in the Business contract as well, irrespective of the type and size of the third variable.

```solidity
contract SneakyContract {
    address dummyVar1;
    address dummyVar2;
    address owner;
    
    function setTime(uint _time) public{
        owner = address(_time);
    }
}

contract LibraryContract {
    uint storedTime;  

    function setTime(uint _time) public {
        storedTime = _time;
  }
}

contract Preservation {
    address public timeZone1Library;
    address public timeZone2Library;
    address public owner; 
    uint storedTime;
    uint _timestamp = 100000;
 
    function setFirstTime(uint _timeStamp) public {
      timeZone1Library.delegatecall(abi.encodePacked(setTimeSignature, _timeStamp));
    }

    function setSecondTime(uint _timeStamp) public {
      timeZone2Library.delegatecall(abi.encodePacked(setTimeSignature, _timeStamp));
    }
}
```

In above code snippet the **Preservation** contract is the business contract and the **LibraryContact** and **SneakyContract** are library contracts. If the *setFirstTime* function in the **Preservation** contract calls the *setTime* function in the **LibraryContract** then the first state variable viz. *timeZone1Library* will be altered since logic in the library contract affects the first declared variable. However if the *setTime* function from the **SneakyContract** is called then the third variable viz. ***owner*** will be set as the library contract is setting the third state variable declared in the library.

Note that calling *setSecondTime* will have the same effect i.e. setting the first state variable as it is using the same library code.

## Level 17 - Recovery

A contract can create another contract using the new operation in solidity. The address of this new contract can be determined using the address of the contract and its nonce. Note that the nonce of a contract address increases with the number of contracts it creates, whereas the nonce of an externally owned account increases with every transaction.

One can deterministically calculate the address of a new contract that a given contract will create as follows -

```solidity
address nonce1= address(uint160(uint256(keccak256(abi.encodePacked(byte(0xd6), byte(0x94), address(this), byte(0x01))))));
address nonce2= address(uint160(uint256(keccak256(abi.encodePacked(byte(0xd6), byte(0x94), address(this), byte(0x02))))));
```

The first two arguments are constants used in RLP encoding, the third argument is the contract or any address and the last argument is the nonce. Note that the nonce seems simple only till a max value after which RLP encoding looks slightly different from the actual value. Note that the nonce for all contracts start with 1, therefore the first contract that will be created will have its address calculated with nonce set to 1. [note](https://ethereum.stackexchange.com/questions/760/how-is-the-address-of-an-ethereum-contract-computed)

## Level 18 - Magic Number

To create a new contract we can send a transaction with no `to` field or `to` field set as 0x0 to the chain. Such a transaction is interpreted by network as a contract creating transaction. The `data` field in the transaction contains the *bytecode* of the contract. This bytecode has two parts -
1. initialization code - these instructions set the instruction pointers, execute the contract constructor and returns the runtime code from memory which is then associated with the address of the contract.
2. Runtime code - is the set of instructions that actually form the execution code of the contract.

Note that the constructor of the contract is executed at the time of initialization and is not stored unlike the runtime code which is associated with the address of the contract. More about this can be read [here](https://blog.openzeppelin.com/deconstructing-a-solidity-contract-part-i-introduction-832efd2d7737/).

To solve this level we need to send the data which creates a contract that returns 42 "the meaning of life" when it is invoked. The assembly of this code looks like

```
# Initialization
PUSH1 0x0a # push the size of the runtime code in bytes (10) into the stack
PUSH1 0x0c # push the byte from which the runtime code will begin in this data (12) into the stack
PUSH1 0x00 # push the memory location where to place the runtime code (0) into the stack
CODECOPY # place the 10 bytes in msg.data from 12th byte at the 0th position in memory 
PUSH1 0x0a # push the size of the data in memory to be returned in bytes into the stack
PUSH1 0x00 # push the location in memory from which data has to be returned into the stack
RETURN # return 10 bytes from the 0th location in memory
# Run Time
PUSH1 0x42 # push the result 42 in stack
PUSH1 0x80 # push the memory location 80 in stack
MSTORE # store 42 in memory location 80
PUSH1 0x20 # push the size of data to be returned from memory (32 bytes) in the stack
PUSH1 0x80 # push the location of data to be returned from the memory (80) in the stack
RETURN # return 32 bytes of data from the 80th location in memory
#
```
The bytecode of this assembly code is `"0x600a600d600039600a6000f300602a60805260206080f3"`. Bytecode is constructed by concatenating the assembly code for the opcodes and data values. The opcodes can be found in the [yellow paper](http://gavwood.com/paper.pdf). The EVM is a stack based machine.

We send a transaction with this data to the network to create a contract

```javascript
web3.eth.sendTransaction({from: player, to: "0x0", data: "0x600a600d600039600a6000f300602a60805260206080f3"})
```

## Level 19 - Alien Codex

Dynamic arrays and mappings do not follow the same rules as state variables in the storage. Their storage layout is defined according to [these rules](https://docs.soliditylang.org/en/v0.8.5/internals/layout_in_storage.html#mappings-and-dynamic-arrays).

Note that the contract storage has 2^256 slots and entirety of this can be accessed using appropriate slot number if the storage contains a bytes32, or an equivalent sized, array.

## Level 20 - Denial

Always be careful while sending funds out of smart contracts. We can never be sure if the receiving address is a user or a smart contract. A malicious contract may contain code in receive or fallback functions to drain our smart contract of funds by creating an endless loop of withdrawal or failing actions.

Therefore always limit such transaction with limited gas fee. Prefer using `transfer` over `send` operation and try to follow the [check-effects-integration pattern](http://solidity.readthedocs.io/en/latest/security-considerations.html#use-the-checks-effects-interactions-pattern) to prevent redundancy attacks.

## Level 21 - Shop

Build a function that returns different values based on the reading of *isSold* public variable in Shop contract.

## Level 22 - Dex

This level can be completed using price manipulation. Note that this does not exploit any technical loophole in the technology, but in the business logic. By modifying the ratio of the two tokens in DEX, one can transfer more and more tokens from the DEX's balance to one's own balance. This can be done by carrying out a sequence of transactions.

1. Add liquidity to the Token 2 in DEX, use all the Token 2 available
2. Buy Token 2 using all the Token 1 available
3. Buy Token 1 using all the Token 2 available
4. Buy Token 2 using all the Token 1 available
5. ...
6. ...
7. ...
8. ...
9. ...
10. Buy all the Token 1 available in the DEX using te required number of Token 2

Note that this process may end sooner or later than 10 steps and you may not also need to use all your tokens in the last step.


First by adding to the liquidity of Token 2, we reduced its price wrt Token 1, then by using all the token  that we had we bought Token 2. With this repurchase we ended up with more Token 2 than we started with, thus we siphoned Token 2 from DEX to ourselves. this made Token 1 cheaper wrt. Token 2, then we used all of our token 2 to buy token 1, since token 1 was cheaper than earlier and we had more token 2 to spare than earlier we were able to buy more token 1 than we had spent in the first place, thus we siphoned token 1 out of DEX even though the DEX restored it's token 2 balance. But now the token 2 is cheaper than token 1 and we have more token 1...

We continue to iterate these steps till the time we're able to buy all the token 1 present in the dex.
