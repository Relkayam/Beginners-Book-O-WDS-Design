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

- **Energy Grade Line (EGL)** represents the total energy head: $H_{EGL} = z + \\frac{p}{\\gamma} + \\frac{v^2}{2g}$
- **Hydraulic Grade Line (HGL)** represents the piezometric head: $H_{HGL} = z + \\frac{p}{\\gamma}$
- **Pressure Head** is the difference between the HGL and the pipe elevation: $\\frac{p}{\\gamma} = H_{HGL} - z$

where $z$ is elevation, $p$ is pressure, $\\gamma$ is the specific weight of water, $v$ is velocity, and $g$ is gravity. The HGL is what determines the pressure in the pipes.

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
6.  **Evaluate Design**: Check if the pressures and velocities meet the required design constraints.

### 3.3 Key Design Considerations

When selecting pipe diameters, engineers must balance multiple factors:
- **Minimum Pressure**: Typically 20–35 m (200–350 kPa) must be maintained at all demand nodes.
- **Maximum Velocity**: Velocities are often kept below 3 m/s to prevent issues like water hammer and excessive head loss.
- **Economic Optimization**: The goal is to minimize total cost, which includes both the initial pipe installation cost and long-term pumping (energy) costs.
- **Fire Flow Demands**: The system must be able to provide higher flows during emergencies, which can be a critical factor in pipe sizing.

---

## 4. Python Implementation Guide

Here, we provide Python functions to implement the hydraulic analysis.

### 4.1 Required Libraries

These libraries are essential for data handling and creating visualizations.

```python
import pandas as pd
import numpy as np
import plotly.graph_objects as go
from plotly.subplots import make_subplots
```

### 4.2 Head Loss Function

This function implements the Hazen-Williams equation.

```python
def calculate_head_loss(L, Q, C, d):
    """Calculates head loss using the Hazen-Williams equation.
    
    Args:
        L (float): Pipe length (m).
        Q (float): Flow rate (m³/s).
        C (float): Hazen-Williams roughness coefficient.
        d (float): Pipe diameter (m).
    
    Returns:
        float: Frictional head loss (m).
    """
    if d <= 0:
        return np.inf  # Avoid division by zero
    
    return (10.67 * L * Q**1.852) / (C**1.852 * d**4.8704)
```

### 4.3 Velocity Function

This function calculates the flow velocity inside a pipe.

```python
def calculate_velocity(Q, d):
    """Calculates the flow velocity in a circular pipe.
    
    Args:
        Q (float): Flow rate (m³/s).
        d (float): Pipe diameter (m).
    
    Returns:
        float: Flow velocity (m/s).
    """
    if d <= 0:
        return 0
    
    area = np.pi * (d/2)**2
    return Q / area
```

### 4.4 Network Analysis Function

This function orchestrates the entire analysis, from calculating losses to determining nodal pressures.

```python
def analyze_network(pipes_dict, reservoir_head):
    """Performs a complete hydraulic analysis of a branched network.
    
    Args:
        pipes_dict (dict): A dictionary containing the data for each pipe.
        reservoir_head (float): The total head at the upstream reservoir (m).
    
    Returns:
        pd.DataFrame: A DataFrame containing the analysis results for each node.
    """
    # First, calculate head loss and velocity for each pipe
    for pipe_id, pipe_data in pipes_dict.items():
        pipe_data['head_loss'] = calculate_head_loss(
            pipe_data['L'], pipe_data['Q'], 
            pipe_data['C'], pipe_data['d']
        )
        pipe_data['velocity'] = calculate_velocity(
            pipe_data['Q'], pipe_data['d']
        )
    
    # Sequentially calculate HGL and pressures
    results = []
    cumulative_distance = 0
    current_hgl = reservoir_head
    
    # Assuming pipes are sorted upstream to downstream
    for pipe_id in sorted(pipes_dict.keys()):
        pipe = pipes_dict[pipe_id]
        
        # Node at the start of the pipe
        results.append({
            'Node': f'Node_{pipe_id}_start',
            'Distance': cumulative_distance,
            'Elevation': pipe['z_start'],
            'HGL': current_hgl,
            'Pressure_Head': current_hgl - pipe['z_start'],
            'Velocity': pipe['velocity'],
            'Pipe_ID': pipe_id
        })
        
        # Update values for the end of the pipe
        cumulative_distance += pipe['L']
        current_hgl -= pipe['head_loss']
        
        # Node at the end of the pipe
        results.append({
            'Node': f'Node_{pipe_id}_end',
            'Distance': cumulative_distance,
            'Elevation': pipe['z_end'],
            'HGL': current_hgl,
            'Pressure_Head': current_hgl - pipe['z_end'],
            'Velocity': pipe['velocity'],
            'Pipe_ID': pipe_id
        })
    
    return pd.DataFrame(results)
```

---

## 5. Interpreting the Results

### 5.1 Hydraulic Performance Metrics

The primary outputs to check are:
- **Minimum Pressure Head**: Does it meet the service requirements at all nodes?
- **Maximum Velocity**: Does it stay within acceptable limits to avoid system damage?
- **Total Head Loss**: How much energy is lost? This directly impacts pumping costs.

### 5.2 Visualization and Analysis

Plotting the results is the best way to understand them:
- The **Hydraulic Grade Line (HGL)** plot shows the available pressure along the network.
- The **Energy Grade Line (EGL)** plot illustrates total energy dissipation due to friction.
- An **Elevation Profile** provides the physical context for the pressure analysis.

### 5.3 Using the Tool for Design

This analysis tool empowers you to:
1.  **Identify Bottlenecks**: Quickly find nodes with low pressure.
2.  **Evaluate Alternatives**: Compare the hydraulic performance of different diameter combinations.
3.  **Optimize for Cost**: Provide the hydraulic data needed to balance pipe costs and energy costs.
4.  **Verify Constraints**: Ensure your final design meets all hydraulic requirements.

---

## 6. Practical Applications

### 6.1 Sensitivity Analysis

You can systematically vary pipe diameters to understand their impact on system pressures and velocities. This helps in identifying the most critical pipes in the network.

### 6.2 Optimization Input

The calculated head losses and pressures are essential inputs for optimization algorithms that aim to find the least-cost design while satisfying all hydraulic constraints. This topic is covered in the next chapters.

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

