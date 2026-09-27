# Lecture 11: Numerical Solution of Algebraic Equations

## Learning Objectives

By the end of this lecture, you should be able to:

- Solve systems of linear algebraic equations using matrix methods and MATLAB's built-in functions
- Understand root-finding concepts and apply them to single nonlinear equations
- Use `fzero()` to find roots of single nonlinear functions
- Implement and compare iterative root-finding methods (bisection, secant, Newton-Raphson)
- Assess convergence criteria and estimate numerical errors
- Solve systems of nonlinear equations using `fsolve()`
- Handle multiple solutions and scaling considerations
- Apply equation-solving techniques to chemical engineering problems (equilibrium calculations, reactor design, thermodynamic equations)
- Compare symbolic `solve` vs numeric `fzero`/`fsolve` and choose appropriate method
---

## Why This Topic Matters

Many chemical engineering problems reduce to solving algebraic equations:

- **Material balances**: Finding unknown stream compositions or flow rates
- **Thermodynamics**: Solving equations of state, vapor-liquid equilibrium, or phase boundary conditions
- **Reaction engineering**: Determining conversion, reactor size, or residence time from rate equations
- **Process control**: Finding steady-state operating points
- **Heat transfer**: Solving energy balance equations with complex nonlinear terms

While some equations have closed-form analytical solutions, most real-world engineering problems involve nonlinear equations with no algebraic solution. Numerical methods allow us to find accurate solutions efficiently. This lecture teaches you both direct methods (for linear systems) and iterative methods (for nonlinear problems).

---

## Linear Algebraic Equations

### Concept

A system of linear algebraic equations has the form:

$$
\begin{align*}
a_{11}x_1 + a_{12}x_2 + \cdots + a_{1n}x_n &= b_1 \\
a_{21}x_1 + a_{22}x_2 + \cdots + a_{2n}x_n &= b_2 \\
&\vdots \\
a_{m1}x_1 + a_{m2}x_2 + \cdots + a_{mn}x_n &= b_m
\end{align*}
$$

This can be written compactly in matrix form as:

$$\mathbf{A} \mathbf{x} = \mathbf{b}$$

where:
- $\mathbf{A}$ is the coefficient matrix (m × n)
- $\mathbf{x}$ is the vector of unknowns
- $\mathbf{b}$ is the right-hand side vector



### MATLAB Syntax

**Method 1: Using the backslash operator (recommended for most cases)**

```matlab
x = A \ b;
```

**Method 2: Using matrix inverse (less efficient, but useful for understanding)**

```matlab
x = inv(A) * b;
```

**Method 3: Using `solve()` (available with Symbolic Math Toolbox)**

```matlab
x = solve(A*x_sym == b);
```

### Syntax Breakdown

| Component | Meaning |
|-----------|---------|
| `A` | Coefficient matrix (m × n), where each row is one equation |
| `b` | Right-hand side vector (m × 1) |
| `\` | MATLAB's linear equation solver (uses LU decomposition or QR for rectangular systems) |
| `x` | Solution vector containing values of the unknowns |
| `;` | Suppresses command-window output |

### Simple Example: Two-Equation System

**Problem:** Solve the system:
$$
\begin{align*}
2x_1 + 3x_2 &= 8 \\
4x_1 + x_2 &= 10
\end{align*}
$$

**MATLAB Code:**

```matlab
% Define the coefficient matrix A
A = [2  3;
     4  1];

% Define the right-hand side vector b
b = [8;
     10];

% Solve using backslash
x = A \ b;

% Display the result
disp(x);
```

**Expected Output:**

```
    2.2000
    1.2000
```

### Example Walkthrough

1. **Matrix setup**: We organize coefficients into matrix `A` and constants into vector `b`
2. **Backslash operation**: `A \ b` solves the system using numerical linear algebra
3. **Result**: `x` contains $x_1 = 2.2$ and $x_2 = 1.2$
4. **Verification**: We can check: $2(2.2) + 3(1.2) = 4.4 + 3.6 = 8$ and $4(2.2) + 1(1.2) = 8.8 + 1.2 = 10$

---

### Chemical Engineering Example: Three-Component Stream Mixing

**Problem:** Three inlet streams mix in a tank:
- Stream 1: 100 kg/h (20% A, 80% B)
- Stream 2: 150 kg/h (10% A, 90% B)
- Stream 3: unknown flow rate (5% A, 95% C)

The outlet stream must have exactly 500 kg/h total flow. Set up and solve for the unknown feed rate $F_3$ and the outlet composition.

**Analysis:**

Let $F_1 = 100$ kg/h, $F_2 = 150$ kg/h, $F_3 = $ unknown, and $F_{out} = 500$ kg/h, with outlet mass fractions $x_A$, $x_B$, $x_C$ (unknowns — not assumed).

Total balance: $F_1 + F_2 + F_3 = F_{out}$

Component A balance: $0.20F_1 + 0.10F_2 + 0.05F_3 = x_A F_{out}$

Component B balance: $0.80F_1 + 0.90F_2 = x_B F_{out}$

Component C balance: $0.95F_3 = x_C F_{out}$

These four equations in four unknowns ($F_3, x_A, x_B, x_C$) form a linear system $Ax = b$.

**MATLAB Code:**

```matlab
% Material balance for 3-stream mixing tank
% Unknowns: F3, xA_out, xB_out, xC_out
F1 = 100;   % kg/h, stream 1 (20% A, 80% B)
F2 = 150;   % kg/h, stream 2 (10% A, 90% B)
Fout = 500; % kg/h, outlet (fixed)
% Stream 3: 5% A, 95% C

% Variables x = [F3; xA; xB; xC]
% Eq1 (total):      F3                     = Fout - F1 - F2
% Eq2 (A balance):  0.05*F3 - Fout*xA       = -(0.20*F1 + 0.10*F2)
% Eq3 (B balance): -Fout*xB                 = -(0.80*F1 + 0.90*F2)
% Eq4 (C balance):  0.95*F3 - Fout*xC       = 0

A = [1     0      0      0;
     0.05 -Fout   0      0;
     0     0     -Fout   0;
     0.95  0      0     -Fout];

b = [Fout - F1 - F2;
     -(0.20*F1 + 0.10*F2);
     -(0.80*F1 + 0.90*F2);
     0];

x = A \ b;
F3 = x(1); xA = x(2); xB = x(3); xC = x(4);

fprintf('Stream 3 flow rate: %.2f kg/h\n', F3);
fprintf('Outlet composition - A: %.2f%%, B: %.2f%%, C: %.2f%%\n', ...
        xA*100, xB*100, xC*100);
fprintf('Composition sum check: %.2f%% (should be 100)\n', (xA+xB+xC)*100);

% Verification: species-by-species in vs out
A_in = 0.20*F1 + 0.10*F2 + 0.05*F3;
B_in = 0.80*F1 + 0.90*F2;
C_in = 0.95*F3;
fprintf('A: in=%.2f, out=%.2f\n', A_in, xA*Fout);
fprintf('B: in=%.2f, out=%.2f\n', B_in, xB*Fout);
fprintf('C: in=%.2f, out=%.2f\n', C_in, xC*Fout);
```

**Expected Output:**

```
Stream 3 flow rate: 250.00 kg/h
Outlet composition - A: 9.50%, B: 43.00%, C: 47.50%
Composition sum check: 100.00% (should be 100)
A: in=47.50, out=47.50
B: in=215.00, out=215.00
C: in=237.50, out=237.50
```

---
### Common Mistakes

1. **Dimension mismatch**: Forgetting that `b` must be a column vector (m × 1), not a row vector
   ```matlab
   % Wrong: b = [8, 10];        % 1 × 2 row vector
   % Correct:
   b = [8; 10];                  % 2 × 1 column vector
   ```

2. **Singular or ill-conditioned matrices**: If the coefficient matrix is singular (determinant = 0), the system has no unique solution
   ```matlab
   A = [1 2; 2 4];  % Second row is 2× the first: singular!
   b = [3; 6];
   x = A \ b;       % This will fail or give unexpected results
   ```

3. **Incorrectly ordering equations or unknowns**: Each row of `A` must correspond to one equation, and each column must correspond to one unknown

### Key Takeaways

- Use **matrix form** $\mathbf{A} \mathbf{x} = \mathbf{b}$ to organize linear systems
- Use the **backslash operator** `A \ b` for numerical stability
- **Verify** your solution by substituting back into the original equations
- Check matrix **rank** and **determinant** to ensure the system is well-posed


## Nonlinear Equations

### Concept

A nonlinear equation has the form:

$$f(x) = 0$$

where $f(x)$ is a nonlinear function of $x$. Unlike linear equations, these cannot generally be solved by direct algebraic manipulation.

**Examples from chemical engineering:**

- Vapor pressure (Antoine equation): $\log_{10} P = A - \frac{B}{C + T}$, solved for $T$ given $P$
- Compressibility factor: $Z = 1 + \frac{BP}{RT}$, solved for $Z$ or $P$
- Reaction equilibrium: $K = \frac{x_A^2}{(1-x_A)^2}$, solved for conversion $x_A$
- Energy balance with nonlinear dissipation: $Q = h A (T_s - T_{\infty}) + \sigma \epsilon A (T_s^4 - T_{\infty}^4)$

### Why Numerical Methods Are Essential

Many engineering equations are **transcendental** (involve exponentials, logarithms, trigonometric functions) or **polynomial** (degree ≥ 3), making analytical solutions impossible or impractical.

### Root-Finding Concepts

**Root**: A value $x^*$ such that $f(x^*) = 0$.

**Bracketing**: Finding an interval $[a, b]$ where $f(a)$ and $f(b)$ have opposite signs, guaranteeing a root exists (by the Intermediate Value Theorem).

**Tolerance/Convergence**: Stopping when $|f(x_n)| < \varepsilon$ or $|x_{n+1} - x_n| < \varepsilon$ for some small $\varepsilon$.

---

### MATLAB's `fzero()` Function

`fzero()` finds a root of a nonlinear function near a starting point or within a bracketing interval.

**Syntax:**

```matlab
root = fzero(fun, x0);
root = fzero(fun, [a, b]);
root = fzero(fun, x0, options);
```

**Syntax Breakdown:**

| Component | Meaning |
|-----------|---------|
| `fun` | Function handle (e.g., `@(x) x^2 - 4` or a named function) |
| `x0` | Initial guess (single value) |
| `[a, b]` | Bracketing interval where `f(a)` and `f(b)` have opposite signs |
| `options` | Optional optimization structure (e.g., `optimset('TolX', 1e-8)`) |
| `root` | Returned root (scalar) |

### Simple Example: Solving $x^2 - 4 = 0$

**MATLAB Code:**

```matlab
% Define the function as an anonymous function
f = @(x) x^2 - 4;

% Find root near x0 = 1 (should find root at x = 2)
root1 = fzero(f, 1);

% Find root using bracketing [0, 3]
root2 = fzero(f, [0, 3]);

% Display results
fprintf('Root near x=1: %.6f\n', root1);
fprintf('Root in [0,3]: %.6f\n', root2);

% Verification
fprintf('f(root1) = %.2e\n', f(root1));
```

**Expected Output:**

```
Root near x=1: 2.000000
Root in [0,3]: 2.000000
f(root1) = 0.00e+00
```

**Note**: The function is satisfied to machine precision (≈ 10^-16).

### Example Walkthrough

1. **Function definition**: We use an anonymous function `f = @(x) x^2 - 4`
2. **Root finding**: `fzero(f, 1)` searches for a root starting from `x = 1`
3. **Result**: Returns `x = 2.0000` (since $2^2 - 4 = 0$)
4. **Bracketing alternative**: `fzero(f, [0, 3])` uses interval bracketing (**faster if you know an interval containing the root**)
5. **Verification**: Evaluating `f(2)` gives essentially zero (machine precision error only)

---

### Chemical Engineering Example: Antoine Equation and Vapor Pressure

**Problem**: Find the temperature at which water boils at atmospheric pressure (1 atm = 101.325 kPa).

The Antoine equation is:
$$\log_{10} P = A - \frac{B}{C + T}$$

where $P$ is vapor pressure (bar), $T$ is temperature (°C), and for water (range 0–60°C):
- $A = 5.08354$
- $B = 1663.125$
- $C = -45.622$

We need to find $T$ such that $P = 1.01325$ bar (≈ 1 atm).

**MATLAB Code:**

```matlab
% Antoine equation parameters for water
A = 5.08354;
B = 1663.125;
C = -45.622;

% Define the objective function: f(T) = P_calc - P_target
P_target = 1.01325;  % bar, atmospheric pressure

% Anonymous function: Antoine equation
% Rearranged as: f(T) = 10^(A - B/(C+T)) - P_target = 0
f = @(T) 10^(A - B/(C + T)) - P_target;

% Find boiling point
% We expect it around 100°C, so use that as starting point
T_boil = fzero(f, 350);

fprintf('Boiling point of water at 1 atm: %.2f K\n', T_boil);
fprintf('Verification: P at %.2f K = %.5f bar\n', T_boil, 10^(A - B/(C + T_boil)));
```

**Expected Output:**

```
Boiling point of water at 1 atm: 373.15 K
Verification: P at 373.15 K = 1.01325 bar
```
### Chemical Engineering Example — Ideal-Gas Cubic (van der Waals form)

```matlab
% van der Waals: (P + a/V^2)(V - b) = RT
% Rearranged as f(V) = 0
a = 1.366;      % bar·L²/mol²  (CO2 example values)
b = 0.0386;     % L/mol
R = 0.08314;    % bar·L/(mol·K)
T = 300;        % K
P = 10;         % bar

f = @(V) (P + a./V.^2).*(V - b) - R*T;
V_root = fzero(f, 2);             % initial guess near ideal-gas volume
fprintf('Molar volume = %.4f L/mol\n', V_root);
```

---

### Common Mistakes

1. **Forgetting the function handle (`@`) or incorrect syntax**:
   ```matlab
   % Wrong: root = fzero(x^2 - 4, 1);
   % Correct:
   root = fzero(@(x) x^2 - 4, 1);
   ```

2. **Using a single starting point when the function has multiple roots**: You may find any root near your starting point, not necessarily the one you want
   ```matlab
   % f = x^2 - 4 has roots at x = ±2
   root1 = fzero(@(x) x^2 - 4, 1);    % Returns x ≈ 2
   root2 = fzero(@(x) x^2 - 4, -1);   % Returns x ≈ -2
   ```

3. **Bracketing interval not containing a root**: If `f(a)` and `f(b)` have the same sign, `fzero` will fail
   ```matlab
    % Wrong: bracketing interval [0, 1.5] for f(x) = x^2 - 4
    % f(0) = -4, f(1.5) = -1.75 
    % Correct: use [0, 3] where f(0) = -4, f(3) = 5 (opposite signs)
    root = fzero(@(x) x^2 - 4, [1, 2]);
   ```

4. **Not verifying the solution**: Always check that `f(root) ≈ 0`
   ```matlab
   root = fzero(f, x0);
   residual = f(root);
   if abs(residual) > 1e-6
       warning('Root may not be accurate: f(root) = %e', residual);
   end
   ```



## Iterative Root-Finding Methods

### Concept

Iterative methods generate a **sequence of approximations** $x_0, x_1, x_2, \ldots$ that converge to the true root $x^*$.

**Each iteration uses information from previous estimates to compute a better approximation.** This section compares three widely-used methods.

### Bisection Method

**Idea**: Repeatedly halve the interval containing the root.

**Algorithm**:
1. Start with bracketing interval $[a, b]$ where $f(a) \cdot f(b) < 0$
2. Compute midpoint: $c = \frac{a + b}{2}$
3. If $f(c) = 0$ or interval is small enough, stop (root found)
4. If $f(a) \cdot f(c) < 0$, the root is in $[a, c]$, so set $b = c$
5. Otherwise, the root is in $[c, b]$, so set $a = c$
6. Return to step 2

**Advantages**:
- Guaranteed convergence if initial bracket is valid
- Simple to implement
- Robust (always works if root exists in bracket)

**Disadvantages**:
- Slow convergence (linear)
- Requires a bracketing interval
- Number of iterations predictable: $n = \lceil \log_2 \frac{b - a}{\varepsilon} \rceil$

### MATLAB Implementation: Bisection

```matlab
function [root, iters] = bisection(f, a, b, tol)
    % BISECTION  Find root of f(x) using bisection method
    % 
    % Syntax:  [root, iters] = bisection(f, a, b, tol)
    % 
    % Inputs:
    %   f     - Function handle
    %   a, b  - Bracketing interval
    %   tol   - Convergence tolerance
    % 
    % Outputs:
    %   root  - Approximate root
    %   iters - Number of iterations
    
    % Check that bracketing interval is valid
    if f(a) * f(b) > 0
        error('Function must have opposite signs at a and b');
    end
    
    iters = 0;
    
    while abs(b - a) > tol
        c = (a + b) / 2;  % Midpoint
        
        if f(c) == 0
            root = c;
            return;
        end
        
        % Choose new interval
        if f(a) * f(c) < 0
            b = c;  % Root is in [a, c]
        else
            a = c;  % Root is in [c, b]
        end
        
        iters = iters + 1;
    end
    
    root = (a + b) / 2;
end
```

**Usage Example**:

```matlab
f = @(x) exp(x) - cos(x);
[root, iters] = bisection(f, -1, 3, 1e-6);
fprintf('Bisection: root = %.8f, iterations = %d\n', root, iters);

x_p = -5:0.001:5;
y_p = f(x_p);

figure
plot(x_p,y_p)
ylim([-1,3])
yline(0)
```

**Output**:
```
Bisection: root = 0.00000000, iterations = 1
```
---
### Fixed-Point Iteration

**Idea**: Rewrite the equation $f(x)=0$ in the form $x=g(x)$, then repeatedly evaluate

$$x_{n+1}=g(x_n)$$

The iterations converge to a fixed point, which is also a root of $f(x)$, when the function $g$ is chosen appropriately.

**Algorithm**:
1. Rearrange $f(x)=0$ as $x=g(x)$
2. Choose an initial guess $x_0$
3. Compute $x_{n+1}=g(x_n)$
4. Stop when $|x_{n+1}-x_n|<\text{tol}$ or $|f(x_{n+1})|<\text{tol}$
5. Otherwise, use $x_{n+1}$ as the next iterate

**Convergence**:
- Local convergence is generally expected if $|g'(x)|<1$ near the fixed point
- Smaller values of $|g'(x^*)|$ usually give faster convergence
- If $|g'(x)|>1$, the iteration may diverge

**Advantages**:
- Simple to understand and implement
- Does not require a derivative of $f$
- Useful when a natural physical or mathematical rearrangement gives a stable $g(x)$

**Disadvantages**:
- Convergence depends strongly on the choice of $g(x)$ and the initial guess
- Usually has linear convergence
- A poor rearrangement can diverge even when the original equation has a root

### MATLAB Implementation: Fixed-Point Iteration

```matlab
function [root, iters, history] = fixedPoint(g, f, x0, tol, max_iters)
    % FIXEDPOINT  Find a root using fixed-point iteration x = g(x)
    %
    % Inputs:
    %   g           - Fixed-point function
    %   f           - Original root function, f(x) = 0
    %   x0          - Initial guess
    %   tol         - Convergence tolerance
    %   max_iters   - Maximum number of iterations
    %
    % Outputs:
    %   root        - Approximate root
    %   iters       - Number of iterations taken
    %   history     - Iteration history [iteration, x, f(x)]

    if nargin < 4
        tol = 1e-8;
    end
    if nargin < 5
        max_iters = 100;
    end

    history = zeros(max_iters, 3);
    x = x0;

    for k = 1:max_iters
        x_new = g(x);
        history(k, :) = [k, x_new, f(x_new)];

        if abs(x_new - x) < tol || abs(f(x_new)) < tol
            root = x_new;
            iters = k;
            history = history(1:k, :);
            return;
        end

        x = x_new;
    end

    warning('Maximum iterations reached without convergence');
    root = x_new;
    iters = max_iters;
    history = history(1:max_iters, :);
end
```

**Usage Example**:

```matlab
% Solve x - cos(x) = 0 using x = cos(x)
f = @(x) x - cos(x);
g = @(x) cos(x);
[root, iters, history] = fixedPoint(g, f, 0.5, 1e-8, 100);
fprintf('Fixed-point iteration: root = %.8f, iterations = %d\n', root, iters);

figure
plot(history(:, 1), history(:, 2), 'o-')
grid on
xlabel('Iteration')
ylabel('Approximation')
title('Fixed-Point Iteration')

figure
fplot(@(x) x)
hold on
fplot(@(x) cos(x))
grid on
xlabel('x')
ylabel('y')
legend('y = x', 'y = cos(x)', 'Location', 'best')
hold off
```

**Output**:
```
Fixed-point iteration: root = 0.73908513, iterations = 44
```
---
### Secant Method

**Idea**: Approximate the derivative using two recent function evaluations, similar to Newton-Raphson but without computing the actual derivative.

**Algorithm**:
1. Start with two initial guesses: $x_0$ and $x_1$
2. Compute the next approximation:
   $$x_{n+1} = x_n - f(x_n) \frac{x_n - x_{n-1}}{f(x_n) - f(x_{n-1})}$$
3. Repeat until convergence

**Advantages**:
- Faster convergence than bisection (superlinear, order ≈ 1.618)
- Does not require derivative
- Only requires two initial guesses (not a bracketing interval)

**Disadvantages**:
- Can diverge if initial guesses are poor
- Not guaranteed to converge
- Requires evaluating the function twice per iteration 

### MATLAB Implementation: Secant Method

```matlab
function [root, iters] = secant(f, x0, x1, tol, max_iters)
    % SECANT  Find root of f(x) using secant method
    % 
    % Syntax:  [root, iters] = secant(f, x0, x1, tol, max_iters)
    % 
    % Inputs:
    %   f           - Function handle
    %   x0, x1      - Initial guesses
    %   tol         - Convergence tolerance
    %   max_iters   - Maximum number of iterations
    % 
    % Outputs:
    %   root        - Approximate root
    %   iters       - Number of iterations taken
    
    if nargin < 5
        max_iters = 100;
    end
    
    iters = 0;
    
    for k = 1:max_iters
        % Secant method iteration
        f_x0 = f(x0);
        f_x1 = f(x1);
        
        % Check for zero denominator
        if abs(f_x1 - f_x0) < eps
            warning('Function values too close; terminating');
            root = x1;
            return;
        end
        
        % Secant update
        x2 = x1 - f_x1 * (x1 - x0) / (f_x1 - f_x0);
        
        % Check convergence
        if abs(x2 - x1) < tol || abs(f(x2)) < tol
            root = x2;
            iters = k;
            return;
        end
        
        % Prepare for next iteration
        x0 = x1;
        x1 = x2;
    end
    
    warning('Maximum iterations reached without convergence');
    root = x1;
    iters = max_iters;
end
```

**Usage Example**:

```matlab
f = @(x) exp(x) - cos(x);
[root, iters] = secant(f, 1, 3, 1e-6, 100);
fprintf('Secant: root = %.8f, iterations = %d\n', root, iters);

x_p = -5:0.001:5;
y_p = f(x_p);

figure
plot(x_p,y_p)
ylim([-1,3])
yline(0)
```

**Output**:
```
Secant: root = 0.00000001, iterations = 8
```
---
### Newton-Raphson Method

#### Explanation

Newton-Raphson uses derivative: $x_{new}=x - f(x)/f'(x)$. Fast quadratic convergence near root, but needs derivative and good initial guess, may diverge.

Algorithm:

1. Initial guess $x_0$
2. For iter=1..maxIter: $x_{new}=x - f(x)/f'(x)$
3. If $|x_{new}-x|<\text{tol}$ or $|f(x_{new})|<\text{tol}$ → converged
4. Else $x=x_{new}$, continue

**Convergence:** Quadratic, doubles correct digits each iteration near root, but fails if $f'(x)=0$ or guess far from root.

**Derivative:** Can provide analytical or approximate numerically: $f'(x) \approx (f(x+h)-f(x))/h$

#### MATLAB Implementation

```matlab
function [root, iter, history] = newtonRaphson(func, dfunc, x0, tol, maxIter)
%NEWTONRAPHSON Newton-Raphson root finding
%   func: f(x), dfunc: f'(x), x0 initial guess
    if nargin<4, tol=1e-8; end
    if nargin<5, maxIter=100; end
    
    x = x0;
    history = [];
    for iter=1:maxIter
        fx = func(x);
        dfx = dfunc(x);
        if abs(dfx) < eps
            error('Derivative near zero, Newton fails');
        end
        x_new = x - fx/dfx;
        history = [history; iter x fx dfx x_new];
        
        if abs(x_new - x) < tol || abs(fx) < tol
            root = x_new;
            return;
        end
        x = x_new;
    end
    warning('Max iterations reached');
    root = x;
end
```
Example:
```matlab
% Example: x^2-2, f'(x)=2x
f = @(x) x^2 - 2;
df = @(x) 2*x;
[root, iter] = newtonRaphson(f, df, 1, 1e-10, 100);
fprintf('Root sqrt(2) ~ %.10f in %d iterations\n', root, iter);

% Chemical: bubble point with derivative via symbolic or finite difference
% For Antoine sum, derivative dP/dT = sum x_i * Psat_i * ln(10)*B/(T+C)^2
```

## Convergence Criteria and Error Estimation

**Convergence Criteria**:

Stop iterating when one of the following is satisfied:

1. **Absolute error**: $|x_{n+1} - x_n| < \varepsilon_a$ (e.g., $\varepsilon_a = 10^{-8}$)
2. **Relative error**: $\frac{|x_{n+1} - x_n|}{|x_{n+1}|} < \varepsilon_r$ (e.g., $\varepsilon_r = 10^{-8}$)
3. **Function residual**: $|f(x_n)| < \varepsilon_f$ (e.g., $\varepsilon_f = 10^{-8}$)
4. **Maximum iterations**: Stop if $n > n_{\max}$ (safety check)

**Error Estimation**:

The **true error** is $e_n = x_n - x^*$ (unknown, since $x^*$ is not known).

The **approximate error** is $\varepsilon_n = |x_n - x_{n-1}|$ (computable).

For iterative methods, a common rule of thumb is:

$$\text{Error} \approx c \cdot \varepsilon_n^p$$

where $p$ is the **order of convergence**:
- Bisection: $p = 1$ (linear)
- Fixed-point iteration: $p = 1$ (linear, when it converges)
- Secant: $p \approx 1.618$ (superlinear)
- Newton-Raphson: $p = 2$ (quadratic)

### Comparison of Root-Finding Methods

| Method | Convergence | Bracket Required? | Derivative Needed? | Robustness | Speed |
|--------|-------------|------|------|-----------|-------|
| Bisection | Linear (slow) | Yes | No | High | Slow |
| Fixed-point iteration | Linear (conditional) | No | No | Low to Moderate | Slow to Moderate |
| Secant | Superlinear | No | No | Moderate | Fast |
| Newton-Raphson | Quadratic | No | Yes | Moderate | Very Fast |
| `fzero` | Mixed | No or Yes | No | High | Fast |

**Practical Guidance**:

- Use **bisection** when you have a bracketing interval and need guaranteed convergence (e.g., table lookup or initial bracket finding)
- Use **fixed-point iteration** when you can choose a rearrangement $x=g(x)$ with $|g'(x)|<1$ near the expected root
- Use **secant** when you have two reasonable guesses but no derivative
- Use **Newton-Raphson** when the derivative is available (usually via symbolic differentiation or user-provided)
- Use **`fzero`** for most practical problems (combines bracketing and interpolation internally)

---

### Example: Comparing Methods on Antoine Equation

```matlab
% Antoine equation for water
A = 5.08354;
B = 1663.125;
C = -45.622;
P_target = 1.01325;

f = @(T) 10.^(A - B./(C + T)) - P_target;

% Method 1: Bisection
tol = 1e-10;
[root_bisect, iters_bisect] = bisection(f, 300, 500, tol);

% Method 2: Fixed-Point
g = @(T) B./(A - log10(P_target)) - C;
[root_fixed_point, iters_fixed_point] = fixedPoint(g, f, 300, tol, 100);


% Method 3: Secant
[root_secant, iters_secant] = secant(f, 300, 500, tol, 100);

% Method 4: Built-in fzero
root_fzero = fzero(f, 300);

fprintf('Root-Finding Method Comparison\n');
fprintf('%-15s | %-12s | %-10s\n', 'Method', 'Root (°C)', 'Iterations');
fprintf('%-15s | %.8f | %d\n', 'Bisection', root_bisect, iters_bisect);
fprintf('%-15s | %.8f | %d\n', 'Fixed-Point', root_fixed_point, iters_fixed_point);
fprintf('%-15s | %.8f | %d\n', 'Secant', root_secant, iters_secant);
fprintf('%-15s | %.8f | (built-in)\n', 'fzero', root_fzero);

figure
plot(300:0.1:400,f(300:0.1:400))
grid on
xlabel('Temperature (°C)')
ylabel('Pressure Error (bar)')
title('Antoine Equation Root Function')
yline(0, 'k--')
```

**Expected Output**:

```
Root-Finding Method Comparison
Method          | Root (°C)    | Iterations
Bisection       | 373.14914560 | 41
Fixed-Point     | 373.14914560 | 1
Secant          | 373.14914560 | 18
fzero           | 373.14914560 | (built-in)
```
---
### Common Mistakes

1. **Starting without a bracketing interval**: Always verify that your initial guesses bracket a root before using methods that assume this
   ```matlab
   f = @(x) x^2 + 1;  % No real roots!
   root = fzero(f, 1);  % Will fail or behave unexpectedly
   ```

2. **Not setting convergence tolerances appropriately**: Too loose and you get inaccurate results; too tight and you waste iterations
   ```matlab
   % Reasonable: 1e-9 to 1e-12 for most engineering problems
   tol = 1e-32;  % May be unnecessarily tight
   ```

3. **Confusing the convergence of $x_n$ with convergence of $f(x_n)$**: They can behave differently
   ```matlab
   % Always verify: abs(f(root)) should be small
   residual = abs(f(root));
   ```
## Polynomial Roots: roots()

The `roots()` function finds all roots of a polynomial:

```matlab
% Polynomial: x^3 - 6*x^2 + 11*x - 6 = 0
% Coefficients in descending order: [1, -6, 11, -6]
coeffs = [1, -6, 11, -6];
all_roots = roots(coeffs);

disp(all_roots)
% Output: [3; 2; 1]

% Verify
for x = all_roots'
    fprintf("x = %.6f, f(x) = %.2e\n", x, x^3 - 6*x^2 + 11*x - 6)
end
```

### Syntax Breakdown

```matlab
r = roots(p);
```

- `p` — polynomial coefficients in **descending order of powers**
- Output: roots (may be complex)

Example: 2x⁴ + 3x³ - x - 5 → coefficients = [2, 3, 0, -1, -5]

### Using poly() to Create Polynomial from Roots

```matlab
% Create polynomial from known roots
desired_roots = [1, 2, 3];
coeffs = poly(desired_roots);

disp(coeffs)    % [1, -6, 11, -6]
% This gives the polynomial (x-1)(x-2)(x-3)
```

## Systems of Nonlinear Equations

### Concept

A system of nonlinear equations has the form:

$$
\begin{align*}
f_1(x_1, x_2, \ldots, x_n) &= 0 \\
f_2(x_1, x_2, \ldots, x_n) &= 0 \\
&\vdots \\
f_n(x_1, x_2, \ldots, x_n) &= 0
\end{align*}
$$

In compact form: $\mathbf{F}(\mathbf{x}) = \mathbf{0}$

where $\mathbf{F}: \mathbb{R}^n \to \mathbb{R}^n$ is a vector function and $\mathbf{x} = [x_1, x_2, \ldots, x_n]^T$ is the unknown vector.

### Why Systems of Nonlinear Equations Arise

Many chemical engineering problems naturally lead to systems:

- **Equilibrium calculations**: Material and energy balances + phase equilibrium relations
- **Reactor networks**: Material balances around multiple reactors with interconnections
- **Distillation**: Simultaneous energy and material balances on multiple trays
- **Thermodynamic property calculations**: Solving equations of state and fugacity relations
- **Process optimization**: Setting gradients to zero in optimization problems

## MATLAB's `fsolve()` Function

`fsolve()` solves systems of nonlinear equations using trust-region or Levenberg-Marquardt methods.

**Syntax:**

```matlab
x = fsolve(fun, x0);
x = fsolve(fun, x0, options);
[x, fval, exitflag, output] = fsolve(fun, x0);
```

**Syntax Breakdown:**

| Component | Meaning |
|-----------|---------|
| `fun` | Function handle returning a vector of residuals (one per equation) |
| `x0` | Initial guess (vector of length $n$) |
| `options` | Optional structure (e.g., `optimset('Display', 'off')`) |
| `x` | Solution vector |
| `fval` | Vector of function values at the solution (should be ≈ 0) |
| `exitflag` | Status indicator (> 0 = converged, ≤ 0 = failed) |
| `output` | Structure with iteration count, algorithm info, etc. |

---
### Simple Example: Two Nonlinear Equations

**Problem**: Solve the system:
$$
\begin{align*}
x^2 + y^2 &= 1 \\
x - y &= 0.5
\end{align*}
$$

**MATLAB Code:**

```matlab
% Define the system of equations as a vector function
F = @(vars) [vars(1)^2 + vars(2)^2 - 1;    % x^2 + y^2 = 1
             vars(1) - vars(2) - 0.5];     % x - y = 0.5

% Initial guess
x0 = [1; 0.5];

% Solve using fsolve
[solution, fval] = fsolve(F, x0);

% Extract solutions
x_sol = solution(1);
y_sol = solution(2);

% Display results
fprintf('Solution:\n');
fprintf('  x = %.6f\n', x_sol);
fprintf('  y = %.6f\n', y_sol);
fprintf('Residuals:\n');
fprintf('  f1 = %e\n', fval(1));
fprintf('  f2 = %e\n', fval(2));

% Verification
fprintf('\nVerification:\n');
fprintf('  x^2 + y^2 = %.6f (should be 1.0)\n', x_sol^2 + y_sol^2);
fprintf('  x - y = %.6f (should be 0.5)\n', x_sol - y_sol);
```

**Expected Output:**

```
Equation solved.

fsolve completed because the vector of function values is near zero
as measured by the value of the function tolerance, and
the problem appears regular as measured by the gradient.

<stopping criteria details>
Solution:
  x = 0.911438
  y = 0.411438
Residuals:
  f1 = 8.417451e-10
  f2 = 7.638334e-14
Convergence status: 1 (positive = success)

Verification:
  x^2 + y^2 = 1.000000 (should be 1.0)
  x - y = 0.500000 (should be 0.5)
```

### Example Walkthrough

1. **System definition**: We encode two equations into a single vector function that returns residuals
2. **Initial guess**: We provide $[x_0; y_0] = [1; 0.5]$
3. **Solving**: `fsolve` iteratively refines the guess
4. **Result**: Returns solution vector and residuals
5. **Verification**: We check that both equations are satisfied

---
### Chemical Engineering Example: Vapor-Liquid Equilibrium (VLE) Calculation

For binary mixture at fixed T, P: Find mole fractions in liquid and vapor phases

```matlab
function F = flash_equations(vars)
    x1 = vars(1);    % Liquid mole fraction of component 1
    y1 = vars(2);    % Vapor mole fraction of component 1
    
    % Equilibrium constants (simplified)
    K1 = 2.5;   % Component 1
    K2 = 0.8;   % Component 2
    
    % Rachford-Rice equations
    F = [K1*x1 - y1;                    % Equilibrium
         (x1) + (1-x1) - 1];            % Closure
end

x0 = [0.4, 0.6];
solution = fsolve(@flash_equations, x0);

fprintf("Liquid composition: x1 = %.4f\n", solution(1))
fprintf("Vapor composition: y1 = %.4f\n", solution(2))
```



## Initial Guesses and Multiple Solutions

**Challenge**: Nonlinear systems can have multiple solutions. The solution found depends on the initial guess $\mathbf{x}_0$.

**Strategy for finding multiple solutions**:

```matlab
F = @(x) exp(x) - cos(x);

% Multiple initial guesses
x0_candidates = -5:0.1:5;

solutions = [];

for i = 1:length(x0_candidates)

    x0 = x0_candidates(i);

    [x, fval, exitflag] = fsolve(F, x0);

    if exitflag > 0 && abs(fval) < 1e-8

        % Check whether this root is already in solutions
        if isempty(solutions) || all(abs(solutions - x) > 1e-4)

            solutions(end+1) = x;

        end
    end
end

% Sort the solutions
solutions = sort(solutions);

fprintf('Found %d distinct solution(s)\n', length(solutions));
disp(solutions');


% Plot
x_p = -5:0.001:5;
y_p = F(x_p);

figure
plot(x_p, y_p, 'LineWidth', 1.5)
hold on
yline(0, '--')
ylim([-1 3])
grid on
xlabel('x')
ylabel('F(x)')
```

### Scaling Considerations

**Why scaling matters**: If variables have very different magnitudes, the solver may struggle.

**Example**:
```matlab
% Poorly scaled system
F = @(vars) [vars(1) + 1e-6*vars(2) - 1;
             1e-6*vars(1) + vars(2) - 1];
```

Here, $x_1$ and $x_2$ have very different relative importance.

**Solution: Rescale variables**:

```matlab
% Original variables:
% x1 ~ 1e3,  x2 ~ 1e-3

% Define scaled variables:
% u1 = x1/1e3,  u2 = x2/1e-3

F_scaled = @(u) [u(1) + u(2) - 2;
                  2*u(1) - u(2) - 1];

% Solve for scaled variables
u = fsolve(F_scaled, [1, 1]);

% Convert back to original variables
x = [u(1)*1e3, u(2)*1e-3];
```

### Setting Options

```matlab
% Create options
options = optimoptions("fsolve", ...
    "Display", "off", ...
    "TolFun", 1e-10, ...      % Function tolerance
    "TolX", 1e-10, ...        % Variable tolerance
    "MaxIterations", 10000);  % Maximum iterations

% Solve with options
solution = fsolve(@equations, x0, options);
```

## Common Mistakes

1. **Incorrect function output**: `fsolve` expects a vector of residuals, not individual equations
   ```matlab
    % Wrong: equations are returned separately
    F = @(x) [x(1)^2 - 4, x(2)^2 - 9];

    % Correct: return a column vector
    F = @(x) [x(1)^2 - 4;
            x(2)^2 - 9];

    % Solve
    x = fsolve(F, [1; 1]);
   ```

2. **Non-square systems**: `fsolve` works best when the number of equations equals the number of unknowns
   ```matlab
   % If you have 3 equations and 2 unknowns (overdetermined),
   % use least-squares methods instead: lsqnonlin
   ```

3. **Poor initial guess**: Especially for systems with multiple solutions


4. **Not checking `exitflag`**: A negative exit flag means the solver failed to converge
   ```matlab
   [x, fval, exitflag] = fsolve(F, x0);
   if exitflag <= 0
       warning('fsolve did not converge');
   end
   ```


## Worked Example: Complete Reactor Design Problem

**Problem**: A continuous stirred-tank reactor (CSTR) with an exothermic reaction needs to be designed. The reaction is irreversible and first-order:

$$A \to B, \quad r = k e^{-E/RT} C_A$$

**Given**:
- Feed concentration: $C_{A,in} = 2$ mol/L
- Feed temperature: $T_{in} = 300$ K
- Feed flow rate: $F = 10$ L/min
- Reactor volume: $V = 100$ L
- Reaction parameters: $k = 4 \times 10^{13}$ min$^{-1}$, $E = 60$ kJ/mol, $R = 8.314$ J/(mol·K)
- Heat of reaction: $\Delta H_r = -100$ kJ/mol of A reacted
- Overall heat-transfer coefficient: $U A = 5000$ J/(min·K)
- Coolant temperature: $T_{cool} = 290$ K

**Find**: Steady-state reactor conditions (conversion, temperature, concentrations)

**Equations**:

Material balance on A:
$$F C_{A,in} - F C_A - V k e^{-E/RT} C_A = 0$$

Energy balance:
$$F \rho c_p (T_{in} - T) - V(-\Delta H_r) k e^{-E/RT} C_A - U A (T - T_{cool}) = 0$$

Assume $\rho c_p \approx 4$ J/(g·K) = 4000 J/(L·K)

**MATLAB Solution**:

First we build the model as a separate function:
```matlab
% Define the reactor model as a system of equations
% Variables: [C_A, T]
function residuals = reactor_model(vars)

    % Given parameters
    C_A_in = 2;         % mol/L, inlet concentration
    T_in = 300;         % K, inlet temperature
    F = 10;             % L/min, feed flow rate
    V = 100;            % L, reactor volume
    R = 8.314;          % J/(mol·K), gas constant
    E = 60000;          % J/mol, activation energy (converted from 60 kJ/mol)
    k0 = 4e13;          % min^-1, pre-exponential factor
    dH_r = -100000;     % J/mol, heat of reaction (converted from -100 kJ/mol)
    UA = 5000;          % J/(min·K), overall heat-transfer coefficient × area
    T_cool = 290;       % K, coolant temperature
    rho_cp = 4000;      % J/(L·K), volumetric heat capacity
    
    C_A = vars(1);
    T = vars(2);
    
    % Reaction rate constant (Arrhenius)
    k = k0 * exp(-E / (R * T));
    
    % Material balance
    r_A = F * C_A_in - F * C_A - V * k * C_A;
    
    % Energy balance
    r_E = F * rho_cp * (T_in - T) - V * (-dH_r) * k * C_A - UA * (T - T_cool);
    
    residuals = [r_A; r_E];
end
```
Main script:
```matlab
% Reactor design problem: exothermic first-order reaction in CSTR

% Given parameters
C_A_in = 2;         % mol/L, inlet concentration
T_in = 300;         % K, inlet temperature
F = 10;             % L/min, feed flow rate
V = 100;            % L, reactor volume
R = 8.314;          % J/(mol·K), gas constant
E = 60000;          % J/mol, activation energy (converted from 60 kJ/mol)
k0 = 4e13;          % min^-1, pre-exponential factor
dH_r = -100000;     % J/mol, heat of reaction (converted from -100 kJ/mol)
UA = 5000;          % J/(min·K), overall heat-transfer coefficient × area
T_cool = 290;       % K, coolant temperature
rho_cp = 4000;      % J/(L·K), volumetric heat capacity


% Initial guess: assume moderate conversion and small temperature rise
x0 = [0.5; 310];  % C_A = 0.5 mol/L, T = 310 K

% Options for fsolve
options = optimset('Display', 'iter', 'TolFun', 1e-8);

% Solve the reactor equations
[solution, fval, exitflag, output] = fsolve(@reactor_model, x0, options);

C_A_sol = solution(1);
T_sol = solution(2);

% Calculate reaction rate and conversion
k_sol = k0 * exp(-E / (R * T_sol));
conversion = (C_A_in - C_A_sol) / C_A_in;
r_sol = k_sol * C_A_sol;

% Display results
fprintf('\n========== REACTOR DESIGN RESULTS ==========\n');
fprintf('Steady-State Conditions:\n');
fprintf('  Outlet concentration: C_A = %.4f mol/L\n', C_A_sol);
fprintf('  Outlet temperature:   T = %.2f K\n', T_sol);
fprintf('  Conversion:           X_A = %.2f %%\n', conversion * 100);
fprintf('  Reaction rate:        r_A = %.4f mol/(L·min)\n', r_sol);
fprintf('  Temperature rise:     ΔT = %.2f K\n', T_sol - T_in);

fprintf('\nVerification (residuals should be ≈ 0):\n');
fprintf('  Material balance residual: %e\n', fval(1));
fprintf('  Energy balance residual:   %e\n', fval(2));
fprintf('  Exit flag (>0 = success): %d\n', exitflag);

% Additional engineering calculations
heat_generated = V * (-dH_r) * r_sol;  % J/min
heat_removed = UA * (T_sol - T_cool);  % J/min
fprintf('\nHeat Transfer:\n');
fprintf('  Heat generated by reaction: %.2e J/min\n', heat_generated);
fprintf('  Heat removed by cooling:    %.2e J/min\n', heat_removed);
fprintf('  Energy balance check (diff): %.2e J/min\n', abs(heat_generated - heat_removed));

% Residence time
tau = V / F;  % min
fprintf('\nReactor Residence Time: τ = %.1f min\n', tau);
```

**Expected Output** (approximate):

```
========== REACTOR DESIGN RESULTS ==========
Steady-State Conditions:
  Outlet concentration: C_A = 0.0101 mol/L
  Outlet temperature:   T = 254.67 K
  Conversion:           X_A = 99.50 %
  Reaction rate:        r_A = 0.1990 mol/(L·min)
  Temperature rise:     ΔT = -45.33 K

Verification (residuals should be ≈ 0):
  Material balance residual: -6.394885e-14
  Energy balance residual:   -6.053597e-09
  Exit flag (>0 = success): 2

Heat Transfer:
  Heat generated by reaction: 1.99e+06 J/min
  Heat removed by cooling:    -1.77e+05 J/min
  Energy balance check (diff): 2.17e+06 J/min

Reactor Residence Time: τ = 10.0 min
```
## Worked Project: Transient 2-D conduction
A 1 by 2 cm ceramic strip [k = 3.0 W/m . °C] is embedded in a high-thermal-conductivity material, as shown in Figure Example 4-13 (Holman, "Heat Transfer" (10th ed.)), so that the sides are maintained at a constant temperature of 300 C. The bottom surface of the ceramic is insulated, and the top surface is exposed to a convection environment with h = 200 W/m2 . °C and T_inf = 50 °C. At time zero the ceramic is uniform in temperature at 300 °C. Calculate the temperatures at nodes 1 to 9 after a time of 12 s. For the ceramic ρ = 1600 kg/m3 and c = 0.8 kJ/kg . °C. Also calculate the total heat loss in this time.

```matlab
%% ========================================================================
%  Holman, "Heat Transfer" (10th ed.) - Example 4-13
%  Transient 2-D conduction in a ceramic strip, solved by the IMPLICIT
%  (backward-Euler) finite-difference method: at every time step the
%  nodal energy-balance equations are assembled into a residual function
%  F(Tnew) = 0 and solved with fsolve().
%
%  Geometry (see Fig. Example 4-13):
%   - 1 cm (y) x 2 cm (x) ceramic strip, k = 3.0 W/m.C
%   - Both sides (x = 0, x = 2 cm)  : held at Tw = 300 C
%   - Bottom (y = 1 cm)             : insulated
%   - Top (y = 0)                   : convection, h, Tinf
%   - Initial condition             : T = 300 C everywhere
%
%  3x3 nodal grid (dx = dy = 0.5 cm). By left-right symmetry T1=T3,
%  T4=T6, T7=T9, leaving 6 unknown nodal temperatures:
%
%        1 --- 2 --- 3      (top row,    convection boundary)
%        |     |     |
%        4 --- 5 --- 6      (middle row, fully interior)
%        |     |     |
%        7 --- 8 --- 9      (bottom row, insulated boundary)
%
%  Implicit backward-Euler energy balance for node i:
%       C_i * (Tnew_i - Told_i) / tau  =  sum_j (Tnew_j - Tnew_i) / R_ij
%  (all neighbor temperatures Tnew_j taken at the NEW time level, which
%  is what makes the method unconditionally stable and requires an
%  implicit solve instead of a simple explicit update.)
%
% ========================================================================

clear; clc; close all;

%% ---------------------- 1. Problem data -------------------------------
k       = 3.0;          % thermal conductivity, W/m.C
rho     = 1600;         % density, kg/m^3
c       = 0.8e3;        % specific heat, J/kg.C
h       = 200;          % convection coefficient, W/m^2.C
Tinf    = 50;           % ambient temperature, C
Tw      = 300;          % side-wall temperature, C
T0      = 300;          % initial temperature, C

dx      = 0.5e-2;       % nodal spacing, m
dy      = 0.5e-2;       % nodal spacing, m
depth   = 1;            % unit depth (per metre of strip length), m

t_final = 12;           % total time to simulate, s
tau     = 2.0;          % time step, s (implicit method is
                        % unconditionally stable, so this is a
                        % free choice - kept equal to the book's
                        % explicit-method value for comparison)

%% ------------------- 2. Thermal resistances ----------------------------
R_int   = dx / (k * dy * depth);          % interior lateral/vertical link, 0.3333 C/W
R_edge  = dx / (k * (dy/2) * depth);      % edge-row lateral link,          0.6667 C/W
R_conv  = 1 / (h * dx * depth);           % top-surface convective link,    1.0000 C/W

fprintf('Resistances  :  R_int = %.4f C/W , R_edge = %.4f C/W , R_conv = %.4f C/W\n', ...
         R_int, R_edge, R_conv);

%% ------------------- 3. Thermal (lumped) capacities ---------------------
C_full = rho * c * dx * dy * depth;        % interior node (4,5)
C_edge = rho * c * dx * (dy/2) * depth;    % edge node (1,2,7,8) - half cell

fprintf('Capacities   :  C_full = %.2f J/C , C_edge = %.2f J/C\n\n', C_full, C_edge);

%% ------------------- 4. Assemble the 6-node network ---------------------
% Unknown vector order: [T1 T2 T4 T5 T7 T8]
idx.T1 = 1; idx.T2 = 2; idx.T4 = 3; idx.T5 = 4; idx.T7 = 5; idx.T8 = 6;
names  = {'1','2','4','5','7','8'};
N = 6;

C = [C_edge; C_edge; C_full; C_full; C_edge; C_edge];   % nodal capacities

% Each row of a node's "links" matrix = [neighbor_index, R_ij, fixedT]
% neighbor_index = 0  ->  fixed-temperature reservoir (wall or ambient),
% with its value given in the 3rd column (NaN when the link goes to
% another unknown node instead of a reservoir).

net(idx.T1).links = [0,      R_edge,  Tw;     % left wall
                      idx.T2, R_edge,  NaN;    % -> node 2
                      idx.T4, R_int,   NaN;    % -> node 4
                      0,      R_conv,  Tinf];  % convection to ambient

net(idx.T2).links = [idx.T1, R_edge,  NaN;     % -> node 1 (mirror of node 3)
                      idx.T1, R_edge,  NaN;    % -> node 3 (= node 1 by symmetry)
                      idx.T5, R_int,   NaN;    % -> node 5
                      0,      R_conv,  Tinf];  % convection to ambient

net(idx.T4).links = [0,      R_int,   Tw;      % left wall
                      idx.T5, R_int,   NaN;    % -> node 5
                      idx.T1, R_int,   NaN;    % -> node 1
                      idx.T7, R_int,   NaN];   % -> node 7

net(idx.T5).links = [idx.T4, R_int,   NaN;     % -> node 4 (mirror of node 6)
                      idx.T4, R_int,   NaN;    % -> node 6 (= node 4 by symmetry)
                      idx.T2, R_int,   NaN;    % -> node 2
                      idx.T8, R_int,   NaN];   % -> node 8

net(idx.T7).links = [0,      R_edge,  Tw;      % left wall
                      idx.T8, R_edge,  NaN;    % -> node 8
                      idx.T4, R_int,   NaN];   % -> node 4
                      % bottom insulated -> no link

net(idx.T8).links = [idx.T7, R_edge,  NaN;     % -> node 7 (mirror of node 9)
                      idx.T7, R_edge,  NaN;    % -> node 9 (= node 7 by symmetry)
                      idx.T5, R_int,   NaN];   % -> node 5
                      % bottom insulated -> no link

%% ------------------- 5. Implicit time march using fsolve ----------------
nSteps      = round(t_final / tau);
T           = T0 * ones(N,1);
Thist       = zeros(N, nSteps+1);
Thist(:,1)  = T;

opts = optimset('Display','off', 'TolFun',1e-10, 'TolX',1e-10);

for p = 1:nSteps
    Told  = T;                      % known temperatures at start of step
    Tnew0 = Told;                   % initial guess for fsolve = previous step
    Tnew  = fsolve(@(Tg) nodalResiduals(Tg, Told, C, net, tau), Tnew0, opts);
    T = Tnew;
    Thist(:,p+1) = T;
end

%% ------------------- 6. Print the result table --------------------------
fprintf('Node temperature history (deg C) - IMPLICIT (backward-Euler) method:\n');
fprintf('  step   t(s)     T1       T2       T4       T5       T7       T8\n');

for p = 0:nSteps
    fprintf('  %3d   %5.1f  %7.2f  %7.2f  %7.2f  %7.2f  %7.2f  %7.2f\n', ...
        p, p*tau, Thist(1,p+1), Thist(2,p+1), Thist(3,p+1), ...
        Thist(4,p+1), Thist(5,p+1), Thist(6,p+1));
end

%% ------------------- 7. Total heat loss over 0 -> t_final ---------------
T1f = Thist(1,end); T2f = Thist(2,end);
T4f = Thist(3,end); T5f = Thist(4,end);
T7f = Thist(5,end); T8f = Thist(6,end);

q = C_edge*( 2*(T0-T1f) + (T0-T2f) + 2*(T0-T7f) + (T0-T8f) ) ...
  + C_full*( 2*(T0-T4f) + (T0-T5f) );

qdot_avg = q / t_final;

fprintf('\nTotal heat loss over %.0f s   : q     = %8.1f J   (per m of strip length)\n', t_final, q);
fprintf('Average heat-loss rate        : q/tau = %8.1f W   (per m of strip length)\n', qdot_avg);

%% ------------------- 8. Plots -------------------------------------------
figure('Name','Example 4-13 (implicit): Nodal temperature histories');
t_vec = (0:nSteps)*tau;
plot(t_vec, Thist','-o','LineWidth',1.4,'MarkerSize',4); grid on;
xlabel('Time, s'); ylabel('Temperature, ^{\circ}C');
title('Transient nodal temperatures - Holman Example 4-13 (implicit, fsolve)');
legend({'T_1 (=T_3)','T_2','T_4 (=T_6)','T_5','T_7 (=T_9)','T_8'}, 'Location','southwest');

figure('Name','Example 4-13 (implicit): Final temperature field');
Tfield = [T1f T2f T1f; T4f T5f T4f; T7f T8f T7f];
x_cm = [0.5 1.0 1.5];
y_cm = [0.0 0.5 1.0];
imagesc(x_cm, y_cm, Tfield); axis image; set(gca,'YDir','normal');
colorbar; colormap('turbo');
xlabel('x, cm'); ylabel('y, cm');
title(sprintf('Temperature field at t = %.0f s (^{\\circ}C)', t_final));
for ii = 1:3
    for jj = 1:3
        text(x_cm(jj), y_cm(ii), sprintf('%.1f', Tfield(ii,jj)), ...
             'HorizontalAlignment','center','Color','w','FontWeight','bold');
    end
end

%% ========================================================================
%  Local function: nodal energy-balance residuals for the implicit scheme
%  This is the system F(Tnew) = 0 that fsolve() solves at every time
%  step. Each row implements one node's backward-Euler energy balance:
%
% ========================================================================
function R = nodalResiduals(Tnew, Told, C, net, tau)
    N = numel(Tnew);
    R = zeros(N,1);
    for i = 1:N
        L = net(i).links;           % this node's [neighbor, R_ij, fixedT] rows
        q_net = 0;                  % net conductive/convective heat IN to node i
        for r = 1:size(L,1)
            j   = L(r,1);
            Rij = L(r,2);
            if j == 0
                Tj = L(r,3);        % fixed reservoir (wall or ambient)
            else
                Tj = Tnew(j);       % neighbor's NEW (unknown) temperature
            end
            q_net = q_net + (Tj - Tnew(i)) / Rij;
        end
        R(i) = C(i) * (Tnew(i) - Told(i)) / tau - q_net;
    end
end
```

---

### Important MATLAB Syntax Reference

| Purpose | MATLAB Syntax |
|---------|---|
| Solve linear system $\mathbf{A}\mathbf{x} = \mathbf{b}$ | `x = A \ b;` |
| Alternative (less efficient) | `x = inv(A) * b;` |
| Find single root of $f(x) = 0$ (bracketing) | `root = fzero(f, [a, b]);` |
| Find single root (initial guess) | `root = fzero(f, x0);` |
| Set convergence tolerance for `fzero` | `opts = optimset('TolX', 1e-8); root = fzero(f, x0, opts);` |
| Solve system $\mathbf{F}(\mathbf{x}) = \mathbf{0}$ | `x = fsolve(F, x0);` |
| Get convergence info from `fsolve` | `[x, fval, exitflag] = fsolve(F, x0);` |
| Check for convergence | `if exitflag > 0, solution converged; end` |
| Set options for `fsolve` | `opts = optimset('Display', 'iter', 'TolFun', 1e-8); fsolve(F, x0, opts);` |

### Common Pitfalls to Avoid

1. Not verifying solutions by substituting back into original equations
2. Using incorrect dimensions (e.g., row vs. column vectors)
3. Choosing poor initial guesses without physical reasoning
4. Ignoring scaling issues when variables have vastly different magnitudes
5. Not checking convergence flags (`exitflag`, residual values)


## Homework

### Problem

Develop a **pressure drop calculator** for flow in pipes using the **Darcy-Weisbach equation**:

$$f = \begin{cases} 
64/Re & \text{laminar} (Re < 2300) \\
0.316 Re^{-0.25} & \text{turbulent} (Re > 2300)
\end{cases}$$

$$\Delta P = f \frac{L}{D} \frac{\rho v^2}{2}$$

**Required Tasks:**

1. Write function to calculate friction factor from Reynolds number
2. Create solver to find velocity given desired pressure drop
3. Solve for pipe diameter given flow and pressure drop constraints
4. Handle transitions between laminar and turbulent flow
5. Create a lookup table for common scenarios

**Concepts Being Tested:**

- Function definition and evaluation
- Root finding with fzero()
- Piecewise functions (laminar vs. turbulent)
- Iterative solutions
- Engineering data organization

**Hints:**

- Reynolds number: Re = ρ*v*D/μ
- Velocity and Re are interdependent
- Use fzero with velocity bracket [0.1, 10] m/s
- Iterate if needed for convergence

<details>
<summary><strong>Detailed Solution — Click to Reveal</strong></summary>

## Solution

### Concept

**Pipe flow calculations** require solving nonlinear equations because Reynolds number and velocity are coupled.

### MATLAB Code

```matlab
clear; clc;

fprintf("===== PIPE FLOW PRESSURE DROP ANALYSIS =====\n\n")

% Properties of water at 20°C
rho = 1000;          % kg/m³
mu = 0.001;          % Pa·s (dynamic viscosity)
nu = mu / rho;       % m²/s (kinematic viscosity)

% Pipe parameters
D = 0.05;            % 5 cm diameter
L = 100;             % 100 m length
dP_target = 50000;   % 50 kPa desired pressure drop

% ===== Function: Calculate friction factor =====
function f = friction_factor(Re)
    if Re < 2300
        f = 64 / Re;        % Laminar
    else
        f = 0.316 * Re^(-0.25);  % Turbulent (Blasius)
    end
end

% ===== Function: Pressure drop given velocity =====
function dP = pressure_drop(v, D, L, rho, mu)
    % Calculate Reynolds number
    Re = rho * v * D / mu;
    
    % Get friction factor
    f = friction_factor(Re);
    
    % Darcy-Weisbach equation
    dP = f * (L/D) * (rho * v^2 / 2);
end

% ===== Solve for velocity =====
fprintf("Finding velocity for dP = %.0f Pa:\n", dP_target)

fun = @(v) pressure_drop(v, D, L, rho, mu) - dP_target;
v_required = fzero(fun, [0.1, 5]);

% Verify solution
Re_final = rho * v_required * D / mu;
dP_final = pressure_drop(v_required, D, L, rho, mu);
Q = v_required * pi * D^2 / 4;  % Flowrate

fprintf("  Velocity: %.4f m/s\n", v_required)
fprintf("  Reynolds: %.0f\n", Re_final)
fprintf("  Flowrate: %.4f L/min\n", Q * 60000)
fprintf("  Pressure drop: %.0f Pa (%.1f kPa)\n", dP_final, dP_final/1000)

% ===== Sensitivity analysis =====
fprintf("\n Velocity vs. Pressure drop:\n")
v_range = linspace(0.1, 3, 20);
dP_range = arrayfun(@(v) pressure_drop(v, D, L, rho, mu), v_range);

fprintf("  v (m/s)   dP (kPa)\n")
for i = 1:length(v_range)
    fprintf("  %.2f      %.1f\n", v_range(i), dP_range(i)/1000)
end

% Plot
figure
plot(v_range, dP_range/1000, "b-", "LineWidth", 2.5)
hold on
plot(v_required, dP_target/1000, "ro", "MarkerSize", 10)
xlabel("Velocity (m/s)", "FontSize", 12)
ylabel("Pressure drop (kPa)", "FontSize", 12)
title("Pipe Pressure Drop: D=5cm, L=100m", "FontSize", 12)
grid on
```

### Expected Result

```
===== PIPE FLOW PRESSURE DROP ANALYSIS =====

Finding velocity for dP = 50000 Pa:
  Velocity: 1.6358 m/s
  Reynolds: 81790
  Flowrate: 192.7125 L/min
  Pressure drop: 50000 Pa (50.0 kPa) ...
```

</details>

---


