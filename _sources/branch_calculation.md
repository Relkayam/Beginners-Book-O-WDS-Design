# Branch Network Hydraulic Analysis

---

## Learning Objectives
By the end of this chapter, students will be able to:

* Analyze hydraulic grade lines and energy grade lines in branched systems
* Evaluate the impact of pipe diameter selection on system performance
* Use computational tools to optimize pipe sizing decisions
* Interpret hydraulic analysis results for design decision-making


## 1. Introduction
Water distribution networks are complex systems that require careful hydraulic analysis to ensure adequate service to all consumers. This chapter focuses on the hydraulic analysis of branched networks, which are common in suburban and rural water distribution systems. Unlike looped networks, branched systems have a tree-like structure with a single path from the source to each demand node.
The primary objective of this exercise is to develop a computational tool that allows engineers to evaluate different pipe diameter combinations and observe their effects on system hydraulics. This approach provides valuable insight into the trade-offs between pipe costs and hydraulic performance, forming the foundation for optimal network design.





# Branch Network Hydraulic Analysis

## Learning Objectives

By the end of this chapter, students will be able to:
- Apply the Hazen-Williams equation to calculate head losses in pipe networks
- Analyze hydraulic grade lines and energy grade lines in branched systems
- Evaluate the impact of pipe diameter selection on system performance
- Use computational tools to optimize pipe sizing decisions
- Interpret hydraulic analysis results for design decision-making

---

## 2. Theoretical Background

### 2.1 Hazen-Williams Equation

The Hazen-Williams equation is widely used in water distribution system analysis due to its simplicity and reasonable accuracy for water flow in pipes. The equation relates head loss to flow rate, pipe characteristics, and fluid properties:

$$h_f = \frac{10.67 \cdot L \cdot Q^{1.852}}{C^{1.852} \cdot d^{4.8704}}$$

where:
- $h_f$ = head loss due to friction (m)
- $L$ = pipe length (m)
- $Q$ = flow rate (m³/s)
- $C$ = Hazen-Williams roughness coefficient (dimensionless)
- $d$ = pipe internal diameter (m)

### 2.2 Hazen-Williams Roughness Coefficients

The roughness coefficient $C$ depends on pipe material and condition:

| Pipe Material | New Pipe | 10+ Years Service |
|---------------|----------|-------------------|
| Ductile Iron | 130-140 | 100-130 |
| Steel | 120-130 | 80-120 |
| Concrete | 120-130 | 85-120 |
| PVC | 140-150 | 130-140 |
| HDPE | 140-150 | 130-140 |

### 2.3 Energy and Hydraulic Grade Lines

Understanding energy relationships in pipe flow is crucial for network analysis:

- **Total Head (Energy Grade Line)**: $H = z + \frac{p}{\gamma} + \frac{v^2}{2g}$
- **Piezometric Head (Hydraulic Grade Line)**: $H_{HGL} = z + \frac{p}{\gamma}$
- **Pressure Head**: $\frac{p}{\gamma} = H_{HGL} - z$

where $z$ is elevation, $p$ is pressure, $\gamma$ is specific weight of water, $v$ is velocity, and $g$ is gravitational acceleration.

---

## 3. Methodology

### 3.1 Network Representation

The branched network is represented using a dictionary-based data structure where each pipe segment contains:
- Geometric properties (length, diameter)
- Hydraulic properties (flow rate, roughness coefficient)
- Calculated results (head loss, velocity)

### 3.2 Solution Algorithm

The analysis follows these sequential steps:

1. **Initialize Network Geometry**: Define pipe lengths, elevations, and connectivity
2. **Specify Design Parameters**: Set flow rates and select pipe diameters
3. **Calculate Head Losses**: Apply Hazen-Williams equation to each pipe segment
4. **Compute Hydraulic Grade Line**: Starting from the source, subtract head losses moving downstream
5. **Determine Pressure Heads**: Calculate pressure at each node using energy equation
6. **Evaluate Design Adequacy**: Check minimum pressure requirements and system constraints

### 3.3 Design Considerations

When selecting pipe diameters, engineers must consider:
- **Minimum Pressure Requirements**: Typically 20-35 m (200-350 kPa) at demand nodes
- **Maximum Velocity Limits**: Generally 1.5-3 m/s to minimize head losses and water hammer effects
- **Economic Optimization**: Balance between pipe costs and pumping costs
- **Fire Flow Requirements**: Ensure adequate capacity for emergency demands

---

## 4. Implementation Guide

### 4.1 Required Libraries

```python
import pandas as pd
import numpy as np
import plotly.graph_objects as go
from plotly.subplots import make_subplots
```

### 4.2 Head Loss Function

```python
def calculate_head_loss(L, Q, C, d):
    """
    Calculate head loss using Hazen-Williams equation
    
    Parameters:
    L (float): Pipe length (m)
    Q (float): Flow rate (m³/s)
    C (float): Hazen-Williams coefficient
    d (float): Pipe diameter (m)
    
    Returns:
    float: Head loss (m)
    """
    
    return (10.67 * L * Q**1.852) / (C**1.852 * d**4.8704)
```

### 4.3 Velocity Calculation

```python
def calculate_velocity(Q, d):
    """
    Calculate flow velocity in pipe
    
    Parameters:
    Q (float): Flow rate (m³/s)
    d (float): Pipe diameter (m)
    
    Returns:
    float: Velocity (m/s)
    """
    if d <= 0:
        return 0
    
    area = np.pi * (d/2)**2
    return Q / area
```

### 4.4 Network Analysis Function

```python
def analyze_network(pipes_dict, reservoir_head):
    """
    Perform complete hydraulic analysis of branched network
    
    Parameters:
    pipes_dict (dict): Dictionary containing pipe data
    reservoir_head (float): Total head at reservoir (m)
    
    Returns:
    pandas.DataFrame: Analysis results
    """
    # Calculate head losses
    for pipe_id, pipe_data in pipes_dict.items():
        pipe_data['head_loss'] = calculate_head_loss(
            pipe_data['L'], pipe_data['Q'], 
            pipe_data['C'], pipe_data['d']
        )
        pipe_data['velocity'] = calculate_velocity(
            pipe_data['Q'], pipe_data['d']
        )
    
    # Calculate hydraulic grade line
    results = []
    cumulative_distance = 0
    current_total_head = reservoir_head
    
    for pipe_id in sorted(pipes_dict.keys()):
        pipe = pipes_dict[pipe_id]
        
        # Node at start of pipe
        results.append({
            'Node': f'Node_{pipe_id}_start',
            'Distance': cumulative_distance,
            'Elevation': pipe['z_start'],
            'Total_Head': current_total_head,
            'Pressure_Head': current_total_head - pipe['z_start'],
            'Velocity': pipe['velocity'],
            'Pipe_ID': pipe_id
        })
        
        # Update for end of pipe
        cumulative_distance += pipe['L']
        current_total_head -= pipe['head_loss']
        
        results.append({
            'Node': f'Node_{pipe_id}_end',
            'Distance': cumulative_distance,
            'Elevation': pipe['z_end'],
            'Total_Head': current_total_head,
            'Pressure_Head': current_total_head - pipe['z_end'],
            'Velocity': pipe['velocity'],
            'Pipe_ID': pipe_id
        })
    
    return pd.DataFrame(results)
```

---

## 5. Results Interpretation

### 5.1 Hydraulic Performance Metrics

Key performance indicators include:
- **Minimum Pressure Head**: Must exceed service pressure requirements
- **Maximum Velocity**: Should remain within acceptable limits
- **Total Head Loss**: Affects pumping energy requirements
- **Pressure Variation**: Indicates system stability

### 5.2 Visualization and Analysis

The analysis generates interactive plots showing:
- **Hydraulic Grade Line**: Shows available pressure throughout the system
- **Energy Grade Line**: Illustrates total energy dissipation
- **Elevation Profile**: Provides context for pressure analysis

### 5.3 Design Optimization

Use the tool to:
1. **Identify Bottlenecks**: Locate nodes with inadequate pressure
2. **Evaluate Alternatives**: Compare different diameter combinations
3. **Optimize Costs**: Balance pipe costs against energy costs
4. **Verify Constraints**: Ensure all design criteria are satisfied

---

## 6. Practical Applications

### 6.1 Sensitivity Analysis

Systematically vary pipe diameters to understand:
- Impact on minimum system pressure
- Changes in total head loss
- Velocity distribution effects
- Economic implications

### 6.2 Design Scenarios

Consider multiple operating conditions:
- **Average Day Demand**: Typical consumption patterns
- **Peak Hour Demand**: Maximum system stress
- **Fire Flow Conditions**: Emergency capacity requirements
- **Future Growth**: Projected demand increases

### 6.3 Regulatory Compliance

Ensure designs meet:
- Local building codes and standards
- Water utility design criteria
- Environmental regulations
- Safety requirements

---

## 7. Limitations and Assumptions

### 7.1 Model Assumptions

- Steady-state flow conditions
- Uniform pipe roughness
- Incompressible flow
- Negligible minor losses at fittings

### 7.2 Hazen-Williams Limitations

- Applicable primarily to water at normal temperatures
- Accuracy decreases for very high or low velocities
- Does not account for pipe aging effects explicitly
- May not be suitable for all pipe materials

### 7.3 Recommended Validation

- Compare results with other hydraulic models
- Verify against field measurements when available
- Consider uncertainty in input parameters
- Perform sensitivity analysis on critical assumptions

---

## 8. Summary and Key Takeaways

This chapter presented a systematic approach to analyzing branched water distribution networks using the Hazen-Williams equation. Key learning points include:

1. **Hydraulic Fundamentals**: Understanding energy relationships and head loss mechanisms
2. **Computational Methods**: Implementing systematic analysis procedures
3. **Design Evaluation**: Using tools to assess and optimize system performance
4. **Engineering Judgment**: Balancing multiple criteria in design decisions

The methods presented here form the foundation for more advanced network analysis techniques, including looped systems, dynamic analysis, and optimization algorithms. Students should practice with various network configurations to develop intuition about hydraulic behavior and design trade-offs.

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

## References

1. Walski, T.M., et al. (2003). *Advanced Water Distribution Modeling and Management*. Haestad Press.
2. Rossman, L.A. (2000). *EPANET 2 Users Manual*. EPA/600/R-00/057.
3. Mays, L.W. (2000). *Water Distribution Systems Handbook*. McGraw-Hill.
4. AWWA (2012). *Water Distribution System Design*. Manual M32, American Water Works Association.

