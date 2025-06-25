# Chapter 5: Branch Network Optimization – Full Example

This chapter provides a step-by-step example of using the `branch_calculation` package to analyze and optimize a branch water network. You will learn how to:
- Load a network from a CSV file
- Analyze the network hydraulically
- Perform both full section and classic optimization for pipe diameters

## Setup

1. Download the required data files from [BNHA data_templates](https://github.com/Relkayam/BNHA/tree/main/data_templates):
   - `network_tree_template.csv`
   - `pipe_prices.xlsx`
2. Place these files in a directory named `data_templates` at the same level as this file.
3. Edit `network_tree_template.csv` to define your network (branches, diameters, lengths, elevations, etc.).
4. Edit `pipe_prices.xlsx` to match your pipe cost data.
5. Install the required package:
   ```bash
   pip install branch_calculation
   ```

---

## Load Network Data

```python
import pandas as pd
import os
from branch_calculation.network import BranchNetwork

data_dir = os.path.join(os.getcwd(), "data_templates")
df = pd.read_csv(os.path.join(data_dir, "network_tree_template.csv"))
pipe_prices = pd.read_excel(os.path.join(data_dir, "pipe_prices.xlsx"))

net = BranchNetwork()
net.set_system_data(reservoir_elevation=300, reservoir_total_head=300)
net.load_from_dataframe(df)
```

---

## Analyze the Network

```python
from branch_calculation.analysis import analyze_network
from branch_calculation.plots import plot_branches

results = analyze_network(net)
df_results = results['df_res']
print(df_results)

# Print summary
for k, v in results['summary'].items():
    print(f"{k}: {v}")

print('Here are the available keys in results through "analyze_network" function:')
print(results.keys())

for key, value in results.items():
    if isinstance(value, pd.DataFrame):
        print(f"{key}:\n{value}\n")
    else:
        print(f"{key}: {value}")

plots = plot_branches(results['results_branch'], minimum_pressure_constraint=2)
```

*Review the resulting DataFrame to see pressures, velocities, and other hydraulic properties for each branch.*

---

## Full Section Optimization

```python
from branch_calculation.optimizer import full_section_optimal_diameter
from branch_calculation.plots import plot_branches

print('=== Optimization for full section and classic methods  ===')
# Perform optimization for full section and classic methods
results = full_section_optimal_diameter(net, pipe_prices, minimum_pressure_constraint=2)
print('Here are the available keys in results through "full_section_optimal_diameter" function:')
print(results.keys())

for key, value in results.items():
    if isinstance(value, pd.DataFrame):
        print(f"{key}:\n{value}\n")
    else:
        print(f"{key}: {value}")

plot_branches(results['results_branch'], minimum_pressure_constraint=2)
```

*This step finds the optimal diameters for each branch using the full section method.*

---

## Classic Optimization

```python
from branch_calculation.optimizer import classic_optimal_diameter_optimization
from branch_calculation.plots import plot_branches

print('=== Classic Optimization for optimal diameter  ===')
# perform classic optimization
results = classic_optimal_diameter_optimization(net, pipe_prices, minimum_pressure_constraint = 2)
print('The available keys in results are:')
print(results.keys())
plot_branches(results['results_branch'], minimum_pressure_constraint=2)

print('Here are the available keys in results through "classic_optimal_diameter_optimization" function:')

for key, value in results.items():
    if isinstance(value, pd.DataFrame):
        print(f"{key}:\n{value}\n")
    else:
        print(f"{key}: {value}")
```

*This step finds the optimal diameters using the classic method.*

