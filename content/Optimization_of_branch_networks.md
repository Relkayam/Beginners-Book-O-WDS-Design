# Designing Branched Water Networks with `BNHA`

## Introduction

This chapter focuses on the design and analysis of branched water distribution systems, which form tree-like structures without loops, as commonly found in rural water supply, irrigation, or preliminary urban planning. We introduce `BNHA` (Branched Network Hydraulic Analysis), a Python-based tool developed to streamline the optimization and hydraulic analysis of these systems. `BNHA` automates pipe sizing and performance calculations, making it a practical resource for engineers working on cost-effective and hydraulically sound designs. This chapter covers the tool’s functionality, underlying mathematics, and practical applications, complemented by a Jupyter Notebook for hands-on implementation.

## Overview of the `BNHA` Package

The `BNHA` package is an open-source Python tool designed for the efficient design and analysis of branched water distribution networks. It supports engineers in optimizing pipe diameters and performing hydraulic calculations, reducing reliance on manual methods. Key features include:

- **Pipe Diameter Optimization:** Selects cost-effective pipe sizes while meeting hydraulic constraints like minimum pressure.
- **Hydraulic Analysis:** Computes head loss, velocity, and pressure across the network.
- **Diagnostics and Verification:** Exports optimization models and constraint details for troubleshooting and validation.

Built on Python libraries such as `pandas` for data management, `NumPy` for numerical operations, and `PuLP` for optimization, `BNHA` is accessible and robust. Source code and documentation are available at [https://github.com/Relkayam/BNHA](https://github.com/Relkayam/BNHA).

## Notation

The following notation is used in `BNHA`’s mathematical models and hydraulic calculations:

- $ \mathcal{P} $: Set of all pipes in the network.
- $ \mathcal{B} $: Set of terminal branch paths (from source to end nodes).
- $ \mathcal{D}_i $: Set of available diameters for pipe $ i \in \mathcal{P} $.
- $ \mathcal{S}_i $: Set of sub-segments for pipe $ i $ in the continuous model.
- $ dz_p $: Elevation drop along path $ p \in \mathcal{B} $.
- $ h_{\min} $: Minimum required pressure head at terminal nodes.
- $ \varepsilon $: Pressure tolerance for numerical feasibility.
- $ L_i $: Length of pipe $ i $.
- $ c_{i,d} $: Cost of using diameter $ d $ for pipe $ i $.
- $ h_{i,d} $: Head loss for pipe $ i $ with diameter $ d $.
- $ x_{i,d} $: Binary variable; 1 if diameter $ d $ is chosen for pipe $ i $, 0 otherwise.
- $ l_j $: Length of sub-segment $ j $ in the continuous model.
- $ c_j $: Cost per meter for sub-segment $ j $.
- $ h'_j $: Head loss per meter for sub-segment $ j $.

Refer to this notation when working through the optimization models or the Jupyter Notebook.

## Optimization Methodologies

`BNHA` provides two optimization approaches for pipe sizing: Discrete Diameter Optimization and Continuous-Length Optimization. These methods address different design needs and are detailed below.

### Discrete Diameter Optimization (MIP)

This method selects a single diameter per pipe from a predefined catalog, optimizing for cost while ensuring hydraulic feasibility.

**Objective:** Minimize total cost:
$$
\min \sum_{i \in \mathcal{P}} \sum_{d \in \mathcal{D}_i} x_{i,d} \cdot c_{i,d}
$$

**Constraints:**
1. **Single Diameter Selection:** Each pipe is assigned one diameter:
   $$
   \sum_{d \in \mathcal{D}_i} x_{i,d} = 1 \quad \forall i \in \mathcal{P}
   $$
2. **Pressure Constraint:** Head loss along each path must not exceed the available elevation drop minus minimum pressure:
   $$
   \sum_{i \in p} \sum_{d \in \mathcal{D}_i} x_{i,d} \cdot h_{i,d} \leq dz_p - h_{\min}
   $$

This Mixed-Integer Programming (MIP) problem is solved using `PuLP`.

### Continuous-Length Optimization (LP)

This approach allows pipes to be divided into segments with varying diameters, enabling designs like tapered pipelines for enhanced efficiency.

**Objective:** Minimize total cost:
$$
\min \sum_{j} l_j \cdot c_j
$$

**Constraints:**
1. **Length Preservation:** Segment lengths sum to the pipe’s total length:
   $$
   \sum_{j \in \mathcal{S}_i} l_j = L_i \quad \forall i \in \mathcal{P}
   $$
2. **Pressure Constraint:** Head loss along each path respects elevation and pressure limits:
   $$
   \sum_{j \in p} l_j \cdot h'_j \leq dz_p - h_{\min} - \varepsilon
   $$

This Linear Programming (LP) problem is computationally efficient and suitable for complex designs.

### Hydraulic Calculations

Both optimization methods use the Hazen-Williams equation for head loss:
$$
h = 10.67 \cdot \left( \frac{L \cdot Q^{1.852}}{C^{1.852} \cdot D^{4.871}} \right)
$$
Where:
- $ L $: Pipe length (m)
- $ Q $: Flow rate (m³/s)
- $ C $: Roughness coefficient
- $ D $: Diameter (m)

Additional calculations include:
- **Velocity:** $ v = \dfrac{4Q}{\pi D^2} $
- **Pressure Head:** $ H_{\text{pressure}} = H_{\text{source}} - \sum h_i - z $

Outputs are provided as structured tables (`df_res`, `results_branch`, `pipes_summery`) with debugging tools for constraint analysis.

## Practical Usage Examples

`BNHA` supports a range of practical applications, demonstrated through the following examples. These will be implemented in the accompanying Jupyter Notebook.

1. **Hydraulic Analysis of Existing Networks:**
   For an existing branched network, such as a rural water system, `BNHA` calculates head loss, velocity, and pressure. Input pipe lengths, diameters, and flows to generate a table of pressures at terminal nodes, verifying compliance with design standards or identifying issues.

2. **Pipe Diameter Optimization for New Designs:**
   When designing a new system, like an irrigation network, use the Discrete Diameter Optimization model. Provide elevation data, flow requirements, and a diameter catalog, and `BNHA` will select cost-optimal pipe sizes that maintain required pressures.

3. **Tapered Pipeline Design:**
   For long pipelines, such as those supplying remote areas, the Continuous-Length Optimization model designs tapered systems. By allowing multiple diameters per pipe, `BNHA` optimizes cost and hydraulic performance, ideal for large-scale projects.

These examples leverage `pandas` for input and output management, ensuring clear and actionable results. The Jupyter Notebook provides sample code and data to explore these scenarios.

## Next Steps

This chapter equips you with the tools to design and analyze branched water networks using `BNHA`. The accompanying Jupyter Notebook offers practical implementations of the examples above, guiding you through setup, optimization, and result interpretation. Explore different network configurations and parameters to deepen your understanding. `BNHA` is a versatile tool for both engineering practice and project development.

**Source code and documentation:** [https://github.com/Relkayam/BNHA](https://github.com/Relkayam/BNHA)
