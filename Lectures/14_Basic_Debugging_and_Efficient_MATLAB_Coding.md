# Lecture 14: Basic Debugging and Efficient MATLAB Coding

## Learning Objectives

By the end of this lecture, you should be able to:

- Identify and interpret different types of MATLAB errors (syntax, runtime, logical).
- Use MATLAB's debugging tools effectively, including breakpoints and step-by-step execution.
- Inspect and monitor variables during code execution.
- Apply vectorization techniques to improve code performance.
- Avoid unnecessary calculations and use preallocation for efficiency.
- Write readable, maintainable MATLAB code.
- Verify code correctness through intermediate result checking and comparison with analytical solutions.
- Apply dimensional and physical consistency checks to ensure engineering validity.



## Why This Topic Matters

Professional MATLAB programming requires more than just writing code that runs. Real engineering problems demand:

1. **Reliability**: Code must produce correct results consistently.
2. **Efficiency**: Complex simulations and calculations must run in reasonable time.
3. **Maintainability**: Team members and future versions of yourself must understand the code.
4. **Verifiability**: Engineers must be confident that numerical results are physically meaningful.

Debugging is not an afterthought—it is a fundamental skill. Similarly, efficient coding becomes essential when processing large datasets, running iterative simulations, or controlling real-time processes. This lecture teaches the practical skills that separate professional engineers from beginners.



## Debugging

### Concept

Debugging is the process of finding and fixing errors in your code. Every programmer writes code with errors. The difference between experienced and inexperienced programmers is that experienced engineers:

- Anticipate common mistakes.
- Recognize error messages quickly.
- Use systematic techniques to isolate the problem.
- Understand the difference between superficial fixes and root causes.

### Three Types of Errors

**1. Syntax Errors**

Syntax errors violate the rules of the MATLAB language. MATLAB detects these immediately when you run the code.

**Example:**

```matlab
x = [1 2 3
y = x + 1;
```

**Error message:**

```
Error: File: myScript.m Line: 1 Column: 10
Unsupported use of the '=' operator. To compare values for equality, use '=='. To specify name-value arguments, check that name is a valid identifier with no
surrounding quotes.
```
```
Error: File: test.m Line: 5 Column: 1
This statement is incomplete.
```

Syntax errors are usually easy to fix because MATLAB tells you exactly where the problem is.

**2. Runtime Errors**

Runtime errors occur during execution. The code is syntactically correct, but something goes wrong when MATLAB tries to execute it.

**Example:**

```matlab
x = [1 2 3];
y = x(5);  % x only has 3 elements
```

**Error message:**

```
Index exceeds the number of array elements. Index must not exceed 3.

Error in test (line 4)
y = x(5);  % x only has 3 elements
    ^^^^
```

Runtime errors usually indicate:
- Incorrect array dimensions
- Undefined variables
- Attempting invalid operations (e.g., division by zero)
- Calling non-existent functions

**3. Logical Errors**

Logical errors are **the most dangerous** because your code runs without crashing, but produces wrong results. The program executes perfectly—but solves the wrong problem or uses incorrect mathematics.

**Example:**

```matlab
% Intended: convert Celsius to Kelvin
% Correct: T_K = T_C + 273.15
T_K = T_C * 273.15;  % Logical error: multiplies instead of adding
```

This code runs fine. It will not crash. But it produces wrong temperatures.

## MATLAB Syntax for Error Handling

While full exception handling is beyond the scope of this lecture, MATLAB provides basic error detection:

**Check for undefined variables:**

```matlab
if ~exist('variable_name', 'var')
    error('variable_name is not defined');
end
```

**Catch division by zero:**

```matlab
if denominator == 0
    error('Division by zero');
end
```

**Use assertions for validation:**

```matlab
assert(T > 0, 'Temperature must be positive');
assert(length(x) == length(y), 'x and y must have same length');
```

### Syntax Breakdown

```matlab
exist('variable_name', 'var')
```

- `exist()` — checks if a variable exists in the workspace
- `'variable_name'` — the name of the variable to check (as a string)
- `'var'` — specifies that we are checking for a variable (not a file or function)
- Returns 1 if the variable exists, 0 if it does not

```matlab
assert(condition, 'message')
```

- `assert()` — MATLAB function that throws an error if the condition is false
- `condition` — logical expression that should be true
- `'message'` — error message to display if condition is false

---
### Common MATLAB Mistakes

**Mistake 1: Forgetting semicolons when testing**

When debugging, you often want to see intermediate values:

```matlab
x = [1 2 3];
y = x + 1;  % No semicolon—see the result
```

This is acceptable during development, but remove semicolons before submitting code if not intentional.

**Mistake 2: Using = instead of == in conditions**

```matlab
% WRONG: this assigns instead of comparing
if x = 5
    disp('x is 5');
end

% CORRECT: this compares
if x == 5
    disp('x is 5');
end
```

**Mistake 3: Not checking array dimensions**

```matlab
A = [1 2 3];
B = [4 5];
C = A + B;  % Error: arrays must be same size for element-wise operations
```

**Mistake 4: Assuming MATLAB indexing starts at 0**

Unlike Python or C, MATLAB uses 1-based indexing:

```matlab
x = [10 20 30];
x(0)   % Error: index must be at least 1
x(1)   % Correct: returns 10
```
## Code Profiling: Identifying Bottlenecks

### `profile()` Function

```matlab
% Profile a function
profile on

% Call your function
for i = 1:1000
    my_slow_function(i);
end

profile off

% View results
profview
```

### `tic` and `toc`

```matlab
% Simple timing

% Method 1: Loop
tic
for i = 1:1000
    y = i^2;
end
time1 = toc;

% Method 2: Vectorized
tic
i = 1:1000;
y = i.^2;
time2 = toc;

fprintf("Loop time: %.6f s\n", time1)
fprintf("Vector time: %.6f s\n", time2)
fprintf("Speedup: %.0fx\n", time1 / time2)
```
### `timeit` : more accurate, runs multiple times

```matlab
% timeit: more accurate, runs multiple times
f = @() exp(-0.3*(0:0.1:100));
t = timeit(f) % median time
```


## Debugging Tools

### Concept

MATLAB provides built-in tools to observe program execution step by step. Rather than guessing what went wrong, you can watch your code run and inspect variables in real time.

### Using Breakpoints

A **breakpoint** is a marker telling MATLAB to pause execution at a specific line. While paused, you can:

- Inspect variables
- Step through code line by line
- Change variables and continue

**Setting a breakpoint (GUI):**

1. Click on the line number in the editor where you want to pause.
2. A red dot appears.
3. Run the script. MATLAB stops at that line.

**Setting a breakpoint (command):**

```matlab
dbstop in myScript.m at 15
```

Pauses execution at line 15 of `myScript.m`.

## Step-by-Step Execution

Once execution is paused at a breakpoint, use:

| Command | Effect |
|---|---|
| `dbstep` | Execute the current line, pause at the next line |
| `dbstep in` | If the current line is a function call, step into that function |
| `dbstep out` | Execute until the current function returns |
| `dbcont` | Continue execution until the next breakpoint |
| `dbquit` | Exit debug mode and stop execution |
| `dbclear` | Remove breakpoint |

**At the command prompt:**

```matlab
K>> dbstep      % Step to next line
K>> x           % View the variable x
K>> y = x + 1   % Modify a variable (temporary)
K>> dbcont      % Continue execution
```

The `K>>` prompt indicates you are in debug mode.

### Inspecting Variables

During debugging, type any variable name to see its current value:

```matlab
K>> x
x =
    1.5000    2.3000    3.1000

K>> size(x)
ans =
     1     3

K>> whos
  Name      Size            Bytes  Class     Attributes
  x         1x3                24  double    
  y         1x1                 8  double    
```

- `whos` — displays all variables in the current workspace with their sizes and types

---

### Chemical Engineering Example: Debugging a Reactor Calculation

Suppose you are calculating conversion in a CSTR (continuously stirred-tank reactor) and getting an unrealistic result:

```matlab
function X = cstr_conversion(F_in, C_A0, k, V)
    % F_in: volumetric flow rate (L/min)
    % C_A0: inlet concentration (mol/L)
    % k: reaction rate constant (1/min)
    % V: reactor volume (L)
    
    tau = V / F_in;  % residence time (min)
    X = k * tau^2 / (1 + k * tau);  % conversion
end

% Test
F = 10;         % L/min
C_A0 = 2;       % mol/L
k = 0.5;        % 1/min
V = 50;         % L

X = cstr_conversion(F, C_A0, k, V);
disp(['Conversion: ', num2str(X)]);
```

If X is physically impossible (e.g., X > 1), set a breakpoint at the line `X = k * tau / (1 + k * tau)` and inspect `tau`, `k`, and the numerator/denominator separately to identify the issue.

---
### try-catch Error Handling

```matlab
try
    % Code that might fail
    result = 1 / 0;           % Division by zero
    
    % Or file operations
    data = readtable("missing_file.csv");
    
catch ME
    % ME is the exception object
    fprintf("Error: %s\n", ME.message)
    fprintf("ID: %s\n", ME.identifier)
    
    % Provide fallback or cleanup
    result = NaN;
end
```

### Input Validation

```matlab
function output = safe_operation(x, y)
    % Validate inputs
    if ~isnumeric(x) || ~isnumeric(y)
        error("Inputs must be numeric")
    end
    
    if length(x) ~= length(y)
        error("Inputs must have same length")
    end
    
    if any(y == 0)
        warning("y contains zeros; division by zero possible")
    end
    
    % Proceed with valid inputs
    output = x ./ y;
end

% Safe calling
try
    result = safe_operation([1, 2, 3], [2, 4, 6]);
    disp(result)
catch ME
    fprintf("Operation failed: %s\n", ME.message)
end
```

## Efficient Coding

### Concept

MATLAB is fast at performing operations on arrays (vectors and matrices). MATLAB is slow at repeated operations in loops. **Vectorization** means restructuring code to replace loops with array operations, dramatically improving performance.

## Vectorization Techniques

### Technique 1: Replace Element-by-Element Loops with Array Operations

**Inefficient (slow loop):**

```matlab
x = [1 2 3 4 5];
y = zeros(size(x));

for i = 1:length(x)
    y(i) = x(i) * 2 + sin(x(i));
end
```

**Efficient (vectorized):**

```matlab
x = [1 2 3 4 5];
y = x .* 2 + sin(x);
```

Both give the same result, but the vectorized version is 10–100× faster for large arrays.

```matlab
x = 1:50000000;
y = zeros(size(x));

tic
for i = 1:length(x)
    y(i) = x(i) * 2 + sin(x(i));
end
toc

tic
x = [1 2 3 4 5];
y = x .* 2 + sin(x);
toc
```

**Key difference:**

- Loop version: evaluates `x(i) * 2 + sin(x(i))` for each element separately (50000000 iterations)
- Vectorized version: MATLAB performs the entire calculation on all 50000000 elements at once

---

### Technique 2: Use Logical Indexing Instead of Loops

**Inefficient:**

```matlab
data = [1.5 2.3 1.8 3.2 0.9 2.1];
threshold = 2.0;
high_values = [];

for i = 1:length(data)
    if data(i) > threshold
        high_values = [high_values data(i)];
    end
end
```

**Efficient:**

```matlab
data = [1.5 2.3 1.8 3.2 0.9 2.1];
threshold = 2.0;
high_values = data(data > threshold);
```
For large arrays:

```matlab
rng(40)
data = 1 + 2*rand(1,100000);
threshold = 2.0;
high_values = [];

tic
for i = 1:length(data)
    if data(i) > threshold
        high_values = [high_values data(i)];
    end
end
toc

rng(40)
data = 1 + 2*rand(1,100000);
threshold = 2.0;
tic
high_values2 = data(data > threshold);
toc
```

---

### Technique 3: Preallocation

When you must use a loop, pre-allocate (reserve space) before the loop:

**Inefficient (growing array dynamically):**

```matlab
n = 10000;
result = [];

for i = 1:n
    result = [result i^2];  % Array grows each iteration
end
```

This forces MATLAB to reallocate memory each iteration, slowing execution dramatically.

**Efficient (pre-allocated array):**

```matlab
n = 10000;
result = zeros(1, n);  % Reserve space upfront

for i = 1:n
    result(i) = i^2;   % Just fill in values
end
```

The pre-allocated version is 100–1000× faster for large loops.

```matlab
n = 10000;
result = [];

for i = 1:n
    result = [result i^2];  % Array grows each iteration
end
toc

tic
n = 10000;
result = zeros(1, n);  % Reserve space upfront

for i = 1:n
    result(i) = i^2;   % Just fill in values
end
```
---

### MATLAB Syntax for Efficient Operations

**Element-wise operations (use `.` operator):**

```matlab
A .* B      % Element-wise multiplication
A ./ B      % Element-wise division
A .^ 2      % Element-wise power
```

**Logical indexing:**

```matlab
x(x > 0)           % Elements greater than 0
x(x > 0 & x < 10)  % Elements between 0 and 10
x(isnan(x)) = 0    % Replace NaN with 0
```

**Preallocation:**

```matlab
A = zeros(m, n);   % Pre-allocate matrix of zeros
B = ones(1, k);    % Pre-allocate vector of ones
C = NaN(p, q);     % Pre-allocate with NaN (useful for tracking uninitialized values)
```

### Syntax Breakdown

```matlab
y = x .* 2 + sin(x)
```

- `x` — input vector
- `.` — indicates element-wise operations
- `.*` — element-wise multiplication (each element of x multiplied by 2)
- `+` — addition (element-wise, since we're adding a scalar)
- `sin(x)` — vectorized sine function (applies to all elements)
- `y` — output vector, same size as x

```matlab
high_values = data(data > threshold)
```

- `data > threshold` — creates a logical array (true/false for each element)
- `data(...)` — indexing with the logical array
- Returns only elements where the condition is true

## Summary of techniques

### Common Vectorization Patterns

```matlab
% Pattern 1: Element-wise operations
data = 1:1000;

% WRONG: Loop
result = [];
for x = data
    result = [result, x*2 + 1];
end

% CORRECT: Vectorized
result = data * 2 + 1;

% Pattern 2: Conditional operations
T = [25, 35, 45, 55, 65];  % Temperature array

% WRONG: Loop
status = {};
for i = 1:length(T)
    if T(i) > 50
        status{i} = "High";
    else
        status{i} = "Low";
    end
end

% CORRECT: Vectorized
status = cell(size(T));
status(T > 50) = {"High"};
status(T <= 50) = {"Low"};

% Pattern 3: Logical indexing
values = [10, 20, 15, 30, 25];

% WRONG: Loop
selected = [];
for v = values
    if v > 18
        selected = [selected, v];
    end
end

% CORRECT: Logical indexing
selected = values(values > 18);
```
**Memory Efficiency**

```matlab
% SLOW: Growing array
tic
y_slow = [];
for i = 1:10000
    y_slow = [y_slow, sin(i)];  % Reallocates memory each time!
end
t_slow = toc;

% FAST: Pre-allocate
tic
y_fast = zeros(1, 10000);
for i = 1:10000
    y_fast(i) = sin(i);
end
t_fast = toc;

fprintf("Growing: %.4f s\n", t_slow)
fprintf("Pre-allocated: %.4f s\n", t_fast)
fprintf("Speedup: %.2fx\n", t_slow / t_fast)
```

**`arrayfun` — Apply function to array elements (loop hidden, not faster, but concise):**

```matlab
T = [300 350 400];
k = arrayfun(@(T) arrhenius(1e7, 60000, T), T) % same as vectorized arrhenius if vectorized
% arrayfun not faster than vectorized, but useful when function not vectorized
```

## Verification

### Concept

After writing and debugging code, you must verify that the results are correct. This requires multiple approaches:

1. **Intermediate result checking** — Does each step produce reasonable values?
2. **Dimensional consistency** — Do units match throughout?
3. **Physical consistency** — Are results physically plausible?
4. **Comparison with analytical solutions** — For simple cases, does MATLAB match hand calculations?

### Checking Intermediate Results

Break calculations into steps and inspect each result:

```matlab
function X = cstr_design(F, V, k, C_A0)
    % Calculate CSTR conversion
    
    tau = V / F;
    disp(['Residence time: ', num2str(tau), ' min']);
    
    denom = 1 + k * tau;
    disp(['Denominator: ', num2str(denom)]);
    
    X = k * tau / denom;
    disp(['Conversion: ', num2str(X)]);
    
    % Check if result is reasonable (0 ≤ X ≤ 1)
    if X < 0 || X > 1
        warning('Conversion is outside physical range [0, 1]');
    end
end
```

### Dimensional Consistency

Always check that units make sense:

```matlab
% Temperature conversion
T_C = 25;           % °C
T_K = T_C + 273.15; % K (correct)
% NOT: T_K = T_C * 273.15 (wrong units)

% Flow rate and volume
F = 10;    % L/min
V = 100;   % L
tau = V / F;  % (L) / (L/min) = min (correct)
```

### Physical Consistency

Check whether results make physical sense:

```matlab
% Reaction conversion must be between 0 and 1
if X < 0 || X > 1
    error('Conversion outside physical range');
end

% Temperature cannot be negative (in absolute scale)
if T < 0
    error('Absolute temperature cannot be negative');
end

% Pressure gradient in flow must be negative (pressure decreases in flow direction)
if dP > 0
    warning('Pressure increases in flow direction (unphysical)');
end
```

### Comparison with Analytical Solutions

For simple problems, solve by hand or use known formulas and compare:

```matlab
% Ideal gas law: PV = nRT
% Compare MATLAB calculation with analytical solution

P = 5;      % bar
V = 2;      % L
T = 298;    % K
R = 0.08314;  % L·bar/(mol·K)

% Solve numerically with MATLAB
n_numerical = (P * V) / (R * T);

% Compare with direct calculation
n_analytical = 5 * 2 / (0.08314 * 298);

error = abs(n_numerical - n_analytical);
disp(['Numerical result: ', num2str(n_numerical)]);
disp(['Analytical result: ', num2str(n_analytical)]);
disp(['Difference: ', num2str(error)]);
```
---

### Chemical Engineering Application: Mixer Material Balance Verification

Material balance to a mixer, where multiple streams are mixed into one outlet. We can verify the calculation:

```matlab
function x_out = mixer_balance(z, F)
    % z: matrix/vector of inlet compositions (mole fraction)
    % F: vector of inlet flow rates (mol/h)
    % x_out: outlet composition (mole fraction)
    
    % Total outlet flow
    F_out = sum(F);
    
    % Component balance
    x_out = sum(z .* F) / F_out;
    
    % Verification 1: Overall balance
    overall_in = sum(F);
    overall_out = F_out;
    balance_error = abs(overall_in - overall_out);
    
    if balance_error > 1e-6
        warning('Overall material balance not satisfied');
    end
    
    % Verification 2: Composition range
    if any(x_out < 0) || any(x_out > 1)
        error('Outlet composition outside [0,1] range');
    end
    
    % Verification 3: Composition sum
    if abs(sum(x_out) - 1) > 1e-6
        warning('Outlet mole fractions do not sum to 1');
    end
end
```



## Worked Example: Debugging and Optimizing a Heat Transfer Calculation

**Problem:** Calculate the outlet temperature of a liquid flowing through a heated pipe. The code produces reasonable results but runs slowly.

**Initial Code (inefficient and without verification):**

```matlab
function T_out = pipe_heat_transfer(T_in, m_dot, Q, n_segments)
    % T_in: inlet temperature (K)
    % m_dot: mass flow rate (kg/s)
    % Q: total heat input (W)
    % n_segments: number of pipe segments for calculation
    
    cp = 4186;  % Specific heat capacity (J/kg·K)
    Q_segment = Q / n_segments;  % Heat per segment (W)
    
    T = T_in;
    
    % Calculate temperature rise in each segment
    for i = 1:n_segments
        dT = Q_segment / (m_dot * cp);
        T = T + dT;
    end
    
    T_out = T;
end

% Test
T_in = 298;      % K
m_dot = 2;       % kg/s
Q = 50000;       % W
n_segments = 100;

T_out = pipe_heat_transfer(T_in, m_dot, Q, n_segments);
disp(['Outlet temperature: ', num2str(T_out), ' K']);
```

**Issues with this code:**

1. **No verification** — No checks that results are physically reasonable
2. **Inefficient loop** — Recalculates the same `dT` in every iteration (since Q and mass flow are constant)
3. **No intermediate inspection** — Cannot easily see what's happening

**Optimized and Verified Code:**

```matlab
function T_out = pipe_heat_transfer_optimized(T_in, m_dot, Q, n_segments)
    % T_in: inlet temperature (K)
    % m_dot: mass flow rate (kg/s)
    % Q: total heat input (W)
    % n_segments: number of pipe segments (for numerical stability)
    
    cp = 4186;  % Specific heat capacity (J/kg·K)
    
    % Verification 1: Check input validity
    assert(T_in > 0, 'Inlet temperature must be positive');
    assert(m_dot > 0, 'Mass flow rate must be positive');
    assert(Q >= 0, 'Heat input cannot be negative');
    assert(n_segments > 0, 'Number of segments must be positive');
    
    % Calculate outlet temperature (vectorized - no loop needed)
    % Energy balance: Q = m_dot * cp * (T_out - T_in)
    T_out = T_in + Q / (m_dot * cp);
    
    % Verification 2: Check physical reasonableness
    if Q > 0 && T_out <= T_in
        warning('Heat input but outlet temperature not higher than inlet');
    end
    
    if T_out > 500
        warning('Outlet temperature unusually high (>500 K)');
    end
    
    disp(['Inlet temperature: ', num2str(T_in), ' K']);
    disp(['Heat input: ', num2str(Q), ' W']);
    disp(['Outlet temperature: ', num2str(T_out), ' K']);
    disp(['Temperature rise: ', num2str(T_out - T_in), ' K']);
end

% Test with verification
T_in = 298;      % K
m_dot = 2;       % kg/s
Q = 50000;       % W

T_out = pipe_heat_transfer_optimized(T_in, m_dot, Q, 100);

% Analytical verification
% Energy balance: Q = m_dot * cp * DT
% Therefore: DT = Q / (m_dot * cp)
DT_expected = Q / (2 * 4186);
T_out_expected = T_in + DT_expected;

disp(['Expected outlet temperature (analytical): ', num2str(T_out_expected), ' K']);
disp(['Difference: ', num2str(abs(T_out - T_out_expected)), ' K']);
```

**Key improvements:**

1. **Removed unnecessary loop** — The loop served no purpose (same dT each iteration). 
2. **Added assertions** — Catches invalid inputs immediately.
3. **Added physical checks** — Warns if results are unreasonable.
4. **Added analytical comparison** — Verifies result against known solution.
5. **Added intermediate output** — Easy to inspect intermediate values.


## Key Concepts

| Concept | Purpose | Approach |
|---|---|---|
| **Syntax errors** | Violate MATLAB language rules | MATLAB catches immediately; read error message |
| **Runtime errors** | Occur during execution (e.g., wrong array size) | Use breakpoints and step through code |
| **Logical errors** | Code runs but produces wrong results | Verify against known solutions; check physical plausibility |
| **Breakpoints** | Pause execution to inspect variables | Set in editor or use `dbstop` |
| **Vectorization** | Replace loops with array operations | Use `.` operator and logical indexing |
| **Preallocation** | Reserve array memory upfront | Use `zeros()`, `ones()`, `NaN()` before loops |
| **Verification** | Confirm results are correct | Check intermediate values, dimensions, and physical consistency |

## Key MATLAB Syntax Reference

| Purpose | MATLAB Syntax | Example |
|---|---|---|
| Set breakpoint | `dbstop in file.m at line` | `dbstop in myScript.m at 15` |
| Step to next line | `dbstep` | `K>> dbstep` |
| Step into function | `dbstep in` | `K>> dbstep in` |
| Continue execution | `dbcont` | `K>> dbcont` |
| Exit debug | `dbquit` | `K>> dbquit` |
| Check if variable exists | `exist('name', 'var')` | `if exist('x', 'var')` |
| Assert condition | `assert(condition, 'msg')` | `assert(T > 0, 'T must be positive')` |
| Throw error | `error('message')` | `error('Invalid input')` |
| Issue warning | `warning('message')` | `warning('This may fail')` |
| Element-wise multiply | `A .* B` | `y = x .* 2` |
| Element-wise divide | `A ./ B` | `z = A ./ B` |
| Element-wise power | `A .^ n` | `y = x .^ 2` |
| Logical indexing | `x(condition)` | `high = x(x > 10)` |
| Preallocate zeros | `zeros(m, n)` | `A = zeros(100, 50)` |
| Preallocate ones | `ones(m, n)` | `B = ones(1, 100)` |
| Get array size | `size(A)` | `[m, n] = size(A)` |
| Get vector length | `length(v)` | `n = length(x)` |
| Vectorized sine | `sin(x)` | `y = sin(x)` (x can be a vector) |
| Vectorized exponential | `exp(x)` | `y = exp(-k*t)` (t can be vector) |

---
### Quick Debugging Checklist

-  Did I remove all syntax errors (MATLAB no longer complains)?
-  Do all variables have reasonable values (check with breakpoints)?
-  Are array dimensions correct (use `size()` to verify)?
-  Did I check for division by zero or NaN values?
-  Do results make physical sense for my engineering application?
-  Does a simple test case produce expected output?
-  Have I compared numerical results against analytical solutions when possible?

### Quick Efficiency Checklist

-  Did I replace loops with vectorized operations where possible?
-  Did I preallocate arrays in loops (use `zeros()` before the loop)?
-  Did I use logical indexing instead of loops for selection?
-  Did I profile the code to find slow sections before optimizing?
-  Is my code still readable after optimization?


## Homework

### Problem

Write a MATLAB function that calculates the conversion of reactant A in a batch reactor following first-order kinetics. The function should:

1. Accept inputs: initial concentration (C_A0 in mol/L), reaction rate constant (k in 1/min), and time vector (t in minutes).
2. Calculate concentration at each time: C_A(t) = C_A0 * exp(-k*t)
3. Calculate conversion: X = (C_A0 - C_A) / C_A0
4. Return the conversion vector.
5. Include verification checks for physical consistency.
6. Optimize the code to use vectorization (avoid loops).

**Required Tasks:**

1. Write the function with input validation.
2. Verify the output makes physical sense (0 ≤ X ≤ 1).
3. Test the function with realistic chemical engineering data.
4. Compare a simple numerical case (by hand) with the MATLAB result.
5. Comment your code explaining key steps.

**Concepts Being Tested:**

- Function writing
- Input validation with `assert()`
- Vectorization
- Error checking
- Physical verification
- Comment documentation

**Hints:**

- Use `exp()` function for the exponential (vectorized automatically)
- Remember MATLAB works on entire arrays at once
- Check that conversion reaches reasonable limits (C_A → 0 as t → ∞)
- Include intermediate `disp()` statements to verify intermediate results

<details>
<summary><strong>Detailed Solution — Click to Reveal</strong></summary>

## Solution

### Concept

First-order reaction kinetics follow the integrated rate law:

$$C_A(t) = C_{A0} \cdot e^{-kt}$$

Conversion is defined as:

$$X = \frac{C_{A0} - C_A(t)}{C_{A0}} = 1 - e^{-kt}$$

The key insights:
1. Use vectorization (not loops) to calculate conversion at all times simultaneously
2. Verify that 0 ≤ X ≤ 1 for all times
3. Check that conversion increases with time (monotonic)
4. For large times, conversion should approach 1 (complete reaction)

### Approach / Idea

1. Validate all inputs (positive values, correct dimensions)
2. Calculate concentration at each time using vectorized exponential
3. Calculate conversion using vectorized arithmetic
4. Add verification checks:
   - Conversion in valid range
   - Conversion monotonically increasing
   - Physical warning if unexpected behavior
5. Return the conversion vector

### Syntax

```matlab
function X = first_order_conversion(C_A0, k, t)
```

- Input validation with `assert()`
- Vectorized exponential: `exp(-k*t)` applies to all elements of t
- Logical indexing for verification: `X(end)` (final conversion)

### Syntax Breakdown

```matlab
C_A = C_A0 * exp(-k * t)
```

- `C_A0` — scalar (initial concentration)
- `k` — scalar (rate constant)
- `t` — vector of times (row vector)
- `(-k * t)` — element-wise multiplication (produces vector of exponents)
- `exp(...)` — vectorized exponential (applied to each exponent)
- `C_A` — output vector, same length as t, containing concentrations

```matlab
X = (C_A0 - C_A) / C_A0
```

- `(C_A0 - C_A)` — scalar minus vector (broadcasting: subtracts from each element)
- Result `X` — vector of conversions, same length as t

```matlab
assert(k > 0, 'Rate constant must be positive')
```

- `k > 0` — logical condition to test
- Throws an error with message if condition is false
- `assert()` stops execution immediately if validation fails

### MATLAB Code

```matlab
function X = first_order_conversion(C_A0, k, t)
    % First-order batch reactor conversion calculation
    %
    % INPUTS:
    %   C_A0   - initial concentration (mol/L), scalar
    %   k      - reaction rate constant (1/min), scalar
    %   t      - time vector (min), row or column vector
    %
    % OUTPUT:
    %   X      - conversion vector, same size as t
    %
    % EXAMPLE:
    %   C_A0 = 2.0;              % mol/L
    %   k = 0.1;                 % 1/min
    %   t = 0:10:100;            % times from 0 to 100 min
    %   X = first_order_conversion(C_A0, k, t);
    %   plot(t, X);
    %   xlabel('Time (min)');
    %   ylabel('Conversion');
    
    % ========================================
    % VERIFICATION: Input validation
    % ========================================
    
    % Check that all inputs are provided
    assert(exist('C_A0', 'var') == 1, 'C_A0 not provided');
    assert(exist('k', 'var') == 1, 'k not provided');
    assert(exist('t', 'var') == 1, 't not provided');
    
    % Check that values are physically reasonable
    assert(C_A0 > 0, 'Initial concentration must be positive');
    assert(k > 0, 'Rate constant must be positive');
    assert(all(t >= 0), 'Time values must be non-negative');
    
    % Check that t is a vector
    assert(isvector(t), 't must be a vector');
    
    % ========================================
    % CALCULATION (vectorized, no loops)
    % ========================================
    
    % Calculate concentration at each time: C_A(t) = C_A0 * exp(-k*t)
    C_A = C_A0 * exp(-k * t);
    
    % Display intermediate result
    disp('Concentration calculation (vectorized):');
    disp(['  Initial concentration: ', num2str(C_A0), ' mol/L']);
    disp(['  Rate constant: ', num2str(k), ' 1/min']);
    disp(['  Final concentration at t_max: ', num2str(C_A(end)), ' mol/L']);
    
    % Calculate conversion: X = (C_A0 - C_A) / C_A0 = 1 - exp(-k*t)
    X = (C_A0 - C_A) / C_A0;
    
    % ========================================
    % VERIFICATION: Check physical consistency
    % ========================================
    
    % Conversion must be between 0 and 1 for all times
    if any(X < 0) || any(X > 1)
        error('Conversion outside physical range [0, 1]');
    end
    
    % At t=0, conversion should be 0 (no reaction at start)
    if X(1) > 1e-6
        warning('Conversion at t=0 should be ~0');
    end
    
    % Conversion should be monotonically increasing (first-order kinetics)
    X_diff = diff(X);
    if any(X_diff < 0)
        error('Conversion must increase monotonically with time');
    end
    
    % For large times, conversion should approach 1
    X_final = X(end);
    if X_final < 0.9 && t(end) > 100 / k
        warning('Conversion has not approached 1 at large times');
    end
    
    % Display verification results
    disp('Verification results:');
    disp(['  Conversion range: [', num2str(min(X)), ', ', num2str(max(X)), ']']);
    disp(['  Conversion at t=0: ', num2str(X(1))]);
    disp(['  Conversion at t_max: ', num2str(X(end))]);
    disp(['  Conversion is monotonically increasing: YES']);
    
end
```

### Code Explanation

**Lines 1-21: Function header and comments**

```matlab
function X = first_order_conversion(C_A0, k, t)
```
Defines a function that accepts three inputs (initial concentration, rate constant, time vector) and returns conversion vector X.

**Lines 23-33: Input validation**

```matlab
assert(C_A0 > 0, 'Initial concentration must be positive');
assert(k > 0, 'Rate constant must be positive');
assert(all(t >= 0), 'Time values must be non-negative');
```

Each `assert()` checks a condition. If false, MATLAB throws an error with the specified message and stops execution. This prevents garbage output from invalid inputs.

**Line 37-39: Vectorized calculation**

```matlab
C_A = C_A0 * exp(-k * t);
```

This is the key vectorized step. Instead of a loop like:

```matlab
for i = 1:length(t)
    C_A(i) = C_A0 * exp(-k * t(i));
end
```

We calculate all values at once. MATLAB's `exp()` function automatically applies the exponential to each element of the input vector `(-k * t)`.

**Line 42: Calculate conversion**

```matlab
X = (C_A0 - C_A) / C_A0;
```

This is also vectorized. The scalar `C_A0` is broadcast (subtracted from each element of C_A), then the entire result is divided by `C_A0`.

**Lines 47-62: Physical verification**

```matlab
if any(X < 0) || any(X > 1)
    error('Conversion outside physical range [0, 1]');
end
```

Uses `any()` to check if any element of X violates the physical constraint. If so, an error stops execution.

### Expected Result

**Test case:**

```matlab
C_A0 = 2.0;              % mol/L
k = 0.1;                 % 1/min (half-life ≈ 6.93 min)
t = [0 10 50 100];       % Four specific times

X = first_order_conversion(C_A0, k, t);

disp("X = ")
disp(X)
```

**Output (console):**

```
Concentration calculation (vectorized):
  Initial concentration: 2 mol/L
  Rate constant: 0.1 1/min
  Final concentration at t_max: 9.08e-05 mol/L
Verification results:
  Conversion range: [0, 0.99995]
  Conversion at t=0: 0
  Conversion at t_max: 0.99995
  Conversion is monotonically increasing: YES
X = 
         0    0.6321    0.9933    1.0000
```

**Interpretation of results:**

- At t=0 min: X ≈ 0 (no reaction yet)
- At t=10 min: X ≈ 0.63 (63.2% conversion)
- At t=50 min: X ≈ 0.993 (99.3% conversion)
- At t=100 min: X ≈ 0.9999 (99.99% conversion, approaching equilibrium)

For first-order kinetics, conversion follows an exponential approach to 1 (complete reaction is asymptotic—conversion never reaches exactly 100% in finite time).

**Comparison with half-life:**

The half-life for first-order kinetics is:

$$t_{1/2} = \frac{\ln(2)}{k} = \frac{0.693}{0.1} ≈ 6.93 \text{ min}$$

At t = 6.93 min, conversion should be 0.5:

```matlab
X_at_half_life = first_order_conversion(2.0, 0.1, 6.93);
% Result: X ≈ 0.5 (as predicted)
```

This confirms the calculation is correct.

### Engineering Interpretation

**Physical meaning:**

In a batch reactor with first-order kinetics:

1. **Early times:** Reaction rate is fast (high concentration), conversion increases quickly
2. **Late times:** Reaction rate slows (low concentration), conversion increases slowly
3. **Asymptotic behavior:** Conversion approaches 1 but never reaches it (mathematical property of exponential)

**Practical implication:**

- At k=0.1 min⁻¹, you achieve 90% conversion in ~23 minutes
- At k=0.1 min⁻¹, you achieve 99% conversion in ~46 minutes
- The benefit of running longer decreases (diminishing returns)

**Design application:**

If your process requires 95% conversion and k=0.1 min⁻¹, you need:

$$0.95 = 1 - e^{-0.1 \cdot t}$$
$$e^{-0.1 \cdot t} = 0.05$$
$$t = -\frac{\ln(0.05)}{0.1} ≈ 30 \text{ min}$$

This type of calculation is essential for reactor sizing and process economics.

</details>

---

