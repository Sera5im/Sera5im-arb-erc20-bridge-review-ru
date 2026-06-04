# Global Review

Этот review фиксирует system-level guarantees, которые связывают deposit, withdraw и surrounding config surface в одну bridge security model.

## Global Invariants

- destination-side asset credit или release не должен происходить без source-side accounting
- asset movement должен идти только через intended bridge lifecycle paths
- L1/L2 token correspondence должна оставаться согласованной по всему bridge lifecycle
- disabled hook и extra-data branches не должны тихо менять normal bridge semantics
- router-level routing model не должна расходиться с gateway-level handling model
- counterpart-gated finalization должна оставаться обязательной для destination-side asset movement
- recovery и fallback branches не должны конфликтовать с normal branch
- privileged, config, init и upgrade surface не должны тихо ломать bridge guarantees
