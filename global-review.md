# Global Review

Этот review фиксирует system-level guarantees, которые объединяют deposit, withdraw и surrounding config surface в одну bridge security model.

## Global Invariants

- destination-side asset credit/release не должен происходить без source-side accounting
- asset movement должен происходить только через intended bridge lifecycle paths
- L1/L2 token correspondence должна оставаться согласованной на уровне whole bridge lifecycle
- disabled hook / extra-data branches не должны silently менять normal bridge semantics
- router-level routing model не должна расходиться с gateway-level handling model
- counterpart-gated finalization должна оставаться обязательной для destination-side asset movement
- recovery / fallback branches не должны конфликтовать с normal branch
- privileged / config / init / upgrade surface не должна silently ломать bridge guarantees
