# Lecture 12: Numerical Differentiation and Integration

## Learning Objectives

By the end of this lecture, you should be able to:

- Explain the difference between numerical and symbolic differentiation
- Apply forward, backward, and central difference formulas for numerical differentiation
- Implement finite-difference approximations in MATLAB
- Distinguish between different numerical integration methods
- Use `trapz()` for trapezoidal integration
- Use `integral()` for adaptive numerical integration
- Integrate experimental/tabulated data
- Understand how step size and discretization affect accuracy
- Apply these methods to chemical engineering problems (rate calculations, material balances, energy balances)

---

## Why This Topic Matters

Many chemical engineering calculations involve derivatives and integrals:

- **Reaction rates** are derivatives of concentration with respect to time
- **Energy balances** often require integration over time or space
- **Transport phenomena** fundamentally involve differentiation (heat flux, mass flux)
- **Process control** relies on rate-of-change calculations
- **Experimental data** is often discrete and noisy, requiring numerical approximations

In practice, you will rarely have perfect analytical expressions. Instead, you will work with:

- Tabulated experimental data
- Results from numerical simulations
- Discretized process variables
- Approximate or complex functions that cannot be integrated analytically

Numerical differentiation and integration allow you to extract useful information and perform calculations with imperfect, real-world data.

---

## Numerical Differentiation

### Concept

**Numerical differentiation** approximates the derivative of a function at a point using discrete data points rather than the limit definition:

$$\frac{df}{dx} = \lim_{\Delta x \to 0} \frac{f(x + \Delta x) - f(x)}{\Delta x}$$

Since we cannot take the limit to zero, we use a small but finite step size $h$ and calculate:

$$\frac{df}{dx} \approx \frac{f(x + h) - f(x)}{h}$$

This is called a **finite-difference approximation**.

### Why Numerical Differentiation?

1. **Experimental data** — Often you have measurements at discrete points, not an analytical function
2. **Complex functions** — Some functions are difficult or impossible to differentiate analytically
3. **Noisy data** — Small step sizes amplify noise; finite differences provide a balance
4. **Computational efficiency** — Sometimes computing the derivative numerically is faster than using symbolic math

### Numerical vs. Symbolic Differentiation

| Aspect | Numerical | Symbolic |
|--------|-----------|----------|
| Requires | Function values at discrete points | Analytical function expression |
| Best for | Experimental data, complex/implicit functions | Analytical functions, exact results |
| Accuracy | Limited; improves with smaller step size | Exact (up to machine precision) |
| Speed | Fast for small datasets | Can be slow for complicated functions |
| Error analysis | Must account for truncation and round-off error | No truncation error |

## Finite-Difference Methods

There are three primary finite-difference approaches:

#### **Forward Difference**

Approximates the derivative using the current point and the next point:

$$f'(x) \approx \frac{f(x + h) - f(x)}{h}$$

**Pros:**
- Simple; only requires one forward step
- Useful at the beginning of a dataset

**Cons:**
- Lowest accuracy (first-order)
- Biased toward the forward side

#### **Backward Difference**

Approximates the derivative using the current point and the previous point:

$$f'(x) \approx \frac{f(x) - f(x - h)}{h}$$

**Pros:**
- Simple; useful at the end of a dataset
- Same accuracy as forward difference

**Cons:**
- First-order accuracy
- Biased toward the backward side

#### **Central Difference**

Approximates the derivative using points on both sides:

$$f'(x) \approx \frac{f(x + h) - f(x - h)}{2h}$$

**Pros:**
- Second-order accuracy (better than forward/backward)
- Symmetric, so unbiased
- Most accurate for regularly spaced data

**Cons:**
- Requires data on both sides (cannot be used at boundaries)

---
### MATLAB Syntax for Numerical Differentiation

MATLAB does not have a built-in function for general finite differences (unlike `diff()` in other languages).

However, you can use:

1. **`diff()` function** — Computes differences between consecutive elements
2. **Manual finite-difference formulas** — Implement forward, backward, or central differences explicitly

#### Using `diff()`

```matlab
dy = diff(y);           % First differences
dx = diff(x);           % Spacing differences
dydx = dy ./ dx;        % Approximate derivative
```

**Note:** `diff()` computes forward differences. The result has one fewer element than the input.

#### Manual Central Difference Implementation

```matlab
h = x(2) - x(1);        % Step size (assuming uniform spacing)
dydx_central = (y(3:end) - y(1:end-2)) / (2*h);
```

This computes the derivative at all interior points using central differences.

### Syntax Breakdown

```matlab
dy = diff(y);
```

- `diff(y)` — Computes differences: `y(2)-y(1)`, `y(3)-y(2)`, ..., `y(n)-y(n-1)`
- Result has length `length(y) - 1`

```matlab
dydx = dy ./ dx;
```

- `dy` — Vector of differences in y
- `dx` — Vector of differences in x (typically uniform)
- `./` — Element-wise division
- `dydx` — Approximate derivatives at intermediate points

### Simple Example: Temperature Change

Suppose a reactor is heated, and we measure temperature at regular time intervals:

```matlab
t = [0, 1, 2, 3, 4, 5];        % Time (minutes)
T = [20, 35, 48, 58, 65, 70];  % Temperature (°C)

% Compute the rate of temperature change (dT/dt)
dt = diff(t);                   % Time intervals
dT = diff(T);                   % Temperature changes
dT_dt = dT ./ dt;               % Rate of change (°C/min)

disp('Time (min):    ');
disp(t(1:end-1));
disp('Rate (°C/min): ');
disp(dT_dt);

figure
plot(t,T)
grid on
xlabel('Time (min)')
ylabel('Temperature (°C)')
title('Temperature vs. Time')
```

**Output:**
```
Time (min):    0    1    2    3    4
Rate (°C/min): 15   13   10    7    5
```

This shows the heating rate decreasing over time.

### Example Walkthrough

Let's break down the calculation:

1. `dt = diff(t)` gives `[1, 1, 1, 1, 1]` — the time intervals
2. `dT = diff(T)` gives `[15, 13, 10, 7, 5]` — the temperature changes
3. `dT_dt = dT ./ dt` gives `[15, 13, 10, 7, 5]` — the rates

Each value in `dT_dt` represents the approximate derivative at the midpoint of each interval. For example:
- Between t=0 and t=1: rate ≈ 15 °C/min
- Between t=1 and t=2: rate ≈ 13 °C/min

---

### Chemical Engineering Example: Reaction Rate from Concentration Data

In a batch reactor, we measure the concentration of reactant A at different times. We want to estimate the reaction rate:

```matlab
% Experimental data: Concentration of A vs. time
time = [0, 5, 10, 15, 20, 25, 30];        % s
C_A = [1.0, 0.85, 0.73, 0.62, 0.54, 0.47, 0.41];  % mol/L

% Forward difference (simple)
dt_fwd = diff(time);
dC_fwd = diff(C_A);
rate_fwd = -dC_fwd ./ dt_fwd;  % Negative because A is consumed

% Central difference (more accurate)
% We can only use this for interior points
dt_cent = time(3:end) - time(1:end-2);
rate_cent = -(C_A(3:end) - C_A(1:end-2)) ./ dt_cent;

% Display results
fprintf('Forward Difference Reaction Rates:\n');
for i = 1:length(rate_fwd)
    fprintf('  At t = %d s: -dC_A/dt ≈ %.4f mol/(L·s)\n', time(i), rate_fwd(i));
end

fprintf('\nCentral Difference Reaction Rates (interior points only):\n');
for i = 1:length(rate_cent)
    time_mid = time(i+1);
    fprintf('  At t = %d s: -dC_A/dt ≈ %.4f mol/(L·s)\n', time_mid, rate_cent(i));
end
```

**Key observations:**
- Reaction rate is positive (concentration decreasing)
- Rate decreases with time (typical for first-order reactions at high conversion)
- Central differences can only be computed at interior points

## gradient() for General Derivatives

MATLAB's `gradient` computes numerical gradient using central differences for interior points, forward/backward for endpoints, and handles non-uniform spacing.

The `gradient()` function handles edge cases automatically:

### Syntax Breakdown

```matlab
dF = gradient(F, X);
```

- `F` — function values (vector)
- `X` — corresponding x values
- Output: derivative at each point
- Uses one-sided differences at endpoints, central elsewhere

```matlab
% Compute gradient of a function
x = linspace(0, 2*pi, 100);
y = sin(x);
dy_dx = gradient(y, x);

% At boundaries: one-sided difference
% At interior: central difference

figure
plot(x, y, "b-", "LineWidth", 4, "DisplayName", "sin(x)")
hold on
plot(x, dy_dx, "r-", "LineWidth", 6, "DisplayName", "d/dx[sin(x)]")
plot(x, cos(x), "k--", "LineWidth", 3, "DisplayName", "cos(x) (exact)")

xlabel("x")
ylabel("Value")
legend()
grid on
```

### Second Derivatives

```matlab
% Compute second derivative (acceleration)
x = linspace(0, 10, 100);
y = sin(x);

% First derivative
dy = gradient(y, x);

% Second derivative
d2y = gradient(dy, x);

plot(x, y, "b-", "LineWidth", 1.5, "DisplayName", "y")
hold on
plot(x, dy, "g-", "LineWidth", 1.5, "DisplayName", "dy/dx")
plot(x, d2y, "r-", "LineWidth", 1.5, "DisplayName", "d²y/dx²")

legend()
grid on
```
**When to use `gradient` vs `diff`:**

- `diff(F)` — returns differences $F_{i+1}-F_i$, length $n-1$, not divided by $h$, not derivative. $diff(F)./diff(x)$ is forward difference, length $n-1$.
- `gradient(F,x)` — returns derivative same length as $F$, central for interior, more accurate, handles non-uniform $x$, preferred for $dC/dz$, $dT/dt$.

---

### Engineering Example: Reaction Rate from Concentration

```matlab
% Experimental concentration data over time
t = [0, 1, 2, 3, 4, 5];      % minutes
C = [2.0, 1.52, 1.18, 0.92, 0.71, 0.55];  % mol/L

% Reaction rate: -dC/dt
dCdt = -gradient(C, t);

fprintf("Reaction rate analysis:\n")
fprintf("Time   Conc    Rate\n")
fprintf("(min)  (M)     (M/min)\n")
for i = 1:length(t)
    fprintf("%.1f    %.2f    %.4f\n", t(i), C(i), dCdt(i))
end

% Plot
figure
tiledlayout(1,2)

nexttile
plot(t, C, "bo-", "MarkerSize", 8, "LineWidth", 2)
xlabel("Time (min)")
ylabel("Concentration (mol/L)")
title("Reactant Concentration")
grid on

nexttile
plot(t, dCdt, "r^-", "MarkerSize", 8, "LineWidth", 2)
xlabel("Time (min)")
ylabel("Reaction Rate (mol/L·min)")
title("Reaction Rate (Numerical Derivative)")
grid on
```

---
### Common Mistakes

1. **Forgetting the sign** — A decreasing quantity (like reactant concentration) has a negative derivative. Don't forget the negative sign when computing rates of consumption.

2. **Using the wrong step size** — Make sure `h` is consistent. If time is in seconds, use the actual time difference.

3. **Boundary issues** — Forward and backward differences give different results at boundaries. Central differences cannot be used at the first or last point.

4. **Confusing element-wise and matrix operations** — Use `./` for element-wise division, not `/` (which is matrix division).

5. **Loss of accuracy due to noise** — Numerical differentiation amplifies noise. If data is noisy, consider smoothing first or use a coarser step size.

6. **Assuming uniform spacing** — If data points are not evenly spaced, be careful when computing derivatives. The step size `h` varies.



## Numerical Integration

### Concept

**Numerical integration** approximates the definite integral of a function:

$$\int_a^b f(x) \, dx \approx \text{sum of area elements}$$

Instead of finding an antiderivative analytically, we approximate the area under the curve using geometric shapes (rectangles, trapezoids, etc.).

### Why Numerical Integration?

1. **No analytical antiderivative** — Many functions cannot be integrated in closed form
2. **Experimental data** — You have measurements, not an equation
3. **Complex domains** — Integration limits or function behavior may be complicated
4. **Efficiency** — Numerical methods are often faster than symbolic computation

### Numerical vs. Symbolic Integration

| Aspect | Numerical | Symbolic |
|--------|-----------|----------|
| Requires | Function values at discrete points (or a function handle) | Analytical function expression |
| Best for | Experimental/tabulated data, complex/non-elementary functions | Functions with known antiderivatives |
| Accuracy | Adjustable; improves with finer discretization | Exact result |
| Speed | Fast for large datasets | Variable; can be very slow for complex functions |
| Automatic | Adaptive methods adjust spacing automatically | Result is exact (no adjustment needed) |

---

### Trapezoidal Integration

The simplest and most common numerical integration method is **trapezoidal integration**.

**Idea:** Approximate the area under the curve as a series of trapezoids.

For data points $(x_1, y_1), (x_2, y_2), \ldots, (x_n, y_n)$, the area of each trapezoid is:

$$A_i = \frac{h_i (y_i + y_{i+1})}{2}$$

where $h_i = x_{i+1} - x_i$ is the width.

Total integral:

$$\int_a^b f(x) \, dx \approx \sum_{i=1}^{n-1} \frac{h_i (y_i + y_{i+1})}{2}$$

For **uniform spacing** ($h_i = h$ for all $i$):

$$\int_a^b f(x) \, dx \approx \frac{h}{2} \left( y_1 + 2y_2 + 2y_3 + \cdots + 2y_{n-1} + y_n \right)$$

---
### MATLAB Syntax for Numerical Integration

MATLAB provides two main functions:

#### `trapz()` — Trapezoidal Integration

```matlab
I = trapz(x, y);       % Integral of y with respect to x
I = trapz(y);          % If x is [1, 2, ..., n], default unit spacing
I = trapz(x, y, dim);  % Integrate along dimension dim (for matrices)
```
### Syntax Breakdown

```matlab
I = trapz(x, y);
```

- `x` — Vector of x-coordinates (independent variable)
- `y` — Vector of function values at each x
- `I` — Approximate value of the integral $\int_a^b y \, dx$

**Important:** `x` and `y` must have the same length.

---

#### `integral()` — Adaptive Numerical Integration

```matlab
I = integral(fun, a, b);
I = integral(fun, a, b, 'AbsTol', tol, 'RelTol', reltol);

% quad: older method
I_quad = quad(fun, -5, 5);
```

- `fun` — function handle
- `a, b` — integration limits
- `'AbsTol'` — absolute tolerance (default 1e-10)
- `'RelTol'` — relative tolerance
- `Inf` — supported as limit

`integral()` uses adaptive quadrature (automatically adjusts step size) and requires a function handle, not tabulated data.

```matlab
I = integral(@(x) sin(x), 0, pi);
```

- `@(x)` — Anonymous function syntax
- `sin(x)` — The integrand
- `0, pi` — Lower and upper limits
- `I` — Approximate value of $\int_0^\pi \sin(x) \, dx = 2$

---
### Simple Example: Computing Work Done by a Gas

In thermodynamics, work done by an expanding gas is:

$$W = \int P \, dV$$

Suppose we have experimental data from a process:

```matlab
% Experimental data: Pressure vs. Volume
V = [1, 1.5, 2, 2.5, 3, 3.5, 4];        % m³
P = [300, 210, 150, 120, 100, 90, 70];  % kPa

% Compute work using trapezoidal integration
W = trapz(V, P);  % Work = integral of P dV

fprintf('Work done by the gas: %.1f kJ\n', W);

figure
plot(P, V, LineWidth=2)
xlabel('Pressure (kPa)')
ylabel('Volume (m^3)')
grid on
```

**Calculation:**
The trapezoid rule integrates the pressure-volume curve:

$$W \approx \frac{0.5}{2}(300 + 210) + \frac{0.5}{2}(210 + 150) + \cdots + \frac{0.5}{2}(90 + 70)$$
$$W \approx 427.5 \text{ kJ}$$

### Example Walkthrough

Breaking down the trapezoid rule:

1. `V = [1, 1.5, 2, 2.5, 3, 3.5, 4]` — Volume data
2. `P = [300, 210, 150, 120, 100, 90, 70]` — Pressure data
3. Trapezoids between consecutive points:
   - Between V=1 and V=1.5: Area ≈ 0.5 × (300+210)/2 = 127.5
   - Between V=1.5 and V=2: Area ≈ 0.5 × (210+150)/2 = 90.0
   - ... and so on
4. `trapz(V, P)` sums all these areas automatically

---
### Chemical Engineering Example: Cumulative Moles from Flow Rate Data

In a continuous process, we measure the mass flow rate of product over time. To find the total moles produced, we integrate:

$$n = \int_0^T \left(\frac{\dot{m}}{M}\right) dt$$

where $\dot{m}$ is the mass flow rate and $M$ is the molar mass.

```matlab
% Data: Molar flow rate of product vs. time
time = [0, 10, 20, 30, 40, 50, 60];        % min
F_A = [0, 2.5, 4.0, 3.8, 3.2, 1.5, 0];    % mol/min (product formation rate)

% Total moles produced
n_total = trapz(time, F_A);  % Integrate rate over time

fprintf('Total moles of product: %.1f mol\n', n_total);

% We can also find cumulative moles at each time point using cumtrapz
n_cumulative = cumtrapz(time, F_A);
fprintf('\nCumulative moles vs. time:\n');
for i = 1:length(time)
    fprintf('  At t = %d min: n = %.2f mol\n', time(i), n_cumulative(i));
end
```

**Output:**
```
Total moles of product: 150.0 mol

Cumulative moles vs. time:
  At t = 0 min: n = 0.00 mol
  At t = 10 min: n = 12.50 mol
  At t = 20 min: n = 45.00 mol
  At t = 30 min: n = 84.00 mol
  At t = 40 min: n = 119.00 mol
  At t = 50 min: n = 142.50 mol
  At t = 60 min: n = 150.00 mol
```

**The `cumtrapz()` function returns cumulative integration at each point, which is useful for understanding how the process evolves over time.**

---
### Integrating Experimental Data with Varying Step Size

If your data is not uniformly spaced, `trapz()` still works correctly because it uses the actual spacing:

```matlab
% Non-uniform time intervals
time = [0, 1, 3, 6, 10, 15, 20];  % Seconds (irregular spacing)
flux = [100, 95, 80, 65, 50, 40, 35];  % mol/(m²·s)

% Trapezoidal integration handles non-uniform spacing automatically
total_flux = trapz(time, flux);  % Total moles per m²

fprintf('Total flux integrated over time: %.1f mol/m²\n', total_flux);
```

**Chemical engineering — PFR volume from discrete rate data:**

$V = F \int_{C_{out}}^{C_{in}} dC/(-r_A)$

Given experimental data $C_A$ vs $r_A$, compute $1/(-r_A)$ vs $C_A$ and integrate.

```matlab
% Example: first-order r_A = -k*C, but we have discrete data
C_A = linspace(0.2, 1.0, 10); % mol/L from outlet to inlet
k=0.3; r_A = -k*C_A; % mol/L/h
% PFR volume: V = F * integral_{C_out}^{C_in} dC/(-r_A)
F = 10; % L/h
integrand = 1./(-r_A); % h, since -r_A = k*C
integrand_sorted = 1./(k*C_sorted);
V = F * trapz(C_sorted, integrand_sorted);
fprintf('PFR volume via trapz: %.2f L, analytical F/k*ln(C_in/C_out)=%.2f L\n', V, F/k*log(1.0/0.2));

% Cumulative volume vs conversion
C_vec = linspace(0.2, 1.0, 100);
integrand_vec = 1./(k*C_vec);
V_cum = F * cumtrapz(C_vec, integrand_vec); % cumulative from C_out
plot(C_vec, V_cum, 'b-', 'LineWidth',2); xlabel('C_A (mol/L)'); ylabel('V (L)'); title('PFR Volume vs C_A'); grid on
```

---

### Simpson's Rule

More accurate than trapezoidal, fits parabola through 3 points: $\int_{x_0}^{x_2} f dx \approx (h/3)(f_0+4f_1+f_2)$ for uniform $h$, error $O(h^4)$, fourth-order.

Requires even number of intervals (odd number of points).

**Manual Simpson:**

```matlab
function I = simpson(x, y)
%SIMPSON Simpson's 1/3 rule, x uniform, y same length, length odd
    n = length(x);
    if mod(n,2)==0
        error('Simpson requires odd number of points (even intervals)');
    end
    h = x(2)-x(1); % assume uniform
    I = y(1) + y(end);
    for i=2:2:n-1
        I = I + 4*y(i);
    end
    for i=3:2:n-2
        I = I + 2*y(i);
    end
    I = I * h/3;
end
```

```matlab
% Test
x = linspace(0, pi, 11); % 11 points, 10 intervals even
y = sin(x);
I_simp = simpson(x, y)
I_trap = trapz(x, y)
fprintf('Simpson: %.8f, trapz: %.8f, exact 2\n', I_simp, I_trap);
```

Simpson more accurate than trapezoidal for smooth functions, but **requires uniform spacing and odd points.**

**MATLAB does not have built-in Simpson for vectors, but `integral` uses adaptive Simpson.**

---

### Advanced Integration with `integral()`

For functions (not discrete data), use adaptive quadrature: `integral` (recommended), `quad` (old), `integral2` for 2D, `integral3` for 3D.

If you have an analytical function (**not just data points**), use `integral()` for automatic, adaptive integration:
```matlab
% Syntax
I = integral(fun, a, b)
I = integral(fun, a, b, 'RelTol',1e-6, 'AbsTol',1e-10)

% 2D integral
I= integral2(fun,xmin,xmax,ymin,ymax)

% 3D integral
I3 = integral3(fun,xmin,xmax,ymin,ymax,zmin,zmax)
```

```matlab
% Example: integral sin from 0 to pi
fun = @(x) sin(x);
I = integral(fun, 0, pi)  % 2

% With parameters
k=0.3; fun = @(C) 1./(k*C);
I = integral(fun, 0.2, 1.0)  % same as PFR integral

% 2D integral
fun2 = @(x,y) x.*y;
I2 = integral2(fun2, 0,1, 0,1)  % int_0^1 int_0^1 x*y dx dy = 0.25

```

```matlab
% Example: Integrate an exponential decay (radioactive decay)
lambda = 0.1;  % Decay constant (1/s)
A_0 = 100;     % Initial amount

% Define the function
A = @(t) A_0 * exp(-lambda * t);

% Integrate from t=0 to t=50
total_decayed = integral(A, 0, 50);

fprintf('Total atoms decayed from t=0 to t=50: %.2f\n', total_decayed);

% Compare with analytical result
A_analytical = A_0 / lambda * (1 - exp(-lambda * 50));
fprintf('Analytical result: %.2f\n', A_analytical);
fprintf('Error: %.2e\n', abs(total_decayed - A_analytical));
```

### Common Mistakes

1. **Wrong order of arguments** — Always use `trapz(x, y)`, not `trapz(y, x)`. The first argument is the independent variable.

2. **Mismatched vector lengths** — `x` and `y` must have the same length. Otherwise, MATLAB throws an error.

3. **Ignoring units** — Make sure the units make sense. If y is in mol/L and x is in seconds, the integral has units of mol·s/L. Account for this in your interpretation.

4. **Using `integral()` with data** — `integral()` requires a function handle, not vectors. Use `trapz()` for tabulated data.

5. **Not understanding step size effects** — Finer discretization improves accuracy but increases computational cost. Use adaptive methods (`integral()`) when precision is critical.

6. **Forgetting about negative areas** — If the function dips below zero, those areas are counted as negative. This is correct but can be counterintuitive.



## Accuracy and Error

### Sources of Numerical Error

Two types of error affect numerical differentiation and integration:

#### Truncation Error

Arises from replacing the true derivative/integral with a finite-difference/finite-sum approximation.

- **Smaller step size** → **Lower truncation error**
- Forward/backward differences: $O(h)$ error (first-order)
- Central differences: $O(h^2)$ error (second-order)
- Trapezoidal rule: $O(h^2)$ error (second-order)

#### Round-Off Error

Arises from finite-precision arithmetic in computers.

- **Smaller step size** → **Higher round-off error** (because divisions by very small numbers amplify rounding)
- **Larger step size** → **Lower round-off error**

**The trade-off:** Too small a step size increases round-off error; too large a step size increases truncation error. The optimal step size balances both.

### Step Size Effects on Accuracy

**For differentiation:**

```matlab
% Function: f(x) = sin(x)
% Analytical: f'(x) = cos(x)
% At x = 1: f'(1) = cos(1) ≈ 0.5403

x_test = 1;
exact = cos(x_test);

% Try different step sizes
h_values = [0.1, 0.01, 0.001, 0.0001, 0.00001, 0.000001, 0.0000001, 0.00000001, 0.000000001, 0.00000000000001];

fprintf('Central Difference Error vs. Step Size:\n');
fprintf('Step Size | Approximation | Error\n');
fprintf('-------------+----------+-------\n');

for h = h_values
    approx = (sin(x_test + h) - sin(x_test - h)) / (2*h);
    error = abs(approx - exact);
    fprintf('%e | %f | %e\n', h, approx, error);
end
```

As $h$ decreases, the error decreases initially, then increases due to round-off.

**For integration:**

Finer discretization improves accuracy, but with diminishing returns:

```matlab
% Integrate sin(x) from 0 to pi (exact answer = 2)
tic
x = 0:0.1:pi;
I_coarse = trapz(x, sin(x));
toc

tic
x = 0:0.01:pi;
I_fine = trapz(x, sin(x));
toc

tic
x = 0:0.0001:pi;
I_finer = trapz(x, sin(x));
toc

tic
x = 0:1e-5:pi;
I_very_fine = trapz(x, sin(x));
toc

tic
x = 0:1e-8:pi;
I_super_fine = trapz(x, sin(x));
toc

tic
x = 0:0.5e-8:pi;
I_extremely_fine = trapz(x, sin(x));
toc

disp("============================================")
fprintf('Trapezoidal Integration Accuracy:\n');
fprintf('Step  | Integral | Error\n');
fprintf('0.1   | %f | %e\n', I_coarse, abs(I_coarse - 2));
fprintf('0.01  | %f | %e\n', I_fine, abs(I_fine - 2));
fprintf('0.0001| %f | %e\n', I_finer, abs(I_finer - 2));
fprintf('1e-5  | %f | %e\n', I_very_fine, abs(I_very_fine - 2));
fprintf('1e-8  | %f | %e\n', I_super_fine, abs(I_super_fine - 2));
fprintf('5e-9  | %f | %e\n', I_extremely_fine, abs(I_extremely_fine - 2));
```
Output:
```
Elapsed time is 0.006819 seconds.
Elapsed time is 0.001219 seconds.
Elapsed time is 0.001370 seconds.
Elapsed time is 0.006766 seconds.
Elapsed time is 4.300799 seconds.
Elapsed time is 13.054765 seconds.
============================================
Trapezoidal Integration Accuracy:
Step  | Integral | Error
0.1   | 1.997469 | 2.531073e-03
0.01  | 1.999982 | 1.793496e-05
0.0001| 2.000000 | 5.959010e-09
1e-5  | 2.000000 | 2.018785e-11
1e-8  | 2.000000 | 1.532108e-14
5e-9  | 2.000000 | 8.881784e-16
```
### Accuracy Considerations

1. **Validation** — Compare numerical results with known analytical solutions when possible
2. **Multiple methods** — Use both forward and central differences (or adaptive integration) to check consistency
3. **Convergence** — Refine your discretization and verify that results stabilize
4. **Physical plausibility** — Does the result make physical sense?



## Worked Example: Application to Reaction Rates

Batch reactor: $r_A = -dC_A/dt$. From discrete $C_A(t)$ data, compute rate.

```matlab
% Batch data: t (h), C_A (mol/L)
t = [0 0.5 1.0 1.5 2.0 2.5 3.0];
C_A = [1.0 0.8 0.64 0.51 0.41 0.33 0.26]; % first-order decay

% Compute dC/dt via gradient
dCdt = gradient(C_A, t);  % mol/L/h
r_A = -dCdt;  % rate positive for consumption? Actually r_A = dC/dt = -k*C, so -dC/dt = k*C

% Plot
figure;
yyaxis left; plot(t, C_A, 'b-o', 'LineWidth',2); ylabel('C_A (mol/L)')
yyaxis right; plot(t, r_A, 'r--s', 'LineWidth',2); ylabel('r_A = -dC/dt (mol/L/h)')
xlabel('Time (h)'); title('Batch: Concentration and Rate from Gradient'); grid on

% For first-order, r = k*C, so k = r/C
k_est = r_A./C_A;
fprintf('Estimated k = r/C: '); disp(k_est)
% Should be ~0.446? Actually for data C=exp(-0.446*t)? Let's check average k
k_mean = mean(k_est(2:end-1))  % exclude endpoints where gradient less accurate
```

**Noisy data issue:**

```matlab
% Add noise to C_A
C_noisy = C_A + 0.02*randn(size(C_A));
dCdt_noisy = gradient(C_noisy, t);
% dCdt_noisy will be very noisy, amplified

% Smooth first
C_smooth = smoothdata(C_noisy, 'movmean', 3);  % moving average 3 points
% Or
C_smooth2 = smoothdata(C_noisy, 'sgolay', 3);  % Savitzky-Golay filter preserves shape
dCdt_smooth = gradient(C_smooth2, t);

figure;
plot(t, C_A, 'k-', 'LineWidth',3, 'DisplayName','True'); hold on
plot(t, C_noisy, 'b-o', 'LineWidth',4, 'DisplayName','Noisy');
plot(t, C_smooth2, 'r--s', 'LineWidth',2, 'DisplayName','Smoothed'); hold off
legend; xlabel('Time'); ylabel('C_A'); title('Smoothing Before Differentiation'); grid on

figure;
plot(t, gradient(C_A,t), 'k-', 'LineWidth',3, 'DisplayName','True rate'); hold on
plot(t, dCdt_noisy, 'b-o', 'LineWidth',4, 'DisplayName','Noisy rate (amplified)');
plot(t, dCdt_smooth, 'r--s', 'LineWidth',2, 'DisplayName','Smoothed rate'); hold off
legend; xlabel('Time'); ylabel('dC/dt'); title('Differentiation Amplifies Noise'); grid on
```

**Lesson:** Differentiation amplifies high-frequency noise. Always smooth data before differentiating, or fit model then differentiate model.

## Worked Example — Batch Rate from Data and PFR Volume via Integration

Combine differentiation and integration for complete reactor analysis.

```matlab
% batch_pfr_diff_int.m

clear; clc;

% --- Batch data: t, C_A ---
t = [0 0.5 1.0 1.5 2.0 2.5 3.0 3.5 4.0]; % h
C_A_true = exp(-0.5*t); % true first-order k=0.5
% Add noise
rng(0);
C_A = C_A_true + 0.02*randn(size(t));
C_A(C_A<0)=0;

% --- Differentiation: rate r = -dC/dt ---
% Without smoothing
dCdt_raw = gradient(C_A, t);
r_raw = -dCdt_raw;

% With smoothing
C_smooth = smoothdata(C_A, 'sgolay', 3);
dCdt_smooth = gradient(C_smooth, t);
r_smooth = -dCdt_smooth;

% True rate
r_true = 0.5*exp(-0.5*t);

figure;
tiledlayout(1,2)

nexttile
plot(t, C_A_true, 'k-', 'LineWidth',2, 'DisplayName','True C_A'); hold on
plot(t, C_A, 'b-o', 'DisplayName','Noisy C_A');
plot(t, C_smooth, 'r--s', 'DisplayName','Smoothed'); hold off
xlabel('Time (h)'); ylabel('C_A (mol/L)'); title('Concentration Data'); 
legend; grid on

nexttile
plot(t, r_true, 'k-', 'LineWidth',2, 'DisplayName','True r=k*C'); hold on
plot(t, r_raw, 'b-o', 'DisplayName','Raw r=-dC/dt');
plot(t, r_smooth, 'r--s', 'DisplayName','Smoothed r'); hold off
xlabel('Time (h)'); ylabel('r_A (mol/L/h)'); title('Rate from Differentiation (Noise Amplified)'); 
legend; grid on

sgtitle('Batch: Differentiation Amplifies Noise - Smooth First')

% --- Integration: PFR volume from rate data ---
% Suppose we have rate vs concentration from batch: r vs C_A (same as above)
% For PFR: V = F * integral_{C_out}^{C_in} dC/(-r)

F = 10; % L/h
C_in = 1.0; C_out = 0.2;

% Create integrand 1/(-r) vs C from batch data (using true rate for accuracy)
C_data = C_A_true; % use true for demo
r_data = 0.5*C_data; % first-order
integrand_data = 1./r_data;

% Sort by C ascending for integration from C_out to C_in
[C_sorted, idx] = sort(C_data);
integrand_sorted = integrand_data(idx);

% Only include C between C_out and C_in
mask = (C_sorted>=C_out) & (C_sorted<=C_in);
C_int = C_sorted(mask);
integrand_int = integrand_sorted(mask);

V_trapz = F * trapz(C_int, integrand_int);
fprintf('PFR volume via trapz from discrete data: %.2f L\n', V_trapz);

% Analytical: V = F/k * ln(C_in/C_out) =10/0.5*ln(5)=20*1.609=32.18 L
V_anal = F/0.5*log(C_in/C_out);
fprintf('Analytical V = F/k*ln(C_in/C_out): %.2f L\n', V_anal);

% Using integral for function
fun = @(C) 1./(0.5*C);
V_integral = F * integral(fun, C_out, C_in);
fprintf('PFR volume via integral: %.2f L\n', V_integral);

% Cumulative volume vs conversion
C_vec = linspace(C_out, C_in, 100);
V_cum = F * cumtrapz(C_vec, 1./(0.5*C_vec));
X_vec = 1 - C_vec/C_in;

figure;
plot(X_vec, V_cum, 'b-', 'LineWidth',2); 
xlabel('Conversion X'); ylabel('V (L)'); title('PFR Volume vs Conversion'); 
grid on

% --- Energy integration ---
a=30; b=0.02; c=1e-5;
Cp_fun = @(T) a + b*T + c*T.^2;
T1=300; T2=400;
dH_integral = integral(Cp_fun, T1, T2);
dH_anal = a*(T2-T1)+b/2*(T2^2-T1^2)+c/3*(T2^3-T1^3);
fprintf('\nEnthalpy change via integral: %.2f J/mol, analytical: %.2f J/mol\n', dH_integral, dH_anal);
```

Shows noise amplification in differentiation and integration for PFR volume.

---

### Important MATLAB Syntax

| Purpose | MATLAB Syntax |
|---------|---------------|
| Compute first differences | `dy = diff(y);` |
| Forward difference derivative | `dydx = diff(y) ./ diff(x);` |
| Central difference derivative | `dydx = (y(3:end) - y(1:end-2)) / (2*h);` |
| Trapezoidal integration | `I = trapz(x, y);` |
| Cumulative trapezoidal integration | `I_cum = cumtrapz(x, y);` |
| Adaptive numerical integration | `I = integral(@(x) f(x), a, b);` |
| Integration with tolerance | `I = integral(@(x) f(x), a, b, 'AbsTol', 1e-6);` |

---

## Homework

### Problem

****Fermentation Rate and Yield Calculation****

A pharmaceutical company ferments a microorganism to produce an antibiotic. The following experimental data was collected from a 50-liter batch reactor:

| Time (h) | Biomass (g/L) | Substrate (g/L) | Product (g/L) |
|----------|---------------|-----------------|---------------|
| 0        | 0.5           | 100             | 0             |
| 2        | 1.2           | 92              | 3.5           |
| 4        | 2.8           | 78              | 8.2           |
| 6        | 5.5           | 58              | 14.1          |
| 8        | 9.2           | 35              | 19.8          |
| 10       | 13.5          | 12              | 23.4          |
| 12       | 14.8          | 2               | 24.2          |

### Required Tasks

1. **Calculate specific growth rate** — Use numerical differentiation to estimate $\mu = \frac{1}{X}\frac{dX}{dt}$ (specific growth rate) at each time point, where $X$ is biomass concentration.

2. **Calculate substrate consumption rate** — Use numerical differentiation to estimate $\frac{dS}{dt}$ (substrate consumption rate) at each time point, where $S$ is substrate concentration.

3. **Calculate volumetric product formation rate** — Use numerical differentiation to estimate $\frac{dP}{dt}$ (product formation rate) at each time point.

4. **Calculate cumulative substrate consumed** — Use numerical integration to find the total substrate consumed from t=0 to t=12 hours.

5. **Calculate overall yield** — Determine $Y_{P/S} = \frac{\Delta P}{\Delta S}$ (product yield from substrate).

### Concepts Being Tested

- Numerical differentiation using finite differences
- Selecting appropriate difference methods (forward/central/backward)
- Element-wise operations and vector indexing
- Numerical integration with `trapz()`
- Interpretation of chemical engineering rates

### Hints

- Use **central differences** for interior points and **forward/backward** at boundaries
- Remember to account for time units (hours) in your rate calculations
- Cumulative consumption = initial concentration minus current concentration, or integrate the rate
- Yield = (change in product) / (change in substrate consumed)


<details>
<summary><strong>Detailed Solution — Click to Reveal</strong></summary>

## Solution

### Step-by-step MATLAB code

```matlab
% -------------------------------------------------
% 1. Enter the experimental data
% -------------------------------------------------
time = [0  2  4  6  8  10  12];          % hours
X    = [0.5 1.2 2.8 5.5 9.2 13.5 14.8]; % biomass (g/L)
S    = [100 92  78  58  35  12   2  ];  % substrate (g/L)
P    = [0   3.5 8.2 14.1 19.8 23.4 24.2]; % product (g/L)

% -------------------------------------------------
% 2. Numerical differentiation with gradient
%    (spacing = 2 hours)
% -------------------------------------------------
dXdt = gradient(X, 2);   % dX/dt  (g/(L·h))
dSdt = gradient(S, 2);   % dS/dt  (g/(L·h))
dPdt = gradient(P, 2);   % dP/dt  (g/(L·h))

% Specific growth rate μ = (1/X) * (dX/dt)
mu = (1 ./ X) .* dXdt;   % (1/h)

% Substrate consumption rate (make it positive)
consumption_rate = -dSdt;   % g/(L·h)

% Product formation rate
formation_rate = dPdt;      % g/(L·h)

% -------------------------------------------------
% 3. Cumulative substrate consumed
% -------------------------------------------------
% Easy way – just look at the concentrations
S_consumed = S(1) - S(end);          % 100 - 2 = 98 g/L

% Alternative way – integrate the consumption rate
S_consumed_trapz = trapz(time, consumption_rate);

% -------------------------------------------------
% 4. Overall yield Y_P/S
% -------------------------------------------------
Y_PS = (P(end) - P(1)) / (S(1) - S(end));

% -------------------------------------------------
% 5. Display everything clearly
% -------------------------------------------------
fprintf('\n=== FERMENTATION ANALYSIS ===\n\n');

fprintf('Task 1 – Specific growth rate μ (h^{-1})\n');
fprintf('Time (h) | Biomass (g/L) |   μ (h^{-1})\n');
fprintf('---------|---------------|------------\n');
for i = 1:length(time)
    fprintf('%7.1f  | %13.2f | %10.4f\n', time(i), X(i), mu(i));
end

fprintf('\nTask 2 – Substrate consumption rate -dS/dt (g/(L·h))\n');
fprintf('Time (h) | -dS/dt\n');
fprintf('---------|--------\n');
for i = 1:length(time)
    fprintf('%7.1f  | %6.2f\n', time(i), consumption_rate(i));
end

fprintf('\nTask 3 – Product formation rate dP/dt (g/(L·h))\n');
fprintf('Time (h) | dP/dt\n');
fprintf('---------|-------\n');
for i = 1:length(time)
    fprintf('%7.1f  | %5.2f\n', time(i), formation_rate(i));
end

fprintf('\nTask 4 – Cumulative substrate consumed\n');
fprintf('From concentrations : %.2f g/L\n', S_consumed);
fprintf('From trapz          : %.2f g/L\n', S_consumed_trapz);

fprintf('\nTask 5 – Overall yield\n');
fprintf('Y_{P/S} = %.4f g product / g substrate\n', Y_PS);
```

### What the numbers mean 

| Quantity                        | Typical behaviour in this batch |
|---------------------------------|---------------------------------|
| Specific growth rate μ          | Starts high (~0.7 h⁻¹) and falls as cells enter stationary phase |
| Substrate consumption rate      | Rises, peaks around 8–10 h, then drops when substrate is almost gone |
| Product formation rate          | Highest in the middle of the fermentation, then slows down |
| Total substrate consumed        | 98 g/L |
| Yield Y<sub>P/S</sub>           | ≈ 0.247 g product per g substrate consumed |

</details>

---
