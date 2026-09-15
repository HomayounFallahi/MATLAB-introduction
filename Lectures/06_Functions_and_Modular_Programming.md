# Functions and Modular Programming

## Learning Objectives

By the end of this lecture, you should be able to:

- Understand the purpose and benefits of functions in MATLAB
- Write and call custom functions with multiple inputs and outputs
- Distinguish between local, global, and persistent variables
- Create anonymous functions and function handles
- Organize MATLAB code into reusable modules and projects
- Develop well-structured engineering functions with error checking
- Appreciate how functions improve code readability, reusability, and maintainability

---

## Why This Topic Matters

As your MATLAB programs grow in complexity—solving material balances, simulating reactors, analyzing experimental data—you'll find that your code becomes longer and harder to follow. Functions allow you to break complex problems into smaller, manageable pieces.

**Benefits of functions:**

- **Code reuse**: Write once, use many times
- **Clarity**: Each function has a single, clear purpose
- **Testing**: Easier to identify and fix bugs in isolated functions
- **Collaboration**: Team members can work on different functions simultaneously
- **Maintainability**: Changes are localized to specific functions

In chemical engineering, you might create functions to calculate reaction rates, perform material balances, or check if a solution is physically reasonable. Rather than rewriting the same calculation every time, you define it once in a function and call it whenever needed.

---

## Introduction to Functions

### What is a Function?

A function is a reusable block of code that performs a specific task. It accepts input arguments, processes them, and returns output values.

**Basic concept:**

Input → Function → Output

### Purpose of Functions

Functions serve several key purposes:

1. **Encapsulation**: Hide complexity inside a function
2. **Reusability**: Call the same function multiple times
3. **Organization**: Group related calculations together
4. **Testing**: Verify each function independently
5. **Modularity**: Build large programs from small pieces

### Function Syntax

The most common MATLAB function has this structure:

```matlab
function [output1, output2] = functionName(input1, input2)
    % Function body: calculations go here
    output1 = calculation1;
    output2 = calculation2;
end
```

**Syntax breakdown:**

| Component | Meaning |
|-----------|---------|
| `function` | Keyword declaring a function |
| `[output1, output2]` | Output variables (use square brackets for multiple outputs) |
| `=` | Assignment operator |
| `functionName` | Name of the function (must match the filename) |
| `(input1, input2)` | Input arguments (parameters) |
| Function body | Code that performs calculations |
| `end` | Closes the function (required) |

### Simple Function Example

Let's create a function that converts temperature from Kelvin to Celsius:

```matlab
function T_celsius = kelvinToCelsius(T_kelvin)
    % Convert temperature from Kelvin to Celsius
    % Input: T_kelvin (scalar or vector in K)
    % Output: T_celsius (same dimensions as input in °C)
    T_celsius = T_kelvin - 273.15;
end
```

**Using the function:**

```matlab
T_K = 350;                          % Temperature in Kelvin
T_C = kelvinToCelsius(T_K);         % Call the function
disp(['Temperature: ' num2str(T_C) ' °C'])
```

**Output:**
```
Temperature: 76.85 °C
```

### Local Variables and Function Scope

Variables created inside a function are **local**—they exist only within that function.

```matlab
function volume = calculateVolume(diameter, height)
    radius = diameter / 2;          % radius is local
    volume = pi * radius^2 * height;
end
```

When you call this function, the variable `radius` is created temporarily, used, and then destroyed. Outside the function, `radius` does not exist. This is called **scope**.

**Why this matters:**
- Local variables don't interfere with other parts of your code
- You can use the same variable name in different functions without conflicts
- Functions are self-contained and predictable

---

## Types of MATLAB Functions

### Inline Functions and Anonymous Functions

An **anonymous function** is a small, unnamed function defined in a single line. Use them for simple calculations.

**Syntax:**

```matlab
functionHandle = @(input1, input2) expression
```

**Example: Ideal Gas Law**

```matlab
% Anonymous function for ideal gas law: PV = nRT
% Solve for pressure: P = nRT/V
idealGasPressure = @(n, R, T, V) (n * R * T) / V;

% Use the function
n = 2;              % moles
R = 8.314;          % J/(mol·K)
T = 350;            % K
V = 0.050;          % m³
P = idealGasPressure(n, R, T, V);

disp(['Pressure: ' num2str(P/1e5) ' bar'])
```

**Output:**
```
Pressure: 1.1688 bar
```

**When to use anonymous functions:**

- Simple, one-line calculations
- Temporary functions used in a script
- Functions passed as arguments to other functions (like solvers)

**When NOT to use anonymous functions:**

- Complex calculations requiring multiple lines
- Functions you'll use repeatedly in different scripts
- When you need to add comments and documentation

### Function Files (.m files)

For more substantial functions, create a separate `.m` file. This is the standard approach for most engineering work.

**File structure:**

Each function should be in its own file named `functionName.m`.

```matlab
% Filename: reactorVolume.m
function V = reactorVolume(flowRate, residenceTime)
    % Calculate reactor volume for a continuous stirred-tank reactor (CSTR)
    %
    % Inputs:
    %   flowRate (m³/s): volumetric feed flow rate
    %   residenceTime (s): desired residence time
    %
    % Output:
    %   V (m³): required reactor volume
    %
    % Equation: V = Q * τ
    
    % Input validation
    if flowRate <= 0 || residenceTime <= 0
        error('flowRate and residenceTime must be positive')
    end
    
    % Calculate volume
    V = flowRate * residenceTime;
end
```

**To use this function in a script:**

```matlab
% script_cstr_design.m
Q = 0.050;      % m³/s
tau = 60;       % s (1 minute residence time)

V_reactor = reactorVolume(Q, tau);
disp(['Required reactor volume: ' num2str(V_reactor) ' m³'])
```

### Subfunctions

You can define multiple functions within a single `.m` file. The first function is the **main function**; any additional functions are **subfunctions** visible only to the main function.

**Example:**

```matlab
% File: reactor_design.m (main function)
function V = reactor_design(F, C_in, C_out, T, type)
%REACTOR_DESIGN Wrapper for CSTR/PFR volume
    k = arrhenius_local(T);  % call subfunction
    
    switch lower(type)
        case "cstr"
            V = cstr_calc(F, C_in, C_out, k);
        case "pfr"
            V = pfr_calc(F, C_in, C_out, k);
        otherwise
            error('Unknown type %s', type);
    end
end

% --- Subfunction 1 ---
function k = arrhenius_local(T)
    A=1e7; Ea=75000; R=8.314;
    k = A*exp(-Ea/(R*T));
end

% --- Subfunction 2 ---
function V = cstr_calc(F, C_in, C_out, k)
    V = F*(C_in - C_out)/(k*C_out);
end

% --- Subfunction 3 ---
function V = pfr_calc(F, C_in, C_out, k)
    % PFR: V = F/k * log(C_in/C_out) for first order
    V = F/k * log(C_in/C_out);
end
```

**Benefits of subfunctions:**

- Keep related calculations together
- Prevent subfunctions from being called outside their context
- Organize complex workflows

### Function Handles

A **function handle** is a reference to a function that can be stored in a variable or passed to other functions.

**Syntax:**

```matlab
handle = @functionName;     % Reference to existing function
handle = @(x) x^2 + 2*x + 1;  % Anonymous function as a handle
```

**Example: Using function handles with a solver**

```matlab
% Define reaction rate constant expression as a function handle
k = @(T) 0.1e-7 * exp(50000 / 8.314 / T);  % Arrhenius equation

% Evaluate at different temperatures
T_values = [300, 350, 400];
k_values = arrayfun(k, T_values);

disp('Temperature (K) |  Rate Constant (1/s)')
disp('--------------------------------------')
for i = 1:length(T_values)
    fprintf('%14.1f  |  %10.6f\n', T_values(i), k_values(i))
end
```

**Output:**
```
Temperature (K) |  Rate Constant (1/s)
--------------------------------------
         300.0  |    5.082621
         350.0  |    0.289975
         400.0  |    0.033851
```

---

## Variable Scope and Function Memory

### Local Variables

By default, all variables created in a function are **local**—they exist only within that function and are destroyed when the function ends.

```matlab
function result = exampleFunction()
    x = 10;         % local variable
    y = 20;         % local variable
    result = x + y;
end

% In the main script:
exampleFunction();
disp(x);    % ERROR: x is not defined outside the function
```

### Global Variables

A **global variable** can be accessed and modified from anywhere in your code. Use sparingly—global variables can make code hard to debug.

**Syntax:**

```matlab
global variableName
```

**Example (not recommended for most cases):**

```matlab
% script1.m
global simulationTime
simulationTime = 0;

while simulationTime < 100
    simulationTime = simulationTime + 1;
    processData();
end

function processData()
    global simulationTime
    disp(['Simulation time: ' num2str(simulationTime)])
end
```

**When to use global variables:**

- Configuration parameters shared across many functions
- Rarely—prefer passing variables as arguments instead

### Persistent Variables

A **persistent variable** retains its value between function calls. It is local to the function but does not reset each time the function is called.

**Syntax:**

```matlab
persistent variableName
```
---

**Example: Tracking cumulative reactor feed**

```matlab
function totalFeed = accumulateFeed(feedThisStep)
    % Track cumulative feed to reactor across multiple calls
    persistent cumulativeFeed
    
    % Initialize on first call
    if isempty(cumulativeFeed)
        cumulativeFeed = 0;
    end
    
    % Add this step's feed
    cumulativeFeed = cumulativeFeed + feedThisStep;
    totalFeed = cumulativeFeed;
end

% Usage in a simulation:
for timestep = 1:10
    Q_in = 0.050;               % m³/s for this step
    Q_total = accumulateFeed(Q_in);
    fprintf('Step %2d: Cumulative feed = %.3f m³\n', timestep, Q_total)
end
```

**Output:**
```
Step  1: Cumulative feed = 0.050 m³
Step  2: Cumulative feed = 0.100 m³
Step  3: Cumulative feed = 0.150 m³
...
Step 10: Cumulative feed = 0.500 m³
```

**When to use persistent variables:**

- Tracking state across function calls (e.g., in iterative solvers)
- Avoiding global variables while maintaining state
- Less common than local or global—use with care

---

## Practical Function Development

### Function Definition and Calling

**Definition best practices:**

1. One function per file (plus subfunctions)
2. Filename matches function name exactly (case-sensitive on Linux/Mac)
3. First line is H1 line: `%FUNCTIONNAME one-line description`
4. Help text explains inputs, outputs, units, example
5. Validate inputs, assign outputs
6. Use local variables for constants

**Template:**

```matlab
function [out1, out2] = myFunction(in1, in2, in3)
%MYFUNCTION One-line description
%   Detailed explanation
%   [out1, out2] = myFunction(in1, in2, in3) description
%
%   Inputs:
%       in1 - description, units
%       in2 - description, units
%   Outputs:
%       out1 - description, units
%
%   Example:
%       [a,b] = myFunction(1,2,3)

    % Input validation (optional but good)
    if nargin < 3
        error('myFunction requires 3 inputs');
    end
    
    % Calculations with local variables
    R = 8.314;
    out1 = in1 * exp(-in2/(R*in3));
    out2 = out1 * 2;
end
```

**Calling:**

```matlab
[outA, outB] = myFunction(1e7, 75000, 350);
% Or with single output
outA = myFunction(1e7, 75000, 350);  % outB still computed but not returned? Actually second output ignored
```

**Multiple outputs:**

```matlab
function [Vm, n] = tankMoles(V_tank, P, T)
%TANKMOLES Compute molar volume and total moles
    R = 0.08314;  % L bar / mol K
    Vm = R*T./P;
    n = V_tank./Vm;
end

[Vm, n] = tankMoles(500, 50, 300)  % Vm=0.4988, n=1002
[Vm] = tankMoles(500, 50, 300)     % only Vm
```

### Input/Output Arguments, varargin, varargout

**Variable number of inputs/outputs** for flexible functions.

- `varargin` — cell array of extra inputs
- `varargout` — cell array of extra outputs
- `nargin` — number of inputs actually passed
- `nargout` — number of outputs requested

```matlab
function varargout = flexibleAntoine(varargin)
%FLEXIBLEANTOINE Antoine with variable inputs
%   Psat = flexibleAntoine(T) uses default coefficients (water)
%   Psat = flexibleAntoine(T, A,B,C) uses provided coeffs
%   [Psat, logPsat] = flexibleAntoine(...) returns both

    % Default coefficients water (mmHg, C)
    A_def=8.07131; B_def=1730.63; C_def=233.426;
    
    if nargin == 1
        T = varargin{1};
        A=A_def; B=B_def; C=C_def;
    elseif nargin == 4
        T = varargin{1};
        A = varargin{2}; B = varargin{3}; C = varargin{4};
    else
        error('Need 1 or 4 inputs: T or T,A,B,C');
    end
    
    logPsat = A - B./(T + C);
    Psat = 10.^logPsat;
    
    if nargout <=1
        varargout{1}=Psat;
    else
        varargout{1}=Psat;
        varargout{2}=logPsat;
    end
end

% Usage
Psat = flexibleAntoine(80)  % uses water defaults
Psat = flexibleAntoine(80, 8.07, 1730.6, 233.4)  % custom
[Psat, logP] = flexibleAntoine(80)  % two outputs
```

**More common pattern — optional name-value pairs:**

```matlab
function k = arrhenius_opt(T, varargin)
%ARRHENIUS_OPT with optional A, Ea
    % defaults
    A = 1e7; Ea = 75000; R = 8.314;
    % parse varargin as pairs
    for i=1:2:length(varargin)
        name = varargin{i};
        val = varargin{i+1};
        switch lower(name)
            case 'a'
                A=val;
            case 'ea'
                Ea=val;
            case 'r'
                R=val;
            otherwise
                warning('Unknown parameter %s', name);
        end
    end
    k = A*exp(-Ea./(R*T));
end

% Call
k = arrhenius_opt(350)  % defaults
k = arrhenius_opt(350, 'A', 2e7, 'Ea', 80000)  % custom
```

This pattern is used by MATLAB built-ins like `plot(x,y,'LineWidth',2)`.

---

### Reusable Engineering Functions

Create a library of functions you'll use repeatedly:

```matlab
% Library of thermodynamic functions for chemical engineering

function P = idealGasPressure(n, T, V)
    % Ideal gas law: P = nRT/V
    R = 8.314;  % J/(mol·K)
    P = n * R * T / V;
end

function Keq = vantHoffEquation(K0, dH, T0, T)
    % Van't Hoff equation: ln(K/K0) = -ΔH/R * (1/T - 1/T0)
    R = 8.314;  % J/(mol·K)
    Keq = K0 * exp(-dH / R * (1/T - 1/T0));
end

function k = arrheniusRate(A, Ea, T)
    % Arrhenius equation: k = A * exp(-Ea / RT)
    R = 8.314;  % J/(mol·K)
    k = A * exp(-Ea / (R * T));
end

% Usage in a script:
n = 2;          % moles
T = 350;        % K
V = 0.050;      % m³
P = idealGasPressure(n, T, V);
```

### Organizing MATLAB Projects

For larger projects, organize your functions into a folder structure:

```
MyReactorProject/
├── main_simulation.m          % Main script
├── functions/
│   ├── calculateMaterialBalance.m
│   ├── calculateEnergyBalance.m
│   ├── solveReactorODE.m
│   └── plotResults.m
├── data/
│   └── experimental_data.csv
└── results/
    └── output.txt
```

**In your main script:**

```matlab
% main_simulation.m
addpath('functions')    % Make functions visible to script

% Run simulation
[X, Y, T] = calculateMaterialBalance(inputs);
```

---

## Worked Example: Reaction Rate Calculator for Chemical Engineering

Let's build a complete set of functions for analyzing a first-order reaction in a CSTR.

**Problem:** You're designing a CSTR to convert a reactant. You have the reaction rate constant at a reference temperature and want to quickly evaluate conversion at different operating conditions.

**Functions needed:**
1. Calculate rate constant at any temperature (Arrhenius)
2. Calculate conversion in CSTR (isothermal, first-order)
3. Check physical feasibility of the design

```matlab
% Filename: reactorDesign.m
% Complete reactor design module

% Main function
function [X, tau, k] = designReactor(Xdesired, k_ref, Ea, T, T_ref)
    % Design a CSTR to achieve desired conversion
    %
    % Inputs:
    %   Xdesired: target conversion (0 to 1)
    %   k_ref (1/s): reaction rate constant at reference temperature
    %   Ea (J/mol): activation energy
    %   T (K): reactor operating temperature
    %   T_ref (K): reference temperature for k_ref
    %
    % Outputs:
    %   X: actual conversion achieved
    %   tau (s): required residence time
    %   k (1/s): rate constant at operating temperature
    
    % Get rate constant at operating temperature
    k = calculateRateConstant(k_ref, Ea, T, T_ref);
    
    % Calculate required residence time
    tau = calculateResidenceTime(Xdesired, k);
    
    % Verify the design is feasible
    X = checkDesignFeasibility(tau, k, Xdesired);
    
    % Display results
    fprintf('\n--- CSTR Design Results ---\n')
    fprintf('Operating Temperature: %.1f K\n', T)
    fprintf('Rate Constant: %.6f 1/s\n', k)
    fprintf('Required Residence Time: %.1f s\n', tau)
    fprintf('Actual Conversion: %.2f%%\n', X*100)
end

% Subfunctions

function k = calculateRateConstant(k_ref, Ea, T, T_ref)
    % Calculate rate constant at new temperature using Arrhenius
    R = 8.314;  % J/(mol·K)
    k = k_ref * exp(-Ea/R * (1/T - 1/T_ref));
end

function tau = calculateResidenceTime(X, k)
    % Calculate residence time needed for desired conversion
    % For first-order: τ = X / (k(1-X))
    if X >= 1
        error('Conversion must be less than 1.0')
    end
    tau = X / (k * (1 - X));
end

function X = checkDesignFeasibility(tau, k, Xdesired)
    % Verify that the residence time gives the desired conversion
    % X = τk / (1 + τk)
    X = (tau * k) / (1 + tau * k);
    
    % Check for inconsistency (shouldn't happen, but good practice)
    if abs(X - Xdesired) > 0.01
        warning('Conversion mismatch: check calculations')
    end
end
```

**Using the module:**

```matlab
% Reactor design calculation
k_ref = 0.01;       % 1/s at 300 K
Ea = 40000;         % J/mol
T_ref = 300;        % K
T_op = 350;         % K (operating temperature)
X_target = 0.60;    % 60% conversion desired

[X, tau, k] = designReactor(X_target, k_ref, Ea, T_op, T_ref);

% Calculate reactor volume (given feed flow)
Q_feed = 0.050;     % m³/s
V_reactor = Q_feed * tau;
fprintf('Required Reactor Volume: %.3f m³\n', V_reactor)
```

**Output:**
```
--- CSTR Design Results ---
Operating Temperature: 350.0 K
Rate Constant: 0.038762 1/s
Required Residence Time: 20.4 s
Actual Conversion: 60.00%
Required Reactor Volume: 1.020 m³
```
---

**Example: Material Balance on a Mixer**

```matlab
function [m_exit, X_exit, T_exit] = mixStreams(m1, X1, T1, m2, X2, T2)
    % Combine two streams and calculate exit properties
    %
    % Inputs:
    %   m1, m2 (kg/s): mass flow rates of stream 1 and 2
    %   X1, X2 (dimensionless): component composition in each stream
    %   T1, T2 (K): temperatures of each stream
    %
    % Outputs:
    %   m_exit (kg/s): exit mass flow rate
    %   X_exit (dimensionless): exit composition (assuming ideal mixing)
    %   T_exit (K): exit temperature (adiabatic mixing)
    
    % Total mass flow
    m_exit = m1 + m2;
    
    % Composition (mass balance on component)
    X_exit = (m1 * X1 + m2 * X2) / m_exit;
    
    % Temperature (energy balance, assuming adiabatic)
    T_exit = (m1 * T1 + m2 * T2) / m_exit;
end

% Usage:
m_stream1 = 100;    % kg/s
X_stream1 = 0.30;   % 30% component A
T_stream1 = 320;    % K

m_stream2 = 50;     % kg/s
X_stream2 = 0.50;   % 50% component A
T_stream2 = 400;    % K

[m_out, X_out, T_out] = mixStreams(m_stream1, X_stream1, T_stream1, ...
                                     m_stream2, X_stream2, T_stream2);

fprintf('Exit flow: %.1f kg/s\n', m_out)
fprintf('Exit composition: %.1f%%\n', X_out * 100)
fprintf('Exit temperature: %.1f K\n', T_out)
```

**Output:**
```
Exit flow: 150.0 kg/s
Exit composition: 36.7%
Exit temperature: 346.7 K
```

---

## Common Mistakes

### Mistake 1: Forgetting to Save Functions in `.m` Files

Functions must be saved in their own `.m` file named exactly as the function.

```matlab
% WRONG: Function in script file (only works as anonymous function)
T_celsius = T_kelvin - 273.15;  % This isn't a function definition

% CORRECT: Saved in file named kelvinToCelsius.m
function T_celsius = kelvinToCelsius(T_kelvin)
    T_celsius = T_kelvin - 273.15;
end
```

### Mistake 2: Misunderstanding Variable Scope

Variables inside a function don't exist outside it.

```matlab
% WRONG: Expecting 'radius' to exist outside the function
function V = calcVolume(D, H)
    radius = D/2;
    V = pi * radius^2 * H;
end

V = calcVolume(1, 2);
disp(radius)  % ERROR: radius is not defined
```

### Mistake 3: Confusing Multiple Outputs

Remember square brackets for multiple outputs.

```matlab
% WRONG: No brackets for multiple outputs
function a, b, c = myfunc(x)  % Syntax error!
    a = x;
    b = 2*x;
    c = 3*x;
end

% CORRECT: Use square brackets
function [a, b, c] = myfunc(x)
    a = x;
    b = 2*x;
    c = 3*x;
end

% CORRECT: Call with square brackets
[out1, out2, out3] = myfunc(5);
```
```matlab
% WRONG (confusing which output is which)
function [a, b, c, d, e, f] = analyze(data)
    a = mean(data);
    b = std(data);
    c = min(data);
    d = max(data);
    e = median(data);
    f = range(data);
end

[m, s, minv, maxv, med, rng] = analyze(x);  % Hard to remember order

% BETTER (return a structure)
function stats = analyze(data)
    stats.mean = mean(data);
    stats.std = std(data);
    stats.min = min(data);
    stats.max = max(data);
    stats.median = median(data);
    stats.range = max(data) - min(data);
end

stats = analyze(x);
fprintf(''Mean: %.2f, Std: %.2f\n'', stats.mean, stats.std)
```

### Mistake 4: Not Validating Inputs

Skipping input validation leads to cryptic error messages.

```matlab
% WRONG: No error checking
function result = divide(a, b)
    result = a / b;
end

divide(10, 0)  % MATLAB error: Inf (unhelpful)

% CORRECT: Check for invalid inputs
function result = divide(a, b)
    if b == 0
        error('Cannot divide by zero')
    end
    result = a / b;
end

divide(10, 0)  % Clear error message
```

### Mistake 5: Global Variables Causing Unexpected Behavior

Using too many global variables makes code hard to debug.

```matlab
% PROBLEMATIC: Global state
global T
T = 300;

function out = calc1()
    global T
    T = T + 10;  % Modifies global T
    out = T;
end

function out = calc2()
    global T
    out = T^2;   % Depends on what calc1 did
end

% Hard to trace which function changed T!
```

---

## Chemical Engineering Application — Complete Modular Flash Calculation

Combine functions, handles, and selection.

Goal: Given $z_i$, $K_i(T,P)$, find $\psi$ (vapor fraction) using Rachford-Rice, then compute $x_i$, $y_i$.

We create functions:

- `rachford_rice.m` — computes $f(\psi)$
- `flash_calc.m` — solves for $\psi$ using fzero with handle

```matlab
% --- File: rachford_rice.m ---
function f = rachford_rice(psi, z, K)
%RACHFORD_RICE Rachford-Rice function f(psi)= sum z_i*(K_i-1)/(1+psi*(K_i-1))
    f = sum( z.*(K-1) ./ (1 + psi*(K-1)) );
end

% --- File: flash_calc.m ---
function [psi, x, y, status] = flash_calc(z, K)
%FLASH_CALC Isothermal flash using Rachford-Rice
%   Inputs: z mole fractions feed, K K-values
%   Outputs: psi vapor fraction, x liquid, y vapor, status string
    % Check two-phase
    f0 = rachford_rice(0, z, K);
    f1 = rachford_rice(1, z, K);
    
    if f0*f1 > 0
        % No two-phase
        if f0 < 0
            psi = 0; % all liquid
            x = z; y = K.*x;
            status = "All liquid (bubble point)";
        else
            psi = 1; % all vapor
            y = z; x = y./K;
            status = "All vapor (dew point)";
        end
        return;
    end
    
    % Two-phase, solve f(psi)=0 using fzero with anonymous handle capturing z,K
    fun = @(psi) rachford_rice(psi, z, K);
    psi = fzero(fun, 0.5);  % initial guess 0.5
    
    x = z ./ (1 + psi*(K-1));
    y = K.*x;
    status = "Two-phase flash";
end
```

**Main script:**

```matlab
% main_flash.m
clear; clc;

z = [0.5 0.3 0.2];  % feed
K = [2.5 1.2 0.4];  % at T,P

[psi, x, y, status] = flash_calc(z, K);

fprintf('Status: %s\n', status);
fprintf('Vapor fraction psi=%.4f\n', psi);
fprintf('Liquid x: '); disp(x);
fprintf('Vapor y: '); disp(y);
fprintf('Sum x=%.4f sum y=%.4f (should be 1)\n', sum(x), sum(y));

% Test subcooled case: low K
K_low = [0.5 0.3 0.1];
[psi2, x2, y2, status2] = flash_calc(z, K_low);
fprintf('\nLow K case: %s, psi=%.2f\n', status2, psi2);
```

This modular design separates thermodynamic model (`rachford_rice`) from solver logic (`flash_calc`) from data (`main_flash`) — exactly how industrial simulators are structured.

---

## Worked Example: Reactor Simulation Functions

### Problem

Create a set of functions to simulate a batch reactor for a simple first-order irreversible reaction:

$$A \rightarrow B, \quad r = kC_A$$

Integrate the differential equation: $\frac{dC_A}{dt} = -kC_A$

### Solution

```matlab
% Main simulation script
clear; clc;

% Reaction parameters
k = 0.1;            % Rate constant (1/s)
C_A0 = 1.0;         % Initial concentration (mol/L)
t_final = 50;       % Simulation time (s)
dt = 0.1;           % Time step (s)

% Run simulation
[t, C_A, C_B] = batch_reactor_simulation(C_A0, k, t_final, dt);

% Display results
display_results(t, C_A, C_B);

% Plot results
plot_reactor_profile(t, C_A, C_B);

% =================================================================
% FUNCTION: batch_reactor_simulation
% =================================================================
function [t, C_A, C_B] = batch_reactor_simulation(C_A0, k, t_final, dt)
    % Simulate a batch reactor with first-order reaction A -> B
    % Inputs:
    %   C_A0 - initial concentration of A (mol/L)
    %   k - rate constant (1/s)
    %   t_final - simulation end time (s)
    %   dt - time step (s)
    % Outputs:
    %   t - time vector
    %   C_A - concentration of A over time
    %   C_B - concentration of B over time
    
    % Initialize
    n_steps = ceil(t_final / dt);
    t = zeros(1, n_steps);
    C_A = zeros(1, n_steps);
    C_B = zeros(1, n_steps);
    
    % Initial conditions
    C_A(1) = C_A0;
    C_B(1) = 0;
    t(1) = 0;
    
    % Euler method integration
    for i = 2:n_steps
        t(i) = t(i-1) + dt;
        
        % Rate of reaction
        r = k * C_A(i-1);
        
        % Update concentrations (A decreases, B increases)
        C_A(i) = C_A(i-1) - r * dt;
        C_B(i) = C_B(i-1) + r * dt;
        
        % Prevent negative concentrations
        if C_A(i) < 0
            C_A(i) = 0;
        end
    end
end

% =================================================================
% FUNCTION: display_results
% =================================================================
function display_results(t, C_A, C_B)
    % Display reactor simulation results
    % Inputs:
    %   t - time vector (s)
    %   C_A - concentration of A (mol/L)
    %   C_B - concentration of B (mol/L)
    
    disp(''========== BATCH REACTOR SIMULATION RESULTS ==========='')
    disp('' '')
    
    % Final concentrations
    fprintf(''Initial concentration of A: %.2f mol/L\n'', C_A(1))
    fprintf(''Final concentration of A: %.4f mol/L\n'', C_A(end))
    fprintf(''Final concentration of B: %.4f mol/L\n'', C_B(end))
    disp('' '')
    
    % Conversion
    X_A = (C_A(1) - C_A(end)) / C_A(1);
    fprintf(''Conversion of A: %.1f%%\n'', X_A * 100)
    
    % Time for 50% conversion
    idx_half = find(C_A <= C_A(1)/2, 1);
    if ~isempty(idx_half)
        fprintf(''Half-life (50%% conversion): %.2f s\n'', t(idx_half))
    end
end

% =================================================================
% FUNCTION: plot_reactor_profile
% =================================================================
function plot_reactor_profile(t, C_A, C_B)
    % Plot concentration profiles from reactor simulation
    % Inputs:
    %   t - time vector
    %   C_A - concentration of A
    %   C_B - concentration of B
    
    figure(''Position'', [100 100 800 500])
    
    plot(t, C_A, ''b-'', ''LineWidth'', 2, ''DisplayName'', ''A'')
    hold on
    plot(t, C_B, ''r-'', ''LineWidth'', 2, ''DisplayName'', ''B'')
    
    xlabel(''Time (s)'', ''FontSize'', 12)
    ylabel(''Concentration (mol/L)'', ''FontSize'', 12)
    title(''Batch Reactor Concentration Profiles'', ''FontSize'', 14)
    grid on
    legend(''FontSize'', 11, ''Location'', ''best'')
    
    % Add annotations
    text(t(end)*0.6, C_A(1)*0.7, ''A decreases'', ''FontSize'', 11)
    text(t(end)*0.6, C_B(end)*0.7, ''B increases'', ''FontSize'', 11)
end
```

---

## Summary

Functions are the foundation of well-organized, maintainable MATLAB code. Key takeaways:

- **Functions encapsulate** calculations and make code reusable
- **Local variables** are default; they exist only within their function
- **Anonymous functions** are quick and useful for simple calculations
- **Function files** (.m files) are standard for substantial code
- **Input validation** prevents errors downstream
- **Engineering functions** should include clear documentation and error handling
- **Organization** matters: structure your code into logical modules

For chemical engineering applications, build function libraries for thermodynamics, material balances, and process calculations. Your future self will thank you!

---

## Key MATLAB Syntax Reference

| Purpose | Syntax |
|---------|--------|
| Define a function | `function [out1, out2] = funcName(in1, in2)` |
| Anonymous function | `handle = @(x) expression` |
| Local variable | Automatically local inside a function |
| Global variable | `global varName` (inside function) |
| Persistent variable | `persistent varName` (inside function) |
| Function documentation | `% Comments describing inputs/outputs` |
| Call a function | `[out1, out2] = funcName(in1, in2)` |
| Number of inputs | `nargin` (number of arguments in) |
| Function handle | `@functionName` or `@(x) expression` |
| Error handling | `if condition; error('message'); end` |
| Warning message | `warning('message')` |

---

## Homework

### Problem

You are developing MATLAB tools for a chemical engineering lab. Your team needs a set of reusable functions for common thermodynamic calculations.

**Create a function file called `thermodynamicCalculations.m` that includes:**

1. A main function `convertTemperature(T, fromUnit, toUnit)` that converts temperature between Kelvin (K), Celsius (C), and Fahrenheit (F)
   - Inputs: T (temperature), fromUnit (string: 'K', 'C', or 'F'), toUnit (string)
   - Output: T_converted (converted temperature)
   - Error checking: Reject absolute temperatures below 0 K

2. A subfunction `kelvinToCelsius(T_k)` that converts K → °C

3. A subfunction `celsiusToKelvin(T_c)` that converts °C → K

4. A subfunction `celsiusToFahrenheit(T_c)` that converts °C → °F

5. A subfunction `fahrenheitToCelsius(T_f)` that converts °F → °C

**Required tasks:**

- Write the complete function file with proper documentation
- Include error checking for invalid input units and physically impossible temperatures
- Test your function with the following conversions:
  - 298 K to °C
  - 25 °C to K
  - 77 °F to K
- Display results clearly with units

**Concepts being tested:**

- Creating function files (.m files)
- Subfunctions and function organization
- Multiple input/output handling
- Input validation and error checking
- String comparison in MATLAB

**Hints:**

- Use `strcmp()` or `switch`/`case` to compare unit strings
- Remember that 0 K = -273.15 °C = -459.67 °F
- Conversions: °C = K - 273.15; °F = °C × 9/5 + 32

<details>
<summary><strong>Detailed Solution — Click to Reveal</strong></summary>

## Solution

### Concept

This problem teaches:
- Function file structure with main function and subfunctions
- String-based conditional logic to choose conversion paths
- Input validation for physical constraints
- Proper documentation and error handling

### Idea

Create a main function that:
1. Validates the input temperature (checks for absolute zero violations)
2. Routes the conversion based on fromUnit and toUnit
3. Calls appropriate subfunctions to perform conversions
4. Returns the converted temperature

Use subfunctions to handle individual conversions, avoiding code duplication.

### Syntax

**String comparison in MATLAB:**
```matlab
if strcmp(fromUnit, 'K')        % strcmp returns true/false
    % ...
end
```

**Switch statement with strings:**
```matlab
switch fromUnit
    case 'K'
        % ...
    case 'C'
        % ...
end
```

### Syntax Breakdown

- `function [out] = mainFunc(in1, in2)` — Define main function
- `function result = subFunc(input)` — Define subfunction (inside same file)
- `if strcmp(str1, str2)` — Compare strings
- `error('message')` — Throw an error and stop execution
- `warning('message')` — Display a warning but continue

### MATLAB Code

```matlab
% Filename: thermodynamicCalculations.m

function T_converted = convertTemperature(T, fromUnit, toUnit)
    % Convert temperature between Kelvin, Celsius, and Fahrenheit
    %
    % Inputs:
    %   T (scalar): temperature value
    %   fromUnit (string): 'K' (Kelvin), 'C' (Celsius), 'F' (Fahrenheit)
    %   toUnit (string): 'K', 'C', or 'F'
    %
    % Output:
    %   T_converted (scalar): converted temperature
    %
    % Example:
    %   T_celsius = convertTemperature(298, 'K', 'C');
    
    % Input validation
    if ~isnumeric(T) || ~ischar(fromUnit) || ~ischar(toUnit)
        error('Invalid input types: T must be numeric, units must be strings')
    end
    
    % Check for absolute zero violation
    switch fromUnit
        case 'K'
            if T < 0
                error('Temperature cannot be below absolute zero (0 K)')
            end
        case 'C'
            if T < -273.15
                error('Temperature cannot be below absolute zero (-273.15 °C)')
            end
        case 'F'
            if T < -459.67
                error('Temperature cannot be below absolute zero (-459.67 °F)')
            end
        otherwise
            error('Unknown unit: %s. Use K, C, or F', fromUnit)
    end
    
    % Convert from input unit to Celsius (intermediate step)
    switch fromUnit
        case 'K'
            T_celsius = kelvinToCelsius(T);
        case 'C'
            T_celsius = T;
        case 'F'
            T_celsius = fahrenheitToCelsius(T);
        otherwise
            error('Unknown unit: %s. Use K, C, or F', fromUnit)
    end
    
    % Convert from Celsius to output unit
    switch toUnit
        case 'K'
            T_converted = celsiusToKelvin(T_celsius);
        case 'C'
            T_converted = T_celsius;
        case 'F'
            T_converted = celsiusToFahrenheit(T_celsius);
        otherwise
            error('Unknown unit: %s. Use K, C, or F', toUnit)
    end
end

% ========== Subfunctions ==========

function T_c = kelvinToCelsius(T_k)
    % Convert Kelvin to Celsius: °C = K - 273.15
    T_c = T_k - 273.15;
end

function T_k = celsiusToKelvin(T_c)
    % Convert Celsius to Kelvin: K = °C + 273.15
    T_k = T_c + 273.15;
end

function T_f = celsiusToFahrenheit(T_c)
    % Convert Celsius to Fahrenheit: °F = °C × 9/5 + 32
    T_f = T_c * 9/5 + 32;
end

function T_c = fahrenheitToCelsius(T_f)
    % Convert Fahrenheit to Celsius: °C = (°F - 32) × 5/9
    T_c = (T_f - 32) * 5/9;
end
```

### Code Explanation

1. **Main function signature:** `convertTemperature(T, fromUnit, toUnit)` takes temperature and two unit strings

2. **Input validation:** Checks that inputs are numeric/strings and that temperature isn't below absolute zero for each unit system

3. **Two-step conversion:** First converts from any unit to Celsius (common intermediate), then to the target unit

4. **Subfunctions:** Each handles one conversion direction with a simple formula

5. **Error handling:** Invalid units or temperatures trigger `error()`, stopping execution with a descriptive message

### Line-by-Line Explanation

```matlab
% Check input types
if ~isnumeric(T) || ~ischar(fromUnit) || ~ischar(toUnit)
    error(...)
end
```
The `~` operator means "NOT"; `~isnumeric(T)` is true if T is NOT numeric.

```matlab
% Check absolute zero for Kelvin
if T < 0
    error('Temperature cannot be below absolute zero (0 K)')
end
```
In Kelvin, absolute zero is 0 K; any negative value is impossible.

```matlab
switch fromUnit
    case 'K'
        T_celsius = kelvinToCelsius(T);
    case 'C'
        T_celsius = T;
    case 'F'
        T_celsius = fahrenheitToCelsius(T);
end
```
Route to the appropriate subfunction based on input unit.

### Expected Result

Running the test cases:

```matlab
% Test 1: 298 K to Celsius
T1 = convertTemperature(298, 'K', 'C');
fprintf('298 K = %.2f °C\n', T1)

% Test 2: 25 Celsius to Kelvin
T2 = convertTemperature(25, 'C', 'K');
fprintf('25 °C = %.2f K\n', T2)

% Test 3: 77 Fahrenheit to Kelvin
T3 = convertTemperature(77, 'F', 'K');
fprintf('77 °F = %.2f K\n', T3)
```

**Output:**
```
298 K = 24.85 °C
25 °C = 298.15 K
77 °F = 298.15 K
```

### Engineering Interpretation

These conversions are critical in chemical engineering:

- **298 K (≈ 25 °C):** Standard reference temperature (STP) used in thermodynamic tables
- **Consistency check:** 77 °F also equals 25 °C, which matches our 298 K result (with rounding), confirming our conversions are correct
- **Engineering use:** Quickly look up properties at standard conditions or convert experimental data from different temperature scales

The function's flexibility allows you to convert between any pair of units without writing separate code for each combination.

</details>

---
