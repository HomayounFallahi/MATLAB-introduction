# Lecture 10: Symbolic Mathematics

## Learning Objectives

By the end of this lecture, you should be able to:

- Create symbolic variables and expressions in MATLAB using the Symbolic Math Toolbox
- Perform algebraic manipulations: simplification, expansion, and factorization
- Solve symbolic equations and systems of equations analytically
- Perform symbolic differentiation and integration
- Evaluate limits symbolically
- Apply symbolic mathematics to derive and solve chemical engineering equations analytically
- Combine numerical and symbolic approaches for complex engineering problems

---

## Why This Topic Matters

So far, you have used MATLAB to solve problems **numerically** — computing approximate answers using algorithms and iteration. Symbolic mathematics allows you to work with equations, expressions, and functions **symbolically**, manipulating them as mathematical objects rather than numbers.

In chemical engineering, this is powerful because:

1. **Exact solutions**: Sometimes you can derive an exact, closed-form solution rather than an approximation.
2. **Insight**: Symbolic derivations reveal how variables relate to each other mathematically.
3. **Verification**: You can use symbolic mathematics to check numerical results.
4. **Derivation of equations**: You can symbolically manipulate material and energy balance equations to isolate unknowns.
5. **Parameter sensitivity**: Understanding how parameters appear in the solution helps predict system behavior.

For example, you might want to derive the analytical solution for concentration in a reactor, or simplify a complex energy balance equation before solving it numerically.

---

## Symbolic Variables and Expressions

### Concept

In numerical MATLAB, variables store numbers:

```matlab
x = 5;
y = x + 3;  % y = 8
```

In symbolic MATLAB, variables represent mathematical symbols. They can be manipulated as mathematical objects without assigning numerical values.

```matlab
syms x
y = x + 3;  % y is now the expression x+3 (not a number)
```

### The Symbolic Math Toolbox

MATLAB's **Symbolic Math Toolbox** provides the tools to create and manipulate symbolic expressions.

If you do not have the Symbolic Math Toolbox installed, some commands will not work. Check by typing:

```matlab
ver symbolic
```

### Creating Symbolic Variables

**Syntax:**

```matlab
% Define symbolic variables
syms x y z

% Define with assumptions
syms T positive real       % Temperature is positive and real
syms C_A nonnegative      % Concentration is non-negative
syms k positive           % Rate constant is positive
```

#### Syntax Breakdown

```matlab
syms var1 var2 var3 [conditions]
```

- `var1, var2, var3` — symbolic variables
- `real` — variable is real-valued
- `positive` — variable > 0
- `integer` — variable is an integer
- `nonnegative` — variable ≥ 0

You can also create individual variables:

```matlab
x = sym('x');
y = sym('y');
```

**Syntax Breakdown:**

- `sym('x')` creates a single symbolic variable with the name `x`
- The result is assigned to the MATLAB variable `x`

### Assumptions: Important for simplification and solving.

```matlab
syms x
assume(x>0)  % or syms x positive
% Or
syms x positive
syms T real

% Clear assumptions
assume(x,'clear')
```

### Creating Symbolic Expressions

Once you have symbolic variables, you can create expressions:

```matlab
syms x
expr = x^2 + 3*x + 2;
```

The variable `expr` now holds the symbolic expression $x^2 + 3x + 2$.

### Symbolic Constants

You can also create symbolic constants:

```matlab
syms pi_sym
pi_sym = sym(pi);  % The true mathematical constant pi
```

Or more commonly, just use `sym()` for mathematical constants:

```matlab
syms x
expr = sqrt(2) * x;  % sqrt(2) is treated symbolically
```

### Simple Example

```matlab
% Create symbolic variables
syms T P R n

% Create an ideal gas law expression
% PV = nRT, solve for V
V = n * R * T / P;

% Display the expression
disp(V)
```

**Output:**
```
(R*T*n)/P
```

### Example Walkthrough

1. `syms T P R n` declares four symbolic variables representing thermodynamic quantities
2. `V = n * R * T / P` constructs the symbolic expression for volume
3. `disp(V)` displays the expression in simplified mathematical notation
4. The expression is not evaluated numerically; it remains as a formula

### Chemical Engineering Example

Suppose you have the Clausius-Clapeyron equation (vapor pressure as a function of temperature):

$$\ln P_{\text{vap}} = A - \frac{B}{T + C}$$

You can create this symbolically:

```matlab
syms T P_vap A B C

% Clausius-Clapeyron equation
clausius_clapeyron = log(P_vap) - (A - B/(T + C));
disp(clausius_clapeyron)
```

**Output:**
```
log(P_vap) - A + B/(T + C)
```

### Common Mistakes

1. **Forgetting to declare variables as symbolic:**
   ```matlab
   expr = x^2 + 1;  % ERROR: x is undefined
   ```
   Fix: Always use `syms` first.

2. **Confusing symbolic and numerical computation:**
   ```matlab
   syms x
   y = x + 3;
   z = y * 2;  % z is 2*(x+3), not a number
   ```
   The expression is not simplified automatically; you must call `simplify()`.

3. **Using the wrong operator:**
   ```matlab
   syms x
   y = x ^ 2;       % Correct
   y = x ** 2;      % ERROR: ** is not valid in MATLAB
   ```


## Printing and Display

### Pretty Print

```matlab
syms x y

expr = (x^2 + 2*x + 1) / (x + 1);
pretty(expr)        % Displays in readable format

% LaTeX format
latex_form = latex(expr);
disp(latex_form)    % LaTeX code
```

### Symbolic to Numeric Conversion

```matlab
syms x
expr = sqrt(2);

% Double precision numeric
value = double(expr);   % 1.4142...

% High precision numeric
value_hp = vpa(expr, 50);  % 50 significant digits
disp(value_hp)
```

## Symbolic Algebra

### Concept

Once you have symbolic expressions, you can manipulate them algebraically: simplify, expand, factor, and substitute values.

### Simplification

**Purpose:** Make an expression shorter or more recognizable.

**Syntax:**

```matlab
simplified_expr = simplify(expr);
```

**Explanation:**

- `simplify()` applies algebraic rules to reduce the expression
- The result may be shorter, combined, or in a standard form

### Simple Example

```matlab
syms x
expr = (x^2 + 2*x + 1) / (x + 1);
simplified = simplify(expr);
disp(simplified)
```

**Output:**
```
x + 1
```

The expression simplifies because $\frac{(x+1)^2}{x+1} = x + 1$.

### Expansion

**Purpose:** Multiply out brackets and express a polynomial in standard form.

**Syntax:**

```matlab
expanded_expr = expand(expr);
```

### Simple Example

```matlab
syms x y
expr = (x + y)^3;
expanded = expand(expr);
disp(expanded)
```

**Output:**
```
x^3 + 3*x^2*y + 3*x*y^2 + y^3
```

### Factorization

**Purpose:** Express a polynomial as a product of simpler factors.

**Syntax:**

```matlab
factored_expr = factor(expr);
```

### Simple Example

```matlab
syms x
expr = x^2 - 5*x + 6;
factored = factor(expr);
disp(factored)
```

**Output:**
```
(x - 2)*(x - 3)
```

### Collecting Terms

```matlab
syms x y
expr = x^2*y + x*y^2 + 2*x*y + x^2 + y^2;

% Collect terms in x
collected = collect(expr, x);
% Result: x^2*(y + 1) + x*(y^2 + 2*y) + y^2

% Collect terms in y
collected = collect(expr, y);
% Result: y^2*(x + 1) + y*(x^2 + 2*x) + x^2
```

### Substitution

**Purpose:** Replace a symbol with a numerical value or another expression.

**Syntax:**

```matlab
result = subs(expr, symbol, value);
```

**Explanation:**

- `subs()` is the substitution function
- `expr` is the symbolic expression
- `symbol` is the variable to replace
- `value` is what to replace it with (a number or another expression)

### Simple Example

```matlab
syms x
expr = x^2 + 3*x + 2;

% Substitute x = 5
result = subs(expr, x, 5);
disp(result)  % Output: 42
```

You can also substitute multiple variables:

```matlab
syms x y
expr = x^2 + y^2;

result = subs(expr, {x, y}, {2, 3});
disp(result)  % Output: 13
```

### Example Walkthrough

```matlab
syms T

% Energy required to heat water from T1 to T2 (simplified)
% Q = m * c * (T2 - T1)
% Expressed as a function of T (where T = final temperature)
Q = 1000 * 4.18 * (T - 25);

% Simplify
Q_simplified = simplify(Q);
disp(Q_simplified)

% Substitute T = 85 (final temperature in Celsius)
Q_at_85 = subs(Q_simplified, T, 85);
disp(Q_at_85)  % Energy needed in Joules
```

**Output:**
```
4180*T - 104500
 
250800
```

### Chemical Engineering Example

Consider the van der Waals equation for a real gas:

$$\left(P + \frac{a}{V_m^2}\right)(V_m - b) = RT$$

where $V_m$ is molar volume. Rearranging to solve for pressure:

```matlab
syms P Vm R T a b

% Van der Waals equation rearranged for P
van_der_waals = P - R*T/(Vm - b) + a/Vm^2;

% Solve for P
P_solution = solve(van_der_waals, P);
disp(P_solution)

% Simplify
P_simplified = simplify(P_solution);
disp(P_simplified)

% For methane: a = 2.283 atm·L²/mol², b = 0.04267 L/mol, R = 0.08206 L·atm/(mol·K), T = 350 K, Vm = 2 L/mol
P_value = subs(P_simplified, {R, T, a, b, Vm}, {0.08206, 350, 2.283, 0.04267, 2});
disp(P_value)
disp(double(P_value))
```

**Output:**
```
(R*T)/(Vm - b) - a/Vm^2
 
(R*T)/(Vm - b) - a/Vm^2
 
11041541561/782932000
 
   14.1028
```

### Syntax Table: Common Algebraic Operations

| Operation | MATLAB Syntax | Example |
|---|---|---|
| Simplify | `simplify(expr)` | `simplify((x+1)^2 - x^2 - 2*x)` |
| Expand | `expand(expr)` | `expand((x+2)^3)` |
| Factor | `factor(expr)` | `factor(x^2 - 5*x + 6)` |
| Substitute | `subs(expr, var, val)` | `subs(x^2 + 1, x, 3)` |
| Collect terms | `collect(expr, var)` | `collect((x+1)^3, x)` |
| Simplify fractions | `simplifyFraction(expr)` | `simplifyFraction((x^2-1)/(x-1))` |

### Common Mistakes

1. **Using `simplify()` too early:**
   ```matlab
   syms x
   expr = (x + 1)^2;
   % Don't do: simplified = simplify(expr);
   % MATLAB will not automatically expand this.
   % If you want the expansion, use expand() first.
   ```

2. **Forgetting the assignment:**
   ```matlab
   syms x
   expr = x^2 + 2*x + 1;
   simplify(expr);  % This runs simplify but does NOT store the result!
   
   % Correct:
   simplified = simplify(expr);
   ```

3. **Substituting into the wrong variable:**
   ```matlab
   syms x y
   expr = x^2 + y;
   result = subs(expr, y, 5);  % Correct
   result = subs(expr, x, 5);  % Different result
   ```

---

## Symbolic Equations and Solving

### Concept

A symbolic equation is a statement of equality between two expressions. You can solve equations symbolically to find exact solutions.

### Creating and Solving Equations

**Syntax:**

```matlab
syms x
equation = x^2 + 3*x + 2 == 0;
solution = solve(equation, x);
disp(solution)
```

**Explanation:**

- `==` defines an equation (one equals sign `=` is assignment; two equals signs `==` define an equation)
- `solve(equation, variable)` solves the equation for the specified variable
- The result is the solution (or solutions)

### Simple Example

```matlab
syms x
equation = x^2 - 5*x + 6 == 0;
x_solutions = solve(equation, x);
disp(x_solutions)
```

**Output:**
```
  2
  3
```

MATLAB found two solutions: $x = 2$ and $x = 3$.

### Systems of Equations

You can solve multiple equations simultaneously:

```matlab
syms x y
eq1 = 2*x + 3*y == 12;
eq2 = x - y == 1;

solution = solve([eq1, eq2], [x, y]);
disp(double(solution.x))
disp(double(solution.y))
```

**Output:**
```
solution.x = 3
solution.y = 2
```

### Example Walkthrough

Material balance for a mixer:

$$F_1 + F_2 = F_3$$
$$x_1 F_1 + x_2 F_2 = x_3 F_3$$

where $F_i$ are flow rates and $x_i$ are mass fractions.

Given: $F_1 = 100$ kg/s, $x_1 = 0.2$, $F_2 = 50$ kg/s, $x_2 = 0.8$. Find: $F_3$ and $x_3$.

```matlab
syms F1 F2 F3 x1 x2 x3

% Material balance equations
eq1 = F1 + F2 == F3;
eq2 = x1*F1 + x2*F2 == x3*F3;

% Solve the system
solution = solve([eq1, eq2], [F3, x3]);

% Substitute known values
F1_val = 100;  % kg/s
x1_val = 0.2;
F2_val = 50;   % kg/s
x2_val = 0.8;

F3_result = subs(solution.F3, {F1, x1, F2, x2}, {F1_val, x1_val, F2_val, x2_val});
x3_result = subs(solution.x3, {F1, x1, F2, x2}, {F1_val, x1_val, F2_val, x2_val});

disp(['F3 = ', num2str(double(F3_result)), ' kg/s']);
disp(['x3 = ', num2str(double(x3_result))]);
```

**Output:**
```
F3 = 150 kg/s
x3 = 0.4
```

### Chemical Engineering Example: Antoine Equation

The Antoine equation gives vapor pressure as a function of temperature:

$$\log_{10} P_{\text{vap}} = A - \frac{B}{T + C}$$

For water: $A = 5.08354$, $B = 1663.125$, $C = -45.622$ (T in °C, P in bar).

Find the boiling point (where $P_{\text{vap}} = 1$ bar):

```matlab
syms T

% Antoine equation for water
A = 5.08354;
B = 1663.125;
C = -45.622;
P_vap = 1;  % We want P_vap = 1 bar (normal boiling point)

% Set up equation
antoine_eq = log10(P_vap) == A - B/(T + C);

% Solve for T
T_boiling = solve(antoine_eq, T);
disp(['Boiling point of water: ', num2str(double(T_boiling)-273.15), ' °C']);
```

**Output:**
```
Boiling point of water: 99.63 °C
```

### Handling Multiple Solutions

Some equations have multiple solutions. `solve()` returns all of them:

```matlab
syms x
equation = x^3 - 6*x^2 + 11*x - 6 == 0;
solutions = solve(equation, x);
disp(solutions)
```

**Output:**
```
  1
  2
  3
```

You can access individual solutions:

```matlab
sol1 = solutions(1);  % First solution
sol2 = solutions(2);  % Second solution
```

### Syntax Table: Solving Equations

| Operation | MATLAB Syntax | Example |
|---|---|---|
| Solve single equation | `solve(equation, variable)` | `solve(x^2 - 4 == 0, x)` |
| Solve system | `solve([eq1, eq2, ...], [var1, var2, ...])` | `solve([x+y==5, x-y==1], [x, y])` |
| Obtain structure | `solution.variable` | `sol.x` |
| Count solutions | `length(solutions)` | `length(solve(...))` |

### Common Mistakes

1. **Using single `=` instead of `==`:**
   ```matlab
   % WRONG:
   eq = x^2 + 1 = 0;   % ERROR: = is assignment, not equality
   
   % Correct:
   eq = x^2 + 1 == 0;  % == defines an equation
   ```

2. **Forgetting to specify the variable to solve for:**
   ```matlab
   syms x y
   eq = x + y == 5;
   sol = solve(eq);  % MATLAB may not know which variable to solve for
   
   % Correct:
   sol = solve(eq, x);  % Solve for x
   ```

3. **Assuming all equations have real solutions:**
   ```matlab
   syms x
   eq = x^2 + 1 == 0;
   sol = solve(eq, x);
   disp(sol)  % Returns [i, -i] (complex solutions)
   ```

### Key Takeaways

- Use `==` to define symbolic equations
- `solve(equation, variable)` finds the solution(s)
- Systems of equations are solved simultaneously
- Use `subs()` to evaluate solutions numerically
- Some equations have no real solutions (only complex)

---



## Symbolic Calculus

### Concept

Symbolic mathematics allows you to compute derivatives and integrals exactly, without numerical approximation.

### Symbolic Differentiation

**Purpose:** Find the derivative of an expression with respect to a variable.

**Syntax:**

```matlab
derivative = diff(expr, variable);
```

**Explanation:**

- `diff()` computes the derivative
- `expr` is the symbolic expression
- `variable` is the variable to differentiate with respect to

### Simple Example

```matlab
syms x
expr = x^3 + 2*x^2 - 5*x + 7;
derivative = diff(expr, x);
disp(derivative)
```

**Output:**
```
3*x^2 + 4*x - 5
```

### Higher-Order Derivatives

To find the second derivative, differentiate twice:

```matlab
syms x
expr = x^4 - 3*x^2 + 2;
first_deriv = diff(expr, x);
second_deriv = diff(first_deriv, x);
disp(second_deriv)
```

**Output:**
```
12*x^2 - 6
```

Or specify the order directly:

```matlab
second_deriv = diff(expr, x, 2);
disp(second_deriv)% Second derivative
third_deriv = diff(expr, x, 3);   % Third derivative
disp(third_deriv)
```

### Partial Derivatives

For functions of multiple variables, specify which variable to differentiate:

```matlab
syms x y
expr = x^2*y + x*y^2 + y^3;

% Partial derivative with respect to x
dfdx = diff(expr, x);
disp(dfdx)

% Partial derivative with respect to y
dfdy = diff(expr, y);
disp(dfdy)
```

**Output:**
```
y^2 + 2*x*y
 
x^2 + 2*x*y + 3*y^2
```

### Symbolic Integration

**Purpose:** Find the antiderivative of an expression.

**Syntax:**

```matlab
integral_result = int(expr, variable);
```

### Simple Example

```matlab
syms x
expr = 3*x^2 + 2*x + 1;
integral = int(expr, x);
disp(expand(integral))
```

**Output:**
```
x^3 + x^2 + x
```

### Definite Integrals

Integrate between two limits:

```matlab
syms x
expr = x^2;
definite_integral = int(expr, x, 0, 2);  % Integral from x=0 to x=2
disp(definite_integral)
```

**Output:**
```
8/3
```

which is $\int_0^2 x^2 \, dx = \frac{x^3}{3} \big|_0^2 = \frac{8}{3}$.

### Limits

**Purpose:** Evaluate the limit of an expression as a variable approaches a value.

**Syntax:**

```matlab
limit_result = limit(expr, variable, point);
```

### Simple Example

```matlab
syms x
expr = sin(x) / x;
limit_result = limit(expr, x, 0);
disp(limit_result)
```

**Output:**
```
1
```

This is a famous limit: $\lim_{x \to 0} \frac{\sin x}{x} = 1$.

### Example Walkthrough: Reaction Rate

Consider a first-order reaction: $C(t) = C_0 e^{-kt}$

Find the rate of change of concentration:

```matlab
syms t C0 k

% Concentration as a function of time
C = C0 * exp(-k*t);

% Rate of change (first derivative with respect to time)
rate = diff(C, t);
disp(rate)
```

**Output:**
```
-C0*k*exp(-k*t)
```

The rate is $-C_0 k e^{-kt}$, which is proportional to the concentration (first-order kinetics).

### Chemical Engineering Example: Enzyme Kinetics

The Michaelis-Menten equation describes enzyme kinetics:

$$v = \frac{V_{\max} [S]}{K_m + [S]}$$

Find $\frac{dv}{d[S]}$ (how velocity changes with substrate concentration):

```matlab
syms S Vmax Km

% Michaelis-Menten velocity
v = Vmax * S / (Km + S);

% Derivative with respect to substrate concentration
dvdS = diff(v, S);
disp(dvdS)

% Simplify
dvdS_simplified = simplify(dvdS);
disp(dvdS_simplified)
```

**Output:**
```
Vmax/(Km + S) - (S*Vmax)/(Km + S)^2
 
(Km*Vmax)/(Km + S)^2
```

This shows that the rate of change is highest at low substrate concentrations and decreases as substrate increases.

### Another Example: Enthalpy Calculation

Enthalpy as a function of temperature (constant pressure):

$$H(T) = H_0 + \int_{T_0}^{T} C_p \, dT$$

If $C_p = a + bT + cT^2$, find $H(T)$:

```matlab
syms T T0 H0 a b c

% Heat capacity
Cp = a + b*T + c*T^2;

% Enthalpy is H0 plus the integral of Cp from T0 to T
dH = int(Cp, T, T0, T);
H = H0 + dH;
disp(H)
```

**Output:**
```
(c*T^3)/3 + (b*T^2)/2 + a*T + H0 - T0*a - (T0^2*b)/2 - (T0^3*c)/3
```

### Syntax Table: Calculus Operations

| Operation | MATLAB Syntax | Example |
|---|---|---|
| First derivative | `diff(expr, var)` | `diff(x^3 + 2*x, x)` |
| Higher derivatives | `diff(expr, var, n)` | `diff(x^4, x, 2)` |
| Partial derivative | `diff(expr, var)` | `diff(x*y + y^2, x)` |
| Indefinite integral | `int(expr, var)` | `int(2*x, x)` |
| Definite integral | `int(expr, var, a, b)` | `int(x^2, x, 0, 2)` |
| Limit | `limit(expr, var, point)` | `limit(sin(x)/x, x, 0)` |

### Common Mistakes

1. **Differentiating a constant:**
   ```matlab
   syms x
   expr = 5;
   deriv = diff(expr, x);
   disp(deriv)  % Output: 0 (correct)
   ```

2. **Forgetting to specify the variable:**
   ```matlab
   syms x y
   expr = x^2 + y^2;
   deriv = diff(expr);  % MATLAB may not know which variable to use
   
   % Correct:
   deriv = diff(expr, y);  % Specify the variable
   ```

3. **Confusing `int()` with the built-in integer data type:**
   ```matlab
   % This will work in symbolic context:
   syms x
   result = int(x^2, x);  % Integration
   
   % But this is a data type:
   n = int32(5);  % Creates a 32-bit integer
   ```
## Solving Differential Equations

### First-Order ODE

```matlab
syms C_A(t) k

% Define ODE: dC_A/dt = -k*C_A
dC_A_dt = diff(C_A, t);
ode = dC_A_dt == -k * C_A;

% Solve with initial condition
C_A0 = 1;
C_A_solution = dsolve(ode, C_A(0) == C_A0);
disp(C_A_solution)  % exp(-k*t)
```

### Second-Order ODE

```matlab
syms x(t) t

% Simple harmonic oscillator: d²x/dt² = -x
ode = diff(x, t, 2) == -x;
x_solution = dsolve(ode, x(0) == 1, subs(diff(x, t), t, 0) == 0);
disp(x_solution)    % cos(t)
```

### System of ODEs

```matlab
syms y1(t) y2(t) t

% dy1/dt = y2
% dy2/dt = -y1

eqs = [diff(y1, t) == y2, diff(y2, t) == -y1];
initial = [y1(0) == 1, y2(0) == 0];

[y1_sol, y2_sol] = dsolve(eqs, initial);
disp(y1_sol)        % cos(t)
disp(y2_sol)        % -sin(t)
```

## Analytical vs. Numerical Solutions

### When to Use Symbolic Mathematics

Symbolic mathematics is powerful, but not always necessary. Here's when to use each approach:

| Aspect | Symbolic | Numerical |
|---|---|---|
| Solution complexity | Simple equations | Complex equations |
| Solution type | Exact formula | Approximate number |
| Insight | Shows relationships | Shows behavior |
| Speed | May be slow for large systems | Fast |
| Exactness | Mathematically exact | Bounded error |

**Example: Quadratic Equation**

$ax^2 + bx + c = 0$

Symbolic solution: $x = \frac{-b \pm \sqrt{b^2 - 4ac}}{2a}$ (exact formula)

Numerical solution: `x = [-2.5, 0.5]` (specific numbers, faster)

**Example: Transcendental Equation**

$e^x = 3x$

Symbolic: No closed-form solution exists (requires `fsolve` or similar)

Numerical: `x ≈ 1.5121` (found by iteration)

---

## Worked Example: Ideal Gas Law and Non-Ideal Behavior

### Problem

Compare the behavior of an ideal gas and a van der Waals gas.

**Given:**
- Temperature: $T = 350$ K
- Pressure: $P = 10$ bar
- Gas constant: $R = 0.08314$ bar·L/(mol·K)
- For CO₂: $a = 3.658$ bar·L²/mol², $b = 0.04267$ L/mol

**Tasks:**
1. Calculate the molar volume of an ideal gas
2. Calculate the molar volume of CO₂ using the van der Waals equation
3. Compare the results

### Solution Approach

**Step 1: Ideal Gas**

For an ideal gas: $PV_m = RT$, so $V_m = \frac{RT}{P}$

```matlab
syms Vm_ideal

% Ideal gas equation: P*Vm = R*T
R = 0.08314;  % bar·L/(mol·K)
T = 350;      % K
P = 10;       % bar

% Solve for ideal gas molar volume
Vm_ideal = R * T / P;
disp(['Ideal gas molar volume: ', num2str(Vm_ideal, 6), ' L/mol']);
```

**Step 2: van der Waals Gas**

For a van der Waals gas: $\left(P + \frac{a}{V_m^2}\right)(V_m - b) = RT$

```matlab
syms Vm

% van der Waals parameters for CO2
a = 3.658;    % bar·L²/mol²
b = 0.04267;  % L/mol

% van der Waals equation
vdw_eq = (P + a/Vm^2) * (Vm - b) == R * T;

% Solve for Vm
Vm_solutions = solve(vdw_eq, Vm);
disp(['van der Waals solutions:']);
disp(Vm_solutions);

% Convert to numerical values
Vm_numerical = double(Vm_solutions);
disp(['Numerical solutions: ', num2str(Vm_numerical)]);

% Physically meaningful solution (largest positive real value)
Vm_vdw = Vm_numerical(Vm_numerical > 0 & abs(imag(Vm_numerical)) < 1e-6);
Vm_vdw = max(real(Vm_vdw));
disp(['van der Waals molar volume: ', num2str(Vm_vdw, 6), ' L/mol']);
```

**Step 3: Compare**

```matlab
% Comparison
error_percent = 100 * (Vm_ideal - Vm_vdw) / Vm_vdw;
disp(['Percent difference: ', num2str(error_percent, 4), '%']);

% Interpret
disp(' ');
disp('Interpretation:');
disp('The ideal gas law overestimates the molar volume.');
disp('This is because van der Waals forces (attractive interactions)');
disp('reduce the volume compared to an ideal gas.');
```

### Complete Script

```matlab
% Ideal vs van der Waals: CO2 at 350 K, 10 bar
clear; close all

syms Vm

% Parameters
R = 0.08314;  % bar·L/(mol·K)
T = 350;      % K
P = 10;       % bar
a = 3.658;    % bar·L²/mol² (for CO2)
b = 0.04267;  % L/mol (for CO2)

% ======== Ideal Gas ========
Vm_ideal = R * T / P;
fprintf('\n=== IDEAL GAS ===\n');
fprintf('Molar volume: %.4f L/mol\n', Vm_ideal);

% ======== van der Waals ========
fprintf('\n=== VAN DER WAALS GAS ===\n');

% Set up and solve van der Waals equation
vdw_eq = (P + a/Vm^2) * (Vm - b) == R * T;
Vm_solutions = solve(vdw_eq, Vm);

% Find physically meaningful solution
Vm_numerical = double(Vm_solutions);
Vm_real_positive = Vm_numerical(Vm_numerical > 0 & abs(imag(Vm_numerical)) < 1e-10);
Vm_vdw = max(real(Vm_real_positive));

fprintf('van der Waals molar volume: %.4f L/mol\n', Vm_vdw);

% ======== Comparison ========
fprintf('\n=== COMPARISON ===\n');
error_percent = 100 * (Vm_ideal - Vm_vdw) / Vm_vdw;
fprintf('Ideal gas Vm: %.4f L/mol\n', Vm_ideal);
fprintf('vdW gas Vm:   %.4f L/mol\n', Vm_vdw);
fprintf('Difference:   %.2f%%\n', error_percent);
fprintf('\nThe ideal gas law overestimates the molar volume by %.2f%%.\n', error_percent);
fprintf('This is due to repulsive forces between molecules.\n');
```

**Output:**
```
=== IDEAL GAS ===
Molar volume: 2.9099 L/mol

=== VAN DER WAALS GAS ===
van der Waals molar volume: 2.8250 L/mol

=== COMPARISON ===
Ideal gas Vm: 2.9099 L/mol
vdW gas Vm:   2.8250 L/mol
Difference:   3.00%

The ideal gas law overestimates the molar volume by 3.00%.
This is due to repulsive forces between molecules.
```


---

### Important MATLAB Syntax Reference

| Purpose | MATLAB Syntax | Notes |
|---|---|---|
| Declare symbolic variables | `syms x y z` | Creates multiple variables at once |
| Create symbolic variable | `x = sym('x')` | Creates a single variable |
| Simplify expression | `simplify(expr)` | Applies algebraic rules |
| Expand expression | `expand(expr)` | Multiplies out brackets |
| Factor polynomial | `factor(expr)` | Factors into irreducibles |
| Substitute values | `subs(expr, var, val)` | Replaces symbol with number |
| Define equation | `eq = expr1 == expr2` | Uses `==` for equations |
| Solve equation | `solve(eq, var)` | Solves for specified variable |
| Solve system | `solve([eq1, eq2], [x, y])` | Solves multiple equations |
| First derivative | `diff(expr, x)` | With respect to x |
| Higher derivative | `diff(expr, x, n)` | nth derivative |
| Indefinite integral | `int(expr, x)` | Antiderivative |
| Definite integral | `int(expr, x, a, b)` | From a to b |
| Limit | `limit(expr, x, point)` | As x approaches point |
| Convert to number | `double(result)` | Converts symbolic to numeric |

---

## Homework

### Problem

**Decomposition of Nitrogen Pentoxide (N₂O₅)**

Nitrogen pentoxide decomposes according to the first-order reaction:

$$2 N_2O_5 \to 4 NO_2 + O_2$$

The rate law is: $-\frac{d[N_2O_5]}{dt} = k[N_2O_5]$

where $k = 3.46 \times 10^{-5}$ s⁻¹ at 45°C.

**Given:**
- Initial concentration: $[N_2O_5]_0 = 0.50$ M
- Time: $t = 300$ s

**Tasks:**

1. Use symbolic mathematics to derive the integrated rate law for a first-order reaction (analytical solution).
2. Verify that the solution satisfies the differential equation (by differentiation).
3. Calculate the concentration of N₂O₅ at $t = 300$ s using the analytical solution.
4. Calculate the rate of decomposition at $t = 300$ s.
5. Explain physically what your results mean.

### Required Tasks

- Create symbolic variables for concentration, time, and rate constant
- Solve the differential equation symbolically
- Differentiate the solution to verify it satisfies the original equation
- Substitute numerical values and report results with units
- Provide physical interpretation

### Concepts Being Tested

- Creating and solving differential equations symbolically
- Verification by differentiation
- Substitution of numerical values
- Understanding first-order kinetics
- Engineering interpretation

### Hints

- The differential equation is $\frac{d[N_2O_5]}{dt} = -k[N_2O_5]$
- You can use `dsolve()` to solve differential equations symbolically
- Remember that `D[C]` or `diff(C, t)` represents the derivative
- Check your answer by differentiating it and verifying it equals $-k[N_2O_5]$

<details>
<summary><strong>Detailed Solution — Click to Reveal</strong></summary>

## Solution

### Concept

This problem involves solving a **first-order linear differential equation** symbolically. The differential equation governs the concentration of a reactant over time. We must:

1. Derive the integrated rate law (closed-form solution)
2. Verify the solution by differentiation
3. Apply numerical values
4. Interpret the physical meaning

### Approach / Idea

**Strategy:**

1. Create a symbolic differential equation: $\frac{d[N_2O_5]}{dt} = -k[N_2O_5]$
2. Use `dsolve()` to obtain the general solution
3. Apply the initial condition $[N_2O_5](0) = [N_2O_5]_0$ to find the particular solution
4. Differentiate the solution to verify it satisfies the original equation
5. Substitute numerical values to get the answer

### Syntax

```matlab
syms C(t) k C0 t_val

% Define the differential equation
deq = diff(C, t) == -k * C;

% Solve the differential equation with initial condition
C_solution = dsolve(deq, C(0) == C0);

% Substitute numerical values and evaluate
```

### Syntax Breakdown

- `syms C(t) k C0 t_val` — Declares symbolic variables; `C(t)` is a function of `t`
- `diff(C, t)` — Represents $\frac{dC}{dt}$
- `deq = diff(C, t) == -k * C` — Defines the differential equation
- `dsolve(deq, C(0) == C0)` — Solves the ODE with initial condition
- `subs(C_solution, {k, C0}, {k_val, C0_val})` — Substitutes numerical values

### MATLAB Code

```matlab
% First-Order Decomposition of N2O5
clear; close all

syms C(t) k C0

% ======== Symbolic Solution ========
fprintf('\n=== SYMBOLIC SOLUTION ===\n');

% Differential equation: dC/dt = -k*C
deq = diff(C, t) == -k * C;

% Solve with initial condition C(0) = C0
C_solution = dsolve(deq, C(0) == C0);
fprintf('General solution:\n');
disp(C_solution);
fprintf('Or in standard form: [N2O5] = [N2O5]_0 * exp(-k*t)\n');

% ======== Verification by Differentiation ========
fprintf('\n=== VERIFICATION ===\n');

% Differentiate the solution
dC_dt = diff(C_solution, t);
fprintf('Derivative of the solution:\n');
disp(simplify(dC_dt));

% Check that it equals -k*C_solution
rhs = -k * C_solution;
fprintf('Right-hand side (-k*C):\n');
disp(rhs);

% Verify they are equal
verification = simplify(dC_dt - rhs);
fprintf('Difference (should be 0): ');
disp(verification);
fprintf('Solution verified!\n');

% ======== Numerical Calculation ========
fprintf('\n=== NUMERICAL RESULTS ===\n');

% Parameters
k_val = 3.46e-5;        % s⁻¹
C0_val = 0.50;          % M
t_val = 300;            % s

% Concentration at t = 300 s
C_at_300 = subs(C_solution, {k, C0, t}, {k_val, C0_val, t_val});
C_at_300_numeric = double(C_at_300);

fprintf('Rate constant k = %.2e s⁻¹\n', k_val);
fprintf('Initial concentration [N2O5]_0 = %.2f M\n', C0_val);
fprintf('Time t = %.0f s\n\n', t_val);
fprintf('Concentration at t = 300 s: [N2O5] = %.5f M\n', C_at_300_numeric);

% Calculate the amount decomposed
C_decomposed = C0_val - C_at_300_numeric;
percent_decomposed = 100 * C_decomposed / C0_val;
fprintf('Amount decomposed: %.5f M\n', C_decomposed);
fprintf('Percent decomposed: %.2f %%\n\n', percent_decomposed);

% ======== Rate of Decomposition ========
fprintf('=== RATE AT t = 300 s ===\n');

% Rate = -dC/dt = k*C
rate_at_300 = k_val * C_at_300_numeric;
fprintf('Rate of decomposition: ');
fprintf('-d[N2O5]/dt = k*[N2O5] = %.2e M/s\n', rate_at_300);
fprintf('Or: %.4f M/min\n', rate_at_300 * 60);

% ======== Physical Interpretation ========
fprintf('\n=== PHYSICAL INTERPRETATION ===\n');
fprintf('\nThe concentration decreases exponentially over time.\n');
fprintf('The integrated rate law [N2O5] = [N2O5]_0 * exp(-k*t) shows:\n\n');
fprintf('At t=0:      [N2O5] = %.2f M (initial)\n', C0_val);
fprintf('At t=300 s:  [N2O5] = %.5f M\n', C_at_300_numeric);
fprintf('Decay: exponential with time constant τ = 1/k = %.0f s\n', 1/k_val);
fprintf('\n');
fprintf('The half-life (time for concentration to drop to 50%%) is:\n');
t_half = log(2) / k_val;
fprintf('t_1/2 = ln(2)/k = %.0f s = %.2f hours\n', t_half, t_half/3600);
fprintf('\n');
fprintf('After 300 s, about %.1f%% of the N2O5 has decomposed.\n', percent_decomposed);
fprintf('The rate of decomposition is %.2e M/s at t=300 s,\n', rate_at_300);
fprintf('which is slower than at t=0 because less N2O5 remains.\n');

% ======== Plot ========
t_plot = linspace(0, 1000, 200);
C_plot = C0_val * exp(-k_val * t_plot);

figure('Position', [100, 100, 700, 500]);
plot(t_plot, C_plot, 'b-', 'LineWidth', 2);
hold on;
plot(300, C_at_300_numeric, 'ro', 'MarkerSize', 8, 'MarkerFaceColor', 'r');
xlabel('Time (s)', 'FontSize', 12);
ylabel('[N_{2}O_{5}] (M)', 'FontSize', 12);
title('First-Order Decomposition of N_{2}O_{5}', 'FontSize', 14, 'FontWeight', 'bold');
grid on;
legend('C(t) = C_0 exp(-kt)', sprintf('At t=300 s: [N_2O_5] = %.4f M', C_at_300_numeric), ...
       'Location', 'best', 'FontSize', 11);
xlim([0 1000]);
ylim([0 0.55]);
```

### Expected Result

```

=== SYMBOLIC SOLUTION ===
General solution:
C0*exp(-k*t)
 
Or in standard form: [N2O5] = [N2O5]_0 * exp(-k*t)

=== VERIFICATION ===
Derivative of the solution:
-C0*k*exp(-k*t)
 
Right-hand side (-k*C):
-C0*k*exp(-k*t)
 
Difference (should be 0): 0
 
Solution verified!

=== NUMERICAL RESULTS ===
Rate constant k = 3.46e-05 s⁻¹
Initial concentration [N2O5]_0 = 0.50 M
Time t = 300 s

Concentration at t = 300 s: [N2O5] = 0.49484 M
Amount decomposed: 0.00516 M
Percent decomposed: 1.03 %

=== RATE AT t = 300 s ===
Rate of decomposition: -d[N2O5]/dt = k*[N2O5] = 1.71e-05 M/s
Or: 0.0010 M/min

=== PHYSICAL INTERPRETATION ===

The concentration decreases exponentially over time.
The integrated rate law [N2O5] = [N2O5]_0 * exp(-k*t) shows:

At t=0:      [N2O5] = 0.50 M (initial)
At t=300 s:  [N2O5] = 0.49484 M
Decay: exponential with time constant τ = 1/k = 28902 s

The half-life (time for concentration to drop to 50%) is:
t_1/2 = ln(2)/k = 20033 s = 5.56 hours

After 300 s, about 1.0% of the N2O5 has decomposed.
The rate of decomposition is 1.71e-05 M/s at t=300 s,
which is slower than at t=0 because less N2O5 remains.
```

### Engineering Interpretation

**Key Findings:**

1. **Analytical Solution:** The symbolic mathematics gives us the exact integrated rate law:
   $$[N_2O_5] = [N_2O_5]_0 e^{-kt}$$
   This is the foundation for all first-order kinetics calculations.

2. **Slow Decomposition:** With $k = 3.46 \times 10^{-5}$ s⁻¹, the decomposition is very slow. After 300 s, only about 1% has reacted. The half-life is about 5.56 hours.

3. **Exponential Decay:** The concentration follows an exponential decay curve. This is characteristic of **first-order reactions**—the rate depends on the concentration of the reactant.

4. **Rate Dependence:** Even though the rate constant is constant, the decomposition rate decreases over time because the concentration decreases. At $t = 300$ s, the rate is $1.71 \times 10^{-5}$ M/s, compared to the initial rate of $k \times C_0 = 1.73 \times 10^{-5}$ M/s.

5. **Engineering Use:** In a real process, you would use this equation to predict how long to keep reactants at a given temperature, or how to optimize conditions for maximum conversion in a desired timeframe.

**Why Symbolic Mathematics is Valuable Here:**

- We obtained the **exact solution** without numerical approximation
- We verified correctness by showing the solution satisfies the differential equation
- The closed-form solution reveals the **exponential nature** of the decay
- We can easily compute any quantity (concentration, rate, half-life) at any time
- The symbolic form is reusable for different parameter values

</details>

---

