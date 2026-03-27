# Results Analysis

## 1. Which Comes First: `simulation` or `detection`?

They are **separate pipelines**.

### `simulation/` pipeline

This is the path-based offline opportunity search pipeline.

Order inside `simulation/`:

1. `path_builder.py`
   - builds the candidate arbitrage paths
2. `simulate_paths.py`
   - applies pool-state updates and computes profitable opportunities

So inside `simulation`, **`path_builder.py` comes first**.

### `detection/` pipeline

This is the live/on-chain arbitrage detection pipeline.

Each chain folder has a direct detector:

- `detection/arbitrum/arbitrum_single_chain_arbitrage.py`
- `detection/base/base_single_chain_arbitrage.py`
- `detection/optimism/optimism_single_chain_arbitrage.py`

This pipeline does **not** depend on the simulation result files.

### Practical conclusion

- `simulation` is for finding theoretical profitable paths from pool-state updates.
- `detection` is for detecting actual arbitrage transactions on-chain.
- Neither folder strictly comes before the other globally.
- Only inside `simulation`, `path_builder.py` comes before `simulate_paths.py`.

## 2. Result JSON Structure and PnL Fields

### Simulation result files

The simulation result files start with fields like:

- `timestamp_range_start`
- `timestamp_range_stop`
- `number_of_updated_pools`
- `number_of_updated_paths`
- `number_of_paths_with_positive_gains`
- `number_of_simulated_paths`
- `number_of_profitable_non_conflicting_paths`
- `profitable_non_conflicting_paths`
- `total_profit_usd`

So the main PnL field for simulation is:

- `total_profit_usd`

Inside each profitable path entry there is also:

- `profit_usd`
- `amount_usd`

### Detection result files

The detection result files start with fields like:

- `block_number`
- `block_timestamp`
- `transaction`
- `arbitrages`
- `token_balance`
- `eth_usd_price`
- `total_cost_usd`
- `total_gain_usd`
- `total_profit_usd`
- `transaction_cost_usd`

So the main realized PnL field for detection is also:

- `total_profit_usd`

This one is more realistic because it includes transaction cost fields.

## 3. Aggregate PnL Summary

The aggregated metrics below were computed from all exported JSON files in `results/`.

### Simulation: Cross-chain

- Files: 5
- Rows: 43,199
- Positive-PnL rows: 34,886
- Zero-PnL rows: 8,313
- Negative-PnL rows: 0
- Total PnL: `1,764,433.18 USD`
- Average per row: `40.84 USD`
- Max row PnL: `507.15 USD`

### Simulation: Single-chain

- Files: 13
- Rows: 122,401
- Positive-PnL rows: 13,716
- Zero-PnL rows: 108,685
- Negative-PnL rows: 0
- Total PnL: `890.98 USD`
- Average per row: `0.0073 USD`
- Max row PnL: `116.80 USD`

### Detection: Arbitrum

- Files: 1
- Rows: 5,409
- Rows with numeric PnL: 5,235
- Positive-PnL rows: 4,176
- Negative-PnL rows: 1,059
- Total PnL: `1,297.31 USD`
- Average per detected row: `0.248 USD`
- Max row PnL: `290.12 USD`
- Min row PnL: `-2.88 USD`

### Detection: Base

- Files: 3
- Rows: 22,860
- Rows with numeric PnL: 21,905
- Positive-PnL rows: 17,090
- Negative-PnL rows: 4,815
- Total PnL: `27,610.34 USD`
- Average per detected row: `1.26 USD`
- Max row PnL: `9,106.75 USD`
- Min row PnL: `-7.02 USD`

### Detection: Optimism

- Files: 1
- Rows: 7,890
- Rows with numeric PnL: 7,846
- Positive-PnL rows: 7,741
- Negative-PnL rows: 105
- Total PnL: `113.08 USD`
- Average per detected row: `0.014 USD`
- Max row PnL: `7.77 USD`
- Min row PnL: `-0.157 USD`

## 4. Main Findings

### Finding A: Cross-chain simulation is much more profitable than single-chain simulation

Cross-chain simulation produced:

- `1.76M USD` total simulated PnL

Single-chain simulation produced:

- only `890.98 USD`

This gap is extremely large.

Most likely explanation:

- the cross-chain simulation is identifying many theoretical price dislocations
- but it is **not modeling real bridge latency, bridge fees, settlement risk, or execution failure**

So the cross-chain simulation results should be treated as:

- **upper-bound theoretical opportunities**

not realistic executable profit.

### Finding B: Single-chain simulation is much sparser and weaker

Single-chain simulation has:

- 122,401 rows
- only 13,716 positive rows
- average row PnL of `0.0073 USD`

That means most single-chain windows are not materially profitable.

Even among positive single-chain rows, average positive profit is only about:

- `0.065 USD`

So the single-chain simulator is finding mostly tiny opportunities, with only a few larger spikes.

### Finding C: Detection results are much closer to realized trading conditions

Unlike simulation, detection includes:

- transaction cost
- total gain
- total profit

That is why detection has both:

- positive PnL rows
- negative PnL rows

This makes detection more realistic for actual arbitrage execution analysis.

### Finding D: Base detection is the strongest realized dataset

Among the detection folders, `base` stands out:

- highest total detected profit: `27,610.34 USD`
- highest average per detected row: `1.26 USD`
- largest max row profit: `9,106.75 USD`

This suggests that, in the exported detection data:

- Base had the most meaningful realized arbitrage opportunities

### Finding E: Optimism detection is the weakest

Optimism has:

- only `113.08 USD` total detected profit
- very small average profit per detected row

So from the exported results, Optimism appears to have the least attractive realized single-chain arbitrage set.

### Finding F: Arbitrum has meaningful activity but weaker realized average than Base

Arbitrum detection has:

- `1,297.31 USD` total detected profit
- max row profit of `290.12 USD`

So it does show real profitable opportunities, but the overall magnitude is much lower than Base in this dataset.

## 5. Top PnL Examples

The top examples were exported to:

- [top_pnl_examples.json](/c:/Users/Fabian/Desktop/598project/DeFi-Cross-Rollup-Arbitrage/Analysis/top_pnl_examples.json)

### Top simulation cross-chain examples

Top simulated cross-chain windows are all around:

- `506 to 507 USD`

These are clustered around timestamps:

- `1730481168`
- `1730484712` to `1730484950`

This clustering suggests repeated nearby windows with the same or very similar cross-chain pricing imbalance.

### Top simulation single-chain example

Best single-chain simulated row:

- `116.80 USD`
- chain: `arbitrum`
- block: `269990092`

### Top detected Arbitrum example

- `290.12 USD`
- block: `269778683`
- tx hash: `95acb4209325a63a165376e1536726785051a7904bff24ad6c027d7e4fbd514e`

### Top detected Base example

The max detected Base PnL is:

- `9,106.75 USD`

This is a very large outlier relative to the rest of the dataset and is worth manual inspection.

### Top detected Optimism example

- `7.77 USD`

This is much smaller than the best Base and Arbitrum detected trades.

## 6. Interpretation

### Best dataset for realistic profit analysis

Use:

- `results/detection/...`

because those records already include:

- transaction cost
- total gain
- total profit

### Best dataset for strategy idea generation

Use:

- `results/simulation/cross_chain_profitable_paths/...`

because it surfaces many theoretical path opportunities.

But you should not interpret those profits as executable without adding:

- bridge costs
- latency
- slippage during transfer delay
- execution sequencing risk

### Best chain from current results

For realized detected PnL:

- `Base` looks strongest

For simulated path richness:

- `cross-chain` dominates all other categories

## 7. Files Created In `Analysis`

- [results_summary.json](/c:/Users/Fabian/Desktop/598project/DeFi-Cross-Rollup-Arbitrage/Analysis/results_summary.json)
- [top_pnl_examples.json](/c:/Users/Fabian/Desktop/598project/DeFi-Cross-Rollup-Arbitrage/Analysis/top_pnl_examples.json)
- [results_analysis.md](/c:/Users/Fabian/Desktop/598project/DeFi-Cross-Rollup-Arbitrage/Analysis/results_analysis.md)

## 8. Bottom Line

The strongest conclusion from the exported results is:

- **cross-chain simulation finds many large theoretical opportunities**
- **Base detection has the strongest realized single-chain profit**
- **single-chain simulation is much weaker than cross-chain simulation**
- **detection results are the better source for realistic PnL analysis**
