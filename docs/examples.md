# Research Workflows — Curated Fork

This is a maintained selection of illustrative research workflows using Skills included in this fork (**88 skills**). The original cross-disciplinary examples are available in the [upstream project](https://github.com/K-Dense-AI/scientific-agent-skills/blob/main/docs/examples.md).

These are research prompts, not completed or independently validated experiments. Confirm package versions, data provenance, licensing, and execution permissions before running any workflow.

## Materials Science & Chemistry

### Example 10: High-Throughput Materials Discovery for Battery Applications

**Objective**: Discover novel solid electrolyte materials for lithium-ion batteries using computational screening.

**Disciplines**: solid-state chemistry · condensed-matter physics · electrochemistry · machine learning · optimization

**Skills Used**:
- `pymatgen` - Materials analysis and feature engineering
- `scikit-learn` - Machine learning
- `pymoo` - Multi-objective optimization
- `arbor` - Hypothesis-tree search over screening/model configurations without overfitting the dev set
- `sympy` - Symbolic math
- `uncertainty-and-units` - meV/atom, S/cm, eV: dimensional checks and error propagation
- `vaex` - Large dataset handling
- `dask` - Parallel computing
- `matplotlib` - Visualization
- `scientific-writing` - Report generation
- `scientific-visualization` - Publication figures

**Starting prompt**:

```text
Use the pymatgen, scikit-learn, pymoo, uncertainty-and-units, dask, and
scientific-writing skills.

Goal: a Pareto set of candidate solid electrolytes worth attempting to
synthesize, with an honest read on which predictions are trustworthy.
Criteria: state the DFT functional behind every energy; hold out a chemical
family entirely rather than splitting randomly.
Deliver: the screened library, the Pareto front, a top-10 table with predicted
values and intervals, and DFT validation for those 10.
Report: how far each Pareto candidate sits from the training distribution.
Extrapolating a conductivity model into a new anion chemistry is a guess, and
should be labelled one.
```

**Workflow**:

```text
Step 1: Generate candidate materials library
- Use Pymatgen to enumerate compositions:
  * Li-containing compounds (Li₁₋ₓM₁₊ₓX₂)
  * M = transition metals (Zr, Ti, Ta, Nb)
  * X = O, S, Se
- Generate ~10,000 candidate compositions
- Apply charge neutrality constraints

Step 2: Filter by thermodynamic stability
- Query the Materials Project through the current `mp-api` client (the legacy
  pymatgen MPRester endpoints have been retired) and record the database version
- Calculate formation energy from elements and energy above the convex hull
- Filter at E_hull < 50 meV/atom, and describe what that means accurately:
  E_hull = 0 is on the hull; a nonzero value is metastable. The 50 meV/atom line is
  an empirical heuristic for "has been synthesized before at comparable
  metastability", not a stability guarantee. Many known, useful materials sit above
  it, and plenty below it have never been made
- Compare energies only within one functional and one correction scheme. Mixing
  GGA and GGA+U totals across a hull produces meaningless differences
- Use uncertainty-and-units to keep meV/atom, eV/formula-unit, and kJ/mol distinct
  throughout; a silent factor of 96.5 here invalidates the entire screen
- Retain ~2,000 thermodynamically plausible compounds

Step 3: Predict crystal structures
- Use Pymatgen structure predictor
- Generate most likely crystal structures for each composition
- Consider common structure types: LISICON, NASICON, garnet, perovskite
- Calculate structural descriptors

Step 4: Calculate material properties with Pymatgen
- Lattice parameters and volume
- Density
- Packing fraction
- Ionic radii and bond lengths
- Coordination environments

Step 5: Feature engineering with Pymatgen
- Calculate compositional features using Pymatgen's featurizers:
  * Elemental property statistics (electronegativity, ionic radius)
  * Valence electron concentrations
  * Stoichiometric attributes
- Calculate structural features:
  * Pore size distribution
  * Site disorder parameters
  * Partial radial distribution functions

Step 6: Build ML models for Li⁺ conductivity prediction
- Collect training data from literature (experimental conductivities)
- Note what that data is: room-temperature conductivities measured by different
  groups on differently-densified pellets vary by orders of magnitude for the same
  nominal composition. Model on log₁₀(σ) and expect an irreducible error floor
- Train ensemble models with scikit-learn (Random Forest, Gradient Boosting, MLP)
- Split by chemical family, holding out whole anion or framework classes. Random
  5-fold CV on a literature set full of near-duplicate doped variants reports an
  accuracy the model will not reproduce on anything new
- Predict ionic conductivity for all candidates with prediction intervals, and use
  arbor if you want to search systematically over featurization, model, and split
  choices — its held-out merge gate is what keeps that search from quietly tuning
  itself onto the validation set

Step 7: Predict additional properties
- Electrochemical stability window (ML model)
- Mechanical properties (bulk modulus, shear modulus)
- Interfacial resistance (estimate from structure)
- Synthesis temperature (ML prediction from similar compounds)

Step 8: Multi-objective optimization with PyMOO
Define optimization objectives:
- Maximize: ionic conductivity (>10⁻³ S/cm target)
- Maximize: electrochemical window (>4.5V target)
- Minimize: synthesis temperature (<800°C preferred)
- Minimize: cost (based on elemental abundance)

Run NSGA-II to find Pareto optimal solutions
Extract top 50 candidates from Pareto front

Step 9: Analyze Pareto optimal materials
- Identify composition trends (which elements appear frequently)
- Analyze structure-property relationships
- Calculate trade-offs between objectives
- Identify "sweet spot" compositions

Step 10: Validate predictions with DFT calculations
- Select top 10 candidates for detailed study
- Set up DFT calculations using Pymatgen's interface
- Calculate:
  * Formation energies at converged k-point density and cutoff (report both)
  * Li⁺ migration barriers (NEB calculations)
  * Electronic band gap — GGA underestimates gaps substantially, so a GGA gap
    is a lower bound and cannot by itself establish an electrochemical window
  * Elastic constants
- A migration barrier is not a conductivity. Converting one to the other needs the
  attempt frequency and the mobile-carrier concentration via a Nernst-Einstein
  relation, plus an assumption that the migration path found by NEB is the rate-
  limiting one. Ab initio MD at elevated temperature is the stronger check where
  affordable
- Barriers computed in a perfect bulk crystal ignore grain boundaries and interfaces,
  which usually dominate measured conductivity in a real pellet
- Compare DFT results with ML predictions and record where they disagree

Step 11: Literature and patent search
- Search for prior art on top candidates
- PubMed and Google Scholar: "[composition] AND electrolyte"
- USPTO: Check for existing patents on similar compositions
- Identify any experimental reports on related materials

Step 12: Generate materials discovery report
- Summary of screening workflow and statistics
- Pareto front visualization (conductivity vs stability vs cost)
- Structure visualization of top candidates
- Property comparison table
- Composition-property trend analysis
- DFT validation results
- Predicted performance vs state-of-art materials
- Synthesis recommendations
- IP landscape summary
- Prioritized list of 5-10 materials for experimental validation
- Export as publication-ready PDF

Expected Output:
- Screened library of 10,000+ materials
- ML models for property prediction
- Pareto-optimal set of 50 candidates
- Detailed analysis of top 10 materials
- DFT validation results
- Comprehensive materials discovery report
```

---

## Experimental Physics & Data Analysis

### Example 19: Analysis of Particle Physics Detector Data

**Objective**: Analyze experimental data from particle detector to identify signal events and measure physical constants.

**Disciplines**: experimental particle physics · statistics · machine learning · large-scale computing · metrology

**Skills Used**:
- `astropy` - Units and constants
- `uncertainty-and-units` - GUM uncertainty budget, coverage factors, error propagation
- `sympy` - Symbolic mathematics
- `matlab` - Matrix/numerical computing and signal processing
- `statistical-analysis` - Statistical analysis
- `scikit-learn` - Classification
- `stable-baselines3` - Reinforcement learning for optimization
- `matplotlib` - Visualization
- `seaborn` - Statistical plots
- `statsmodels` - Hypothesis testing
- `dask` - Large-scale data processing
- `vaex` - Out-of-core dataframes
- `scientific-visualization` - Publication-quality & interactive visualization

**Starting prompt**:

```text
Use the vaex, dask, scikit-learn, statistical-analysis, statsmodels,
uncertainty-and-units, and scientific-visualization skills.

Goal: a cross-section measurement with a defensible uncertainty budget.
Criteria: the analysis is blinded — every selection, cut, and classifier
threshold is fixed using simulation and sidebands before the signal region is
looked at. Say explicitly when the box is opened.
Deliver: selection efficiency, background estimate, fitted yield, cross
section with statistical and systematic uncertainties itemized separately.
Report: significance via the asymptotic likelihood formula, not S/sqrt(B).
If the search scanned a mass range, give local and global significance.
```

**Workflow**:

```text
Step 1: Load and inspect detector data
- Load ROOT files or HDF5 with raw detector signals
- Use Vaex for out-of-core processing (TBs of data)
- Inspect data structure: event IDs, timestamps, detector channels
- Extract key observables:
  * Energy deposits in calorimeters
  * Particle trajectories from tracking detectors
  * Time-of-flight measurements
  * Trigger information

Step 2: Apply detector calibration and corrections
- Load calibration constants
- Apply energy calibrations to convert ADC to physical units
- Correct for detector efficiency variations
- Apply geometric corrections (alignment)
- Use Astropy units for unit conversions (eV, GeV, MeV)
- Account for dead time and detector acceptance

Step 3: Event reconstruction
- Cluster energy deposits to form particle candidates
- Reconstruct particle trajectories (tracks)
- Match tracks to calorimeter clusters
- Calculate invariant masses for particle identification
- Compute momentum and energy for each particle
- Use Dask for parallel processing across events

Step 4: Event selection and filtering (blinded)
- Define the signal region from the physics hypothesis, then keep it blinded:
  optimize every cut on simulation and on data sidebands only. Tuning selections
  while watching the signal region biases the yield upward and invalidates the
  quoted p-value, because the selection has been fitted to a fluctuation
- Fix and document the full selection before unblinding, and record the moment
- Apply quality cuts:
  * Track quality (chi-squared, number of hits)
  * Fiducial volume cuts
  * Timing cuts (beam window)
  * Particle identification cuts
- Estimate trigger efficiency
- Calculate event weights for corrections

Step 5: Background estimation
- Identify background sources:
  * Cosmic rays
  * Beam-related backgrounds
  * Detector noise
  * Physics backgrounds (non-signal processes)
- Simulate backgrounds using Monte Carlo (if available)
- Estimate background from data in control regions
- Use sideband subtraction method

Step 6: Signal extraction
- Fit invariant mass distributions to extract signal
- Use a binned or unbinned extended maximum-likelihood fit:
  * Signal model: Gaussian, or Breit-Wigner convolved with the detector resolution
    when the natural width is comparable to resolution
  * Background model: polynomial or exponential, with the functional-form choice
    itself carried as a systematic (fit with alternatives, take the spread)
- Compute significance with the asymptotic formula from the profile likelihood ratio,
  Z = sqrt( 2 [ (s+b) ln(1 + s/b) - s ] ), which reduces to S/√B only in the
  large-background limit. S/√B overstates significance exactly where it matters
  most — few events, small b — so quote the likelihood-based number
- Include background-estimate uncertainty in the profile as a nuisance parameter;
  significance computed with b fixed is not the significance you have
- If the fit scanned a mass range, the local significance is inflated by the
  look-elsewhere effect. Report the global significance as well, or say the trials
  factor was not evaluated

Step 7: Machine learning event classification
- Train classifier with scikit-learn to separate signal from background
- Features: kinematic variables, topology, detector response
- Models: Boosted Decision Trees (XGBoost), Neural Networks
- Cross-validate with k-fold CV
- Optimize selection criteria using ROC curves
- Calculate signal efficiency and background rejection

Step 8: Reinforcement learning for trigger optimization
- Use Stable-Baselines3 to optimize trigger thresholds
- Environment: detector simulator
- Action: adjust trigger thresholds
- Reward: maximize signal efficiency while controlling rate
- Train PPO or SAC agent
- Validate on real data

Step 9: Calculate physical observables
- Measure cross-sections:
  * σ = N_signal / (ε × L × BR)
  * N_signal: number of signal events
  * ε: detection efficiency (including acceptance — state whether it is folded in)
  * L: integrated luminosity
  * BR: branching ratio
- Use uncertainty-and-units to carry units and correlated uncertainties through the
  division. Naive quadrature is wrong here: the efficiency and the background
  estimate often share a simulation-modelling systematic, and treating correlated
  terms as independent understates the total
- Derive the propagation symbolically with Sympy where the expression is nontrivial,
  and use Astropy units and constants for the conversions (eV, GeV, barns, pb⁻¹)
- Report a GUM-style budget: Type A (statistical) and Type B (systematic) terms
  itemized, the combined standard uncertainty, and the coverage factor used

Step 10: Statistical analysis and hypothesis testing
- Perform hypothesis tests with statsmodels:
  * Likelihood ratio test for signal vs background-only
  * Calculate p-values and significance levels
  * Set confidence limits (CLs method)
- Bayesian analysis for parameter estimation
- Calculate confidence intervals and error bands

Step 11: Systematic uncertainty evaluation
- Identify sources of systematic uncertainty:
  * Detector calibration uncertainties
  * Background estimation uncertainties
  * Theoretical uncertainties (cross-sections, PDFs)
  * Monte Carlo modeling uncertainties
- Propagate uncertainties through analysis chain
- Combine statistical and systematic uncertainties
- Present as error budget

Step 12: Create comprehensive physics report
- Event displays showing candidate signal events
- Kinematic distributions (momentum, energy, angles)
- Invariant mass plots with fitted signal
- ROC curves for ML classifiers
- Cross-section measurements with error bars
- Comparison with theoretical predictions
- Systematic uncertainty breakdown
- Statistical significance calculations
- Interpretation:
  * Consistency with Standard Model
  * Constraints on new physics parameters
  * Discovery potential or exclusion limits
- Recommendations:
  * Detector improvements
  * Additional data needed
  * Future analysis strategies
- Export publication-ready PDF formatted for physics journal

Expected Output:
- Reconstructed physics events
- Signal vs background classification
- Measured cross-sections and branching ratios
- Statistical significance of observations
- Systematic uncertainty analysis
- Comprehensive experimental physics paper
```

---

## Chemical Engineering & Process Optimization

### Example 20: Optimization of Chemical Reactor Design and Operation

**Objective**: Design and optimize a continuous chemical reactor for maximum yield and efficiency while meeting safety and economic constraints.

**Disciplines**: chemical engineering · reaction kinetics · Bayesian inference · control theory · process economics

**Skills Used**:
- `sympy` - Symbolic equations and reaction kinetics
- `uncertainty-and-units` - Dimensional consistency across kinetics, balances, and economics
- `statistical-analysis` - Numerical analysis
- `pymoo` - Multi-objective optimization
- `simpy` - Process simulation
- `pymc` - Bayesian parameter estimation
- `scikit-learn` - Process modeling
- `stable-baselines3` - Real-time control optimization
- `pufferlib` - High-throughput vectorized RL training for control policies
- `matplotlib` - Process diagrams
- `scientific-visualization` - Publication-quality & interactive visualization
- `fluidsim` - Fluid dynamics simulation
- `openpiv` - Validate simulated mixing against measured velocity fields
- `scientific-writing` - Engineering reports

**Starting prompt**:

```text
Use the sympy, uncertainty-and-units, pymc, scikit-learn, pymoo, simpy, and
scientific-writing skills.

Goal: a reactor design and operating envelope, with the uncertainty in the
kinetics carried all the way through to the economics.
Criteria: dimensional consistency checked on every equation; kinetic
parameters reported as posteriors, not point estimates.
Deliver: validated model, Pareto front, recommended operating point, dynamic
simulation, control design, economics with sensitivity, safety analysis.
Report: propagate the kinetic posterior into the yield prediction. A design
optimized against a point estimate can sit on a cliff edge — show whether it
is robust across the posterior.
```

**Workflow**:

```text
Step 1: Define reaction system and kinetics
- Chemical reaction: A + B → C + D
- Use Sympy to define symbolic rate equations:
  * Arrhenius equation: k = A × exp(-Ea/RT)
  * Rate law: r = k × [A]^α × [B]^β
- Define material and energy balances symbolically
- Include equilibrium constants and thermodynamics
- Account for side reactions and byproducts

Step 2: Develop reactor model
- Select reactor type: CSTR, PFR, batch, or semi-batch
- Write conservation equations:
  * Mass balance: dC/dt = (F_in × C_in - F_out × C)/V + r
  * Energy balance: ρCp × dT/dt = Q - ΔH_rxn × r × V
  * Momentum balance (pressure drop)
- Include heat transfer correlations
- Model mixing and mass transfer limitations

Step 3: Parameter estimation with PyMC
- Load experimental data from pilot reactor
- Bayesian inference to estimate kinetic parameters:
  * Pre-exponential factor (A)
  * Activation energy (Ea)
  * Reaction orders (α, β)
- Reparameterize Arrhenius around a reference temperature —
  k = k_ref · exp[−(Ea/R)(1/T − 1/T_ref)] — before sampling. In the raw form ln A
  and Ea are almost perfectly correlated (the compensation effect), which produces a
  narrow diagonal ridge that MCMC samples badly and that makes both marginals look
  uninformative even when k(T) is well determined
- Use uncertainty-and-units to confirm the rate law is dimensionally consistent: the
  units of A depend on the reaction orders, so a fitted α or β changes what A even
  means, and this is a routine source of silent error
- Incorporate literature priors, and check they are compatible with the data via a
  prior predictive check
- Sample with PyMC; check R-hat, effective sample size, and divergences
- Report posteriors and the parameter correlation structure, not just marginal
  credible intervals — the correlation is what determines the uncertainty in any
  prediction you make downstream

Step 4: Model validation
- Simulate reactor with estimated parameters using scipy.integrate
- Compare predictions with experimental data
- Calculate goodness of fit (R², RMSE)
- Perform sensitivity analysis:
  * Which parameters most affect yield?
  * Identify critical operating conditions
- Refine model if needed

Step 5: Machine learning surrogate model
- Train a fast surrogate with scikit-learn
- Generate training data from the detailed model over a space-filling design (Latin
  hypercube or Sobol), not a grid — grids waste runs and leave diagonal gaps
- Features: T, P, residence time, feed composition, catalyst loading
- Target: yield, selectivity, conversion
- Prefer Gaussian Process Regression here specifically because it returns a
  predictive variance; the optimizer in Step 7 will push toward the design-space
  edges, and you need the surrogate to say when it is extrapolating
- Validate on held-out points, reporting R² and max error. A surrogate is only valid
  inside its sampled envelope; the optimizer must be constrained to that envelope, or
  every optimum it finds will sit in a region the surrogate never saw
- Re-verify the final optimum against the full mechanistic model, never against the
  surrogate alone

Step 6: Single-objective optimization
- Maximize yield with scipy.optimize:
  * Decision variables: T, P, feed ratio, residence time
  * Objective: maximize Y = (moles C produced) / (moles A fed)
  * Constraints:
    - Temperature: 300 K ≤ T ≤ 500 K (safety)
    - Pressure: 1 bar ≤ P ≤ 50 bar (equipment limits)
    - Residence time: 1 min ≤ τ ≤ 60 min
    - Conversion: X_A ≥ 90%
- Use Sequential Least Squares Programming (SLSQP)
- Identify optimal operating point

Step 7: Multi-objective optimization with PyMOO
- Competing objectives:
  1. Maximize product yield
  2. Minimize energy consumption (heating/cooling)
  3. Minimize operating cost (raw materials, utilities)
  4. Maximize reactor productivity (throughput)
- Constraints:
  - Safety: temperature and pressure limits
  - Environmental: waste production limits
  - Economic: minimum profitability
- Run NSGA-II or NSGA-III
- Generate Pareto front of optimal solutions
- Select operating point based on preferences

Step 8: Dynamic process simulation with SimPy
- Model complete plant:
  * Reactors, separators, heat exchangers
  * Pumps, compressors, valves
  * Storage tanks and buffers
- Simulate startup, steady-state, and shutdown
- Include disturbances:
  * Feed composition variations
  * Equipment failures
  * Demand fluctuations
- Evaluate dynamic stability
- Calculate time to steady state

Step 9: Control system design
- Design feedback control loops:
  * Temperature control (PID controller)
  * Pressure control
  * Flow control
  * Level control
- Tune PID parameters using Ziegler-Nichols or optimization
- Implement cascade control for improved performance
- Add feedforward control for disturbance rejection

Step 10: Reinforcement learning for advanced control
- Use Stable-Baselines3 to train RL agent:
  * Environment: reactor simulation (SimPy-based)
  * State: T, P, concentrations, flow rates
  * Actions: adjust setpoints, flow rates, heating/cooling
  * Reward: +yield -energy cost -deviation from setpoint
- Train PPO or TD3 agent
- Compare with conventional PID control
- Evaluate performance under disturbances
- Implement model-free adaptive control

Step 11: Economic analysis
- Calculate capital costs (CAPEX):
  * Reactor vessel cost (function of size, pressure rating)
  * Heat exchanger costs
  * Pumps and instrumentation
  * Installation costs
- Calculate operating costs (OPEX):
  * Raw materials (A, B, catalyst)
  * Utilities (steam, cooling water, electricity)
  * Labor and maintenance
- Revenue from product sales
- Calculate economic metrics:
  * Net present value (NPV)
  * Internal rate of return (IRR)
  * Payback period
  * Levelized cost of production

Step 12: Safety analysis
- Identify hazards:
  * Exothermic runaway reactions
  * Pressure buildup
  * Toxic or flammable materials
- Perform HAZOP-style analysis
- Calculate safe operating limits:
  * Maximum temperature of synthesis reaction (MTSR)
  * Adiabatic temperature rise
  * Relief valve sizing
- Design emergency shutdown systems
- Implement safety interlocks

Step 13: Uncertainty quantification
- Propagate parameter uncertainties from PyMC:
  * How does kinetic parameter uncertainty affect yield?
  * Monte Carlo simulation with parameter distributions
- Evaluate robustness of optimal design
- Calculate confidence intervals on economic metrics
- Identify critical uncertainties for further study

Step 14: Generate comprehensive engineering report
- Executive summary of project objectives and results
- Process flow diagram (PFD) with material and energy streams
- Reaction kinetics and model equations
- Parameter estimation results with uncertainties
- Optimization results:
  * Pareto front for multi-objective optimization
  * Recommended operating conditions
  * Trade-off analysis
- Dynamic simulation results (startup curves, response to disturbances)
- Control system design and tuning
- Economic analysis with sensitivity to key assumptions
- Safety analysis and hazard mitigation
- Scale-up considerations:
  * Pilot to commercial scale
  * Heat and mass transfer limitations
  * Equipment sizing
- Recommendations:
  * Optimal reactor design (size, type, materials of construction)
  * Operating conditions for maximum profitability
  * Control strategy
  * Further experimental studies needed
- Technical drawings and P&ID (piping and instrumentation diagram)
- Export as professional engineering report (PDF)

Expected Output:
- Validated reactor model with parameter uncertainties
- Optimal reactor design and operating conditions
- Pareto-optimal solutions for multi-objective optimization
- Dynamic process simulation results
- Advanced control strategies (RL-based)
- Economic feasibility analysis
- Safety assessment
- Comprehensive chemical engineering design report
```

---

## Scientific Illustration & Visual Communication

### Example 22: Creating Publication-Ready Scientific Figures

**Objective**: Generate, audit, and package scientific illustrations, diagrams, and graphical abstracts while preserving evidence provenance and current venue requirements.

**Disciplines**: scientific communication · visual design · accessibility · research integrity

**Skills Used**:
- `generate-image` - AI image generation and editing
- `matplotlib` - Data visualization
- `scientific-visualization` - Best practices
- `scientific-schematics` - Scientific diagrams
- `scientific-writing` - Figure caption creation
- `scientific-slides` - Presentation materials
- `latex-posters` - Conference posters
- `pptx-posters` - Macro-free `.pptx` posters from approved local manifests

**Starting prompt**:

```text
Use the scientific-visualization, scientific-schematics, matplotlib,
generate-image, and scientific-writing skills.

Goal: a figure package for submission, plus a talk version of the same figures.
Criteria: data figures come from the data, always. Generated imagery is
allowed only for conceptual illustration and must be labelled as such.
Deliver: numbered figures at venue-required resolution, captions, a slide deck,
and a provenance file recording every model, prompt, and edit.
Report: which figures are data-derived and which are illustrative, per figure.
Do not: use image generation to render, alter, or extend anything that
represents observed data — no invented error bars, scale bars, or micrographs.
```

**Workflow**:

```text
Step 1: Plan visual communication strategy
- Separate data figures from conceptual/AI-generated illustrations; never use image
  generation to invent observations, labels, scale, or quantitative evidence
- Confirm authorization before sending any source image or unpublished/sensitive
  content to an external image service
- Identify key concepts that need visual representation:
  * Experimental workflow diagrams
  * Molecular structures and interactions
  * Data visualization (handled by matplotlib)
  * Conceptual illustrations for mechanisms
  * Graphical abstract for paper summary
- Determine appropriate style for target journal/audience
- Sketch rough layouts for each figure

Step 2: Generate experimental workflow diagram
- Use generate-image skill with detailed prompt:
  "Scientific illustration showing a step-by-step experimental 
  workflow for CRISPR gene editing: (1) guide RNA design at computer,
  (2) cell culture in petri dish, (3) electroporation device,
  (4) selection with antibiotics, (5) sequencing validation.
  Clean, professional style with numbered steps, white background,
  suitable for scientific publication."
- Save as workflow_diagram.png
- Record model, prompt, output, edits, and source/permission metadata; have a domain
  expert verify every depicted scientific detail

Step 3: Create molecular interaction schematic
- Generate detailed molecular visualization:
  "Scientific diagram of protein-ligand binding mechanism:
  show receptor protein (blue ribbon structure) with binding pocket,
  small molecule ligand (ball-and-stick, orange) approaching,
  key hydrogen bonds indicated with dashed lines, water molecules
  in binding site. Professional biochemistry illustration style,
  clean white background, publication quality."
- Generate multiple versions with different angles/styles
- Select best representation

Step 4: Edit existing figures for consistency
- Use the generate-image skill's supported host interface to request a background
  or palette edit only after confirming permission to process the source image
- Standardize color schemes across all figures
- Preserve the untouched original and record each transformation
- Verify current journal requirements directly rather than assuming an edit makes
  the figure compliant

Step 5: Generate graphical abstract
- Create comprehensive visual summary:
  "Graphical abstract for cancer immunotherapy paper: left side
  shows tumor cells (irregular shapes, red) being attacked by
  T cells (round, blue). Center shows the drug molecule structure.
  Right side shows healthy tissue (green). Arrow flow from left
  to right indicating treatment progression. Modern, clean style
  with minimal text, high contrast, suitable for journal TOC."
- Ensure dimensions meet journal requirements
- Iterate to highlight key findings

Step 6: Create conceptual mechanism illustrations
- Generate mechanism diagrams:
  "Scientific illustration of enzyme catalysis mechanism:
  Show substrate entering active site (step 1), transition state
  formation with electron movement arrows (step 2), product
  release (step 3). Use standard biochemistry notation,
  curved arrows for electron movement, clear labeling."
- Generate alternative representations for supplementary materials

Step 7: Produce presentation-ready figures
- Create high-impact visuals for talks:
  "Eye-catching scientific illustration of DNA double helix
  unwinding during replication, with DNA polymerase (large
  green structure) adding nucleotides. Dynamic composition,
  vibrant but professional colors, dark background for
  presentation slides."
- Adjust style for poster vs slide format
- Create versions at different resolutions

Step 8: Generate figure panels for multi-part figures
- Create consistent series of related images:
  "Panel A: Normal cell with intact membrane (green outline)
  Panel B: Cell under oxidative stress with damaged membrane
  Panel C: Cell treated with antioxidant, membrane recovering
  Consistent style across all panels, same scale, white background,
  scientific illustration style suitable for publication."
- Ensure visual consistency across panels
- Annotate with panel labels

Step 9: Edit for accessibility
- Use redundant encodings, labels, patterns, and an audited contrast-aware palette
- If an authorized AI edit changes colors, compare it against the original and
  verify that no scientific content or spatial relationship changed
- Add patterns or textures for additional differentiation
- Run contrast checks and manual accessibility review; do not claim that palette
  selection alone establishes accessibility

Step 10: Create supplementary visual materials
- Generate additional context figures:
  "Anatomical diagram showing location of pancreatic islets
  within the pancreas, cross-section view with labeled structures:
  alpha cells, beta cells, blood vessels. Medical illustration
  style, educational, suitable for supplementary materials."
- Create protocol flowcharts and decision trees
- Generate equipment setup diagrams

Step 11: Compile figure legends and captions
- Use scientific-writing skill to create descriptions:
  * Figure number and title
  * Detailed description of what is shown
  * Explanation of symbols, colors, and abbreviations
  * Scale bars and measurement units
  * Statistical information if applicable
- Format according to journal guidelines
- Map factual and numerical caption claims to verified evidence IDs

Step 12: Assemble final publication package
- Organize all figures in publication order
- Verify the target venue's current dimensions, resolution, color-space, font,
  accessibility, and submission requirements before export
- Compile into PDF:
  * Title page with graphical abstract
  * All figures with captions
  * Supplementary figures section
- Create separate folder with individual figure files
- Document all generation prompts for reproducibility
- For a PowerPoint poster, build an author-approved local content/asset manifest,
  validate hashes, provenance, printer rules, reading order, and approval hash,
  then generate and inspect a macro-free `.pptx`; final review/export is manual

Expected Output:
- Complete set of source-traceable, human-reviewed scientific illustrations
- Graphical abstract for table of contents
- Mechanism diagrams and workflow figures
- Edited versions checked against current journal guidance
- Accessibility-reviewed figure versions with redundant encodings
- Figure package with captions and metadata
- Documentation of prompts used for reproducibility
```

---

## Quantum Computing for Chemistry

### Example 23: Variational Quantum Eigensolver for Molecular Ground States

**Objective**: Build and benchmark a reproducible VQE workflow for a small, classically verifiable molecular ground-state problem before considering larger chemistry applications.

**Disciplines**: quantum information · quantum chemistry · numerical optimization · metrology

**Skills Used**:
- `qiskit` - Qiskit Nature mapping, V2 primitives, target-aware transpilation, simulation, and IBM Runtime execution
- `uncertainty-and-units` - Hartree/eV/kcal·mol⁻¹ conversions and shot-noise error budgets
- `matplotlib` - Energy landscape visualization
- `scientific-visualization` - Publication figures
- `scientific-writing` - Quantum chemistry reports

**Starting prompt**:

```text
Use the qiskit, uncertainty-and-units, and scientific-writing skills.

Goal: a VQE result I can trust, on a system where I already know the answer.
Criteria: exact diagonalization baseline first; define the accuracy target
(chemical accuracy, 1.6 mHa) before running anything noisy.
Deliver: convergence history, ideal / noisy / hardware energies side by side,
resource counts, and every version pin and seed.
Report: shot-noise uncertainty on every energy, in consistent units. Do not
convert between Hartree and eV without showing the factor.
Say plainly where mitigation did not help — that is a useful result too.
```

**Workflow**:

```text
Step 1: Define molecular system
- Start with H2 or another small molecule that can be solved exactly
- Record geometry, units, charge, spin, and basis set
- Choose any active-space and freeze-core approximations explicitly
- Calculate spin-orbital and qubit counts after all reductions

Step 2: Construct molecular Hamiltonian
- Use the pinned Qiskit Nature and PySCF integration
- Generate the second-quantized electronic Hamiltonian
- Apply a current mapper such as Jordan-Wigner directly
- Record coefficients, Pauli-term count, nuclear repulsion, and mapper

Step 3: Establish classical and exact-quantum baselines
- Compute Hartree-Fock and exact diagonalization results where tractable
- Run StatevectorEstimator with the same mapped Hamiltonian
- Confirm energy conventions and avoid adding nuclear repulsion twice
- Define an accuracy target before using noisy simulation or hardware

Step 4: Design and validate the ansatz
- Compare a chemistry-motivated ansatz with a shallow hardware-efficient ansatz
- Record parameter order, initial state, depth, and two-qubit operations
- Use a current optimizer object and bounded evaluation budget
- Verify the small-circuit state and expectation values locally

Step 5: Implement VQE with V2 primitives
- Use StatevectorEstimator for the ideal development loop
- Pass parameter arrays through Estimator PUBs
- Compile the parameterized circuit once rather than once per iteration
- Save convergence history, primitive metadata, and package versions

Step 6: Progress from noise model to IBM QPU
- Build a recorded Aer/fake-backend baseline
- Select an accessible BackendV2 by width and target capabilities
- Generate an ISA circuit and apply its layout to every observable
- Use job or batch mode; use sessions only on eligible plans

Step 7: Evaluate mitigation rather than assuming benefit
- Compare Runtime Estimator resilience levels 0 and 1 or 2
- Record requested precision, realized uncertainty, and workload overhead
- Compare both results with the exact small-system baseline
- Report cases where mitigation does not improve the estimate

Step 8: Analyze resources and reproducibility
- Report logical and ISA depth, layout, and native two-qubit operations
- Store QPY circuits, backend, job IDs, seeds, options, and version pins
- Separate ideal, modeled-noise, and hardware results
- Quantify total optimizer evaluations and QPU usage

Step 9: Generate quantum chemistry report
- Energy convergence plots
- Circuit diagrams and ansatz visualizations
- Comparison with exact and classical chemistry baselines
- Accuracy, uncertainty, and execution-cost accounting
- Limitations on extrapolating from small molecules to applications
- Publication-quality figures
- Export comprehensive report

Expected Output:
- Reproducible ideal, noisy, and optional hardware VQE results
- Logical and ISA circuits with mapped observables
- Comparison with exact and classical baselines
- Mitigation A/B analysis with uncertainty and cost
- Versioned quantum chemistry workflow report
```

---

## Open Quantum Systems & Cross-Framework Benchmarking

### Example 24: Dissipative Dynamics of an Excitonic Energy-Transfer Complex

**Objective**: Model coherent energy transfer in a light-harvesting complex coupled to a vibrational bath, and check whether a variational quantum algorithm on the same Hamiltonian reproduces the classically computed result.

**Disciplines**: quantum optics · biophysics · physical chemistry · quantum computing · numerical analysis

Photosynthetic energy transfer sits between disciplines: the system is biological, the
Hamiltonian is physics, the parameters come from spectroscopy, and the interesting
question — whether coherence survives long enough to matter at physiological
temperature — is answerable only by simulating an open quantum system properly. It is
also a well-characterized benchmark, which makes it a good place to test whether a
quantum-computing approach reproduces a known answer before trusting it on an unknown one.

**Skills Used**:
- `qutip` - Open-system dynamics: Lindblad, Redfield, HEOM-style hierarchies, and steady states
- `pennylane` - Variational algorithms and differentiable quantum programming on the same Hamiltonian
- `cirq` - Independent circuit construction and compilation for a second hardware target
- `qiskit` - Third framework for cross-checking transpiled circuit depth and results
- `sympy` - Symbolic derivation of the system-bath Hamiltonian and rate expressions
- `uncertainty-and-units` - cm⁻¹, meV, fs, and kT conversions; the whole problem turns on these
- `statistical-analysis` - Fitting, convergence testing, and comparison statistics
- `matplotlib` / `scientific-visualization` - Population dynamics and coherence plots
- `scientific-writing` - Report with the classical baseline foregrounded

**Starting prompt**:

```text
Use the qutip, pennylane, cirq, qiskit, sympy, uncertainty-and-units, and
scientific-writing skills.

Goal: population dynamics and coherence lifetimes for this complex, plus an
answer to whether a VQE on the same Hamiltonian matches the classical result.
Criteria: convergence in bath-hierarchy depth and time step must be
demonstrated, not assumed. Every energy in cm^-1 and every time in fs, with
conversions shown.
Deliver: population traces, coherence decay with timescales, a
temperature/reorganization-energy sweep, and a classical-versus-quantum
comparison table with circuit resource counts.
Report: state which master equation you used and why it is valid in this
coupling and temperature regime — that choice determines the answer.
Do not: present the quantum-hardware result as an advantage. It is a
correctness check against a classically solvable case.
```

**Workflow**:

```text
Step 1: Assemble the Hamiltonian and get the units right first
- Build the excitonic Hamiltonian: site energies on the diagonal, electronic
  couplings off-diagonal, conventionally in cm⁻¹
- Use sympy to derive the system-bath coupling and the spectral density expression
  symbolically before committing to numbers
- Use uncertainty-and-units for every conversion. This problem is unforgiving about
  it: site energies in cm⁻¹, couplings sometimes in meV, dynamics in fs, and thermal
  energy as kT — at 300 K, kT ≈ 208 cm⁻¹, which is comparable to typical
  reorganization energies. Whether coherence survives depends on that comparison, so
  a botched conversion does not produce a slightly wrong answer, it produces the
  wrong physics
- Record the spectroscopic source for every parameter and its uncertainty

Step 2: Choose the master equation deliberately
- The regime decides the method, and the method decides the answer:
  * Secular Lindblad is fast and guarantees positivity, but assumes weak coupling and
    well-separated timescales — it will underestimate coherence lifetimes here
  * Redfield captures the bath structure better but can produce unphysical negative
    populations outside its validity range
  * A hierarchical (HEOM-style) treatment is appropriate when reorganization energy
    is comparable to electronic coupling, which is the interesting case
- State the regime, state the choice, and run at least two methods so the reader can
  see how much the conclusion depends on it

Step 3: Simulate with QuTiP
- Construct the Liouvillian and propagate the density matrix
- Demonstrate convergence: hierarchy depth (or bath-mode truncation), time step, and
  Hilbert-space truncation each swept until the observable stops moving
- Verify the physics at every step — trace preservation, positivity of the density
  matrix, and relaxation to the correct thermal state at long times. A simulation
  that does not thermalize correctly is wrong regardless of how the early dynamics look
- Extract site populations, exciton populations, and inter-site coherences

Step 4: Sweep the parameters that matter
- Vary temperature, reorganization energy, and bath correlation time
- Report coherence lifetime as a function of each. The scientifically honest framing:
  coherence in these systems is short-lived at physiological temperature, and the
  question is whether it is long enough to affect transfer efficiency — quantify the
  efficiency change, do not just show that oscillations exist
- Propagate the spectroscopic parameter uncertainties into the lifetime estimate

Step 5: Set up the same Hamiltonian as a variational problem
- Map the electronic Hamiltonian to qubits and build a VQE for its ground state
- Implement in PennyLane, using its autodifferentiation for analytic parameter-shift
  gradients rather than finite differences
- Rebuild the same ansatz in Cirq and in Qiskit. This is not redundancy: the three
  compile to different native gate sets and different circuit depths, and comparing
  their transpiled two-qubit counts tells you what the circuit actually costs on a
  given hardware target

Step 6: Validate against the classical answer
- Diagonalize the same Hamiltonian exactly — the system is small, so the true ground
  state is available
- Compare VQE energies from all three frameworks to it, in consistent units, with
  shot-noise uncertainties
- Report circuit depth, two-qubit gate count, and optimizer evaluations per
  framework. Any discrepancy between frameworks on the same Hamiltonian is a bug in
  one of the implementations, and finding it is the point of running three

Step 7: Report
- Lead with the classical result; the quantum implementation is a benchmark against it
- Give the master-equation choice, the convergence evidence, and the parameter
  uncertainties before any conclusion about coherence
- State the limitation plainly: a handful of sites solved exactly on a classical
  machine says nothing about scaling, and no part of this demonstrates quantum advantage

Expected Output:
- Converged open-system dynamics with population and coherence traces
- Coherence lifetime versus temperature and reorganization energy, with uncertainty
- A comparison of at least two master equations on the same system
- VQE ground-state energies from PennyLane, Cirq, and Qiskit against exact diagonalization
- Circuit resource counts per framework and per hardware target
```

---

## Research Grant Writing

### Example 25: NIH R01 Grant Proposal Development

**Objective**: Develop a comprehensive research grant proposal with literature review, specific aims, and budget justification.

**Disciplines**: research strategy · biostatistics · experimental design · scientific writing · research administration

**Skills Used**:
- `database-lookup` - Query ClinicalTrials.gov for preliminary data context
- `paper-lookup` - Search PubMed, OpenAlex for literature and citations
- `research-grants` - Grant writing templates and guidelines
- `scientific-literature-review` - Systematic literature analysis
- `hypothesis-generation` - Scientific hypothesis development
- `experimental-design` - Design, randomization, blinding, and controls for each aim
- `statistical-power` - A priori power analysis and sample-size justification
- `scientific-writing` - Technical writing
- `scientific-critical-thinking` - Research design
- `peer-review` - Self-assessment against the review criteria before submission
- `citation-management` - Reference formatting

**Starting prompt**:

```text
Use the research-grants, scientific-literature-review, hypothesis-generation,
experimental-design, statistical-power, scientific-writing, peer-review, and
citation-management skills.

Goal: a complete R01 package, plus an honest internal review of it.
Criteria: each aim states its hypothesis, its design, its power analysis, and
what result would refute it. Aims must not be contingent on each other.
Deliver: Specific Aims page, Research Strategy, budget (xlsx), timeline,
rigor and reproducibility section, bibliography.
Report: run the peer-review skill against the FOA's review criteria and give
me the critique before I read the draft — including the weaknesses you would
raise if you were reviewer 3.
```

**Workflow**:

```text
Step 1: Define research question and significance
- Use hypothesis-generation skill to refine research questions
- Identify knowledge gaps in the field
- Articulate significance and innovation
- Define measurable outcomes

Step 2: Comprehensive literature review
- Search PubMed for relevant publications (last 10 years)
- Query OpenAlex for citation networks
- Identify key papers and review articles
- Use scientific-literature-review skill to synthesize findings
- Identify gaps that proposal will address

Step 3: Develop specific aims
- Aim 1: Mechanistic studies (hypothesis-driven)
- Aim 2: Translational applications
- Aim 3: Validation and clinical relevance
- Ensure aims are interdependent but not contingent
- Define success criteria for each aim

Step 4: Design research approach
- Use experimental-design to choose the design for each aim — factorial, blocked,
  crossover, or randomized — and to specify randomization, blinding, and the unit of
  randomization (the unit is where most designs quietly go wrong)
- Use scientific-critical-thinking to stress-test the logic connecting aims to claims
- Include positive and negative controls, and state what each one rules out
- Plan the statistical analysis before the data exist, including how missing data and
  multiplicity across aims will be handled
- Identify potential pitfalls and pre-specify the alternative approach for each

Step 5: Preliminary data compilation
- Gather existing data supporting hypothesis
- Search ClinicalTrials.gov for relevant prior work
- Create figures showing preliminary results
- Quantify feasibility evidence

Step 6: Innovation and significance sections
- Articulate what is novel about approach
- Compare to existing methods/knowledge
- Explain expected impact on field
- Address NIH mission alignment

Step 7: Timeline and milestones
- Create Gantt chart for 5-year project
- Define quarterly milestones
- Identify go/no-go decision points
- Plan for personnel and resource allocation

Step 8: Budget development
- Calculate personnel costs (PI, postdocs, students)
- Equipment and supplies estimates
- Core facility usage costs
- Travel and publication costs
- Indirect cost calculation

Step 9: Rigor and reproducibility
- Address biological variables (sex as a biological variable, age, strain) as factors
  in the design, not as a sentence in the text — reviewers check for the difference
- Run a priori power analysis with statistical-power for each aim's primary endpoint:
  state the effect size, its source, alpha, target power, and the resulting n. Power
  computed after the fact from an observed effect is not a power analysis, and adding
  it will cost credibility
- Where the effect size is genuinely unknown, present a power curve across a
  plausible range and name the minimum detectable effect instead of inventing one
- Data management and sharing plan
- Authentication of key resources

Step 10: Format and compile
- Use research-grants templates for NIH format
- Apply citation-management for references
- Create biosketch and facilities sections
- Generate PDF with proper formatting
- Check page limits and formatting requirements

Step 11: Review and revision
- Run the peer-review skill against the proposal, scored on the actual review
  criteria for this mechanism (significance, investigators, innovation, approach,
  environment) rather than on general writing quality
- Ask it for the strongest objection to each aim, not a summary of strengths
- Check for logical flow and clarity
- Verify alignment with FOA requirements, page limits, and formatting rules

Step 12: Final deliverables
- Specific Aims page (1 page)
- Research Strategy (12 pages)
- Bibliography
- Budget and justification
- Biosketches
- Letters of support
- Data management plan
- Human subjects/vertebrate animals sections (if applicable)

Expected Output:
- Complete NIH R01 grant proposal
- Literature review summary
- Budget spreadsheet with justification
- Timeline and milestone chart
- All required supplementary documents
- Properly formatted PDF ready for submission
```

---

## Geospatial & Earth Observation

### Example 27: Remote Sensing for Environmental Monitoring

**Objective**: Combine satellite imagery and vector data to map land-cover change and quantify environmental drivers across a watershed.

**Disciplines**: remote sensing · hydrology · landscape ecology · spatial statistics · machine learning

**Skills Used**:
- `geomaster` - Remote sensing, GIS, and earth-observation workflows
- `geopandas` - Vector data (shapefiles, GeoJSON) and spatial joins
- `zarr-python` - Chunked N-D arrays for large raster/time stacks
- `dask` - Parallel/out-of-core processing of image cubes
- `scikit-learn` - Land-cover classification
- `timesfm-forecasting` - Project index time series forward where a baseline is needed
- `statistical-analysis` - Trend and correlation testing
- `uncertainty-and-units` - Reflectance scaling, area units, and change-area intervals
- `matplotlib` - Mapping and charts
- `scientific-visualization` - Publication-quality & interactive visualization

**Starting prompt**:

```text
Use the geomaster, geopandas, zarr-python, dask, scikit-learn,
statistical-analysis, and uncertainty-and-units skills.

Goal: how much land cover changed in this watershed, where, and what covaries
with it — with an area estimate that has a confidence interval.
Criteria: accuracy assessed on an independent probability sample, not on
training pixels. Reproject everything to one equal-area CRS before measuring area.
Deliver: classified maps per date, a change matrix, area estimates with CIs,
per-sub-catchment statistics, and driver correlations.
Report: pixel-counted area is biased by classification error — give the
error-adjusted area estimate and say which estimator you used.
```

**Workflow**:

```text
Step 1: Acquire and stack imagery
- Use geomaster to pull Sentinel-2/Landsat scenes for the area and time range
- Compute spectral indices (NDVI, NDWI, NBR) per scene
- Store the multi-date raster cube as a chunked Zarr array (zarr-python)

Step 2: Prepare vector layers with GeoPandas
- Load watershed boundaries, land parcels, and road networks
- Reproject all layers to a common CRS
- Clip rasters to the area of interest and rasterize key vector masks

Step 3: Scale processing with Dask
- Lazily load the Zarr cube as Dask arrays
- Map index calculations and cloud masking across chunks in parallel

Step 4: Land-cover classification
- Sample labeled training pixels (forest, cropland, water, urban)
- Train a Random Forest classifier with scikit-learn on spectral + index features
- Validate on an independent probability sample of reference points, not on held-out
  pixels from the same polygons. Neighbouring pixels are spatially autocorrelated, so
  a random pixel split reports an accuracy that will not hold on new ground — use
  spatial block cross-validation for model selection
- Report the confusion matrix, per-class user's and producer's accuracy, and overall
  accuracy. Prefer these to kappa, which is largely redundant with overall accuracy
  and has been argued out of favour in the remote-sensing literature

Step 5: Change detection and zonal statistics
- Compute land-cover transitions between years
- Do not report change area by counting classified pixels. Classification errors are
  asymmetric and change is rare, so pixel counting is badly biased — a 5% error rate
  on a stable class can swamp a 2% real change. Use a stratified estimator with the
  reference sample to produce error-adjusted area estimates with confidence intervals
- Reproject to an equal-area CRS before computing any area; measuring hectares in a
  Web Mercator projection introduces a latitude-dependent error of tens of percent
- Use GeoPandas zonal stats to summarize change per sub-catchment
- Correlate change with covariates (slope, precipitation) via statistical-analysis,
  accounting for spatial autocorrelation — ordinary regression on spatial data
  understates standard errors substantially

Step 6: Generate report
- Time-series maps, change matrices, and trend plots
- Per-zone summary tables and interpretation
- Export publication-quality figures and a PDF report

Expected Output:
- Analysis-ready Zarr raster cube and classified land-cover maps
- Quantified land-cover change with per-zone statistics
- Environmental driver analysis and geospatial report
```

---

## Research Ideation & Study Planning

### Example 34: From Idea to a Powered, Well-Designed Study

**Objective**: Move from open-ended ideation to transparent candidate hypotheses and a statistically powered study plan without treating any candidate as validated or automatically selecting a winner.

**Disciplines**: philosophy of science · experimental design · biostatistics · domain-specific reasoning

**Starting prompt**:

```text
Use the scientific-brainstorming, consciousness-council, what-if-oracle,
hypothesis-generation, experimental-design, and statistical-power skills.

Goal: a preregistration-ready plan built from candidates I can still argue with.
Criteria: for every hypothesis, a rival that predicts something different, and
the observation that would distinguish them. A hypothesis with no rival that
makes a different prediction is not yet testable.
Deliver: candidate set with rivals, discriminating predictions, chosen design
with randomization and blocking, power analysis, and analysis plan.
Report: keep the human decisions visible — which directions were set aside and
why. Do not silently rank or eliminate candidates on my behalf.
```

**Skills Used**:
- `scientific-brainstorming` - Open-ended ideation and gap-finding
- `consciousness-council` - Multi-perspective deliberation on directions
- `what-if-oracle` - Structured scenario/branch analysis
- `hypothesis-generation` - Formalize evidence-bounded candidates and rival predictions
- `hypogenic` - Produce candidate textual patterns from labeled text datasets
- `experimental-design` - Choose design, randomization, and blocking
- `statistical-power` - Sample size, MDE, and power curves

**Workflow**:

```text
Step 1: Diverge — generate ideas
- Use scientific-brainstorming to explore the problem space and interdisciplinary links
- Run a consciousness-council deliberation to weigh competing research directions

Step 2: Stress-test directions
- Use what-if-oracle to explore best/likely/worst/contrarian scenarios for top ideas
- Record assumptions, objections, vetoes, uncertainty, and reasons a human team
  retains, revises, or sets aside a direction

Step 3: Formalize hypotheses
- Convert the chosen direction into testable hypotheses with hypothesis-generation
- If an approved labeled text dataset exists, use HypoGeniC to produce candidate
  textual hypotheses and held-out task statistics
- Do not treat HypoGeniC accuracy as truth, causal evidence, novelty, or scientific
  validation, and do not automatically score, rank, select, accept, or reject hypotheses

Step 4: Design the study
- Use experimental-design to select a design (factorial, RCT, block, crossover),
  define randomization, blocking, and treatment combinations

Step 5: Power and sample size
- Use statistical-power for a priori power analysis, minimum detectable effect,
  and power curves across plausible effect sizes

Step 6: Deliverable
- A human-reviewed, preregistration-ready plan: candidate/rival set, discriminating
  predictions, design diagram, analysis plan, oversight gates, and justified sample size

Expected Output:
- A documented set of candidate hypotheses, rivals, assumptions, and human decisions
- A concrete experimental design with randomization/blocking
- Power analysis and sample-size justification
```

---

## Literature & Knowledge Management

### Example 35: Systematic Literature Review and Research Knowledge Base

**Objective**: Run a multi-source literature search, ingest and organize sources, and synthesize a cited, well-managed review.

**Disciplines**: evidence synthesis · information science · research methodology · science communication

**Skills Used**:
- `research-lookup` - Routed current-research search (web/deep/academic)
- `paper-lookup` - 18 scholarly APIs for literature, citation edges, text-mined entities, deposited artifacts, journals, and organization IDs
- `exa-search` - Semantic web search tuned for technical content
- `parallel-web` - Academic-focused web search/fetch and enrichment
- `bgpt-paper-search` - Structured experimental data extracted from papers
- `paperzilla` - Canonical papers and project recommendations
- `liteparse` - Local PDF/Office parsing with layout/bounding boxes
- `markitdown` - Convert documents to Markdown
- `open-notebook` - Organize sources into AI research notebooks
- `pyzotero` - Manage a Zotero reference library
- `scholar-evaluation` - Qualitative, low-stakes developmental review of works
- `dhdna-profiler` - Optional, non-evaluative characterization of reasoning style in a text
- `citation-management` - Reference formatting
- `scientific-literature-review` - Systematic synthesis

**Starting prompt**:

```text
Use the research-lookup, paper-lookup, exa-search, liteparse, markitdown,
open-notebook, pyzotero, citation-management, and scientific-literature-review skills.

Goal: a systematic review with an auditable search, not a summary of whatever
came back first.
Criteria: record the exact query, database, filters, and date for every search;
state inclusion and exclusion criteria before screening; log every exclusion
with its reason.
Deliver: search log, PRISMA-style counts, evidence table (xlsx), synthesis
with themes and conflicts, and a de-duplicated Zotero library.
Report: which databases returned nothing, and where coverage is thin. A silent
gap reads as "no evidence exists" when it may mean "not indexed here".
```

**Workflow**:

```text
Step 1: Multi-source search
- Write the protocol first: question, inclusion and exclusion criteria, and the
  databases to be searched. A search designed after seeing results is a narrative
  review wearing a systematic review's clothes
- Use research-lookup to route queries and paper-lookup for the bibliographic
  databases; broaden with exa-search and parallel-web for grey literature and
  technical sources the indexes miss
- Record every query verbatim with its database, filters, result count, and access
  date. This log is what makes the review reproducible, and it cannot be
  reconstructed afterwards
- Pull structured study fields (sample sizes, methods, outcomes) via bgpt-paper-search
- Surface canonical references and recommendations with paperzilla
- Search preprint servers deliberately and label preprints as unrefereed. Restricting
  to published work imports publication bias, since null results are published less
- Use companion APIs when the protocol calls for them: OpenCitations for citation
  edges; PubTator3 for text-mined entity candidates; Zenodo, Figshare, and BioStudies
  for deposited data/software/supplements; ROR for affiliation IDs; DOAJ for journal
  inclusion. Verify entity mentions in source text and preserve artifact versions
- Read the corresponding paper-lookup reference before calling: a successful HTTP
  response alone does not prove that a filter was applied or a record exists

Step 2: Ingest and normalize sources
- Parse local PDFs/Office files with liteparse (layout + bounding boxes)
- Convert mixed documents to clean Markdown with markitdown
- Organize everything into an open-notebook research notebook

Step 3: Reference management
- Store and de-duplicate references in Zotero via pyzotero
- Tag by theme, method, and evidence level

Step 4: Critical appraisal
- Use scholar-evaluation for qualitative, evidence-traceable developmental review
  of each authorized scholarly work
- If a predeclared low-stakes rubric is useful, treat optional scores only as
  bounded anchor summaries with uncertainty—not measurements of quality
- Never score or rank authors, reviewers, institutions, or other people, and never
  use the output for consequential personnel, admissions, funding, or award decisions

Step 4b (optional): Characterizing reasoning style in a corpus
- dhdna-profiler extracts cognitive and reasoning patterns from text. In a research
  context its legitimate use is descriptive and corpus-level — for instance,
  characterizing how argumentation differs between a field's theoretical and
  empirical literature, or how a research programme's framing shifted across a decade
- The constraints are the same as for scholar-evaluation, and they are strict: this
  is never a measurement of a person's ability, and its output must not inform any
  personnel, admissions, funding, review-assignment, or award decision
- Do not profile identified individuals without their knowledge and consent, and do
  not present a stylistic characterization as a finding about competence or rigour
- Skip this step entirely if the corpus is small enough that "the corpus" means
  "a few identifiable authors"

Step 5: Synthesize
- Use the scientific-literature-review skill to synthesize themes, gaps, and consensus/conflicts
- Report PRISMA-style counts: records identified, de-duplicated, screened, excluded
  with reasons, and included
- Assess risk of bias with an instrument appropriate to the study designs included,
  and weight the synthesis accordingly rather than counting papers. Six weak studies
  agreeing is not stronger evidence than one strong study disagreeing
- Where studies conflict, examine whether population, dose, endpoint, or analysis
  differs before concluding the literature is simply inconsistent
- Format citations with citation-management and verify every one resolves — check
  that each DOI, PMID, and arXiv ID retrieves the paper you think it does

Step 6: Deliverable
- A cited systematic review with evidence tables and a managed reference library

Expected Output:
- A reproducible search log with per-database queries, filters, and dates
- PRISMA-style flow counts including exclusions with reasons
- Organized, parsed, and reference-managed corpus
- Evidence table with risk-of-bias assessment per study
- Qualitatively appraised, synthesized, fully cited literature review
- An explicit statement of where the evidence is thin or absent
```

---

## Scientific Communication & Tooling

### Example 37: Publication Packaging — Diagrams, Infographics, and Venue Formatting

**Objective**: Turn verified results into an author-reviewed draft manuscript package with source-traceable prose and visuals, current venue checks, and an optional macro-free PPTX poster.

**Disciplines**: scientific writing · publishing standards · visual communication · research integrity

**Skills Used**:
- `markdown-mermaid-writing` - Text-based diagrams and structured docs
- `scientific-writing` - Evidence registry, authorship, confidentiality, and consistency checks
- `scientific-schematics` - Scientific diagrams
- `infographics` - AI-generated infographics with data accuracy checks
- `peer-review` - Internal critique before the manuscript leaves the group
- `venue-templates` - LaTeX templates and submission guidelines
- `markitdown` - Convert drafts/sources to Markdown
- `citation-management` - Reference formatting and verification
- `latex-posters` - Conference poster
- `pptx-posters` - Macro-free PowerPoint poster from an approved local manifest

**Starting prompt**:

```text
Use the scientific-writing, markdown-mermaid-writing, scientific-schematics,
infographics, peer-review, venue-templates, and citation-management skills.

Goal: a submission-ready draft package plus the critique I would get from a
hostile reviewer.
Criteria: every numerical claim in the text maps to an entry in the evidence
registry. Every citation resolves to the paper it claims to.
Deliver: manuscript (PDF + DOCX), figures with captions, supplementary tables
(xlsx), poster, and a reviewer-style critique.
Report: verify the venue's current author instructions and AI-disclosure
policy directly rather than assuming the template is current.
Do not: submit anything. Authors approve declarations and content, and
authorize submission separately.
```

**Workflow**:

```text
Step 1: Structure the manuscript
- Establish an authorized local workspace, source manifest, claim/evidence registry,
  authorship/declaration records, and reporting-guideline coverage
- Draft the document in Markdown with scientific-writing; add Mermaid diagrams
- Convert existing source materials to Markdown with markitdown

Step 2: Build figures and schematics
- Create mechanism/workflow schematics with scientific-schematics
- Produce a one-page infographic summary only from verified, author-approved data
- Obtain explicit authorization before any external image service receives source
  material; record prompt/model/output provenance and manually verify every detail

Step 3: Apply venue formatting
- Use venue-templates to identify a candidate LaTeX template, then verify the current
  official author instructions and AI/disclosure policy for the exact venue and article type
  (Nature/Science/PLOS/IEEE/ACM or a target conference)

Step 4: Generate outputs
- Compile draft manuscript review copies to PDF and DOCX for authorized collaborators
- Build a conference poster with latex-posters
- If PowerPoint is requested, populate the pptx-posters local manifest with exact
  author-approved text/assets, hashes, provenance, printer requirements, reading order,
  alt text, and approval hash; generate and inspect a one-slide macro-free `.pptx`

Step 5: Final check
- Run local claim/reference/consistency checks and verify formatting, figure properties,
  accessibility, and reference style against current official venue requirements
- Accountable human authors resolve scientific issues, approve declarations and content,
  and separately authorize any submission

Expected Output:
- Author-reviewed draft manuscript package (PDF + DOCX) with evidence traceability
- Source-traceable diagrams, schematics, and infographic
- A matching LaTeX poster or inspected macro-free `.pptx` poster
```

---
