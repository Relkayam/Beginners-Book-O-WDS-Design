# Chapter 3: Branch Network Hydraulic Analysis

---

## Learning Objectives
By the end of this chapter, you will be able to:

- **Apply** the Hazen-Williams equation to calculate head losses in pipe networks.
- **Analyze** hydraulic and energy grade lines in branched systems.
- **Evaluate** the impact of pipe diameter selection on system performance.
- **Use** computational tools to inform pipe sizing decisions.
- **Interpret** hydraulic analysis results for design and optimization.

---

## 1. Introduction to Branched Networks

Water distribution networks are complex systems requiring careful hydraulic analysis to ensure reliable service. This chapter focuses on **branched networks**, which feature a tree-like structure where there is only one path from the source to each demand point. These are common in rural and suburban areas.

The goal of this chapter is to build a computational tool for evaluating how different pipe diameters affect system hydraulics. This analysis is fundamental to understanding the trade-offs between pipe costs and hydraulic performance, paving the way for network optimization.

---

## 2. Theoretical Background

### 2.1 Hazen-Williams Equation

The Hazen-Williams equation is a widely-used empirical formula for calculating head loss in water pipes. Its simplicity and reasonable accuracy make it a standard tool in water distribution system analysis.

   $$
h_f = \frac{10.67\,L\,Q^{1.852}}{C^{1.852}\,d^{4.8704}}
$$

where:
- $h_f$ = head loss due to friction (m)
- $L$ = pipe length (m)
- $Q$ = flow rate (m³/s)
- $C$ = Hazen-Williams roughness coefficient (dimensionless)
- $d$ = internal pipe diameter (m)

### 2.2 Hazen-Williams Roughness Coefficients ($C$)

The roughness coefficient depends on the pipe's material and age. Higher values indicate smoother pipes.

| Pipe Material | New Pipe | 10+ Years of Service |
|---------------|----------|----------------------|
| Ductile Iron  | 130–140  | 100–130              |
| Steel         | 120–130  | 80–120               |
| Concrete      | 120–130  | 85–120               |
| PVC           | 140–150  | 130–140              |
| HDPE          | 140–150  | 130–140              |

### 2.3 Energy and Hydraulic Grade Lines (EGL & HGL)

Understanding the energy in a pipe system is crucial for analysis:

- **Energy Grade Line (EGL)** represents the total energy head:


  $$ H_{EGL} = z + \frac{p}{\gamma} + \frac{v^2}{2g} $$


- **Hydraulic Grade Line (HGL)** represents the piezometric head:

  $$ H_{HGL} = z + \frac{p}{\gamma} $$

- **Pressure Head** is the difference between the HGL and the pipe elevation:

  $$ \frac{p}{\gamma} = H_{HGL} - z $$

where $z$ is elevation, $p$ is pressure, ${\gamma}$ is the specific weight of water, $v$ is velocity, and $g$ is gravity. The HGL is what determines the pressure in the pipes.

---

## 3. Methodology for Analysis

### 3.1 Network Representation

A branched network can be efficiently represented using a dictionary or a similar data structure. Each pipe segment (or branch) should store its key properties:
- **Geometric**: Length, diameter, start/end elevations.
- **Hydraulic**: Flow rate, roughness coefficient.
- **Calculated**: Head loss, velocity.

### 3.2 Solution Algorithm

The hydraulic analysis proceeds sequentially from the upstream source to the downstream nodes:

1.  **Initialize Network Geometry**: Define pipe lengths, node elevations, and connectivity.
2.  **Set Design Parameters**: Assign flow rates and select initial pipe diameters for analysis.
3.  **Calculate Head Losses**: Apply the Hazen-Williams equation to each pipe segment.
4.  **Compute Hydraulic Grade Line (HGL)**: Starting from the source with a known head, subtract the calculated head loss for each pipe segment to find the head at the next downstream node.
5.  **Determine Nodal Pressures**: Calculate the pressure at each node by subtracting the node's elevation from its HGL value.

---

## 4. Worked Example

I will provide a worked example in the next section to demonstrate the methodology in practice.

---

## 5. Summary

The methods presented here form the foundation for more advanced network analysis techniques, including looped systems, dynamic analysis, and optimization algorithms. You should practice with various network configurations to develop intuition about hydraulic behavior and design trade-offs.

---

## 9. Exercises

# TODO: add specific data, instructions and results for exercises

### Exercise 1: Basic Analysis
Given a three-pipe branched system, calculate the hydraulic grade line and identify the minimum pressure head location.

### Exercise 2: Diameter Optimization
For a specified minimum pressure requirement, determine the smallest pipe diameters that satisfy the constraint.

### Exercise 3: Sensitivity Study
Analyze how a 20% increase in flow rate affects system performance for different diameter combinations.

### Exercise 4: Design Comparison
Compare the hydraulic performance and estimated costs of three different pipe sizing strategies.

---

