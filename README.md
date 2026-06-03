# Arbitrum ERC20 Bridge Review

Этот репозиторий содержит мои review notes по ERC20 bridge paths в Arbitrum.

## Что Покрывает Этот Review

Arbitrum переносит ERC20 asset между L1 и L2 через router/gateway model:

- на deposit ERC20 токены escrow'ятся на L1 и credit'ятся на L2
- на withdraw L2 token representation burn'ится, а escrowed L1 asset release'ится обратно

Этот review сфокусирован именно на ERC20 deposit и withdrawal paths на bridge contract / application layer.

## Scope

Мой текущий scope здесь:

- L1 router и L1 gateway deposit logic
- L2 gateway deposit finalization logic
- L2 gateway withdrawal initiation logic
- L1 gateway withdrawal finalization logic
- surrounding config, routing, init и upgrade surface

Мой текущий scope здесь сфокусирован на bridge application layer, а не на более глубоком transport-layer разборе Nitro internals. Поэтому такие части, как Inbox, Outbox, Bridge и другие message-delivery / execution internals, здесь рассматриваются только как transport boundary между reviewed L1 и L2 contract-layer paths.

## Структура Review

- [deposit-review.md](deposit-review.md)
  Review ERC20 deposit path: от L1 routing и escrow до L2 final credit.

- [withdraw-review.md](withdraw-review.md)
  Review ERC20 withdrawal path: от L2 burn и message creation до L1 final release.

- [out-of-flow-review.md](out-of-flow-review.md)
  Review surrounding config, routing mutation, init и upgrade surface.

- [global-review.md](global-review.md)
  System-level guarantees, которые связывают deposit, withdrawal и surrounding admin/config layer.
