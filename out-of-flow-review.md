# Out-of-Flow Review

Этот review покрывает config, init, routing-mutation и upgrade surface вокруг deposit/withdraw lifecycle. Он не про normal asset path, а про surrounding admin/config assumptions, на которых этот path держится.

## 1. L1GatewayRouter.initialize(...)

```solidity
function initialize(
    address _owner,
    address _defaultGateway,
    address,
    address _counterpartGateway,
    address _inbox
) public {
    GatewayRouter._initialize(_counterpartGateway, address(0), _defaultGateway);
    owner = _owner;
    WhitelistConsumer.whitelist = address(0);
    inbox = _inbox;
}
```

Что делает:

- инициализирует L1 router
- фиксирует `owner`, `defaultGateway`, `counterpartGateway`, `inbox`

Invariants:

- router initialization должен фиксировать корректный `owner`
- router initialization должен фиксировать корректный `counterpartGateway` и `defaultGateway`
- router initialization должен фиксировать корректный `inbox`

## 2. L1GatewayRouter.setDefaultGateway(...)

```solidity
function setDefaultGateway(
    address newL1DefaultGateway,
    uint256 _maxGas,
    uint256 _gasPriceBid,
    uint256 _maxSubmissionCost
) external payable virtual onlyOwner returns (uint256) {
    return
        _setDefaultGateway(
            newL1DefaultGateway,
            _maxGas,
            _gasPriceBid,
            _maxSubmissionCost,
            msg.value
        );
}
```

Что делает:

- меняет default routing path
- инициирует corresponding L2 router update

Invariants:

- default routing mutation должна быть privileged
- default routing mutation не должна оставаться только на L1

## 3. L1GatewayRouter.setOwner(...)

```solidity
function setOwner(address newOwner) external onlyOwner {
    require(newOwner != address(0), "INVALID_OWNER");
    owner = newOwner;
}
```

Что делает:

- переводит privileged authority над L1 router

Invariants:

- owner rotation должна быть privileged
- owner не должен переводиться в `address(0)`

## 4. L1GatewayRouter.setGateway(...)

```solidity
function setGateway(
    address _gateway,
    uint256 _maxGas,
    uint256 _gasPriceBid,
    uint256 _maxSubmissionCost,
    address _creditBackAddress
) public payable virtual override returns (uint256) {
    return
        _setGatewayWithCreditBack(
            _gateway,
            _maxGas,
            _gasPriceBid,
            _maxSubmissionCost,
            _creditBackAddress,
            msg.value
        );
}
```

Что делает:

- регистрирует gateway для token path
- передает mutation в downstream routing layer

Invariants:

- gateway registration path должен идти только через intended registration logic
- single-token gateway mutation не должна обходить downstream correspondence checks

## 5. L1GatewayRouter.setGateways(...)

```solidity
function setGateways(
    address[] memory _token,
    address[] memory _gateway,
    uint256 _maxGas,
    uint256 _gasPriceBid,
    uint256 _maxSubmissionCost
) external payable virtual onlyOwner returns (uint256) {
    return
        _setGateways(
            _token,
            _gateway,
            _maxGas,
            _gasPriceBid,
            _maxSubmissionCost,
            msg.sender,
            msg.value
        );
}
```

Что делает:

- batch-обновляет router-level gateway mappings
- отправляет corresponding update на L2

Invariants:

- batch routing mutation должна быть privileged
- batch update не должна обходить token/gateway correspondence checks

## 6. L2GatewayRouter.initialize(...)

```solidity
function initialize(
    address _counterpartGateway,
    address _defaultGateway
) public {
    GatewayRouter._initialize(_counterpartGateway, address(1), _defaultGateway);
}
```

Что делает:

- инициализирует L2 router
- фиксирует `counterpartGateway` и `defaultGateway`

Invariants:

- L2 router initialization должен фиксировать counterpart routing model

## 7. L2GatewayRouter.setGateway(...)

```solidity
function setGateway(
    address[] memory _token,
    address[] memory _gateway
) external override onlyCounterpartGateway {
    require(_token.length == _gateway.length, "WRONG_LENGTH");

    for (uint256 i = 0; i < _token.length; i++) {
        l1TokenToGateway[_token[i]] = _gateway[i];
        emit GatewaySet(_token[i], _gateway[i]);
    }
}
```

Что делает:

- принимает counterpart-driven batch routing mutation на L2
- обновляет L2 view of token-to-gateway mapping

Invariants:

- L2 routing mutation должна приходить только от `counterpartGateway`
- batch correspondence между token list и gateway list должна оставаться согласованной

## 8. L2GatewayRouter.setDefaultGateway(...)

```solidity
function setDefaultGateway(address newL2DefaultGateway)
    external
    override
    onlyCounterpartGateway
{
    defaultGateway = newL2DefaultGateway;
    emit DefaultGatewayUpdated(newL2DefaultGateway);
}
```

Что делает:

- меняет destination-side default routing branch на L2

Invariants:

- L2 default routing mutation должна быть counterpart-gated
- default routing mutation должна явно менять stored destination-side route

## 9. L1ArbitrumGateway.postUpgradeInit()

```solidity
function postUpgradeInit() external {
    address proxyAdmin = ProxyUtil.getProxyAdmin();
    require(msg.sender == proxyAdmin, "NOT_FROM_ADMIN");
}
```

Что делает:

- post-upgrade hook для proxy-controlled L1 gateway

Invariants:

- post-upgrade hook должен вызываться только proxy admin

## 10. L1ArbitrumGateway._initialize(...)

```solidity
function _initialize(
    address _l2Counterpart,
    address _router,
    address _inbox
) internal {
    TokenGateway._initialize(_l2Counterpart, _router);
    require(_router != address(0), "BAD_ROUTER");
    require(_inbox != address(0), "BAD_INBOX");
    inbox = _inbox;
}
```

Что делает:

- инициализирует L1 gateway
- фиксирует counterpart, router, inbox

Invariants:

- L1 gateway initialization не должен стартовать без valid router
- L1 gateway initialization не должен стартовать без valid inbox

## 11. L2ArbitrumGateway.postUpgradeInit()

```solidity
function postUpgradeInit() external {
    address proxyAdmin = ProxyUtil.getProxyAdmin();
    require(msg.sender == proxyAdmin, "NOT_FROM_ADMIN");
}
```

Что делает:

- post-upgrade hook для proxy-controlled L2 gateway

Invariants:

- post-upgrade hook должен вызываться только proxy admin

## 12. L2ArbitrumGateway._initialize(...)

```solidity
function _initialize(address _l1Counterpart, address _router) internal override {
    TokenGateway._initialize(_l1Counterpart, _router);
    require(_router != address(0), "BAD_ROUTER");
}
```

Что делает:

- инициализирует L2 gateway
- фиксирует L1 counterpart и router

Invariants:

- L2 gateway initialization не должен стартовать без valid router
