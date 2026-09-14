# Control Structures

## Learning Objectives

By the end of this lecture, you should be able to:

- Write and understand `if`, `elseif`, and `else` conditional statements
- Use `switch`-`case` structures for multi-way selection
- Implement `for` and `while` loops with proper syntax
- Use `break` and `continue` statements to control loop flow
- Nest conditional statements and loops for complex logic
- Apply conditional logic to engineering problems (iterative calculations, process parameter decisions)
- Write chemical engineering computations that involve conditional decisions and repetitive calculations

---

## Why This Topic Matters

Real-world chemical engineering problems rarely follow a single, linear path. Process calculations often require:

- **Decisions**: "If the temperature exceeds the setpoint, adjust cooling." "If the reaction hasn't reached equilibrium, continue iterations."
- **Repetition**: "Calculate the conversion in a reactor over 1000 time steps." "Read sensor data in a loop until the experiment ends."
- **Complex Logic**: Nested decisions and loops work together to solve realistic problems.

Control structures allow you to write programs that:

- Automatically respond to different conditions
- Perform calculations repeatedly without rewriting the same code
- Handle process variability and real-time decisions
- Simulate time-dependent phenomena like reactor dynamics or batch processes

Without control structures, you would be limited to writing scripts that always do the same thing, regardless of the data or conditions. Control structures are essential for building realistic engineering applications.

---

## Conditional Statements

### What Are Conditional Statements?

Conditional statements allow your code to execute different code blocks depending on whether a logical condition is true or false. In MATLAB, the primary conditional statement is the **`if`-`elseif`-`else`** structure.


### MATLAB Syntax: `if`-`elseif`-`else`

```matlab
if condition1
    % code executed if condition1 is true
elseif condition2
    % code executed if condition1 is false AND condition2 is true
elseif condition3
    % code executed if condition1 and condition2 are false AND condition3 is true
else
    % code executed if all previous conditions are false
end
```

### Syntax Breakdown

| Component | Meaning |
|-----------|---------|
| `if` | Keyword that starts the conditional block |
| `condition1` | A logical expression that evaluates to true or false |
| `% code` | MATLAB statements executed only if the condition is true |
| `elseif` | Keyword for additional conditions (optional, can have multiple) |
| `else` | Keyword for the default case when all conditions are false (optional) |
| `end` | Keyword that closes the conditional block (required) |

### Nested Conditions

Conditions can be placed inside other conditions. Keep nesting shallow and use indentation for readability.

```matlab
T = 455;
P = 5.5;

if T > 450
    if P > 5
        fprintf('Both temperature and pressure are high\n');
    else
        fprintf('Only temperature is high\n');
    end
else
    fprintf('Temperature is acceptable\n');
end
```
---

### Simple Example: Temperature Control

```matlab
T = 350;            % Temperature in K
T_setpoint = 320;   % Setpoint in K

if T > T_setpoint
    disp('Temperature too high. Cooling required.')
elseif T < T_setpoint - 5
    disp('Temperature too low. Heating required.')
else
    disp('Temperature within acceptable range.')
end
```

**Output:**
```
Temperature too high. Cooling required.
```

### Example Walkthrough

1. `T = 350` — Store the current temperature.
2. `T_setpoint = 320` — Define the desired setpoint.
3. `if T > T_setpoint` — Check: Is T (350) greater than 320? Yes, so this block executes.
4. `disp('Temperature too high. Cooling required.')` — Print a message.
5. The `elseif` and `else` blocks are skipped because the `if` condition was true.
6. `end` — Close the conditional structure.

### More Complex Example: Reactor Feed Decision

```matlab
% Material balance: decide feed flow based on tank level
h = 2.5;            % Current tank height in meters
h_min = 1.0;        % Minimum safe height in meters
h_max = 3.5;        % Maximum safe height in meters
F_in = 0;           % Inlet flow rate in m³/s

if h < h_min
    F_in = 0.5;     % Increase feed
    status = 'Low level: feeding';
elseif h > h_max
    F_in = 0.1;     % Reduce feed (outlet still drains)
    status = 'High level: reducing feed';
else
    F_in = 0.3;     % Normal operation
    status = 'Normal operation';
end

disp(['Tank height: ' num2str(h) ' m'])
disp(['Feed flow: ' num2str(F_in) ' m³/s'])
disp(['Status: ' status])
```

**Output:**
```
Tank height: 2.5 m
Feed flow: 0.3 m³/s
Status: Normal operation
```
---
### Chemical Engineering Example — Property Correlation Selection

Antoine coefficients change with temperature range. Choose correct set.

```matlab
% Water Antoine coefficients (example, P in mmHg, T in °C)
T_C = 85;  % °C

if T_C < 0
    disp('Invalid: below freezing');
elseif T_C <= 100
    % Low range coefficients
    A = 8.07131; B = 1730.63; C = 233.426;
    Psat = 10^(A - B/(T_C + C));
    fprintf('Low range: Psat = %.1f mmHg at %.0f C\n', Psat, T_C);
elseif T_C <= 200
    % High range
    A = 8.14019; B = 1810.94; C = 244.485;
    Psat = 10^(A - B/(T_C + C));
    fprintf('High range: Psat = %.1f mmHg at %.0f C\n', Psat, T_C);
else
    disp('Temperature above correlation range');
end
```

**Why conditional matters:** Using low-range coefficients at 150°C gives 2x error — conditional ensures correct physical model.

**Second example — Flow regime check:**

```matlab
Re = 2500;  % Reynolds number
if Re < 2100
    regime = "Laminar";
    f = 64/Re;  % friction factor laminar
elseif Re <= 4000
    regime = "Transitional";
    f = 0.316*Re^-0.25;  % approximate Blasius
else
    regime = "Turbulent";
    f = 0.316*Re^-0.25;  % same for demo, actually Colebrook needed
end
fprintf('Re=%.0f: %s, f=%.4f\n', Re, regime, f);
```
---


### Common Mistakes

1. **Forgetting `end`**: Every `if` must have a closing `end`.
   ```matlab
   % WRONG
   if T > 350
       disp('Too hot')  % Missing end
   ```

2. **Using `=` instead of `==` in a condition**:
   ```matlab
   % WRONG
   if T = 350
       % This assigns, not compares!
   end
   
   % CORRECT
   if T == 350
       % This compares.
   end
   ```

3. **Not understanding operator precedence**: Use parentheses to be clear.
   ```matlab
   % Ambiguous
   if T > 300 & P < 5 | R == 2
   
   % Clear
   if (T > 300 && P < 5) || (R == 2)
   ```

4. **Forgetting that conditions must produce a scalar logical value** when using `if`:
   ```matlab
   % WRONG - creates an array of logicals
   T = [300 350 400];
   if T > 325
       % Error: condition must be scalar
   end
   ```

---

## Selection Structures: `switch`-`case`

### What Is a Switch Statement?

A `switch` statement is useful when you have one variable that can take multiple discrete values, and you want to execute different code for each value. It's an alternative to multiple `if-elseif-else` blocks.

### MATLAB Syntax

```matlab
switch variable
    case value1
        % code executed if variable == value1
    case value2
        % code executed if variable == value2
    case {value3, value4, value5}
        % code executed if variable equals any of these values
    otherwise
        % code executed if no case matches
end
```

### Syntax Breakdown

| Component | Meaning |
|-----------|---------|
| `switch` | Keyword that starts the selection structure |
| `variable` | The expression being tested (typically a scalar or string) |
| `case value` | Label for a specific value; code below executes if variable matches |
| `{value1, value2}` | Multiple values can be grouped using cell array syntax |
| `otherwise` | Default case if no case matches (equivalent to `else`) |
| `end` | Closes the `switch` structure |

### Simple Example: Feed Stream Type

```matlab
feed_type = 'reactant';

switch feed_type
    case 'reactant'
        concentration = 1.5;    % mol/L
        fprintf('Feed: Reactant solution at %.2f mol/L\n', concentration)
    case 'solvent'
        concentration = 0;
        fprintf('Feed: Pure solvent\n')
    case 'catalyst'
        concentration = 0.01;
        fprintf('Feed: Catalyst solution at %.2f mol/L\n', concentration)
    otherwise
        fprintf('Unknown feed type\n')
end
```

**Output:**
```
Feed: Reactant solution at 1.50 mol/L
```

### Example Walkthrough

1. `feed_type = 'reactant'` — Store the feed type.
2. `switch feed_type` — Begin the selection structure.
3. `case 'reactant'` — Check if `feed_type` equals `'reactant'`. It does, so this block executes.
4. The remaining `case` blocks are skipped.
5. `end` — Close the structure.

### More Complex Example: Unit Conversion

```matlab
quantity = 5.2;
fromUnit = "bar";
toUnit   = "Pa";

switch fromUnit
    case "bar"
        SI = quantity * 1e5;          % → Pa
    case "atm"
        SI = quantity * 101325;
    case "psi"
        SI = quantity * 6894.76;
    otherwise
        error('Unsupported pressure unit');
end

switch toUnit
    case "Pa"
        result = SI;
    case "kPa"
        result = SI / 1000;
    case "MPa"
        result = SI / 1e6;
    otherwise
        error('Unsupported target unit');
end

fprintf('%.4g %s = %.4g %s\n', quantity, fromUnit, result, toUnit);
```

---

### Chemical Engineering Example — Species Property Database

```matlab
species = "Benzene";

switch species
    case "Benzene"
        MW = 78.11;  % g/mol
        Tb = 80.1;   % C
        Antoine_A = 6.90565; % etc.
    case "Toluene"
        MW = 92.14;
        Tb = 110.6;
        Antoine_A = 6.95464;
    case {"Xylene", "p-Xylene", "o-Xylene"}
        MW = 106.16;
        Tb = 144.4;  % approx
        Antoine_A = 6.99052;
    otherwise
        MW = NaN;
        Tb = NaN;
        fprintf('Species %s not in database\n', species);
end

if ~isnan(MW)
    fprintf('%s: MW=%.2f g/mol, Tb=%.1f C\n', species, MW, Tb);
end
```

**Why `switch` vs `if`?**

- Use `switch` when comparing one variable against many constant values (reactor type, species name, unit operation)
- Use `if` when conditions involve ranges (`T>400`), logical combinations (`T>400 & P>8`)

---

### **Nested switch:** Can be inside `if` and vice versa.

```matlab
phase = "vapor"; species = "Benzene";

if phase == "vapor"
    switch species
        case "Benzene"
            % vapor properties
        case "Toluene"
            % ...
    end
else
    switch species
        case "Benzene"
            % liquid properties
    end
end
```

### Common Mistakes

1. **Forgetting `end`**: Every `switch` must have a closing `end`.

2. **Not using braces for multiple values**: To match multiple values with one case, use cell arrays:
   ```matlab
   % CORRECT
   case {1, 2, 3}
       % code
   end
   ```

3. **Forgetting to use `otherwise`**: If no case matches and you don't have `otherwise`, nothing happens.

4. **Using `switch` with vectors**: `switch` works only with scalars or strings, not arrays:
   ```matlab
   % WRONG
   T = [300 350 400];
   switch T
       % This fails because T is a vector
   ```

---

## Loops: `for` and `while`

### Introduction to Loops

A **loop** allows you to repeat a block of code multiple times. This is essential for:

- Processing arrays element by element
- Simulating time-stepping calculations 
- Iterative numerical methods (Newton-Raphson, bisection)
- Reading experimental data points
- Accumulating sums or products

### The `for` Loop

#### When Should You Use `for`?

Use `for` loops when:

- You know in advance how many times you need to repeat the code
- You need to iterate over a known number of time steps or array elements
- You're calculating something for each point in a data set

#### MATLAB Syntax

```matlab
for index = start:step:end
    % code to repeat
end
```

Or equivalently:

```matlab
for index = array
    % code to repeat, index takes each value from array
end
```

#### Syntax Breakdown

| Component | Meaning |
|-----------|---------|
| `for` | Keyword that starts the loop |
| `index` | Loop variable (counter); takes a new value each iteration |
| `start:step:end` | Range syntax: start value, increment, end value |
| `array` | Alternative: index iterates through elements of array |
| `% code` | Statements executed repeatedly |
| `end` | Closes the loop |

---
#### Simple Example

```matlab
% Sum 1 to 5
total = 0;
for i = 1:5
    total = total + i;
end
disp(total)  % 15

% Loop over temperature vector
T_vec = [300 350 400 450];
k_vec = zeros(size(T_vec));  % preallocate
A = 1e7; Ea = 60000; R = 8.314;
for idx = 1:length(T_vec)
    T = T_vec(idx);
    k_vec(idx) = A*exp(-Ea/(R*T));
end
disp(k_vec)

% Better: vectorized (no loop) for this case, but loop useful when each iteration complex
% Loop over species names
speciesList = ["Benzene" "Toluene" "Xylene"];
for s = speciesList
    fprintf('Processing %s\n', s);
end
```
---

#### Example: Calculate Conversion Over Time

```matlab
% Simulate a first-order reaction: dC/dt = -k*C
k = 0.15;           % Rate constant in 1/min
C0 = 2.0;           % Initial concentration in mol/L
C = C0;
dt = 0.5;           % Time step in min

fprintf('Time (min) \t | \t Concentration (mol/L)\n')
fprintf('-------------------------------------------------\n')

for t = 0:dt:5      % Time from 0 to 5 minutes in 0.5 min increments
    fprintf('%8.1f \t | \t %8.4f\n', t, C)
    C = C - k * C * dt;  % Simple Euler integration step
end
```

**Output:**
```
Time (min) 	 | 	 Concentration (mol/L)
-------------------------------------------------
     0.0 	 | 	   2.0000
     0.5 	 | 	   1.8500
     1.0 	 | 	   1.7113
     1.5 	 | 	   1.5829
     2.0 	 | 	   1.4642
     2.5 	 | 	   1.3544
     3.0 	 | 	   1.2528
     3.5 	 | 	   1.1588
     4.0 	 | 	   1.0719
     4.5 	 | 	   0.9915
     5.0 	 | 	   0.9172
```

#### Example Walkthrough

1. `k = 0.15` — Define the rate constant.
2. `C = C0` — Initialize concentration to the initial value.
3. `for t = 0:dt:5` — Loop: set `t` to 0, then 0.5, then 1.0, ... until 5.0.
4. `fprintf(...)` — Print the current time and concentration.
5. `C = C - k * C * dt` — Update concentration using Euler's method.
6. Loop repeats with the next value of `t`.
7. `end` — Close the loop.

#### More Complex Example: Array Iteration

```matlab
% Calculate conversion in a series of isothermal reactors
F_A = 5.0;              % Inlet molar flow rate of A in mol/s
k = 0.5;                % Rate constant in L/(mol·s)
V = [2, 2, 2];          % Reactor volumes in L
C_A = F_A / (5);        % Initial concentration (assume 5 L/s total volumetric flow)

fprintf('Reactor | Volume (L) | Conversion (%%) | Exit Conc (mol/L)\n')
fprintf('--------|------------|--------|---------------------\n')

for i = 1:length(V)
    % Calculate steady-state conversion in reactor i
    % Using Levenspiel plot concept: tau = V/F_vol
    tau = V(i) / 5;                        % Residence time in s
    C_A_exit = C_A / (1 + k * C_A * tau);  % Exit concentration
    X_A = (C_A - C_A_exit) / C_A * 100;    % Conversion in %
    
    fprintf('   %d    |    %5.1f    | %6.2f  | %8.4f\n', i, V(i), X_A, C_A_exit)
    C_A = C_A_exit;     % Exit of reactor i becomes inlet to reactor i+1
end
```

**Output:**
```
Reactor | Volume (L) | Conversion (%) | Exit Conc (mol/L)
--------|------------|--------|---------------------
   1    |      2.0    |  16.67  |   0.8333
   2    |      2.0    |  14.29  |   0.7143
   3    |      2.0    |  12.50  |   0.6250
```

#### Chemical Engineering Application: Temperature Profile in a Tubular Reactor

```matlab
% Calculate temperature profile in a plug flow reactor with heat transfer
L = 10;                 % Reactor length in m
n_nodes = 11;           % Number of calculation points
dz = L / (n_nodes - 1); % Spatial step in m
T0 = 300;               % Inlet temperature in K
U = 50;                 % Heat transfer coefficient in W/(m²·K)
T_wall = 350;           % Cooling jacket temperature in K
A = 0.1;                % Cross-sectional area in m²
rho_Cp = 1000;          % Heat capacity per unit volume in J/(m³·K)
v = 2;                  % Velocity in m/s

z = 0:dz:L;             % Position array
T = zeros(size(z));     % Temperature array
T(1) = T0;              % Inlet temperature

fprintf('Position (m) | Temperature (K)\n')
fprintf('-------------|----------------\n')

for i = 1:n_nodes-1
    % Heat transfer term: dT/dz = -(U*A/(rho*Cp*v*A)) * (T - T_wall)
    dT_dz = -(U / (rho_Cp * v)) * (T(i) - T_wall);
    T(i+1) = T(i) + dT_dz * dz;
    fprintf('   %8.3f   |    %8.2f\n', z(i), T(i))
end
fprintf('   %8.3f   |    %8.2f\n', z(n_nodes), T(n_nodes))
```

**Output:**
```
Position (m) | Temperature (K)
-------------|----------------
      0.000   |      300.00
      1.000   |      301.25
      2.000   |      302.47
      3.000   |      303.66
      4.000   |      304.82
      5.000   |      305.95
      6.000   |      307.05
      7.000   |      308.12
      8.000   |      309.17
      9.000   |      310.19
     10.000   |      311.18
```
---
### **Preallocation:** Critical for speed. Without preallocation, MATLAB resizes array each iteration.

```matlab
% Slow (growing)
A = [];
for i=1:1000
    A = [A, i];  % reallocates each time
end

% Fast (preallocated)
A = zeros(1,1000);
for i=1:1000
    A(i) = i;
end
```

### Nested For Loops

Loop inside loop — for 2D matrices, reactor networks, etc.

```matlab
% Matrix multiplication manual (for teaching, not for real use)
A = [1 2; 3 4]; B = [5 6; 7 8];
C = zeros(2,2);
for i=1:2
    for j=1:2
        for k=1:2
            C(i,j) = C(i,j) + A(i,k)*B(k,j);
        end
    end
end
disp(C)  % should equal A*B

% Chemical: concentration matrix reactors x time
C_matrix = zeros(3,4);  % 3 reactors, 4 times
for reactor = 1:3
    for t = 1:4
        C_matrix(reactor, t) = 1.0 / (reactor + t);  % dummy calc
    end
end
```

---

### The `while` Loop

#### When Should You Use `while`?

Use `while` loops when:

- You don't know in advance how many iterations are needed
- You want to continue until a condition is met (e.g., "repeat until convergence")
- You want to iterate based on a logical condition rather than a fixed counter
- You're implementing iterative numerical methods (Newton-Raphson)

#### MATLAB Syntax

```matlab
while condition
    % code to repeat
end
```

#### Syntax Breakdown

| Component | Meaning |
|-----------|---------|
| `while` | Keyword that starts the loop |
| `condition` | Logical expression; loop continues as long as this is true |
| `% code` | Statements executed repeatedly |
| `end` | Closes the loop |

---
#### Simple Example

```matlab
% Successive halving until <0.01
x = 1;
while x > 0.01
    x = x/2;
    fprintf('x=%.4f\n', x);
end

% Newton-Raphson like iteration (simplified)
% Solve x^2 = 2, guess x=1, iterate x_new = (x + 2/x)/2
x = 1;
tol = 1e-6;
iter = 0;
while true
    iter = iter + 1;
    x_new = (x + 2/x)/2;
    if abs(x_new - x) < tol
        break;  % exit loop
    end
    x = x_new;
    if iter > 100
        disp('Not converging');
        break;
    end
end
fprintf('sqrt(2) ~ %.8f in %d iterations\n', x_new, iter);
```

---

### Counting with while Loops

```matlab
count = 0;
max_count = 10;

while count < max_count
    disp(count);
    count = count + 1;
end
```

---
#### Simple Example: Iterative Calculation Until Convergence

```matlab
% Find the steady-state conversion in a CSTR using iterative method
X = 0.0;            % Initial guess for conversion
tau = 5.0;          % Residence time in s
k = 0.2;            % Rate constant in 1/s
tolerance = 1e-4;   % Convergence criterion
max_iter = 100;     % Safety limit
iteration = 0;

fprintf('Iteration | Conversion \t | Change\n')
fprintf('---------|---------------|-------\n')

while iteration < max_iter
    iteration = iteration + 1;
    X_old = X;
    % CSTR steady-state: X = (k*tau*X) / (1 + k*tau*(1-X))
    % Rearranged for fixed-point iteration
    X = k * tau / (1 + k * tau);
    
    change = abs(X - X_old);
    fprintf(' %d \t | %.5f \t | %.2e\n', iteration, X, change)
    
    if change < tolerance
        fprintf('\nConverged after %d iterations\n', iteration)
        break
    end
end
```

**Output:**
```
Iteration | Conversion 	 | Change
---------|---------------|-------
 1 	 | 0.50000 	 | 5.00e-01
 2 	 | 0.50000 	 | 0.00e+00

Converged after 2 iterations
```

#### Example Walkthrough

1. Initialize `X = 0.0` as an initial guess.
2. Enter the `while` loop because `iteration < max_iter` is true.
3. Increment `iteration`.
4. Calculate the new conversion `X`.
5. Calculate the change `change`.
6. Print the result.
7. If `change < tolerance`, execute `break` to exit the loop.
8. Otherwise, repeat.

#### More Complex Example: Newton-Raphson Root Finding

```matlab
% Find the root of: f(x) = x^3 - 6*x^2 + 11*x - 6
% Derivative: f'(x) = 3*x^2 - 12*x + 11
x = 3.5;                % Initial guess
tolerance = 1e-6;
max_iter = 100;
iteration = 0;

fprintf('Iteration |    x      |    f(x)    | Change\n')
fprintf('----------|-----------|------------|--------\n')

while iteration < max_iter
    iteration = iteration + 1;
    x_old = x;
    
    % Function and derivative
    f = x^3 - 6*x^2 + 11*x - 6;
    f_prime = 3*x^2 - 12*x + 11;
    
    % Newton-Raphson update
    x = x - f / f_prime;
    
    change = abs(x - x_old);
    fprintf('   %3d    |  %7.5f  |  %9.2e | %8.2e\n', iteration, x, f, change)
    
    if change < tolerance
        fprintf('\nRoot found: x = %.6f\n', x)
        break
    end
end
```

**Output:**
```
Iteration |    x      |    f(x)    | Change
----------|-----------|------------|--------
     1    |  3.17391  |   1.88e+00 | 3.26e-01
     2    |  3.03231  |   4.44e-01 | 1.42e-01
     3    |  3.00146  |   6.78e-02 | 3.09e-02
     4    |  3.00000  |   2.92e-03 | 1.45e-03
     5    |  3.00000  |   6.34e-06 | 3.17e-06
     6    |  3.00000  |   3.01e-11 | 1.51e-11

Root found: x = 3.000000
```

#### Chemical Engineering Application: Iterative Isothermal Flash Calculation

```matlab
% Solve isothermal flash using iterative bubble-point method
% Simplified: Binary system with given compositions and K-values

T = 350;            % Temperature in K
P = 10;             % Pressure in bar
x_A = 0.4;          % Liquid mole fraction of A (known)
K_A = 2.5;          % K-value for A (property, known)
K_B = 0.6;          % K-value for B (property, known)

% Solve for vapor fraction V/F using Rachford-Rice equation
% Sum(z_i*(K_i-1)/(1 + V/F*(K_i-1))) = 0
z_A = 0.5;          % Feed composition
z_B = 0.5;
V_F = 0.5;          % Initial guess for V/F
tolerance = 1e-5;
iteration = 0;

fprintf('Iteration |   V/F    | Residual\n')
fprintf('----------|----------|----------\n')

while iteration < 50
    iteration = iteration + 1;
    V_F_old = V_F;
    
    % Rachford-Rice equation residual
    residual = z_A * (K_A - 1) / (1 + V_F * (K_A - 1)) + ...
               z_B * (K_B - 1) / (1 + V_F * (K_B - 1));
    
    % Derivative for Newton-Raphson
    dR_dVF = -z_A * (K_A - 1)^2 / (1 + V_F * (K_A - 1))^2 - ...
             z_B * (K_B - 1)^2 / (1 + V_F * (K_B - 1))^2;
    
    V_F = V_F - residual / dR_dVF;
    
    % Ensure V_F stays in valid range
    V_F = max(0.001, min(0.999, V_F));
    
    fprintf('   %3d    | %8.5f | %9.2e\n', iteration, V_F, residual)
    
    if abs(V_F - V_F_old) < tolerance
        x_A = z_A / (1 + V_F * (K_A - 1));
        y_A = K_A * x_A;
        fprintf('\nConverged!\n')
        fprintf('Vapor fraction (V/F): %.4f\n', V_F)
        fprintf('Liquid composition: x_A = %.4f\n', x_A)
        fprintf('Vapor composition: y_A = %.4f\n', y_A)
        break
    end
end
```

**Output:**
```
Iteration |   V/F    | Residual
----------|----------|----------
     1    |  0.86269 |  1.79e-01
     2    |  0.91654 |  2.16e-02
     3    |  0.91667 |  5.00e-05
     4    |  0.91667 |  6.26e-13

Converged!
Vapor fraction (V/F): 0.9167
Liquid composition: x_A = 0.2105
Vapor composition: y_A = 0.5263
```

#### Chemical Engineering Example — Recycle Convergence

Classic: reactor with recycle, need to iterate until recycle flow converges.

```matlab
% Simple recycle: Fresh feed F0=100 mol/s, reactor conversion 0.8,
% separator splits 90% product, 10% recycle, recycle mixes with fresh feed

F0 = 100;  % mol/s fresh feed
X = 0.8;   % conversion per pass
split = 0.9; % fraction to product

% Initial guess
F_recycle = 0;  % mol/s
tol = 0.01;     % mol/s tolerance
maxIter = 100;
iter = 0;
converged = false;

while ~converged
    iter = iter + 1;
    F_reactor_in = F0 + F_recycle;
    F_reactor_out = F_reactor_in * (1 - X);  % unreacted left
    F_recycle_new = F_reactor_out * (1 - split);  % recycle is 10% of reactor out
    
    % Check convergence
    if abs(F_recycle_new - F_recycle) < tol
        converged = true;
    end
    
    F_recycle = F_recycle_new;
    
    fprintf('Iter %d: F_recycle=%.3f\n', iter, F_recycle);
    
    if iter > maxIter
        warning('Recycle not converged in %d iterations', maxIter);
        break;
    end
end

fprintf('Converged recycle flow: %.3f mol/s\n', F_recycle);
```

**Why `while` not `for`?** We don't know iterations needed — depends on tolerance and conversion. `while` continues until condition met.


---

### Loop Control: `break` and `continue`

#### `break` Statement

The `break` statement immediately exits the innermost loop.

**Syntax:**
```matlab
break;
```

**Example:**
```matlab
for i = 1:100
    if i == 5
        break;  % Exit loop when i equals 5
    end
    disp(i)
end
```

**Output:**
```
1
2
3
4
```

#### `continue` Statement

The `continue` statement skips the rest of the current iteration and proceeds to the next iteration.

**Syntax:**
```matlab
continue;
```

**Example:**
```matlab
for i = 1:5
    if i == 3
        continue;  % Skip i=3, go directly to i=4
    end
    disp(i)
end
```

**Output:**
```
1
2
4
5
```

#### Chemical Engineering Application: Screening Reactor Conditions

```matlab
% Break on safety violation
T_profile = [300 350 410 390 380];
for i=1:length(T_profile)
    T = T_profile(i);
    if T > 400
        fprintf('Alarm at index %d: T=%.0f K, breaking\n', i, T);
        break;  % stop checking further
    end
    fprintf('T %d OK: %.0f K\n', i, T);
end

% Continue: skip invalid data
conc = [1.0 0.8 NaN 0.6 0.5];
sumValid = 0; countValid = 0;
for i=1:length(conc)
    if isnan(conc(i))
        continue;  % skip NaN
    end
    sumValid = sumValid + conc(i);
    countValid = countValid + 1;
end
meanValid = sumValid / countValid;
fprintf('Mean valid conc: %.3f\n', meanValid);
```

**When to use:**

- `break` for safety, convergence, error — stop loop when further iterations meaningless
- `continue` for data cleaning — skip bad points

### Engineering Applications — Iterative Calculations — Bubble Point

Bubble point temperature requires iteration because $K_i$ depends on $T$.

Simplified: binary mixture, Antoine for Psat, Raoult's law, bubble $T$ when $\sum x_i P_{sat,i}(T) = P$.

```matlab
% Bubble point of Benzene-Toluene at 1 atm, x_Bz=0.4
P_total = 760;  % mmHg
x_Bz = 0.4; x_Tol = 0.6;

% Antoine coefficients (P mmHg, T C): log10(Psat)=A-B/(T+C)
A_Bz=6.90565; B_Bz=1211.033; C_Bz=220.79;
A_Tol=6.95464; B_Tol=1344.8; C_Tol=219.482;

% Function to compute total vapor pressure at T
calcP = @(T_C) x_Bz*10^(A_Bz - B_Bz/(T_C + C_Bz)) + x_Tol*10^(A_Tol - B_Tol/(T_C + C_Tol));

% Iterate T
T_guess = 90;  % C
tol = 0.01;  % mmHg tolerance on pressure
iter=0;
while true
    iter=iter+1;
    P_calc = calcP(T_guess);
    err = P_calc - P_total;
    fprintf('Iter %d: T=%.2f C, P_calc=%.1f, err=%.2f\n', iter, T_guess, P_calc, err);
    if abs(err) < tol
        break;
    end
    % Simple update: if P_calc > P_total, T too high, reduce
    T_guess = T_guess - 0.1*err/10;  % crude proportional update
    if iter>100
        disp('Not converged');
        break;
    end
end
fprintf('Bubble point: %.2f C\n', T_guess);
```

This is precursor to `fzero` in later lectures, but shows `while` logic.

---
## Worked Example: Iterative Root Finding (Newton-Raphson Method)

### Problem

Find the root of the equation:

$$f(x) = x^3 - 2x - 5 = 0$$

Using the Newton-Raphson method, which iterates:

$$x_{n+1} = x_n - \frac{f(x_n)}{f'(x_n)}$$

where $f''(x) = 3x^2 - 2$ is the derivative.

### Solution

```matlab
% Newton-Raphson Root Finding
% Equation: f(x) = x^3 - 2x - 5 = 0
% Derivative: f"(x) = 3x^2 - 2

% Define the function and its derivative
f = @(x) x^3 - 2*x - 5;
f_prime = @(x) 3*x^2 - 2;

% Initial guess and convergence criteria
x = 2.0;                        % Initial guess
tolerance = 1e-6;              % Convergence tolerance
max_iterations = 100;
iterations = 0;
converged = false;

% Display header
disp("Iteration | x_current | f(x) | |f(x)|")
disp("=========================================")

% Newton-Raphson iteration
while iterations < max_iterations
    f_x = f(x);
    f_prime_x = f_prime(x);
    
    % Display iteration info
    fprintf("%9d | %9.6f | %8.4f | %8.4e\n", iterations, x, f_x, abs(f_x))
    
    % Check convergence
    if abs(f_x) < tolerance
        converged = true;
        break;
    end
    
    % Newton-Raphson update
    x = x - f_x / f_prime_x;
    iterations = iterations + 1;
end

% Report results
disp(" ")
if converged
    disp("CONVERGED")
    fprintf("Root found: x = %.8f\n", x)
    fprintf("f(root) = %.2e\n", f(x))
    fprintf("Iterations: %d\n", iterations)
else
    disp("DID NOT CONVERGE")
    fprintf("Final x = %.8f after %d iterations\n", x, iterations)
end

% Verify the solution
disp(" ")
disp("Verification:")
fprintf("f(%.6f) = %.2e\n", x, f(x))
```

**Output:**

```
Iteration | x_current | f(x) | |f(x)|
=========================================
        0 |  2.000000 |  -1.0000 | 1.0000e+00
        1 |  2.100000 |   0.0610 | 6.1000e-02
        2 |  2.094568 |   0.0002 | 1.8572e-04
        3 |  2.094551 |   0.0000 | 1.7398e-09
 
CONVERGED
Root found: x = 2.09455148
f(root) = 1.74e-09
Iterations: 3
 
Verification:
f(2.094551) = 1.74e-09
```

---

### Common Loop Mistakes

1. **Infinite loops**: A `while` loop that never becomes false:
   ```matlab
   % WRONG - this runs forever
   while true
       disp('This never stops')
   end
   ```
   **Fix:** Use a counter or convergence criterion to ensure the loop terminates.

2. **Off-by-one errors**: Starting or ending at the wrong index:
   ```matlab
   % WRONG - misses the last element
   arr = [1, 2, 3, 4, 5];
   for i = 1:length(arr)-1
       disp(arr(i))
   end
   
   % CORRECT
   for i = 1:length(arr)
       disp(arr(i))
   end
   ```

3. **Forgetting `end`**: Every `for` and `while` must have an `end`.

4. **Not initializing accumulator variables**:
   ```matlab
   % WRONG - sum is undefined initially
   for i = 1:10
       sum = sum + i;  % Error if sum doesn't exist
   end
   
   % CORRECT
   sum = 0;
   for i = 1:10
       sum = sum + i;
   end
   ```

5. **Premature loop termination without `break`**:
   ```matlab
   % If you use break, you must check conditions
   while iteration < max_iter
       % ... calculations ...
       if converged
           break;  % Critical to exit explicitly
       end
   end
   ```


| Mistake                              | Consequence                          | Remedy                              |
|--------------------------------------|--------------------------------------|-------------------------------------|
| Forgetting `end`                     | “Unexpected end of file” error       | Match every `for`/`while`/`if`      |
| Changing the loop index inside a `for`| Unpredictable behaviour              | Treat the index as read-only        |
| Infinite `while` loop                | Program hangs                        | Add a maximum-iteration counter     |
| Growing an array inside a loop       | Severe slowdown for large N          | Pre-allocate with `zeros`/`ones`    |
| Using loops when vectorization works | Slower, harder-to-read code          | Prefer array operations             |

---

## Nested Conditional Statements and Loops

Real-world engineering problems often require nesting: placing control structures inside other control structures.

### Nested `if` Statements

**Syntax:**
```matlab
if condition1
    if condition2
        % code if both condition1 and condition2 are true
    end
end
```

### Example: Feed Validation in a Reactor Startup

```matlab
% Check multiple reactor startup conditions before beginning a run
T_ambient = 298;    % Ambient temperature in K
F_in = 2.5;         % Feed flow rate in L/s
P = 8.5;            % Reactor pressure in bar
P_max = 10;         % Maximum safe pressure in bar
F_min = 1.0;        % Minimum feed rate in L/s

if T_ambient > 273 && T_ambient < 323    % Between 0°C and 50°C
    if F_in > F_min                      % Feed flow adequate
        if P < P_max                     % Pressure safe
            disp('All startup checks passed. Safe to proceed.')
            can_start = true;
        else
            disp('Pressure too high. Vent system before startup.')
            can_start = false;
        end
    else
        disp('Feed flow insufficient. Check pump.')
        can_start = false;
    end
else
    disp('Ambient temperature out of range.')
    can_start = false;
end
```

**Output:**
```
All startup checks passed. Safe to proceed.
```

### Nested Loops

**Syntax:**
```matlab
for i = start1:end1
    for j = start2:end2
        % code executed multiple times
    end
end
```

### Example: Material Balance for Multi-Stage Cascade

```matlab
% Multi-stage extraction cascade: track component mass through each stage
n_stages = 3;           % Number of extraction stages
n_components = 2;       % Two components: A and B
F_in = 10;              % Feed rate in kg/s
x_A_feed = 0.6;         % Mole fraction of A in feed

% Distribution coefficients (K = y/x)
K = [2.0, 0.4];         % K_A = 2.0, K_B = 0.4

% Initialize composition
x = zeros(n_stages, n_components);
x(1, 1) = x_A_feed;
x(1, 2) = 1 - x_A_feed;

fprintf('Stage | x_A (liquid) | x_B (liquid) | y_A (vapor)\n')
fprintf('------|--------------|--------------|------------\n')

for stage = 1:n_stages
    % Current liquid composition
    x_A = x(stage, 1);
    x_B = x(stage, 2);
    
    % Vapor composition from distribution coefficients
    y_A = K(1) * x_A;
    y_B = K(2) * x_B;
    
    % Normalize vapor (ensure y sums to 1)
    y_norm = y_A + y_B;
    y_A = y_A / y_norm;
    y_B = y_B / y_norm;
    
    fprintf('  %d   |    %8.4f  |    %8.4f  |    %8.4f\n', ...
            stage, x_A, x_B, y_A)
    
    % Outlet vapor becomes inlet to next stage (equilibrium model)
    if stage < n_stages
        x(stage+1, 1) = y_A;
        x(stage+1, 2) = y_B;
    end
end
```

**Output:**
```
All startup checks passed. Safe to proceed.
Stage | x_A (liquid) | x_B (liquid) | y_A (vapor)
------|--------------|--------------|------------
  1   |      0.6000  |      0.4000  |      0.8824
  2   |      0.8824  |      0.1176  |      0.9740
  3   |      0.9740  |      0.0260  |      0.9947
```

---

## Summary

### Key Concepts

| Concept | Purpose |
|---------|---------|
| **`if-elseif-else`** | Execute different code based on conditions |
| **`switch-case`** | Select from multiple discrete options |
| **`for` loop** | Repeat code a fixed number of times |
| **`while` loop** | Repeat code until a condition is met |
| **`break`** | Exit a loop immediately |
| **`continue`** | Skip to the next iteration |
| **Nesting** | Place control structures inside each other |


### Important MATLAB Syntax

| Purpose | Syntax |
|---------|--------|
| Basic conditional | `if condition ... end` |
| Multiple conditions | `if ... elseif ... else ... end` |
| Discrete selection | `switch var ... case value ... end` |
| Fixed repetition | `for index = start:end ... end` |
| Conditional repetition | `while condition ... end` |
| Exit loop | `break;` |
| Skip to next iteration | `continue;` |

### Avoiding Common Mistakes

1. Always include `end` for every `if`, `for`, and `while`
2. Use `==` to compare; use `=` only to assign
3. Remember that MATLAB indexing starts at 1, not 0
4. Initialize accumulator variables before loops
5. Use meaningful variable names (e.g., `T_setpoint` not `x`)
6. Include units in comments
7. Test edge cases (e.g., what happens at the boundary?)

---

## Homework

### Problem

You are simulating a semi-batch reactor where temperature is ramped. At each time step, you must select the correct heat capacity correlation based on temperature range, check safety, and compute energy required. If temperature exceeds 500 K, stop simulation (break). If concentration is below detection limit, skip energy calculation (continue) but still record time.

Given:
- Time vector $t = 0:1:10$ min (11 points)
- Temperature profile $T = 300 + 20*t$ K → [300, 320, ..., 500] K
- Concentration $C_A = 1.0*exp(-0.15*t)$ mol/L
- Heat capacity correlations:
  - If $T < 350$ K: $C_p = 2.0$ J/g/K
  - Elseif $T < 450$ K: $C_p = 2.0 + 0.01*(T-350)$ J/g/K
  - Else: $C_p = 3.0 + 0.02*(T-450)$ J/g/K
- Energy for heating from $T_{prev}$ to $T$ per gram: $Q = C_p * (T - T_{prev})$, with $T_{prev}=T_{i-1}$, $Q_1=0$ at t=0
- Detection limit $C_{det}=0.2$ mol/L
- Safety limit $T_{safe}=500$ K (break if $T > 500$)

### Required Tasks

1. Create $t$, $T$, $C_A$ vectors
2. Preallocate $C_p$ (1x11) and $Q$ (1x11) with zeros
3. Write `for` loop over indices 1..length(t):
   - Inside, use `if-elseif-else` to select $C_p$ based on $T(i)$
   - If $T(i) > 500$, `fprintf` alarm and `break`
   - If $C_A(i) < 0.2$, `fprintf` below detection, set $C_p$ and $Q$ but `continue` to skip detailed status (or still set status)
   - Else compute $Q(i)$ (0 for first point, else $C_p(i)*(T(i)-T(i-1))$)
   - Use nested `if` or `switch` to assign status string: "Low T", "Medium T", "High T" based on $C_p$ ranges
4. After loop, display table of $t,T,C_A,C_p,Q,status$
5. Use `switch` on reactor operation mode: mode = "Heating" → run above loop, "Cooling" → different Cp (optional)
6. Count how many points were below detection using logical indexing and loop

### Concepts Being Tested

- `if`, `elseif`, `else`, nested `if`
- `switch`, `case`, `otherwise`
- `for` loops with index, preallocation
- `while` vs `for` (here for)
- `break` and `continue`
- Combining logical conditions, relational operators
- Engineering application: conditional property correlations, safety interlock

### Hints

- Preallocate: `Cp = zeros(size(T)); Q = zeros(size(T)); status = strings(size(T));`
- Loop: `for i=1:length(t)` and inside `if T(i) < 350`
- For Q: `if i==1, Q(i)=0; else Q(i)=Cp(i)*(T(i)-T(i-1)); end`
- `break` exits for loop entirely, `continue` skips to next iteration (after continue, code below in loop not executed)
- For status: `if Cp(i) < 2.5, status(i)="Low T", elseif Cp(i) < 3.0, status(i)="Medium T", else status(i)="High T", end`

<details>
<summary><strong>Detailed Solution — Click to Reveal</strong></summary>

### Concept

This homework integrates conditional property selection (common: Cp, Antoine, viscosity correlations depend on T range), safety interlock (break on high T), data quality handling (continue on low concentration), and repetitive calculation (energy per step). It mimics semi-batch reactor heating where you must choose correct correlation and stop if unsafe.

### Idea

1. Build time and temperature vectors, concentration decay
2. Preallocate Cp, Q, status for performance
3. For loop over time: select Cp via if-elseif-else based on T, check safety T>500 → break, check detection C_A<0.2 → continue (but still record), compute Q with previous T, assign status via if or switch
4. After loop, build table, count detection events via logical and via loop with switch to demonstrate both
5. Optional switch on operation mode to show selection structure

### Syntax

```matlab
t = 0:10;
T = 300 + 20*t;
C_A = 1.0*exp(-0.15*t);
Cp = zeros(size(T)); Q = zeros(size(T)); status = strings(size(T));
for i=1:length(t)
    if T(i) < 350
        Cp(i)=2.0;
    elseif T(i) < 450
        Cp(i)=2.0+0.01*(T(i)-350);
    else
        Cp(i)=3.0+0.02*(T(i)-450);
    end
    if T(i) > 500
        fprintf('Alarm at t=%d, break\n', t(i)); break;
    end
    if C_A(i) < 0.2
        status(i)="Below detection";
        continue;
    end
    ...
end
switch mode
    case "Heating"
        ...
    otherwise
end
```

### Syntax Breakdown

- `t = 0:10` — colon creates 0 to 10 inclusive, 11 points
- `T = 300 + 20*t` — vectorized temperature ramp, element-wise scalar multiplication
- `zeros(size(T))` — preallocation same size as T, avoids growing array
- `for i=1:length(t)` — loop index i from 1 to 11, length returns number of elements
- `if T(i) < 350` — scalar condition inside loop, T(i) is scalar
- `elseif T(i) < 450` — second range, only checked if first false
- `else` — remaining, T>=450
- `if T(i) > 500, break; end` — safety interlock, break exits for loop entirely, remaining iterations not executed
- `if C_A(i) < 0.2, continue; end` — continue skips rest of current iteration, goes to next i, but Cp already set before continue
- `if i==1, Q(i)=0; else Q(i)=Cp(i)*(T(i)-T(i-1)); end` — first point no delta, else energy = Cp*deltaT
- `strings(size(T))` — creates string array for status, can hold text
- `switch mode, case "Heating", ... otherwise, end` — selection based on operation mode string
- `sum(C_A < 0.2)` — logical indexing count, alternative to loop counting

### MATLAB Code

```matlab
% homework05_solution.m - Semi-batch heating with conditional Cp

clear; clc;

% --- 1. Vectors ---
t = 0:10;  % min, 0 to 10 inclusive, 11 points
T = 300 + 20*t;  % K, 300 to 500
C_A = 1.0 * exp(-0.15*t);  % mol/L, first-order decay

fprintf('Time: '); disp(t);
fprintf('T: '); disp(T);
fprintf('C_A: '); disp(C_A);

% --- 2. Preallocate ---
Cp = zeros(size(T));  % J/g/K
Q = zeros(size(T));   % J/g per step
status = strings(size(T));

% --- Operation mode selection using switch ---
mode = "Heating";  % try "Cooling" or "Heating"

switch mode
    case "Heating"
        fprintf('Mode: Heating ramp\n');
        % Run main loop below
    case "Cooling"
        fprintf('Mode: Cooling ramp (demo different Cp)\n');
        % For demo, we still run same loop but could have different correlations
    otherwise
        fprintf('Unknown mode %s, default to Heating\n', mode);
        mode = "Heating";
end

% --- 3. For loop with conditionals, break, continue ---
safe_T = 500;  % K, break if > (strictly >)
detect_limit = 0.2; % mol/L

for i = 1:length(t)
    % --- Select Cp based on T range ---
    if T(i) < 350
        Cp(i) = 2.0;
    elseif T(i) < 450
        Cp(i) = 2.0 + 0.01*(T(i)-350);
    else
        Cp(i) = 3.0 + 0.02*(T(i)-450);
    end
    
    % --- Safety check: break if T > 500 ---
    if T(i) > safe_T
        fprintf('ALARM at t=%d min: T=%.0f K > safe limit %d K, breaking loop\n', t(i), T(i), safe_T);
        status(i) = "Aborted: High T";
        % Fill remaining with NaN/aborted if break
        if i < length(t)
            Cp(i+1:end) = NaN;
            Q(i+1:end) = NaN;
            status(i+1:end) = "Not executed (break)";
        end
        break;  % exit for loop
    end
    
    % --- Detection limit: continue if below ---
    if C_A(i) < detect_limit
        fprintf('t=%d min: C_A=%.3f < %.1f below detection, skip detailed calc\n', t(i), C_A(i), detect_limit);
        status(i) = "Below detection";
        % Still compute Q before continue? Requirement says skip energy calc, but we set Q=0 for below detection demo
        % For this solution, compute Q but mark status, then continue to skip status assignment below
        if i==1
            Q(i)=0;
        else
            Q(i)=Cp(i)*(T(i)-T(i-1));
        end
        continue;  % skip remaining code in this iteration (status already set)
    end
    
    % --- Energy calculation ---
    if i==1
        Q(i)=0;  % no heating at start
    else
        Q(i)=Cp(i)*(T(i)-T(i-1));  % J/g, Cp * deltaT
    end
    
    % --- Assign status based on Cp (or T) using if-elseif ---
    if Cp(i) < 2.5
        status(i) = "Low T";
    elseif Cp(i) < 3.0
        status(i) = "Medium T";
    else
        status(i) = "High T";
    end
    
    fprintf('t=%2d min: T=%3.0f K, C_A=%.3f mol/L, Cp=%.2f J/g/K, Q=%.1f J/g -> %s\n', ...
            t(i), T(i), C_A(i), Cp(i), Q(i), status(i));
end

% --- 4. Display table ---
resultTable = table(t', T', C_A', Cp', Q', status', ...
    'VariableNames',{'t_min','T_K','C_A_mol_L','Cp_J_g_K','Q_J_g','Status'});
disp(resultTable);

% --- 5. Count below detection via logical indexing ---
numBelowLogical = sum(C_A < detect_limit);
fprintf('\nBelow detection (logical count): %d/%d\n', numBelowLogical, length(C_A));

% --- 6. Count via loop with switch (demonstrate switch) ---
counts = struct('Low',0,'Medium',0,'High',0,'Below',0,'Aborted',0);
for i=1:length(status)
    % Use switch on status
    switch status(i)
        case "Low T"
            counts.Low = counts.Low + 1;
        case "Medium T"
            counts.Medium = counts.Medium + 1;
        case "High T"
            counts.High = counts.High + 1;
        case "Below detection"
            counts.Below = counts.Below + 1;
        case "Aborted: High T"
            counts.Aborted = counts.Aborted + 1;
        otherwise
            % includes Not executed
    end
end
fprintf('Counts via loop+switch: Low=%d Medium=%d High=%d Below=%d Aborted=%d\n', ...
        counts.Low, counts.Medium, counts.High, counts.Below, counts.Aborted);

% --- 7. While loop demo: find time when C_A drops below 0.3 ---
target = 0.3;
idx = 1;
while idx <= length(C_A) && C_A(idx) >= target
    idx = idx + 1;
end
if idx <= length(C_A)
    fprintf('\nC_A drops below %.1f at t=%d min (C=%.3f)\n', target, t(idx), C_A(idx));
else
    fprintf('\nC_A never drops below %.1f\n', target);
end

% --- 8. Energy total (excluding NaN) ---
totalQ = sum(Q,'omitnan');
fprintf('Total energy for heating (up to break): %.1f J/g\n', totalQ);
```

### Line-by-Line Explanation

- `t=0:10` creates 11 points, `T=300+20*t` ramp to 500 K at t=10, `C_A=exp(-0.15*t)` decays from 1.0 to ~0.223 at t=10
- Preallocation `Cp=zeros(size(T))` etc. avoids growing arrays, critical for performance, also defines size before loop
- `switch mode` demonstrates selection structure for operation mode — even though we only implement Heating, structure shows how to handle multiple modes
- Inside `for i=1:length(t)`: first `if-elseif-else` selects Cp correlation based on T range — mimics real Cp tables where coefficients change with T
- `if T(i) > safe_T, break; end` safety interlock: if temperature exceeds 500 K, break exits loop, remaining points marked Not executed — simulates DCS shutdown. Note condition `>` not `>=` because task says break if >500, and T=500 at last point should not break (500 is allowed)
- `if C_A(i) < detect_limit, continue; end` demonstrates continue: skip detailed status assignment but still record Cp/Q, go to next iteration. In this solution we set status before continue to record Below detection
- `if i==1, Q(i)=0; else Q(i)=Cp(i)*(T(i)-T(i-1)); end` handles first point no deltaT, else energy = Cp*deltaT — typical energy balance for heating step
- Status assignment via `if Cp<2.5 Low, elseif <3.0 Medium, else High` shows nested decision after Cp selection
- Table construction `table(t', ...)` transposes row vectors to columns for table
- `sum(C_A < detect_limit)` counts below detection via logical sum — vectorized alternative to loop
- Second counting via `for` + `switch` on status string demonstrates combining loop and selection for reporting
- While loop demo finds first time C_A drops below target — shows while condition with `&&` short-circuit and index update, alternative to `find`
- `sum(Q,'omitnan')` total energy ignoring NaN from break — robust handling

### Expected Result

```
Time: 0 1 ... 10
T: 300 320 ... 500
C_A: 1.0 0.86 0.74 ... 0.22
Mode: Heating ramp
t=0: T=300, C_A=1.0, Cp=2.0, Q=0 -> Low T
t=1: T=320, Cp=2.0, Q=40 -> Low T
...
t=3: T=360, Cp=2.1, Q=42 -> Low T (Cp 2.1 <2.5)
t=5: T=400, Cp=2.5, Q=50 -> Medium T
...
t=7: T=440, Cp=2.9, Q=58 -> Medium T
t=8: T=460, Cp=3.2, Q=64 -> High T
t=9: C_A=0.259? Actually C_A at t=9 = exp(-1.35)=0.259, still above 0.2
t=10: T=500, C_A=0.223, Cp=4.0? Wait: T=500 => Cp=3.0+0.02*50=4.0, Q=80, status High T, no break because T not >500
Below detection: 0 points? Let's compute: C_A at t=10 = exp(-1.5)=0.223 >0.2, so 0 below. If t extended to 12, would be below.
If we set detect 0.3, then below at t>=8
```

For detect 0.2, actually C_A at t=8 = exp(-1.2)=0.301 >0.2, t=9 0.259 >0.2, t=10 0.223 >0.2, so 0 below. Change detect to 0.25 would show below at t=10. The code handles anyway.

If T_safe=500 and break on >500, no break occurs because max T=500. If set T_safe=490, break at t=10 (T=500>490) and remaining NaN.

### Engineering Interpretation

- Conditional Cp selection reflects real property databases: heat capacity correlations change with temperature range — using wrong correlation gives 20-30% error in energy balance
- Safety break mimics DCS interlock: if T exceeds safe limit, stop heating to prevent runaway — break prevents further energy addition
- Continue on low concentration mimics analyzer detection limit: below limit, measurement unreliable, skip detailed conversion calculation but continue monitoring — avoids using bad data in control
- Energy total `sum(Q)` is integral of Cp dT, needed for heater sizing — preallocation and loop ensure accurate accounting even with aborted points (omitnan)
- While loop to find when C_A drops below target is typical for batch time determination: time to reach desired conversion
- Switch on status for counting demonstrates how to generate shift reports: how many points in Low/Medium/High regime

</details>

---

## Final Review

Before moving to the next lecture, make sure you can:

- [ ] Write `if-elseif-else` statements to make decisions based on conditions
- [ ] Use `switch-case` structures for multi-way selection
- [ ] Write `for` loops with both counter and array iteration syntax
- [ ] Write `while` loops that terminate based on convergence criteria
- [ ] Use `break` to exit loops and `continue` to skip iterations
- [ ] Nest conditional statements and loops
- [ ] Apply control structures to realistic chemical engineering problems
- [ ] Debug control structures to find logic errors

---

## Key MATLAB Syntax Reference

| Purpose | MATLAB Syntax | Example |
|---------|---------------|---------|
| **If condition** | `if condition ... end` | `if T > 350; disp('Hot'); end` |
| **If-elseif-else** | `if ... elseif ... else ... end` | `if T > 350; ... elseif T < 300; ... else; ... end` |
| **Switch selection** | `switch var; case val; ... otherwise; end` | `switch mode; case 1; ... case 2; ... end` |
| **For loop (counter)** | `for i = start:step:end ... end` | `for i = 1:10; sum = sum + i; end` |
| **For loop (array)** | `for x = array ... end` | `for Q = [40 50 60]; ... end` |
| **While loop** | `while condition ... end` | `while error > tol; ... end` |
| **Break** | `break;` | `if converged; break; end` |
| **Continue** | `continue;` | `if X < min; continue; end` |
| **Logical AND** | `&` or `&&` | `if (T > 300) && (P < 10)` |
| **Logical OR** | `\|` or `\|\|` | `if (T > 350) \| (P > 15)` |
| **Logical NOT** | `~` | `if ~isempty(data)` |

