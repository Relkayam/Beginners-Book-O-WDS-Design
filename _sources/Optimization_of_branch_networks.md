# Chapter 4: Optimization of Branched Networks

## 1. Introduction to Network Optimization

In the previous chapter, we learned how to perform a hydraulic analysis of a branched network with known pipe diameters. While essential, analysis is only half the story. The real engineering challenge lies in **design**: selecting the most cost-effective pipe diameters that still meet all hydraulic requirements. This process is known as **optimization**.

This chapter introduces the core concepts and mathematical methods for optimizing branched water networks. We will use the `BNHA` (Branched Network Hydraulic Analysis) Python package, an open-source tool designed specifically for this purpose. `BNHA` automates the complex task of pipe sizing, allowing us to focus on the design principles.

## 2. Overview of the `BNHA` Package

The `BNHA` package is a powerful tool for designing and analyzing branched water distribution networks. Its key features include:

- **Pipe Diameter Optimization:** Automatically selects the least-cost pipe sizes from a given catalog while satisfying hydraulic constraints, such as minimum pressure.
- **Hydraulic Analysis:** Computes head loss, velocity, and pressure across the network for a given design.
- **Diagnostics and Verification:** Exports the underlying optimization models and constraint details, which is invaluable for troubleshooting and validating designs.

It is built on common Python libraries like `pandas` for data management, `NumPy` for numerical operations, and `PuLP` for solving optimization problems. The source code and further documentation are available on [GitHub](https://github.com/Relkayam/BNHA).

## 3. Mathematical Formulation

To understand how optimization works, we must first define the mathematical notation used in the models.

| Notation | Description |
| :--- | :--- |
| $\mathcal{P}$ | The set of all pipes in the network. |
| $\mathcal{B}$ | The set of all unique paths from the source to a terminal node. |
| $\mathcal{D}_i$ | The set of available commercial diameters for pipe $i \in \mathcal{P}$. |
| $\mathcal{S}_i$ | The set of sub-segments for pipe $i$ in the continuous-length model. |
| $dz_p$ | The available static head (elevation drop) along a path $p \in \mathcal{B}$. |
| $h_{\min}$ | The minimum required pressure head at any terminal node. |
| $L_i$ | The length of pipe $i$. |
| $c_{i,d}$ | The cost of selecting diameter $d$ for pipe $i$. |
| $h_{i,d}$ | The head loss in pipe $i$ if diameter $d$ is chosen. |
| $x_{i,d}$ | A binary decision variable: 1 if diameter $d$ is chosen for pipe $i$, 0 otherwise. |
| $l_j$ | The length of a sub-segment $j$ in the continuous-length model. |
| $c_j$ | The cost per meter for a sub-segment $j$. |
| $h'_j$ | The head loss per meter for a sub-segment $j$. |
| $\varepsilon$ | A small pressure tolerance for numerical stability. |

---

## 4. Optimization Methodologies

`BNHA` implements two primary optimization approaches for pipe sizing: Discrete and Continuous.

### 4.1 Discrete Diameter Optimization (Mixed-Integer Programming)

This is the most common design scenario. The model selects **one single diameter** for each pipe from a predefined catalog of commercial sizes to minimize total cost while meeting pressure requirements.

**Objective: Minimize Total Pipe Cost**

$$
\min \sum_{i \in \mathcal{P}} \sum_{d \in \mathcal{D}_i} x_{i,d} \cdot c_{i,d}
$$

**Constraints:**

1.  **Single Diameter per Pipe:** Ensures exactly one diameter is chosen for each pipe.

    $$
    \sum_{d \in \mathcal{D}_i} x_{i,d} = 1 \quad \forall i \in \mathcal{P}
    $$

2.  **Minimum Pressure Requirement:** The total head loss along any path cannot exceed the available head (static head minus minimum required pressure).

    $$
    \sum_{i \in p} \sum_{d \in \mathcal{D}_i} x_{i,d} \cdot h_{i,d} \leq dz_p - h_{\min} \quad \forall p \in \mathcal{B}
    $$

This formulation is a **Mixed-Integer Programming (MIP)** problem, which `BNHA` solves using the `PuLP` library.

### 4.2 Continuous-Length Optimization (Linear Programming)

This advanced method allows a single pipe to be constructed from **multiple segments of different diameters**. This is useful for designing long, tapered pipelines where hydraulic efficiency can be significantly improved.

**Objective: Minimize Total Pipe Cost**

$$
\min \sum_{j} l_j \cdot c_j
$$

**Constraints:**

1.  **Conservation of Length:** The sum of segment lengths must equal the total length of the pipe.

    $$
    \sum_{j \in \mathcal{S}_i} l_j = L_i \quad \forall i \in \mathcal{P}
    $$

2.  **Minimum Pressure Requirement:** The total head loss along any path must be within the available head.

    $$
    \sum_{j \in p} l_j \cdot h'_j \leq dz_p - h_{\min} - \varepsilon
    $$

This is a **Linear Programming (LP)** problem, which is computationally faster to solve than MIP problems.

---

## 5. From Theory to Practice

The next chapter provides a complete, hands-on Jupyter Notebook to guide you through using these optimization methods with `BNHA`. We will explore practical scenarios, including:

1.  **Analyzing an Existing Network:** Calculating pressures and velocities to identify hydraulic bottlenecks.
2.  **Optimizing a New Design:** Using the discrete model to find the most economical pipe diameters for a new irrigation system.
3.  **Designing a Tapered Pipeline:** Applying the continuous model to a long transmission main to optimize performance and cost.

These examples will demonstrate how to structure input data using `pandas` and interpret the results to make informed engineering decisions.

**Ready to get started? Let's move to the next chapter to put this theory into practice.**
