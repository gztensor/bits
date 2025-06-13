# Subsubnet Functionality BIT

## Test Areas

### ✅ Weight Cleanup
- [ ] Weights are cleaned when `subsubnet_limit_in_force` decreases
- [ ] For each subsubnet, when a miner is deregistered or leaves, their weights are cleaned across **all** subsubnets

### ✅ Limit Update Logic
- [ ] Decreasing `desired_subsubnet_limit` reduces `subsubnet_limit_in_force` by no more than `global_subsubnet_decrease_per_subnet_superblock`
- [ ] Validate update timing during the subnet superblock (every 20 tempos)
- [ ] Confirm fallback to `min(desired_subsubnet_limit, global_subsubnet_limit_per_subnet)`

### ✅ Validator Permissions & Hyperparameters
- [ ] Ensure only subnet owners (or sudo) can modify their subnet's subsubnet hyperparameters
- [ ] Max allowed `desired_subsubnet_limit` is 8 (ids 0-7)
- [ ] Validators **cannot** modify other subnets’ parameters
- [ ] `desired_subsubnet_limit` is readable by API per subnet and globally

### ✅ Compatibility with Existing Systems
- [ ] Confirm `subsubnet_id = 0` acts as default fallback for weight operations
- [ ] Validate `set_weights`, `commit_weights`, `reveal_weights` apply correctly on subsubnet_id=0
- [ ] Confirm legacy operations still work as expected when subsubnet feature is not used (regression)

### ✅ Weight Setting Restrictions
- [ ] Prevent weight setting above `subsubnet_limit_in_force`
- [ ] Prevent CR2/CR3 weight commits/reveals for disabled subsubnets
- [ ] Validators **can** set weights above hyperparameter but below `subsubnet_limit_in_force`

### ✅ Miner-Subsubnet Interaction
- [ ] Miner can participate in multiple or all subsubnets (bond, weights, rewards)
- [ ] Support miner existence with no weights on any subsubnet
- [ ] Support subsubnet existence with no miner weights at all
- [ ] Ensure correct weights can be retrieved per miner per subsubnet

### ✅ Emissions and Incentives
- [ ] Ensure `subsubnet_limit_in_force` does not exceed global limit
- [ ] Validate weight independence across subsubnets
- [ ] Check total emission is split among subsubnets based on pre-defined distribution (Fibonacci default):
  - id0 = 1
  - id1 = 2
  - id2 = 3
  - id3 = 5
  - id4 = 8
  - id5 = 13
  - id6 = 21
  - id7 = 34
- [ ] Validate that per-subsubnet incentives are distributed proportionally to miner weights
- [ ] Trust, vtrust, consensus, etc. are calculated per subsubnet and then aggregated
- [ ] Rounding logic does not lose incentive
- [ ] Sum of all subsubnet emissions matches total emission

### ✅ Emergency and Recycling Behavior
- [ ] Empty subsubnet enters "Yuma Emergency Mode", emission distributed by stake
- [ ] If consensus sum is 0 for a subsubnet, trigger emergency fallback
- [ ] Recycling incentives per subsubnet via validator vote by subnet owner ID
- [ ] Miner without any subsubnet weights receives **no** reward

### ✅ Subnet Compatibility Testing
- [ ] Subnets with `desired_subsubnet_limit = 1` (id = 0 only) operate without change
- [ ] Regression tests confirm staking, pruning, emissions, and validator selection work as before
- [ ] Switching subsubnets off restores old behavior without side effects
- [ ] Storage layout and RPC responses remain compatible for older miners/validators
- [ ] Validate upgrade paths: backward compatibility for older nodes (v2+)

### ✅ Governance and Parameter Controls
- [ ] Subnet owner can set incentive proportions per subsubnet
- [ ] Global subsubnet limit is dynamically adjustable
- [ ] Decreasing global limit propagates cleanups in all subnets
- [ ] Subnet owners can update subsubnet-specific hyperparameters
- [ ] Validator vote (via `kappa`) determines miner emission share per subsubnet

---

## References
- Implements part of: Subsubnet BIP (TBD)
- Affects: Miner incentives, emission mechanics, subnet behavior
