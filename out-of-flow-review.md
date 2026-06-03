# Out-of-Flow Review

Этот review покрывает config, init, routing-mutation и upgrade surface вокруг deposit/withdraw lifecycle. Он не про normal asset path, а про surrounding admin/config assumptions, на которых этот path держится.

## 1. L1GatewayRouter.initialize(...)

Что делает:

- инициализирует L1 router
- фиксирует `owner`, `defaultGateway`, `counterpartGateway`, `inbox`

Invariants:

- router initialization должен фиксировать корректный `owner`
- router initialization должен фиксировать корректный `counterpartGateway` и `defaultGateway`
- router initialization должен фиксировать корректный `inbox`

## 2. L1GatewayRouter.setDefaultGateway(...)

Что делает:

- меняет default routing path
- инициирует corresponding L2 router update

Invariants:

- default routing mutation должна быть privileged
- default routing mutation не должна оставаться только на L1

## 3. L1GatewayRouter.setOwner(...)

Что делает:

- переводит privileged authority над L1 router

Invariants:

- owner rotation должна быть privileged
- owner не должен переводиться в `address(0)`

## 4. L1GatewayRouter.setGateway(...)

Что делает:

- регистрирует gateway для token path
- передает mutation в downstream routing layer

Invariants:

- gateway registration path должен идти только через intended registration logic
- single-token gateway mutation не должна обходить downstream correspondence checks

## 5. L1GatewayRouter.setGateways(...)

Что делает:

- batch-обновляет router-level gateway mappings
- отправляет corresponding update на L2

Invariants:

- batch routing mutation должна быть privileged
- batch update не должна обходить token/gateway correspondence checks

## 6. L2GatewayRouter.initialize(...)

Что делает:

- инициализирует L2 router
- фиксирует `counterpartGateway` и `defaultGateway`

Invariants:

- L2 router initialization должен фиксировать counterpart routing model

## 7. L2GatewayRouter.setGateway(...)

Что делает:

- принимает counterpart-driven batch routing mutation на L2
- обновляет L2 view of token-to-gateway mapping

Invariants:

- L2 routing mutation должна приходить только от `counterpartGateway`
- batch correspondence между token list и gateway list должна оставаться согласованной

## 8. L2GatewayRouter.setDefaultGateway(...)

Что делает:

- меняет destination-side default routing branch на L2

Invariants:

- L2 default routing mutation должна быть counterpart-gated
- default routing mutation должна явно менять stored destination-side route

## 9. L1ArbitrumGateway.postUpgradeInit()

Что делает:

- post-upgrade hook для proxy-controlled L1 gateway

Invariants:

- post-upgrade hook должен вызываться только proxy admin

## 10. L1ArbitrumGateway._initialize(...)

Что делает:

- инициализирует L1 gateway
- фиксирует counterpart, router, inbox

Invariants:

- L1 gateway initialization не должен стартовать без valid router
- L1 gateway initialization не должен стартовать без valid inbox

## 11. L2ArbitrumGateway.postUpgradeInit()

Что делает:

- post-upgrade hook для proxy-controlled L2 gateway

Invariants:

- post-upgrade hook должен вызываться только proxy admin

## 12. L2ArbitrumGateway._initialize(...)

Что делает:

- инициализирует L2 gateway
- фиксирует L1 counterpart и router

Invariants:

- L2 gateway initialization не должен стартовать без valid router
