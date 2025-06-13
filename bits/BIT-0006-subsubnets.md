# BIT-0006: Sub-subnets

- **BIT Number:** 0006
- **Title:** Sub-subnets
- **Author(s):** Rhef, Greg Zaitsev
- **Discussions-to:** [URL for discussion thread]
- **Status:** Draft
- **Type:** Core
- **Created:** 2025-06-13
- **Updated:** 2025-06-13

## 🔍 Abstract

This BIT proposes the introduction of subsubnets, a hierarchical layer within each subnet to support multiple, independent weight spaces, emissions, and incentive flows under a single subnet umbrella. Subsubnets enable fine-grained control over miner task allocation, incentive distribution, and validator decision-making. Each subnet can define up to 8 subsubnets, with weights and rewards tracked independently per subsubnet.

## 🔧 Motivation

Subnets today treat miner weights as a single flat structure, limiting expressivity in multi-task or multi-objective networks. Introducing subsubnets allows a single subnet to support multiple distinct task markets, enabling specialized incentive tracking, validator scoring, and emission logic per group. This facilitates complex workloads, parallel task handling, and flexible protocol design without fragmenting network security or duplicating validator logic.

## 🧪 Specification

### Subsubnet Limits
- Each subnet defines a `desired_subsubnet_limit` hyperparameter (default = 1).
- A global limit `global_subsubnet_limit_per_subnet` acts as a ceiling.
- The active value `subsubnet_limit_in_force` is updated every subnet superblock (every 20 tempos) as:

```
subsubnet_limit_in_force = min(
    desired_subsubnet_limit,
    global_subsubnet_limit_per_subnet,
    subsubnet_limit_in_force_last + global_subsubnet_decrease_per_subnet_superblock
)
```

### Weight Operations
- `set_weights`, `commit_weights`, and `reveal_weights` accept a `subsubnet_id` argument.
- Legacy operations default to `subsubnet_id = 0`.
- Weight writes are disallowed above the current `subsubnet_limit_in_force`.

### Validator Permissions
- Only the subnet owner or sudo can change the `desired_subsubnet_limit` or emission proportions.
- Validators may set weights for any subsubnet within the current limit.

### Emission Logic
- Emission is distributed across subsubnets according to a configured ratio (default Fibonacci: [1, 2, 3, 5, 8, 13, 21, 34]).
- Each subsubnet computes trust, consensus, and incentive separately.
- Final vtrust is aggregated across subsubnets weighted by their emissions.
- Rounding is preserved across subsubnet splits to ensure exact conservation of emitted tokens.

### Edge Cases
- If a subsubnet has zero consensus, it enters “Yuma emergency mode” and allocates emission proportional to stake.
- Miners without weights in a subsubnet receive no emission from it.
- Subsubnets with no miners are gracefully handled.

### Compatibility
- Subnets with a single subsubnet behave exactly as today (ID 0).
- Legacy miners/validators interoperate with subsubnet-enabled subnets via `subsubnet_id = 0`.
- Storage and RPC interfaces remain backward-compatible.

## ✅ Rationale

Subsubnets allow a single subnet to support multiple incentive partitions, enabling more advanced use cases (e.g. routing, filtering, classification, multitask models). They preserve validator overhead by avoiding new subnets while enabling greater expressivity. The design enforces strict backward compatibility and safe transitions when limits change.

## 📘 Reference Implementation

- Will be implemented in the `subtensor` core repo.
- Interfaces for weight setting, emission, and validator ranking will be extended to include `subsubnet_id`.

## 🧱 Backward Compatibility

- All existing weight operations apply to `subsubnet_id = 0`.
- Subnets not opting into subsubnets will remain functionally identical.
- Miners and validators on older versions will continue functioning under subsubnet_id 0.

## 📈 Test Cases

See BIT test document `subsubnet_test_plan.bit`.

## 💬 Discussion

- Emission proportion customization per subsubnet opens design space for subnet-specific task prioritization.
- Validator voting on miner subsubnet weights (via kappa) may be used for consensus and reward routing.
- Handling of dynamic emission distribution and cleanups when limits decrease must be conservative and race-free.

## 🛠️ Future Work

- **Custom Emission Proportions**: Subnet owners will be able to customize the proportion of emission allocated to each subsubnet, enabling tailored incentive strategies based on task complexity or utility.

- **Dynamic Global Subsubnet Limit**: A globally enforced ceiling on subsubnet counts will be adjustable over time. Reductions to this limit will automatically trigger cleanup of excess subsubnet data across all subnets.

- **Hyperparameter Governance**: Subnet owners will gain control over additional subsubnet-specific hyperparameters beyond the subsubnet limit, allowing more granular tuning of behavior.

- **Validator-Driven Incentive Routing**: Using the kappa stake majority mechanism, validators may vote to adjust miner incentive shares within subsubnets, supporting flexible prioritization of behaviors and tasks.

- **Additional Governance Extensions**: Future extensions may include subsubnet-specific pruning policies, trust calculation curves, or dynamic validator selection strategies.


## 🔐 Security Considerations

The introduction of subsubnets introduces additional state surfaces and per-subsubnet tracking, which must be secured against manipulation:

- **Permission Enforcement**: Only subnet owners or sudo must be able to modify `desired_subsubnet_limit`, emission proportions, or trigger subsubnet resets. Improper permission checks could allow hostile takeovers of reward logic.
  
- **Weight Isolation**: Weights across subsubnets must remain isolated. Cross-contamination could allow miners to gain rewards in unintended subsubnets.
  
- **Rounding and Overflow**: Emission rounding and aggregation must be implemented carefully to prevent underflow/overflow or token inflation.
  
- **Backward Compatibility**: Any logic paths introduced for subsubnet IDs must default to safe values (e.g., subsubnet_id = 0) to avoid denial-of-service for legacy miners and validators.

- **Cleanups**: When limits are decreased, weight purging must be idempotent and bounded to prevent validator or miner state desynchronization.

## © Copyright

This document is licensed under [The Unlicense](https://unlicense.org/).
