# Introduction to Chemical Engineering MATLAB Projects

## Learning Objectives

By the end of this lecture, you should be able to:

- Develop complete chemical engineering applications in MATLAB
- Integrate multiple concepts from previous lectures
- Design modular, maintainable chemical engineering code
- Solve realistic process engineering problems
- Validate simulations against experimental data
- Create professional documentation and reports
- Present results visually and quantitatively
- Understand workflow from problem to deployed solution



## Why This Topic Matters

This lecture **brings everything together**. Real chemical engineering work requires you to:

- Combine programming, math, and physics
- Manage complex, multi-part problems
- Validate against reality
- Communicate results clearly
- Think about deployment and sustainability


## Problem Formulation

### Understanding the Problem Statement

Every engineering project begins with a **problem statement**—a description of what you need to calculate and why it matters.

A good problem statement includes:

| Element | Purpose | Example |
|---------|---------|---------|
| **Context** | Why does this calculation matter? | "A distillation unit must separate a feed mixture to meet product purity specifications" |
| **Inputs** | What data or parameters are known? | "Feed flowrate: 100 kmol/h; Feed composition: 40 mol% A, 60 mol% B" |
| **Outputs** | What quantity must be calculated? | "Calculate the distillate and bottoms flowrates and compositions" |
| **Constraints** | What physical or economic limits apply? | "Distillate purity ≥ 95 mol% A; Bottoms purity ≥ 90 mol% B" |
| **Assumptions** | What simplifications are made? | "Isothermal operation; ideal solutions; 50 theoretical stages" |

#### Example Problem Statement

> **Problem:** Calculate the exit temperature and composition of a perfectly mixed continuous stirred-tank reactor (CSTR) operating under steady state. The first-order reversible reaction A ⇌ B occurs in liquid phase. Given feed flowrate, inlet concentration, reaction rate constant, reactor volume, and thermal inputs, determine the extent of reaction and verify that the system is physically realizable.

### Translating the Problem into MATLAB Structure

Once you understand the problem, structure your MATLAB solution as follows:

#### 1. **Identify Required Information**

List everything you need:

```
Inputs:
  - Feed flowrate (kmol/h)
  - Inlet concentration of A (kmol/m³)
  - Reaction rate constant k (1/s)
  - Reactor volume (m³)
  - Inlet temperature (K)
  - Heat of reaction (kJ/kmol)

Outputs:
  - Exit concentration of A (kmol/m³)
  - Exit concentration of B (kmol/m³)
  - Exit temperature (K)
  - Conversion of A (%)
  - Residence time (s)
```

#### 2. **State Key Assumptions**

Make your simplifications explicit:

```
Assumptions:
  - Perfect mixing (CSTR ideal behavior)
  - Steady-state operation
  - Constant volume reactor
  - Constant density liquid
  - Negligible kinetic energy and pressure drop effects
  - Constant heat capacity
  - Forward reaction rate: r = k·C_A (mol/(m³·s))
  - Reverse reaction rate neglected (pseudo-first-order)
```

#### 3. **Write the Governing Equations**

Translate chemistry and physics into mathematics:

```
Material Balance (Component A):
  F_in · C_A_in = F_out · C_A_out + r_A · V

where r_A = k · C_A (forward reaction rate)

Energy Balance (assuming no heat loss):
  Inlet energy + Heat of reaction = Outlet energy
  
  F_in · rho · Cp · (T - T_ref) + (-Delta_H_rxn) · r_A · V 
    = F_out · rho · Cp · (T_exit - T_ref)

Conversion:
  X_A = (C_A_in - C_A_out) / C_A_in

Residence Time:
  τ = V / F
```

#### 4. **Define the MATLAB Workflow**

Sketch the logical sequence:

```
1. Define inputs and constants
2. Solve material and energy balances (algebraic or ODE system)
3. Calculate derived quantities (conversion, rate, etc.)
4. Check physical validity
5. Display or save results
6. Visualize key results
```

## Implementation

### Project Structure

Organize your MATLAB code into three layers:

```
project_name/
│
|
├── README.md
|
├── main_script.m          % Top-level workflow
├── functions/
│   ├── input_parameters.m      % Define constants and user inputs
│   ├── cstr_model.m            % Solve reactor equations
│   ├── validate_results.m      % Check plausibility
│   └── generate_report.m       % Display or export results
│
├── data/
│   ├── feed_properties.csv     % Input data
│   └── results.csv             % Output data
│
└── plots/
    ├── conversion_profile.fig
    └── temperature_distribution.fig
```

---

### Building a MATLAB Function Hierarchy

#### Layer 1: Input Parameters

Create a function that centralizes all constants and user-defined data:

```matlab
function [params] = input_parameters()
%INPUT_PARAMETERS  Define reactor and feed properties
%   Returns a structure array containing all parameters for a CSTR
%   calculation

% Physical and chemical properties
params.k = 0.1;              % Reaction rate constant (1/s)
params.Delta_H_rxn = -50000;      % Heat of reaction (J/kmol)
params.Cp = 4180;            % Heat capacity (J/(kg·K))
params.rho = 1000;             % Density (kg/m³)

% Reactor and feed specifications
params.V = 5;                % Reactor volume (m³)
params.F_in = 2.5;           % Feed flowrate (kmol/h)
params.T_in = 298;           % Inlet temperature (K)
params.C_A_in = 10;          % Inlet concentration of A (kmol/m³)

% Conversion target and tolerances
params.tolerance = 1e-6;     % ODE solver tolerance
params.X_target = 0.50;      % Target conversion (50%)

% Reference state
params.T_ref = 298;          % Reference temperature (K)

end
```

**Benefits:**
- Easy to change parameters without editing the main code
- All constants in one place
- Clear units and meanings
- Reusable across multiple analyses

#### Layer 2: Core Calculation Function

Implement the actual engineering model:

```matlab
function [results] = cstr_model(params)
%CSTR_MODEL  Solve steady-state CSTR with reversible reaction
%   Inputs:
%     params - structure containing reactor and feed properties
%   Outputs:
%     results - structure with outlet conditions and performance metrics

% Convert feed flowrate from kmol/h to kmol/s
F_kmol_s = params.F_in / 3600;

% Residence time
tau = params.V / F_kmol_s;

% Solve material balance for outlet concentration
% F·C_in = F·C_out + k·C_out·V
% Rearranging: C_out = (F·C_in) / (F + k·V)

C_A_out = (F_kmol_s * params.C_A_in) / (F_kmol_s + params.k * params.V);

% Calculate conversion
X_A = (params.C_A_in - C_A_out) / params.C_A_in;

% Reaction rate at outlet conditions
r_A = params.k * C_A_out;  % kmol/(m³·s)

% Total production rate of A reacted
rate_A_reacted = r_A * params.V;  % kmol/s

% Energy balance (simplified, assuming isothermal approach)
% Heat released per unit time
Q_rxn = abs(params.Delta_H_rxn) * rate_A_reacted;  % J/s = W

% Temperature rise (assuming adiabatic conditions)
% Q = F · rho · Cp · Delta_T
% Delta_T = Q / (F · rho · Cp)
mass_flow = F_kmol_s * 18;  % Approximate molar mass for water-like fluid (kg/s)
Delta_T_adiabatic = Q_rxn / (mass_flow * params.Cp);

T_exit_adiabatic = params.T_in + Delta_T_adiabatic;

% Store results in a structure
results.C_A_out = C_A_out;        % kmol/m³
results.C_B_produced = params.C_A_in - C_A_out;  % kmol/m³
results.X_A = X_A;                % Conversion fraction (0–1)
results.T_exit = T_exit_adiabatic;  % K
results.tau = tau;                % s
results.r_A = r_A;                % kmol/(m³·s)
results.Q_rxn = Q_rxn / 1000;     % kW

end
```

#### Layer 3: Validation Function

Verify that results make physical sense:

```matlab
function [is_valid, message] = validate_results(params, results)
%VALIDATE_RESULTS  Check physical plausibility of CSTR solution
%   Inputs:
%     params - input parameters structure
%     results - results structure
%   Outputs:
%     is_valid - logical; true if results pass all checks
%     message - character array with validation details

is_valid = true;
message = "";

% Check 1: Outlet concentration cannot exceed inlet
if results.C_A_out > params.C_A_in
    is_valid = false;
    message = [message; "ERROR: Outlet concentration exceeds inlet"];
end

% Check 2: Conversion must be between 0 and 1
if results.X_A < 0 || results.X_A > 1
    is_valid = false;
    message = [message; "ERROR: Conversion outside physical range [0,1]"];
end

% Check 3: Outlet temperature should be higher than inlet (exothermic)
if results.T_exit < params.T_in
    message = [message; "WARNING: Exit temperature lower than inlet (endothermic or cooled)"];
end

% Check 4: Residence time should be positive
if results.tau <= 0
    is_valid = false;
    message = [message; "ERROR: Residence time is non-positive"];
end

% Check 5: Reaction rate should be positive (forward reaction)
if results.r_A < 0
    is_valid = false;
    message = [message; "ERROR: Reaction rate is negative"];
end

% If all checks pass
if is_valid && isempty(message)
    message = "All validation checks passed.";
end

end
```

#### Layer 4: Main Script

Orchestrate the complete workflow:

```matlab
% =========================================================================
%               CSTR REACTOR DESIGN AND ANALYSIS
% =========================================================================
% Purpose: Determine outlet composition and temperature for a steady-state
%          CSTR with an irreversible exothermic first-order reaction
% =========================================================================

clear; clc; close all;
addpath("functions")

% Step 1: Load input parameters
params = input_parameters();

fprintf('\n=========== CSTR ANALYSIS ===========\n');
fprintf('Problem: First-order reaction A → B in liquid CSTR\n\n');

% Step 2: Solve the reactor model
results = cstr_model(params);

% Step 3: Validate physical plausibility
[is_valid, msg] = validate_results(params, results);

if ~is_valid
    fprintf('\n!!! VALIDATION FAILED !!!\n%s\n', msg);
    return;
else
    fprintf(' Validation passed: %s\n\n', msg);
end

% Step 4: Display results
fprintf('RESULTS:\n');
fprintf('--------\n');
fprintf('Outlet [A]:        %8.4f  kmol/m³\n', results.C_A_out);
fprintf('Outlet [B]:        %8.4f  kmol/m³\n', results.C_B_produced);
fprintf('Conversion:        %8.4f  (%.1f%%)\n', results.X_A, results.X_A*100);
fprintf('Exit Temperature:  %8.2f  K\n', results.T_exit);
fprintf('Residence Time:    %8.4f  s\n', results.tau);
fprintf('Reaction Rate:     %8.6f  kmol/(m³·s)\n', results.r_A);
fprintf('Heat Released:     %8.2f  kW\n', results.Q_rxn);

% Step 5: Export results to CSV (optional)
writetable(struct2table(results), 'cstr_results.csv');
```

### Workflow Overview

This modular structure provides several advantages:

1. **Separation of Concerns**: Parameters, physics, and validation are independent
2. **Testability**: Each function can be tested separately
3. **Reusability**: Swap out `cstr_model` for a different reactor type (PFR, batch) easily
4. **Maintainability**: Changing a constant doesn't require rewriting equations
5. **Documentation**: Each function's purpose is clear

---

## Verification and Presentation

### Checking Physical Plausibility

Never trust a number just because MATLAB computed it. Ask:

#### 1. **Dimensional Analysis**

Does every quantity have correct units?

```matlab
%  Correct: mixing units at boundaries
C_out_kmol_m3 = 10;        % kmol/m³
V_m3 = 5;                   % m³
F_kmol_h = 100;            % kmol/h
F_kmol_s = F_kmol_h / 3600;  % Convert to kmol/s before math

%  Incorrect: mixing units without conversion
rate = F_kmol_h * 1000 * V_m3;  % Nonsensical: (kmol/h)·m³ = ???
```

#### 2. **Order of Magnitude**

Is the answer reasonable for the context?

```matlab
% CSTR example:
% - Residence time 5 seconds for a 5 m³ reactor at 2.5 kmol/h?
% - Conversion 50% for a reaction with k=0.1 s⁻¹?
% - Temperature rise 10 K from an exothermic reaction?

% Check against literature or hand calculations
tau_expected = 5 / (2.5 / 3600);  % seconds
fprintf('Expected residence time: %.1f s\n', tau_expected);

% Theoretical max conversion (if infinite residence time)
X_theoretical = 1.0;  % 100% for reversible reaction
if results.X_A > X_theoretical
    warning('Conversion exceeds theoretical maximum');
end
```

#### 3. **Boundary Conditions**

Do edge cases behave as expected?

```matlab
% As reaction rate constant k → 0 (no reaction):
% Expected: X_A → 0, C_out → C_in, T_exit → T_in

% As k → ∞ (instantaneous reaction):
% Expected: X_A → 1, C_out → 0, T_exit → maximum

% Run these limiting cases:
params_slow = input_parameters();
params_slow.k = 1e-6;
results_slow = cstr_model(params_slow);
assert(results_slow.X_A < 0.01, 'Slow reaction should have low conversion');

params_fast = input_parameters();
params_fast.k = 100;
results_fast = cstr_model(params_fast);
assert(results_fast.X_A > 0.99, 'Fast reaction should have high conversion');

fprintf(' Boundary condition tests passed.\n');
```

---

### Comparing with Analytical or Known Results

When possible, validate MATLAB results against:

1. **Analytical Solutions** (if available)
2. **Published Data** (literature values)
3. **Simple Cases** (hand calculations or limiting behavior)
4. **Experimental Data** (measured values)

#### Example: CSTR with First-Order Irreversible Reaction

**Analytical Solution:**

$$X_A = \frac{k \tau}{1 + k \tau}$$

where $k$ is the reaction rate constant and $\tau$ is the residence time.

**MATLAB Verification:**

```matlab
% Analytical solution
tau = results.tau;
k = params.k;
X_A_analytical = (k * tau) / (1 + k * tau);

% Compare with MATLAB results
error_X_A = abs(results.X_A - X_A_analytical) / X_A_analytical * 100;

fprintf('Analytical Conversion:  %.6f\n', X_A_analytical);
fprintf('MATLAB Conversion:      %.6f\n', results.X_A);
fprintf('Relative Error:         %.3f %%\n', error_X_A);

if error_X_A < 0.1
    fprintf(' Excellent agreement with analytical solution\n');
else
    fprintf('⚠ Check for modeling errors\n');
end
```

---

### Error Analysis and Sensitivity Analysis

Understand how uncertainties in inputs propagate to outputs:

#### Sensitivity to Feed Concentration

```matlab
% Baseline
params_base = input_parameters();
results_base = cstr_model(params_base);

% Perturb input
% Test: ±10% variation in inlet concentration
C_A_nominal = params_base.C_A_in;
perturbation = 0.10;  % ±10%

C_A_low = C_A_nominal * (1 - perturbation);
C_A_high = C_A_nominal * (1 + perturbation);

% Calculate results at perturbed conditions
params_low = params_base;
params_low.C_A_in = C_A_low;
results_low = cstr_model(params_low);

params_high = params_base;
params_high.C_A_in = C_A_high;
results_high = cstr_model(params_high);

% Sensitivity: Delta_Output / Delta_Input
sensitivity = (results_high.C_A_out - results_low.C_A_out) / ...
              (C_A_high - C_A_low);

fprintf('\nSensitivity Analysis: Outlet Conc. vs. Inlet Conc.\n');
fprintf('Sensitivity = %.4f\n', sensitivity);
fprintf('Interpretation: 1 kmol/m³ increase in inlet → %.4f kmol/m³ increase in outlet\n', sensitivity);

% Visualization
C_A_range = linspace(C_A_low, C_A_high, 10);
C_A_out_range = zeros(size(C_A_range));

for i = 1:length(C_A_range)
    params_temp = params_base;
    params_temp.C_A_in = C_A_range(i);
    results_temp = cstr_model(params_temp);
    C_A_out_range(i) = results_temp.C_A_out;
end

figure('Name', 'Sensitivity Analysis');
plot(C_A_range, C_A_out_range, 'b-o', 'LineWidth', 2, 'MarkerSize', 6);
xlabel('Inlet Concentration [A] (kmol/m³)');
ylabel('Outlet Concentration [A] (kmol/m³)');
title('Sensitivity: How Outlet Responds to Inlet Variation');
grid on;
```

---

### Professional Presentation

#### Text Report Generation

```matlab
function [] = generate_report(params, results)
%GENERATE_REPORT  Create a professional text report of CSTR analysis

% Create formatted report
fid = fopen('CSTR_Analysis_Report.txt', 'w');

fprintf(fid, '========================================\n');
fprintf(fid, '     CSTR REACTOR ANALYSIS REPORT\n');
fprintf(fid, '========================================\n\n');

fprintf(fid, 'Generated: %s\n\n', datetime('now'));

fprintf(fid, 'REACTOR SPECIFICATIONS\n');
fprintf(fid, '----------------------\n');
fprintf(fid, 'Volume:                 %.2f m³\n', params.V);
fprintf(fid, 'Feed Rate:              %.2f kmol/h\n', params.F_in);
fprintf(fid, 'Residence Time:         %.4f s\n', results.tau);

fprintf(fid, '\nFEED COMPOSITION\n');
fprintf(fid, '----------------\n');
fprintf(fid, 'Temperature:            %.2f K\n', params.T_in);
fprintf(fid, 'Concentration [A]:      %.4f kmol/m³\n', params.C_A_in);

fprintf(fid, '\nREACTION KINETICS\n');
fprintf(fid, '-----------------\n');
fprintf(fid, 'Rate Constant k:        %.6f 1/s\n', params.k);
fprintf(fid, 'Delta_H_rxn:                 %.0f J/kmol\n', params.Delta_H_rxn);

fprintf(fid, '\nOUTLET CONDITIONS\n');
fprintf(fid, '-----------------\n');
fprintf(fid, 'Temperature:            %.2f K  (rise: %.2f K)\n', ...
    results.T_exit, results.T_exit - params.T_in);
fprintf(fid, 'Concentration [A]:      %.4f kmol/m³\n', results.C_A_out);
fprintf(fid, 'Concentration [B]:      %.4f kmol/m³\n', results.C_B_produced);

fprintf(fid, '\nPERFORMANCE METRICS\n');
fprintf(fid, '-------------------\n');
fprintf(fid, 'Conversion:             %.4f (%.2f%%)\n', results.X_A, results.X_A*100);
fprintf(fid, 'Reaction Rate:          %.6f kmol/(m³·s)\n', results.r_A);
fprintf(fid, 'Heat Released:          %.2f kW\n', results.Q_rxn);

fprintf(fid, '\n========================================\n');
fprintf(fid, 'END OF REPORT\n');
fprintf(fid, '========================================\n');

fclose(fid);

fprintf('Report saved to: CSTR_Analysis_Report.txt\n');

end
```



## Worked Example: binary distillation column design

### Problem Statement

A distillation column must separate a benzene (A) + toluene (B) mixture to produce:

- Distillate: ≥ 95 mol% benzene
- Bottoms: ≥ 90 mol% toluene

**Given:**
- Feed composition: 50 mol% benzene, 50 mol% toluene
- Feed rate: 100 kmol/h
- Atmospheric pressure (1 atm)
- Constant molar overflow assumption
- 20 theoretical stages + reboiler and condenser

**Find:** Distillate and bottoms flowrates and compositions.

### Solution Strategy

We will use **Rachford-Rice equation** (shortcut method) to solve the distillation problem.

### Complete MATLAB Implementation

```matlab
% =========================================================================
%         BINARY DISTILLATION COLUMN DESIGN AND ANALYSIS
% =========================================================================
clear; clc; close all;

% --- STEP 1: DEFINE INPUTS ---
F = 100;        % Feed rate (kmol/h)
z_A = 0.50;     % Feed composition (mole fraction benzene)
z_B = 1 - z_A;  % Mole fraction toluene

% Vapor-liquid equilibrium: y = K·x
% Simplified model at 1 atm (benzene more volatile)
K_A = 2.5;      % Equilibrium constant for benzene
K_B = 0.8;      % Equilibrium constant for toluene

% Product specifications
x_D_min = 0.95;  % Minimum distillate purity
x_B_max = 0.10;  % Maximum bottoms composition (max 10% benzene)

% --- STEP 2: SOLVE MATERIAL BALANCE ---

% Overall balance: F = D + B
% Component A balance: F·z_A = D·x_D + B·x_B

% From constraints:
x_D = x_D_min;          % Meet distillate spec exactly
x_B = x_B_max;          % Benzene fraction in bottoms = 0.10

% Solve material balance
% F·z_A = D·x_D + (F-D)·x_B
% F·z_A = D·x_D + F·x_B - D·x_B
% F·(z_A - x_B) = D·(x_D - x_B)

D = F * (z_A - x_B) / (x_D - x_B);  % Distillate flowrate
B = F - D;                           % Bottoms flowrate

% --- STEP 3: CALCULATE COMPOSITION OF OTHER COMPONENT ---
y_A = K_A * x_D;  % Vapor composition leaving condenser (approximately)
y_B = K_B * x_B;  % Vapor composition leaving reboiler (approximately)

% --- STEP 4: VALIDATE RESULTS ---
fprintf('===== DISTILLATION ANALYSIS =====\n\n');

% Check: Material balance
fprintf('Material Balance Check:\n');
fprintf('  F:        %8.2f kmol/h\n', F);
fprintf('  D + B:    %8.2f kmol/h\n', D + B);
fprintf('  Balance:   (difference: %.4e)\n\n', abs(F - (D+B)));

% Check: Component balance
A_in = F * z_A;
A_out = D * x_D + B * x_B;
fprintf('Component A Balance:\n');
fprintf('  Input:    %8.2f kmol/h\n', A_in);
fprintf('  Output:   %8.2f kmol/h\n', A_out);
fprintf('  Balance:   (difference: %.4e)\n\n', abs(A_in - A_out));

% --- STEP 5: DISPLAY RESULTS ---
fprintf('PROCESS RESULTS:\n');
fprintf('----------------\n');
fprintf('Feed:\n');
fprintf('  Rate:                 %8.2f kmol/h\n', F);
fprintf('  Composition (A):      %8.4f  (%.2f%%)\n', z_A, z_A*100);
fprintf('  Composition (B):      %8.4f  (%.2f%%)\n', z_B, z_B*100);

fprintf('\nDistillate Stream:\n');
fprintf('  Rate:                 %8.2f kmol/h\n', D);
fprintf('  Composition A:        %8.4f  (%.2f%%)  [Spec: ≥95%%]\n', ...
    x_D, x_D*100);
fprintf('  Composition B:        %8.4f  (%.2f%%)\n', 1-x_D, (1-x_D)*100);

fprintf('\nBottoms Stream:\n');
fprintf('  Rate:                 %8.2f kmol/h\n', B);
fprintf('  Composition A:        %8.4f  (%.2f%%)  [Spec: ≤10%%]\n', ...
    x_B, x_B*100);
fprintf('  Composition B:        %8.4f  (%.2f%%)\n', 1-x_B, (1-x_B)*100);

% --- STEP 6: VISUALIZATION ---
figure('Name', 'Distillation Analysis');

subplot(1,2,1);
streams = {'Feed', 'Distillate', 'Bottoms'};
compositions_A = [z_A, x_D, x_B];
compositions_B = [z_B, 1-x_D, 1-x_B];
X = categorical(streams);
X = reordercats(X, streams);
bar(X, [compositions_A; compositions_B]', 'stacked');
ylabel('Mole Fraction');
title('Stream Compositions');
legend('Benzene (A)', 'Toluene (B)', 'Location', 'NorthEast');
grid on; grid minor;
ylim([0 1]);

subplot(1,2,2);
flowrates = [F, D, B];
colors = [0.4 0.6 0.9];  % Blue-ish colors
bar(X, flowrates, 'FaceColor', [0.5 0.7 0.9]);
ylabel('Flowrate (kmol/h)');
title('Stream Flowrates');
grid on; grid minor;

% --- STEP 7: SAVE RESULTS ---
results.F = F;
results.D = D;
results.B = B;
results.z_A = z_A;
results.x_D = x_D;
results.x_B = x_B;

% Save to CSV
data = table([F; D; B], [z_A; x_D; x_B], 'VariableNames', ...
    {'Flowrate_kmol_h', 'Composition_A_molfrac'});
writetable(data, 'distillation_results.csv');

fprintf('\nResults exported to: distillation_results.csv\n');

% --- STEP 8: Cheking Infeasible results---
assert(D > 0 && B > 0, 'Infeasible specs: negative stream flow.');
```

**Expected Output:**

```
===== DISTILLATION ANALYSIS =====

Material Balance Check:
  F:          100.00 kmol/h
  D + B:      100.00 kmol/h
  Balance:   (difference: 0.0000e+00)

Component A Balance:
  Input:       50.00 kmol/h
  Output:      50.00 kmol/h
  Balance:   (difference: 7.1054e-15)

PROCESS RESULTS:
----------------
Feed:
  Rate:                   100.00 kmol/h
  Composition (A):        0.5000  (50.00%)
  Composition (B):        0.5000  (50.00%)

Distillate Stream:
  Rate:                    47.06 kmol/h
  Composition A:          0.9500  (95.00%)  [Spec: ≥95%]
  Composition B:          0.0500  (5.00%)

Bottoms Stream:
  Rate:                    52.94 kmol/h
  Composition A:          0.1000  (10.00%)  [Spec: ≤10%]
  Composition B:          0.9000  (90.00%)

Results exported to: distillation_results.csv
```


## Summary

### Key Concepts

**Problem Formulation:**
- Translate engineering language into structured problem definitions
- Clearly state inputs, outputs, assumptions, and constraints
- Identify governing equations from physical and chemical principles

**Implementation:**
- Organize code into modular functions (inputs → physics → validation → output)
- Use structures and named parameters for clarity
- Implement layered architecture: main script → high-level functions → utility functions

**Verification:**
- Perform dimensional analysis to catch unit errors
- Check boundary conditions and limiting cases
- Compare MATLAB results against analytical solutions when available
- Conduct sensitivity and error analysis to understand model robustness

**Presentation:**
- Generate professional reports with clear sections
- Visualize key results for intuitive understanding
- Export data to CSV/Excel for sharing with colleagues
- Document all assumptions and methods

### Workflow Summary

```
1. Problem Statement
   ↓
2. Input Definition (parameters, constants)
   ↓
3. Mathematical Formulation (equations)
   ↓
4. MATLAB Implementation (functions)
   ↓
5. Validation & Verification (checks, limits)
   ↓
6. Sensitivity Analysis (robustness)
   ↓
7. Results Presentation (reports, plots, export)
   ↓
8. Interpretation (engineering conclusions)
```

### Important MATLAB Patterns

| Task | Pattern |
|------|---------|
| Centralize parameters | Use `function params = input_parameters()` |
| Organize calculations | Break into small, testable functions |
| Validate results | Check physical bounds and edge cases |
| Handle units | Convert at function boundaries |
| Document code | Include purpose, units, assumptions |
| Report results | Use `fprintf` with formatted output |
| Export data | `writetable()` for CSV, `save()` for .mat |
| Visualize | `figure`, `subplot`, `bar`, `plot` with labels |




---

## Final Words

You now have **all the tools** to solve real chemical engineering problems with MATLAB:

- **Programming**: variables, functions, control flow
- **Mathematics**: matrices, calculus, numerical methods
- **Engineering**: modeling, validation, optimization
- **Tools**: debugging, version control, AI assistance

The next step is to **apply these skills** to problems you care about. MATLAB is your engineering calculator—use it to innovate.

