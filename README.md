# Fusion-Based Quantum Computing: Resource States, Heralding, and Measurement Branches

Numerical analysis of Bell-state measurements and loss-aware photonic resource preparation. The notebook combines exact small-state calculations, a probabilistic entanglement-swapping model, Monte Carlo validation, and an extension that resolves the quantum-state outcomes of destructive Bell measurements on two graph-state resources.

The models separate ideal projective measurements from externally specified photon-survival and heralding probabilities. All numerical parameters are illustrative; the results are not hardware-calibrated fusion-gate predictions. The tensor indexing convention is explicitly fixed to $q_0$ as the most-significant bit, consistent with the displayed qubit labels.

## Notebook organization

[`Fusion_Based_Quantum_Computing.ipynb`](Fusion_Based_Quantum_Computing.ipynb) is the complete executable analysis, including the extension at the end.

| Section | Content |
| --- | --- |
| **1–3** | Bell and GHZ resource states, stabilizers, Bell-basis projectors, four-outcome entanglement swapping, and a specified measurement-failure example |
| **4–5** | Photon survival, heralded operation success, endpoint phase errors, exact statevector projections, Pauli-frame correction, and density-matrix checks |
| **6–8** | Retry costs, parallel independent attempts, Monte Carlo uncertainties, convergence, and a parameter sweep |
| **9 · Extension** | Bell measurements on two three-qubit graph states, four normalized output branches, bipartite entanglement, alternative local-$Z$ failure branches, and heralded sampling |

The reference simulation functions are **included in the notebook**. No `src/` directory, input dataset, or separate extension file is required. Executing the notebook creates a local `figures/` directory containing plots and a CSV summary.

## Baseline model: heralded Bell-pair chains

Consider $L$ independent initial Bell pairs, each prepared in

$$
|\Phi^+\rangle=\frac{|00\rangle+|11\rangle}{\sqrt{2}}.
$$

An ideal Bell measurement on the adjacent qubits of two pairs entangles the remote endpoints, conditional on the measurement outcome. The measured qubits are consumed. For an outcome labelled $(a,b)$, a known Pauli-frame correction restores $|\Phi^+\rangle$ at the endpoints.

In the resource-accounting model, each of the $2L$ photons independently survives with probability $\eta$; each of the $L-1$ intermediate operations is independently heralded with probability $h$. A complete chain is retained only when every required event succeeds, with no partial recovery or repeat-until-success protocol:

$$
P_{\mathrm{accept}}=\eta^{2L}h^{L-1}.
$$

The surviving endpoint pair experiences independent $Z$ faults with probability $p$ per endpoint. Because a pair of $Z$ faults cancels in the Bell-state overlap,

$$
F_{\mathrm{cond}}=(1-p)^2+p^2,
\qquad
Q=P_{\mathrm{accept}}F_{\mathrm{cond}}.
$$

Here $F_{\mathrm{cond}}$ is the overlap with the corrected target Bell state **conditioned on acceptance**; $Q$ is the acceptance-weighted overlap per complete attempt. Neither is a logical error rate.

Fully provisioned attempts consume $2L$ input photons each, so the mean supplied-photon cost per accepted chain is $2L/P_{\mathrm{accept}}$. For $K$ independent parallel attempts, the probability of at least one accepted output in a round is

$$
P_{\geq1}=1-(1-P_{\mathrm{accept}})^K.
$$

This captures resource consumption but not optical switching, memory lifetime, feed-forward latency, or real timing throughput.

## Extension: explicit measurement branches on graph resources

The baseline samples abstract herald flags without calculating residual resource states after an attempted operation on multi-qubit graph resources. Section 9 adds a small exact-state calculation to resolve this distinction.

Two three-qubit path-graph states are prepared independently, with qubits labelled $0,1,2$ and $3,4,5$. A destructive Bell measurement acts on qubits $2$ and $3$; the remaining state is ordered as $(0,1,4,5)$. For Bell outcome $(a,b)$,

$$
|\psi_{ab}\rangle =
\frac{(\langle\beta_{ab}|_{2,3}\otimes I)|G_A\rangle\otimes|G_B\rangle}
{\sqrt{P_{ab}}}.
$$

The notebook computes all four Born probabilities and verifies their normalization. Each Bell branch has probability $1/4$ in this example. The reduced-state von Neumann entropy across $(0,1)\mid(4,5)$ is one bit for each Bell outcome, confirming entanglement between the initially independent resources.

The statevector calculation also identifies the output state for a representative branch. For ordinary Bell outcome $(a,b)=(0,0)$, local Hadamards on the two outer surviving qubits map the four-qubit output exactly to a GHZ state (unit overlap). Applying a Hadamard to the measured qubit $q_3$ before the Bell projection instead yields the four-qubit path-graph state for outcome $(0,0)$ (unit overlap). The rotated projection represents an ideal measurement basis, not a model of its optical implementation.

As a **separate, illustrative failure instrument**, local $Z$ measurements replace the Bell projection on qubits $2$ and $3$. Their outcomes leave the two residual resources separable across the original cut (zero entanglement entropy). This is not a universal optical fusion-failure rule.

Finally, a sampled acceptance event is defined using independent survival probability $\eta$ for six photons and an **externally assigned** herald probability $h$:

$$
P_{\mathrm{accept,ext}}=\eta^6h.
$$

Given an accepted event, the Bell outcome is sampled from the computed Born probabilities. This hybrid model distinguishes state-dependent outcome statistics from the hardware-dependent event that selects the instrument.

### What the extension establishes

- Explicit normalized Bell-projection branches acting on two graph-state resources.
- Direct output-state checks distinguishing the GHZ-equivalent ordinary Bell branch from a path-graph output obtained using a rotated Bell projection.
- Nonzero entanglement across the original resource partition after successful projection.
- A contrasting local-measurement instrument whose residual resources remain separable across that cut.
- Analytical and Monte Carlo checks of the independently specified acceptance probability.

The extension checks GHZ-equivalence and path-graph equivalence only for the specified $(0,0)$ branches. It does not establish general outcome-dependent feed-forward corrections or model beam splitters, detectors, distinguishability, loss-tolerant graph rewrites, or a scalable fusion architecture. The extension is an independent small-state calculation; the baseline's endpoint $Z$-noise model is not compounded with its four-qubit outputs.

## Run locally

Python 3.10 or newer is recommended. From the repository directory:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install numpy matplotlib networkx jupyterlab ipykernel
jupyter lab Fusion_Based_Quantum_Computing.ipynb
```

On Windows, activate the environment with `.venv\Scripts\activate`. Select the environment's Python kernel and choose **Run All**. Running all cells in order is important because the extension initializes additional statevector variables after the baseline study.

The notebook requires no external data. Running it creates `figures/multiplexing.png`, `figures/link_scaling.png`, `figures/convergence.png`, and `figures/final_experiment.csv` in the current working directory.

## Reproducibility and validation

The numerical checks include Bell-basis orthonormality, projector completeness, stabilizer eigenvalues, all four entanglement-swapping branches, Pauli-frame corrections, density-matrix trace and purity, deterministic loss/noise limits, and Monte Carlo agreement with analytical acceptance and conditional fidelity. Section 9 additionally verifies four normalized projected graph-resource states, tests GHZ- versus path-state equivalence for the $(0,0)$ branches, checks a labelled asymmetric graph against big-endian qubit indexing, compares entanglement and separability, and tests sampled heralding consistency.

Monte Carlo routines use explicit seeds, with distinct seeds for different convergence sample sizes. Normal-approximation standard errors are reported for binary estimators, and the extension checks sampled acceptance rates against a five-standard-error bound; conditional-fidelity uncertainty is calculated with the number of accepted trials rather than the total number of attempts.

For the baseline example $L=3$, $\eta=0.95$, $h=0.5$, and $p=0.03$:

| Quantity | Analytical value |
| --- | ---: |
| Complete-chain acceptance $P_{\mathrm{accept}}$ | 0.183773 |
| Conditional Bell-state fidelity $F_{\mathrm{cond}}$ | 0.941800 |
| Acceptance-weighted overlap $Q$ | 0.173077 |
| Mean supplied photons per accepted output | 32.65 |

With six surviving photons and $\eta=0.95$, the extension's acceptance model predicts $\eta^6 h$; for $h=0.5$ this is approximately $0.36755$. These are calculations under stated assumptions, not measurements from a photonic device.

## Scope and limitations

The baseline treats a successful chain as an ideal endpoint Bell pair with output-only phase errors. It does not propagate noise accumulated at earlier links or calculate a physically specified linear-optical heralding instrument. Failed attempts are discarded. Multiplexed copies are statistically independent and do not share constrained optical hardware.

The extension evaluates ideal projective measurements on two small graph resources. Although entanglement appears across the original resource cut, the verified target-state equivalences are limited to the stated ideal projector branches; no hardware-specific multi-qubit fusion instrument or parameter extraction is established. The notebook therefore supports controlled small-system calculations and resource trade-off studies, **not** fault-tolerance thresholds, large-scale photonic architecture forecasts, or experimentally validated fusion efficiencies.

## References

- S. Bartolucci *et al.*, “Fusion-based quantum computation,” *Nature Communications* **14**, 912 (2023). https://doi.org/10.1038/s41467-023-36493-1
- D. E. Browne and T. Rudolph, “Resource-efficient linear optical quantum computation,” *Physical Review Letters* **95**, 010501 (2005). https://doi.org/10.1103/PhysRevLett.95.010501
- R. Raussendorf and H. J. Briegel, “A one-way quantum computer,” *Physical Review Letters* **86**, 5188 (2001). https://doi.org/10.1103/PhysRevLett.86.5188
