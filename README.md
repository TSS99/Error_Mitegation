# Mathematical Guide to Quantum Error Mitigation

This guide provides a step-by-step mathematical explanation of the quantum error mitigation techniques demonstrated in the notebook [pyquil_error_mitigation_july_1_to_5.executed.ipynb](file:///d:/CDAC%20Projects/Error%20Mitegation/pyquil_error_mitigation_july_1_to_5.executed.ipynb).

---

## 1. Ideal Bell State and Joint Expectation Value

We prepare a 2-qubit Bell state, $|\Phi^+\rangle$, using a Hadamard gate ($H$) on qubit 0 followed by a CNOT gate with qubit 0 as control and qubit 1 as target.

### Step-by-Step State Preparation
Starting from the ground state $|00\rangle$:

1. Apply Hadamard gate $H$ to the first qubit:
$$
(H \otimes I)|00\rangle = \frac{1}{\sqrt{2}}(|0\rangle + |1\rangle) \otimes |0\rangle = \frac{1}{\sqrt{2}}(|00\rangle + |10\rangle)
$$

2. Apply CNOT gate (controlled by qubit 0, targeting qubit 1):
$$
\text{CNOT}_{0 \to 1} \left[ \frac{1}{\sqrt{2}}(|00\rangle + |10\rangle) \right] = \frac{1}{\sqrt{2}}(|00\rangle + |11\rangle) = |\Phi^+\rangle
$$

### Joint Expectation Value $\langle ZZ \rangle$
We want to measure the expectation value of the joint operator $Z \otimes Z$, which measures the correlation between the two qubits. The Pauli-Z operator has eigenvalues $+1$ (for state $|0\rangle$) and $-1$ (for state $|1\rangle$).

The action of $Z \otimes Z$ on the basis states is:
$$
(Z \otimes Z)|00\rangle = (+1)(+1)|00\rangle = |00\rangle
$$
$$
(Z \otimes Z)|11\rangle = (-1)(-1)|11\rangle = |11\rangle
$$

Thus, the action of $Z \otimes Z$ on the Bell state $|\Phi^+\rangle$ is:
$$
(Z \otimes Z)|\Phi^+\rangle = (Z \otimes Z) \left[ \frac{1}{\sqrt{2}}(|00\rangle + |11\rangle) \right] = \frac{1}{\sqrt{2}}(|00\rangle + |11\rangle) = |\Phi^+\rangle
$$

The ideal expectation value is:
$$
\langle ZZ \rangle_{\text{ideal}} = \langle \Phi^+ | Z \otimes Z | \Phi^+ \rangle = \langle \Phi^+ | \Phi^+ \rangle = 1.0
$$

In terms of measurement probabilities, since $P(00) = 0.5$, $P(11) = 0.5$, and $P(01) = P(10) = 0$:
$$
\langle ZZ \rangle = P(00) + P(11) - P(01) - P(10) = 0.5 + 0.5 - 0 - 0 = 1.0
$$

---

## 2. Readout Noise Model

Readout noise occurs when a qubit is correctly prepared and processed, but the measurement device misidentifies the state (e.g., records a $|0\rangle$ as a $|1\rangle$).

### Single-Qubit Readout Noise
We model the measurement error on a single qubit as a symmetric bit-flip channel with error probability $e$. The transition matrix $K$ is:
$$
K = \begin{pmatrix} 1 - e & e \\ e & 1 - e \end{pmatrix}
$$
where:
* $K_{0,0} = P(\text{observe } 0 \mid \text{true state is } 0) = 1 - e$
* $K_{1,0} = P(\text{observe } 1 \mid \text{true state is } 0) = e$

### Two-Qubit Readout Noise (Confusion Matrix)
For a 2-qubit system, assuming the noise on each qubit is independent, the global transition probability matrix $M$ is the tensor product:
$$
M = K \otimes K
$$

$$
M = \begin{pmatrix} 1 - e & e \\ e & 1 - e \end{pmatrix} \otimes \begin{pmatrix} 1 - e & e \\ e & 1 - e \end{pmatrix}
$$

$$
M = \begin{pmatrix}
(1-e)^2 & e(1-e) & e(1-e) & e^2 \\
e(1-e) & (1-e)^2 & e^2 & e(1-e) \\
e(1-e) & e^2 & (1-e)^2 & e(1-e) \\
e^2 & e(1-e) & e(1-e) & (1-e)^2
\end{pmatrix}
$$

### Effect of Noise on Probabilities
Let $\mathbf{x}$ be the vector of true state probabilities and $\mathbf{y}$ be the vector of noisy observed probabilities:
$$
\mathbf{x} = \begin{pmatrix} P_{\text{ideal}}(00) \\ P_{\text{ideal}}(01) \\ P_{\text{ideal}}(10) \\ P_{\text{ideal}}(11) \end{pmatrix} = \begin{pmatrix} 0.5 \\ 0.0 \\ 0.0 \\ 0.5 \end{pmatrix}
$$

The noisy observed probabilities are calculated by $\mathbf{y} = M\mathbf{x}$:
$$
y_{00} = 0.5(1-e)^2 + 0.5e^2 = 0.5(1 - 2e + 2e^2)
$$
$$
y_{01} = 0.5e(1-e) + 0.5e(1-e) = e(1-e)
$$
$$
y_{10} = 0.5e(1-e) + 0.5e(1-e) = e(1-e)
$$
$$
y_{11} = 0.5e^2 + 0.5(1-e)^2 = 0.5(1 - 2e + 2e^2)
$$

For the simulated readout error rate $e = 0.08$ ($8\%$):
$$
y_{00} = 0.5(0.92^2 + 0.08^2) = 0.5(0.8464 + 0.0064) = 0.4264
$$
$$
y_{01} = 0.08 \times 0.92 = 0.0736
$$
$$
y_{10} = 0.0736
$$
$$
y_{11} = 0.4264
$$

The expectation value under readout noise is:
$$
\langle ZZ \rangle_{\text{noisy}} = y_{00} + y_{11} - y_{01} - y_{10}
$$
$$
\langle ZZ \rangle_{\text{noisy}} = 0.4264 + 0.4264 - 0.0736 - 0.0736 = 0.7056
$$

---

## 3. Readout Error Mitigation

Readout mitigation aims to reconstruct the true probability distribution $\mathbf{x}$ from the noisy observed distribution $\mathbf{y}$ using the calibration data contained in the confusion matrix $M$.

### Mathematical Correction
Since the noise model is linear:
$$
\mathbf{y} = M\mathbf{x}
$$

If the confusion matrix $M$ is invertible (which is true as long as $e \neq 0.5$), we can retrieve the ideal probabilities $\mathbf{x}$ by multiplying both sides by the inverse of $M$:
$$
\mathbf{x} = M^{-1}\mathbf{y}
$$

### Implementation Steps
1. Estimate $\mathbf{y}$ by measuring the relative frequencies of outcomes from the quantum device.
2. Solve the linear system $M\mathbf{x} = \mathbf{y}$ to find the mitigated vector $\mathbf{x}$.
3. Due to statistical sampling noise, some values in the calculated vector $\mathbf{x}$ might be slightly negative or exceed $1.0$. We project the vector back to the closest valid probability simplex by:
   * Clipping values to the range $[0, 1]$.
   * Re-normalizing the vector so its elements sum to $1.0$.

---

## 4. CNOT Folding (Noise Scaling)

Zero-Noise Extrapolation (ZNE) requires running the quantum circuit at different levels of noise. We scale the noise of a gate artificially by replacing it with an odd number of identical gates, a process called **gate folding**.

### Mathematical Invariance
Let $\text{CNOT}$ be the unitary operator representing the gate. Because a CNOT gate is its own self-inverse, applying it twice results in the identity operator:
$$
\text{CNOT}^2 = I
$$

For any odd scale factor $S = 2k + 1$ (where $k \ge 0$ is an integer):
$$
\text{CNOT}^{2k+1} = (\text{CNOT}^2)^k \text{CNOT} = I^k \text{CNOT} = \text{CNOT}
$$

Thus, in a noiseless simulator, folding a gate does not change the final quantum state. However, on physical hardware (or in a noisy simulator), applying $S$ gates instead of $1$ gate increases the error rate by a factor of $S$.

---

## 5. Zero-Noise Extrapolation (ZNE)

We collect the noisy expectation values $E(S)$ at different noise scale factors $S \in \{1, 3, 5\}$ and fit a curve to extrapolate the value at $S = 0$ (zero noise).

### Linear Fit Model
We assume that for small scale factors, the expectation value decays linearly with the noise scale:
$$
E(S) = mS + b
$$
where:
* $m$ is the slope of the error decay.
* $b$ is the y-intercept, representing the simulated expectation value at zero noise ($S = 0$):
$$
E(0) = b
$$

### Linear Regression Math
Using the method of least squares, the parameters $m$ and $b$ for $N$ data points $(S_i, E_i)$ are given by:
$$
m = \frac{N \sum (S_i E_i) - (\sum S_i)(\sum E_i)}{N \sum S_i^2 - (\sum S_i)^2}
$$

$$
b = \frac{\sum E_i - m \sum S_i}{N}
$$

Evaluating this linear fit allows us to estimate the gate-error-mitigated expectation value $b$ from the set of noisy measurements.
