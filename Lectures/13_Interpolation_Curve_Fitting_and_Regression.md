# Lecture 13: Interpolation, Curve Fitting, and Basic Regression

## Learning Objectives

By the end of this lecture, you should be able to:

- Understand interpolation concepts and when to use them in engineering applications
- Perform linear interpolation using MATLAB's `interp1` function
- Distinguish between interpolation and extrapolation
- Fit polynomial models to experimental data using `polyfit` and `polyval`
- Evaluate model quality using residuals and the coefficient of determination ($R^2$)
- Compare fitted models with experimental data
- Apply curve fitting to typical chemical engineering problems such as vapor pressure, reaction kinetics, and property estimation


## Why This Topic Matters

In chemical engineering, we frequently encounter data that is incomplete or sampled at discrete points. Whether analyzing experimental measurements, working with correlations from literature, or building predictive models, we need methods to:

1. **Estimate values between known data points** — interpolation
2. **Find continuous relationships in discrete data** — curve fitting
3. **Assess how well our model describes the data** — statistical evaluation

For example, if you have vapor pressure data at a few temperatures, you need to estimate vapor pressure at an intermediate temperature. Or if you measure reaction conversion at different times, you need to fit a kinetic model to predict future conversion.


## Interpolation

### Concept

**Interpolation** is the process of estimating values at points that lie *within the range* of known data points. It assumes that intermediate values follow a smooth, predictable pattern based on the surrounding data.

**Key principle:** With interpolation, we estimate a value at a point between two (or more) known data points.

### Simple Interpolation Example

Suppose we have experimental data for vapor pressure of water at specific temperatures:

| Temperature (K) | Vapor Pressure (kPa) |
|---|---|
| 373.15 | 101.3 |
| 383.15 | 147.4 |
| 393.15 | 198.3 |

What is the vapor pressure at 380 K?

**Conceptually:** We assume the relationship is smooth between 373.15 K and 383.15 K, and we estimate the value at 380 K by interpolation.

---

### MATLAB Syntax: `interp1`

```matlab
y_interp = interp1(x, y, x_query, method);
```

**Syntax Breakdown:**

- `x` — vector of known x-values (must be in ascending order)
- `y` — vector of known y-values
- `x_query` — scalar or vector of query points where we want to estimate y
- `method` — interpolation method:
  - `'linear'` — piecewise linear (default)
  - `'cubic'` — piecewise cubic (smooth)
  - `'spline'` — cubic spline (smooth, extrapol atable)
  - `'pchip'` — monotonic cubic
- `y_interp` — interpolated y-values at `x_query`


**Default method:** Linear interpolation (straight line between points)

The most useful methods for engineering work are:

| Method     | Keyword     | Characteristics                              |
|------------|-------------|----------------------------------------------|
| Linear     | `'linear'`  | Simple, continuous, discontinuous derivative |
| Spline     | `'spline'`  | Cubic, continuous second derivative          |
| PCHIP      | `'pchip'`   | Shape-preserving, no overshoot               |
| Nearest    | `'nearest'` | Piecewise constant                           |

### Example: Linear Interpolation

```matlab
% Known vapor pressure data
T_known = [373.15, 383.15, 393.15];  % K
P_vap_known = [101.3, 147.4, 198.3]; % kPa

% Find vapor pressure at 380 K
T_query = 380;
P_vap_at_380 = interp1(T_known, P_vap_known, T_query);

fprintf('Vapor pressure at 380 K: %.2f kPa\n', P_vap_at_380);
```

**Output:**
```
Vapor pressure at 380 K: 132.88 kPa
```
---

### Interpolation with Multiple Query Points

You can also interpolate at multiple points in a single call:

```matlab
% Known data
x = [1, 2, 3, 4, 5];
y = [2, 5, 7, 8, 10];

% Query at multiple points
x_query = [1.5, 2.3, 3.7, 4.2];
y_interp = interp1(x, y, x_query);

plot(x, y, 'ko', 'MarkerSize', 8, 'DisplayName', 'Known data');
hold on
plot(x_query, y_interp, 'r--^', 'MarkerSize', 8, 'DisplayName', 'Interpolated');
xlabel('x');
ylabel('y');
legend;
grid on;
```

### Chemical Engineering Application: Viscosity Interpolation

Viscosity data for a fluid is often tabulated at specific temperatures. You need viscosity at an intermediate temperature:

```matlab
% Viscosity data for water at different temperatures (approximate)
T = [273.15, 298.15, 323.15, 348.15];      % K
mu = [1.787e-3, 0.891e-3, 0.547e-3, 0.365e-3];  % Pa·s

% Find viscosity at 310 K
T_target = 310;
mu_310 = interp1(T, mu, T_target);

fprintf('Viscosity at 310 K: %.4e Pa·s\n', mu_310);
```

### Engineering Example: Thermodynamic Data

```matlab
% Vapor pressure table (incomplete)
T = [50, 70, 90, 110];                 % °C
P_sat = [12.34, 31.19, 70.38, 143.27]; % kPa

% Find saturation pressure at 80°C
T_query = 80;
P_at_80 = interp1(T, P_sat, T_query, "spline");

fprintf("Saturation pressure at %.0f°C: %.2f kPa\n", T_query, P_at_80)

% Create smooth table for display
T_smooth = 50:5:110;
P_smooth = interp1(T, P_sat, T_smooth, "spline");

fprintf("Interpolated steam table:\n")
fprintf("T (°C)   P_sat (kPa)\n")
for i = 1:length(T_smooth)
    fprintf("%6.0f   %8.2f\n", T_smooth(i), P_smooth(i))
end
```

## Extrapolation: A Warning

**Extrapolation** means estimating outside the range of known data. `interp1` can do this, but it is **highly unreliable** and should be avoided.

```matlab
% Known data spans [373.15, 393.15]
T_known = [373.15, 383.15, 393.15];
P_vap_known = [101.3, 147.4, 198.3];

% Try to extrapolate beyond the range (NOT RECOMMENDED)
T_extrapolate = 410;  % Beyond 393.15!
P_vap_extrapolated = interp1(T_known, P_vap_known, T_extrapolate);

fprintf('Warning: Extrapolated pressure at 410 K: %.2f kPa\n', P_vap_extrapolated);
```

By default, `interp1` will return `NaN` (Not a Number) for points outside the range. To allow extrapolation, you can use:

```matlab
P_vap_extrapolated = interp1(T_known, P_vap_known, T_extrapolate, 'linear', 'extrap');
```

**But be careful!** Extrapolation often gives physically unrealistic results because it assumes the linear trend continues indefinitely.

### Example: Antoine equation: vapor pressure in different temperature ranges

```matlab
%% Antoine equation: vapor pressure in different temperature ranges
% log10(P) = A - B/(T + C),  P in bar, T in K
% Demonstration of the error of extrapolating the Bridgeman & Aldrich
% (304-333 K) coefficients up to 525 K, compared with Liu & Lindsay.
clear; clc; close all;

% Columns: Tmin  Tmax  A  B  C
data = [ ...
    379.0  573.0  3.55959   643.748  -198.043;   % Liu and Lindsay, 1970
    273.0  303.0  5.40221  1838.675   -31.737;   % Bridgeman and Aldrich, 1964
    304.0  333.0  5.20389  1733.926   -39.485;   % Bridgeman and Aldrich, 1964
    334.0  363.0  5.0768   1659.793   -45.854];  % Bridgeman and Aldrich, 1964

labels = { ...
    'Liu & Lindsay 1970 (379-573 K)', ...
    'Bridgeman & Aldrich 1964 (273-303 K)', ...
    'Bridgeman & Aldrich 1964 (304-333 K)', ...
    'Bridgeman & Aldrich 1964 (334-363 K)'};

Pvap = @(T,A,B,C) 10.^(A - B./(T + C));   % bar

n      = size(data,1);
colors = lines(n);

figure('Color','w','Position',[100 100 950 650]); hold on; grid on; box on;

% --- Valid-range curves (solid) ---
for i = 1:n
    T = linspace(data(i,1), data(i,2), 200);
    P = Pvap(T, data(i,3), data(i,4), data(i,5));
    plot(T, P, '-', 'LineWidth', 2, 'Color', colors(i,:), ...
         'DisplayName', labels{i});
end

%% --- Extrapolation of Bridgeman & Aldrich (304-333 K) ---
iB = 3;                                   % row of the 304-333 K set
A_b = data(iB,3); B_b = data(iB,4); C_b = data(iB,5);
iL = 1;                                   % row of Liu & Lindsay
A_l = data(iL,3); B_l = data(iL,4); C_l = data(iL,5);

T_target = 525;                           % K

T_ext = linspace(data(iB,2), 573, 300);   % from end of valid range to 573 K
P_ext = Pvap(T_ext, A_b, B_b, C_b);
plot(T_ext, P_ext, '--', 'LineWidth', 2, 'Color', colors(iB,:), ...
     'DisplayName', 'Bridgeman 304-333 K, EXTRAPOLATED');

P_extrap = Pvap(T_target, A_b, B_b, C_b);   % extrapolated estimate
P_exact  = Pvap(T_target, A_l, B_l, C_l);   % reference (valid range)

abs_err = P_extrap - P_exact;
rel_err = abs_err / P_exact * 100;

plot(T_target, P_extrap, 'o', 'MarkerSize', 9, 'LineWidth', 1.5, ...
     'MarkerFaceColor', colors(iB,:), 'MarkerEdgeColor', 'k', ...
     'DisplayName', sprintf('Extrapolated @ %g K: %.2f bar', T_target, P_extrap));
plot(T_target, P_exact, 's', 'MarkerSize', 9, 'LineWidth', 1.5, ...
     'MarkerFaceColor', colors(iL,:), 'MarkerEdgeColor', 'k', ...
     'DisplayName', sprintf('Liu & Lindsay @ %g K: %.2f bar', T_target, P_exact));
plot([T_target T_target], [P_exact P_extrap], 'k-', 'LineWidth', 1.2, ...
     'HandleVisibility', 'off');

text(T_target-8, sqrt(P_exact*P_extrap), ...
     sprintf('Error = %.2f bar (%.1f %%)', abs_err, rel_err), ...
     'HorizontalAlignment', 'right', 'FontSize', 10, 'FontWeight', 'bold');

set(gca, 'YScale', 'log', 'FontSize', 11);
xlabel('Temperature (K)');
ylabel('Vapor pressure (bar)');
title('Antoine equation ranges and the effect of extrapolation');
legend('Location','southeast','FontSize',9);
xlim([265 580]);

%% --- report ---
fprintf('Extrapolation test at T = %.1f K\n', T_target);
fprintf('  Bridgeman (304-333 K) extrapolated : %.4f bar\n', P_extrap);
fprintf('  Liu & Lindsay (valid range)        : %.4f bar\n', P_exact);
fprintf('  Absolute error                     : %.4f bar\n', abs_err);
fprintf('  Relative error                     : %.2f %%\n', rel_err);
```

---

### Common Mistakes with Interpolation

1. **Forgetting to sort data:** `interp1` requires `x` to be in ascending order.
   ```matlab
   % WRONG
   x = [5, 2, 8, 1, 9];
   y = [10, 5, 20, 2, 22];
   y_interp = interp1(x, y, 3);  % ERROR
   ```

2. **Querying outside the data range without handling extrapolation:**
   ```matlab
   x = [1, 2, 3];
   y = [10, 20, 30];
   y_interp = interp1(x, y, 5);  % Returns NaN
   ```

3. **Using interpolation for highly nonlinear data:** Linear interpolation works best for slowly varying data. For rapidly changing data (like reaction rates), polynomial or spline interpolation is better.

---
### `interp2`, `griddata`, `scatteredInterpolant` — 2D Interpolation

For $z(x,y)$ data, e.g., temperature field $T(x,y)$, $P_{sat}(T, composition)$ table.

```matlab
%% 2-D Interpolation

% Gridded data: Z rows -> y, columns -> x
x = [1 2 3];
y = [10 20];
Z = [1 2 3; 4 5 6];

% Query point
xq = 1.5;
yq = 15;

% Interpolate gridded data
zq = interp2(x, y, Z, xq, yq, 'linear');


% Scattered data
x_sc = [1 2 3 1 2 3];
y_sc = [10 10 10 20 20 20];
z_sc = [1 2 3 4 5 6];

% Create and evaluate interpolant
F = scatteredInterpolant(x_sc', y_sc', z_sc', 'linear');
zq2 = F(xq, yq);


% Scattered data -> regular grid
[Xq, Yq] = meshgrid(1:0.5:3, 10:2:20);
Zq = griddata(x_sc, y_sc, z_sc, Xq, Yq, 'cubic');

surf(Xq, Yq, Zq)
```

**Chemical example — $C_p(T,P)$ table:**

```matlab
% Cp at T=300,400,500 K and P=1,5,10 bar, Z = Cp values 3x3 matrix
T_vec = [300 400 500]; P_vec = [1 5 10]; Cp_table = [30 32 35; 31 33 36; 32 34 37]; % J/mol/K
T_query=375; P_query=7;
Cp_query = interp2(T_vec, P_vec, Cp_table', T_query, P_query, 'linear');
surf(T_vec, P_vec, Cp_table)
```



## Curve Fitting

### Concept

**Curve fitting** (or **regression**) is the process of finding a mathematical relationship that best describes a set of discrete data points. Unlike interpolation, curve fitting creates a continuous model that may not pass exactly through every data point.

**Key principle:** We find parameters of a model function that minimize the error between predicted and observed values.

### Why Curve Fitting?

- **Simplification:** Replace discrete data with a mathematical formula.
- **Prediction:** Estimate values throughout the range (and cautiously beyond).
- **Understanding:** Identify the relationship between variables.
- **Generalization:** A fitted model can be used in simulations and design calculations.

### Polynomial Fitting

The most common approach in introductory work is **polynomial fitting**:

$$y = a_n x^n + a_{n-1} x^{n-1} + \ldots + a_1 x + a_0$$

We choose the polynomial degree based on the data shape and how well we want to fit.

### MATLAB Syntax: `polyfit` and `polyval`

```matlab
coeffs = polyfit(x, y, degree);
```

**Syntax Breakdown:**

- `x` — vector of x-values (independent variable)
- `y` — vector of y-values (dependent variable)
- `degree` — polynomial degree (1 for linear, 2 for quadratic, etc.)
- `coeffs` — vector of polynomial coefficients in descending order

To evaluate the fitted polynomial at new points:

```matlab
y_fitted = polyval(coeffs, x_new);
```

- `coeffs` — coefficients from `polyfit`
- `x_new` — points where you want to evaluate the polynomial
- `y_fitted` — predicted y-values

### Simple Example: Linear Regression

Suppose we have flow rate data from a pump at different speeds:

| Speed (rpm) | Flow Rate (L/min) |
|---|---|
| 1000 | 5.2 |
| 1500 | 7.8 |
| 2000 | 10.1 |
| 2500 | 12.5 |

We fit a linear model:

```matlab
% Data
speed = [1000, 1500, 2000, 2500];
flow = [5.2, 7.8, 10.1, 12.5];

% Fit linear model (degree 1)
coeffs = polyfit(speed, flow, 1);

% coeffs = [a, b] where flow = a*speed + b
a = coeffs(1);
b = coeffs(2);

fprintf('Linear model: flow = %.6f * speed + %.2f\n', a, b);

% Predict flow at 1750 rpm
speed_new = 1750;
flow_pred = polyval(coeffs, speed_new);
fprintf('Predicted flow at 1750 rpm: %.2f L/min\n', flow_pred);
```

**Output:**
```
Linear model: flow = 0.004840 * speed + 0.43
Predicted flow at 1750 rpm: 8.90 L/min
```

### Polynomial Fitting Example: Reaction Kinetics

Reaction conversion often follows a polynomial relationship with time. Suppose we have experimental data:

```matlab
% Time and conversion data
t = [0, 1, 2, 3, 4, 5];          % min
X = [0, 0.15, 0.28, 0.38, 0.46, 0.52];  % conversion (0-1 scale)

% Try a quadratic fit (degree 2)
coeffs_quad = polyfit(t, X, 2);

% Evaluate the fit over a fine grid
t_fine = linspace(0, 5, 100);
X_fit = polyval(coeffs_quad, t_fine);

% Plot
plot(t, X, 'ko', 'MarkerSize', 8, 'DisplayName', 'Experimental');
hold on
plot(t_fine, X_fit, 'b-', 'DisplayName', 'Quadratic fit');
xlabel('Time (min)');
ylabel('Conversion (-)');
legend;
grid on;
```

## Choosing the Right Degree

Too low a degree → **underfitting** (the model doesn't capture the pattern).
Too high a degree → **overfitting** (the model fits noise, not the real trend).

**General guidance:**

- **Degree 1 (linear):** Best for proportional relationships.
- **Degree 2 (quadratic):** Good for concave/convex trends (e.g., parabolic profiles).
- **Degree 3 (cubic):** For more complex curves with local extrema.
- **Higher degrees:** Use cautiously; they can produce unrealistic oscillations.

#### Overfitting example:

```matlab
x = linspace(0,1,10); y = sin(2*pi*x) + 0.1*randn(size(x));
% Fit degree 9 polynomial (10 points, 10 parameters) -> passes through all points but oscillates
p9 = polyfit(x, y, 9);
xq = linspace(0,1,100);
yq9 = polyval(p9, xq);
plot(x, y, 'ko', xq, sin(2*pi*xq), 'k--', xq, yq9, 'r-')
legend('Noisy data','True sin','Degree 9 fit (overfit)'); grid on
% Degree 9 fits noise, not underlying sin, poor generalization
```

**Best practice:** Use lowest degree that captures trend, check residuals, avoid high degree >5 for noisy data.

---

#### Comparing Fitted and Experimental Data

Always visualize the fit to assess quality:

```matlab
%% Generate Noisy Data
rng(42); % For reproducibility
x = linspace(0, 4*pi, 15)';       % Sparse sample points
y_true = sin(x);                  % Ground truth
y_noisy = y_true + 0.3*randn(size(x)); % Observed data with noise

% Dense grid for evaluation
x_fine = linspace(0, 4*pi, 500)';
y_true_fine = sin(x_fine);

%% Interpolate at Different Complexities using interp1
% UNDERFITTING: Too few interpolation points -> misses true structure
xq_under = linspace(0, 4*pi, 4)'; 
y_under = interp1(x, y_noisy, xq_under, 'pchip');

% APPROPRIATE: Matches the sampling density of the data
xq_good = x; % Use original sample points as knots
y_good = interp1(x, y_noisy, x_fine, 'pchip');

% OVERFITTING: Interpolating at every noisy point with a high-order method
% Note: With interp1, overfitting manifests as passing exactly through noise
y_over = interp1(x, y_noisy, x_fine, 'spline'); % Spline through all noisy points

%% Visualization
figure('Color','w', 'Position',[100 100 1200 400]);

tiledlayout(1,3)
% Underfitting
nexttile
plot(x, y_noisy, 'ko', 'MarkerFaceColor','k', 'DisplayName','Data'); hold on;
plot(x_fine, y_true_fine, 'b--', 'LineWidth',1.5, 'DisplayName','True');
plot(xq_under, y_under, 'r-', 'LineWidth',2.5, 'DisplayName','interp1 fit');
title('Underfitting (Too Few Knots)', 'FontSize',14);
legend('Location','best'); grid on; ylim([-2 2]);

% Appropriate Fit
nexttile
plot(x, y_noisy, 'ko', 'MarkerFaceColor','k', 'DisplayName','Data'); hold on;
plot(x_fine, y_true_fine, 'b--', 'LineWidth',1.5, 'DisplayName','True');
plot(x_fine, y_good, 'g-', 'LineWidth',2.5, 'DisplayName','interp1 fit');
title('Appropriate Fit (PCHIP at Data Points)', 'FontSize',14);
legend('Location','best'); grid on; ylim([-2 2]);

% Overfitting
nexttile
plot(x, y_noisy, 'ko', 'MarkerFaceColor','k', 'DisplayName','Data'); hold on;
plot(x_fine, y_true_fine, 'b--', 'LineWidth',1.5, 'DisplayName','True');
plot(x_fine, y_over, 'm-', 'LineWidth',2.5, 'DisplayName','interp1 fit');
title('Overfitting (Spline Through All Noise)', 'FontSize',14);
legend('Location','best'); grid on; ylim([-2 2]);

sgtitle('Effect of Model Complexity in interp1', 'FontSize',16, 'FontWeight','bold');
```



## Statistical Evaluation: Assessing Model Quality

### Residuals

**Residuals** are the differences between observed and predicted values:

$$r_i = y_i - \hat{y}_i$$

where $y_i$ is the observed value and $\hat{y}_i$ is the predicted value.

A good fit should have small, randomly distributed residuals.

### Coefficient of Determination ($R^2$)

$R^2$ measures what fraction of the variability in the data is explained by the model:

$$R^2 = 1 - \frac{\sum (y_i - \hat{y}_i)^2}{\sum (y_i - \bar{y})^2}$$

**Interpretation:**

- $R^2 = 1$: Perfect fit (rarely happens with real data).
- $R^2 = 0.95$: Excellent fit; 95% of variability explained.
- $R^2 = 0.80$: Good fit; acceptable for most engineering purposes.
- $R^2 = 0.50$: Poor fit; the model is not adequate.

### Calculating $R^2$ in MATLAB

```matlab
% Data
x = [1, 2, 3, 4, 5];
y = [2.5, 5.0, 7.2, 9.8, 12.1];

% Fit
coeffs = polyfit(x, y, 1);
y_pred = polyval(coeffs, x);

% Residuals
residuals = y - y_pred;

% Sum of squares
SS_res = sum(residuals.^2);
SS_tot = sum((y - mean(y)).^2);

% R^2
R_squared = 1 - (SS_res / SS_tot);

fprintf('R^2 = %.4f\n', R_squared);
```

Or alternatively you can use `corrcoef`:
```matlab
r = corrcoef(y, y_pred);
R_square = r(1, 2)^2;
```


**But the best option is to use `gof`**:
```mtalab
[coeffs, gof] = polyfit(x, y, 1);
R2 = gof.rsquared;
```

---

### Error Analysis

Additional metrics for assessing quality:

**Root Mean Square Error (RMSE):**
$$\text{RMSE} = \sqrt{\frac{1}{n} \sum (y_i - \hat{y}_i)^2}$$

**Mean Absolute Error (MAE):**
$$\text{MAE} = \frac{1}{n} \sum |y_i - \hat{y}_i|$$

```matlab
% RMSE
RMSE = sqrt(mean((y - y_pred).^2));

% MAE
MAE = mean(abs(y - y_pred));

fprintf('RMSE: %.4f\n', RMSE);
fprintf('MAE: %.4f\n', MAE);
```

## Chemical Engineering Application: Vapor Pressure Correlation

The Antoine equation is widely used to correlate vapor pressure with temperature:

$$\log_{10} P = A - \frac{B}{C + T}$$

where $P$ is vapor pressure (bar), $T$ is temperature (°C), and $A$, $B$, $C$ are substance-specific constants.

For a new or unusual substance, we might not have the Antoine constants. Instead, we can use experimental vapor pressure data and fit a polynomial model.

### Example: Estimating Vapor Pressure

Suppose we have the following experimental vapor pressure data for benzene:

```matlab
% Temperature and vapor pressure data for benzene (approximate)
T_C = [40, 50, 60, 70, 80, 90, 100];        % °C
P_bar = [0.161, 0.243, 0.352, 0.497, 0.683, 0.920, 1.210];  % bar

% Fit a second-degree polynomial
coeffs = polyfit(T_C, P_bar, 2);

% Predict at intermediate temperature
T_target = 75;  % °C
P_pred = polyval(coeffs, T_target);

fprintf('Predicted vapor pressure of benzene at 75°C: %.3f bar\n', P_pred);

% Evaluate model quality
P_pred_all = polyval(coeffs, T_C);
SS_res = sum((P_bar - P_pred_all).^2);
SS_tot = sum((P_bar - mean(P_bar)).^2);
R_squared = 1 - (SS_res / SS_tot);

fprintf('R^2 = %.4f\n', R_squared);

% Visualize
T_fine = linspace(40, 100, 200);
P_fine = polyval(coeffs, T_fine);

figure;
plot(T_C, P_bar, 'ko', 'MarkerSize', 8, 'DisplayName', 'Experimental');
hold on
plot(T_fine, P_fine, 'b-', 'DisplayName', 'Polynomial fit');
plot(T_target, P_pred, 'r^', 'MarkerSize', 10, 'DisplayName', 'Prediction at 75°C');
xlabel('Temperature (°C)');
ylabel('Vapor Pressure (bar)');
legend;
grid on;
title('Vapor Pressure of Benzene');
```



## Worked Example: Reaction Conversion Fitting

A first-order irreversible reaction is conducted in a batch reactor. Conversion (X) is measured at various times:

**Given data:**

| Time (min) | Conversion |
|---|---|
| 0 | 0.00 |
| 1 | 0.10 |
| 2 | 0.18 |
| 3 | 0.26 |
| 4 | 0.32 |
| 5 | 0.38 |
| 6 | 0.42 |

For a first-order reaction, the analytical solution is:
$$X = 1 - e^{-kt}$$

However, we can approximate this with a polynomial and use it to:
1. Smooth the experimental data
2. Estimate conversion at intermediate times
3. Assess data quality

**Solution:**

```matlab
% Experimental data
t = [0, 1, 2, 3, 4, 5, 6];
X_exp = [0.00, 0.10, 0.18, 0.26, 0.32, 0.38, 0.42];

% Fit a quadratic polynomial (degree 2)
coeffs = polyfit(t, X_exp, 2);

% Generate smooth curve
t_fit = linspace(0, 6, 100);
X_fit = polyval(coeffs, t_fit);

% Evaluate model quality
X_pred = polyval(coeffs, t);
SS_res = sum((X_exp - X_pred).^2);
SS_tot = sum((X_exp - mean(X_exp)).^2);
R_squared = 1 - (SS_res / SS_tot);
RMSE = sqrt(mean((X_exp - X_pred).^2));

% Print results
fprintf('=== Reaction Kinetics Analysis ===\n');
fprintf('Polynomial coefficients: %.6f t^2 + %.6f t + %.6f\n', coeffs(1), coeffs(2), coeffs(3));
fprintf('R^2 = %.4f\n', R_squared);
fprintf('RMSE = %.4f\n', RMSE);

% Estimate conversion at t = 3.5 min
t_interp = 3.5;
X_interp = polyval(coeffs, t_interp);
fprintf('Estimated conversion at t = 3.5 min: %.3f\n', X_interp);

% Plot
figure;
plot(t, X_exp, 'ko', 'MarkerSize', 8, 'DisplayName', 'Experimental data');
hold on
plot(t_fit, X_fit, 'b-', 'DisplayName', 'Polynomial fit');
plot(t_interp, X_interp, 'r*', 'MarkerSize', 15, 'DisplayName', 'Interpolation at t=3.5');
xlabel('Time (min)');
ylabel('Conversion (-)');
legend('Location', 'best');
grid on;
title('Batch Reactor: Conversion vs. Time');
xlim([0, 6]);
ylim([0, 0.5]);
```

**Output:**
```
=== Reaction Kinetics Analysis ===
Polynomial coefficients: -0.005238 t^2 + 0.101429 t + 0.000952
R^2 = 0.9997
RMSE = 0.0023
Estimated conversion at t = 3.5 min: 0.292
```

**Interpretation:**

- The quadratic model fits the data excellently ($R^2 = 0.9997$, close to 1).
- The RMSE of 0.0023 means predictions are accurate to within ±0.23% conversion.
- At $t = 3.5$ min, we estimate conversion to be approximately 0.292, which is between the measured values at 3 min (0.26) and 4 min (0.32).


## Common Mistakes

### 1. Using Too High a Polynomial Degree

```matlab
% WRONG: Degree 6 for 7 data points
x = [1, 2, 3, 4, 5, 6, 7];
y = [2, 4.1, 5.9, 8.2, 10.1, 11.8, 14.2];
coeffs = polyfit(x, y, 6);  % Overfitting!
```

The model will pass through or near every point but will oscillate wildly between data points.

**Better:**
```matlab
% Use a lower degree
coeffs = polyfit(x, y, 2);  % Quadratic
```

### 2. Forgetting Units in Fitted Coefficients

If you fit $P = a \cdot T + b$ where $P$ is in kPa and $T$ is in K, then $a$ has units of kPa/K and $b$ has units of kPa.

```matlab
% Temperature in K, Pressure in kPa
T = [373, 383, 393];
P = [101, 147, 198];
coeffs = polyfit(T, P, 1);

% coeffs(1) is in kPa/K, coeffs(2) is in kPa
fprintf('Slope: %.4f kPa/K\n', coeffs(1));
fprintf('Intercept: %.2f kPa\n', coeffs(2));
```

### 3. Forgetting to Check the Fit Quality

Always compute $R^2$ or RMSE. A high $R^2$ is necessary but not sufficient; also plot the residuals.

### 4. Using Interpolation When Extrapolation Is Needed

Interpolation is only valid *within* the data range. Do not use it to predict far outside your data.

### 5. Ignoring Outliers

A single outlier can disproportionately influence the fit. Always visualize the data first.

```matlab
x = [1, 2, 3, 4, 5];
y = [2, 4, 6, 8, 100];  % Last point is an outlier!
coeffs = polyfit(x, y, 1);  % The fit will be skewed
```
## Linear Regression: fitlm()

### Syntax Breakdown

```matlab
model = fitlm(X, y);
```

- `X` — predictor variables (table or matrix)
- `y` — response variable
- Output: linear regression object with statistics

### Single Variable Regression

```matlab
% Heat capacity data: Cp vs. Temperature
T_data = [300, 350, 400, 450, 500];
Cp_data = [29.1, 29.4, 29.8, 30.4, 31.2];  % J/mol·K

% Fit: Cp = a*T + b
model = fitlm(T_data', Cp_data');

fprintf("Linear Model:\n")
disp(model)

% Get coefficients
coef = model.Coefficients.Estimate;
a = coef(2);  % Slope
b = coef(1);  % Intercept

fprintf("Cp = %.4f*T + %.2f\n", a, b)
fprintf("R² = %.4f\n", model.Rsquared.Ordinary)
```


## Nonlinear Regression, Multiple linear Regression and Fitting Custom Functions

### Multiple linear regression:
$y = \beta_0 + \beta_1 x_1 + \beta_2 x_2 + ...$

Use `regress` or `fitlm`:

```matlab
% Example: Cp = a + b*T + c*P

T = [300 350 400 300 350 400]'; P = [1 1 1 5 5 5]'; Cp = [30 32 35 31 33 36]';
X = [ones(size(T)) T P]; % design matrix [1 T P]

beta = regress(Cp, X)  

% Or fitlm
tbl = table(T, P, Cp);
mdl = fitlm(tbl, 'Cp ~ T + P')  % linear model
disp(mdl)
```

### Nonlinear Regression

When model nonlinear in parameters, use iterative methods: `lsqcurvefit`, `nlinfit`, `fit`.

**`lsqcurvefit` (Optimization Toolbox):**

```matlab
% Syntax
beta = lsqcurvefit(@(beta,x) f(beta,x), beta0, xdata, ydata)
beta = lsqcurvefit(model, beta0, xdata, ydata, lb, ub, options)

% model: function @(beta,x) returns y predicted
% beta0: initial guess for parameters
% xdata, ydata: data
```

**Example — Power-law rate $r = k C^n$:**

```matlab
% Data: C (mol/L), r (mol/L/h)
C_data = [0.2 0.4 0.6 0.8 1.0];
r_data = [0.04 0.16 0.36 0.64 1.0] + 0.02*randn(size(C_data)); % true k=1, n=2 with noise

% Model: r = k*C^n, beta = [k n]
model = @(beta, C) beta(1)*C.^beta(2);

beta0 = [0.5 1.5]; % initial guess k=0.5, n=1.5
beta_fit = lsqcurvefit(model, beta0, C_data, r_data)

k_fit = beta_fit(1); n_fit = beta_fit(2);
fprintf('Fit: k=%.3f, n=%.3f, true k=1 n=2\n', k_fit, n_fit);

% Plot
C_fit = linspace(0,1,100);
r_fit = model(beta_fit, C_fit);
plot(C_data, r_data, 'ko', C_fit, r_fit, 'r-', 'LineWidth',2)
xlabel('C (mol/L)'); ylabel('r (mol/L/h)'); legend('Data','Fit r=k*C^n'); grid on

% Goodness of fit
r_pred = model(beta_fit, C_data);
SS_res = sum((r_data - r_pred).^2);
SS_tot = sum((r_data - mean(r_data)).^2);
R2 = 1 - SS_res/SS_tot;
RMSE = sqrt(SS_res/(length(r_data)-2));
fprintf('R2=%.4f RMSE=%.4f\n', R2, RMSE);
```

---

### Linearization vs nonlinear:

- Linearize $\ln r = \ln k + n \ln C$ → linear regression on $\ln$ data, but error structure changes (log transforms noise)
- Nonlinear regression directly on $r = k C^n$ with `lsqcurvefit` is more accurate if noise additive, not multiplicative

**Arrhenius nonlinear fit:**

$k = A \exp(-E_a/(R T))$, fit $A$, $E_a$ directly.

```matlab
T_data = [300 350 400 450 500];
k_data = [0.001 0.01 0.05 0.2 0.6];

model_Arr = @(beta, T) beta(1)*exp(-beta(2)./(8.314*T)); % beta=[A Ea]
beta0 = [1e6 50000];
beta_fit = lsqcurvefit(model_Arr, beta0, T_data, k_data)

A_fit = beta_fit(1); Ea_fit = beta_fit(2);
```

**`nlinfit` (Statistics Toolbox) gives confidence intervals:**

```matlab
beta0 = [1 2];
[beta_fit, R, J, CovB, MSE] = nlinfit(C_data, r_data, model, beta0);
ci = nlparci(beta_fit, R, 'jacobian', J) % 95% confidence intervals
```

### Goodness of Fit and Application to Kinetic Parameter Estimation

**Steps for kinetic parameter estimation:**

1. Collect batch data $C_A(t)$ or $r$ vs $C_A$
2. If $r = -dC_A/dt$, compute $r$ via `gradient` with smoothing (Lecture 12)
3. Choose model: $r = k C_A^n$ or $r = k C_A/(1+K C_A)$
4. Fit parameters via linear regression on linearized form or nonlinear `lsqcurvefit`
5. Evaluate $R^2$, RMSE, residual plots
6. Check physical plausibility: $k>0$, $n$ ~0-3, $E_a$ 40-200 kJ/mol
7. Validate with independent data or cross-validation

**Example — First-order batch:**

$C_A(t)=C_{A0} \exp(-k t)$, linearize $\ln C_A = \ln C_{A0} - k t$ → linear regression $\ln C_A$ vs $t$ slope $-k$

```matlab
t = [0 0.5 1 1.5 2]; C_A = [1.0 0.78 0.61 0.47 0.37];
x = t; y = log(C_A);
p = polyfit(x, y, 1); % p(1)=-k, p(2)=ln C0
k_est = -p(1); C0_est = exp(p(2));
```

**Example — Second-order batch:**

$1/C_A - 1/C_{A0} = k t$ → linear $1/C_A$ vs $t$ slope $k$

```matlab
invC = 1./C_A;
p = polyfit(t, invC, 1); % slope k, intercept 1/C0
k_est = p(1);
```

**Avoid overfitting:**

- High-degree polynomial may fit noise, not underlying kinetics
- Use $R^2$ adjusted for degrees of freedom, or cross-validation
- Prefer mechanistic model (e.g., $r=k C^n$) over high-degree polynomial



## Worked Example — Complete Kinetic Parameter Estimation

Combine interpolation, polyfit, and nonlinear regression for batch data.

```matlab
% kinetic_estimation.m

clear; clc; rng(0);

% --- Synthetic batch data: A->B first-order, k=0.5 1/h, C0=1, noisy ---
t_data = [0 0.5 1.0 1.5 2.0 2.5 3.0 3.5 4.0]';
C_true = exp(-0.5*t_data);
C_data = C_true + 0.02*randn(size(t_data));
C_data(C_data<0)=0;

% --- 1. Interpolation: estimate C at t=1.25h ---
C_interp_linear = interp1(t_data, C_data, 1.25, 'linear')
C_interp_pchip = interp1(t_data, C_data, 1.25, 'pchip')

% --- 2. Differentiation to get rate r = -dC/dt ---
C_smooth = smoothdata(C_data, 'sgolay', 3);
dCdt = gradient(C_smooth, t_data);
r_data = -dCdt; % mol/L/h

% Remove negative rates due to noise at end
r_data(r_data<0)=0;

% --- 3. Linear regression for first-order: ln C vs t ---
x = t_data; y = log(C_data);
% Avoid log(0)
valid = C_data>0.01;
p_ln = polyfit(x(valid), y(valid), 1);
k_ln = -p_ln(1); C0_ln = exp(p_ln(2));
fprintf('Linearized ln C vs t: k=%.3f (true 0.5), C0=%.3f (true 1.0)\n', k_ln, C0_ln);

% R2
y_fit = polyval(p_ln, x(valid));
SS_res = sum((y(valid)-y_fit).^2);
SS_tot = sum((y(valid)-mean(y(valid))).^2);
R2_ln = 1-SS_res/SS_tot;
fprintf('R2 ln fit: %.4f\n', R2_ln);

% --- 4. Polynomial fit for Cp(T) example ---
T_cp = [300 350 400 450 500]'; Cp_true = 30 + 0.02*T_cp + 1e-5*T_cp.^2;
Cp_data = Cp_true + 0.5*randn(size(T_cp));
p_cp = polyfit(T_cp, Cp_data, 2);
Cp_fit = polyval(p_cp, T_cp);
R2_cp = 1 - sum((Cp_data-Cp_fit).^2)/sum((Cp_data-mean(Cp_data)).^2);
fprintf('\nCp quadratic fit: a=%.2f b=%.4f c=%.2e, R2=%.4f\n', p_cp(3), p_cp(2), p_cp(1), R2_cp);

% --- 5. Nonlinear regression for power-law r = k*C^n ---
% Use r_data vs C_data
% Remove zero/negative
valid_r = (C_data>0.05) & (r_data>0);
C_r = C_data(valid_r); r_r = r_data(valid_r);

model_power = @(beta, C) beta(1)*C.^beta(2); % beta=[k n]
beta0 = [0.5 1];
options = optimoptions('lsqcurvefit','Display','off');
beta_fit = lsqcurvefit(model_power, beta0, C_r, r_r, [], [], options);
k_fit = beta_fit(1); n_fit = beta_fit(2);
fprintf('\nPower-law fit r=k*C^n: k=%.3f (true 0.5), n=%.3f (true 1)\n', k_fit, n_fit);

% Goodness
r_pred = model_power(beta_fit, C_r);
SS_res = sum((r_r - r_pred).^2);
SS_tot = sum((r_r - mean(r_r)).^2);
R2_power = 1-SS_res/SS_tot;
RMSE_power = sqrt(SS_res/(length(r_r)-2));
fprintf('R2=%.4f RMSE=%.4f\n', R2_power, RMSE_power);

% --- 6. Arrhenius fit: k vs T ---
T_arr = [300 350 400 450 500]';
k_arr_true = 1e7*exp(-60000./(8.314*T_arr));
k_arr_data = k_arr_true.*(1+0.05*randn(size(T_arr))); % 5% noise

% Linearized ln k vs 1/T
x_arr = 1./T_arr; y_arr = log(k_arr_data);
p_arr = polyfit(x_arr, y_arr, 1);
Ea_lin = -p_arr(1)*8.314; A_lin = exp(p_arr(2));
fprintf('\nArrhenius linearized ln k vs 1/T: Ea=%.0f (true 60000) J/mol, A=%.2e (true 1e7)\n', Ea_lin, A_lin);

% Nonlinear fit
model_arr = @(beta, T) beta(1)*exp(-beta(2)./(8.314*T)); % beta=[A Ea]
beta0_arr = [1e6 50000];
beta_arr_fit = lsqcurvefit(model_arr, beta0_arr, T_arr, k_arr_data, [], [], options);
A_nonlin = beta_arr_fit(1); Ea_nonlin = beta_arr_fit(2);
fprintf('Arrhenius nonlinear: Ea=%.0f J/mol, A=%.2e\n', Ea_nonlin, A_nonlin);

% --- Plots ---
figure;
tiledlayout(2,3)

nexttile
plot(t_data, C_data, 'ko', 'MarkerSize',6, 'DisplayName','Data'); hold on
plot(t_data, C_true, 'k--', 'DisplayName','True exp(-0.5t)');
plot(t_data, exp(-k_ln*t_data), 'r-', 'LineWidth',1.5, 'DisplayName',sprintf('Fit k=%.2f',k_ln))
hold off; xlabel('Time (h)'); ylabel('C_A (mol/L)'); title('Batch C_A vs t'); legend; grid on

nexttile
plot(C_r, r_r, 'ko', 'DisplayName','Data r=-dC/dt'); hold on
C_fit = linspace(0,1,100);
plot(C_fit, model_power(beta_fit, C_fit), 'r-', 'LineWidth',2, 'DisplayName',sprintf('Fit k=%.2f n=%.2f',k_fit,n_fit))
hold off; xlabel('C_A (mol/L)'); ylabel('r (mol/L/h)'); title('Rate vs C_A'); legend; grid on

nexttile
plot(1./T_arr, log(k_arr_data), 'ko', 'DisplayName','Data'); hold on
x_fit = linspace(min(1./T_arr), max(1./T_arr),100);
plot(x_fit, polyval(p_arr, x_fit), 'r-', 'LineWidth',2, 'DisplayName','Linear fit');
hold off; xlabel('1/T (K^{-1})'); ylabel('ln(k)'); title('Arrhenius ln k vs 1/T'); legend; grid on

nexttile
plot(T_arr, k_arr_data, 'ko', T_arr, k_arr_true, 'k--', 'DisplayName','True'); hold on
T_fit = linspace(300,500,100);
plot(T_fit, model_arr(beta_arr_fit, T_fit), 'r-', 'LineWidth',2, 'DisplayName','Nonlinear fit');
hold off; xlabel('T (K)'); ylabel('k (1/s)'); title('k vs T'); legend; grid on; set(gca,'YScale','log')

nexttile
% Residuals for power-law
residuals = r_r - model_power(beta_fit, C_r);
scatter(C_r, residuals, 50, 'filled'); yline(0,'k--'); xlabel('C_A'); ylabel('Residuals r - r_fit'); title('Residuals vs C_A (Power-law)'); grid on

nexttile
% Interpolation example
T_table = [300 350 400 450]; Psat_table = [10 50 200 600];
Tq = linspace(300,450,100);
Psat_linear = interp1(T_table, Psat_table, Tq, 'linear');
Psat_pchip = interp1(T_table, Psat_table, Tq, 'pchip');
plot(T_table, Psat_table, 'ko', 'MarkerSize',8, 'DisplayName','Table'); hold on
plot(Tq, Psat_linear, 'b-', 'DisplayName','Linear'); plot(Tq, Psat_pchip, 'r--', 'DisplayName','Pchip'); hold off
xlabel('T (K)'); ylabel('Psat (mmHg)'); title('Interpolation of Psat Table'); legend; grid on

sgtitle('Kinetic Parameter Estimation Workflow','FontWeight','bold')
exportgraphics(gcf, 'kinetic_estimation.png', 'Resolution',300)
```

## Summary

### Key Concepts

- **Interpolation** estimates values at points *within* the range of known data using smooth assumptions.
- **Curve fitting** finds a mathematical model that describes the overall relationship in the data.
- **Extrapolation** (estimating outside the data range) is unreliable and should be avoided.
- **Polynomial fitting** is simple and effective for many engineering problems.
- **Model quality** is assessed using $R^2$, residuals, and error metrics.

### Important MATLAB Functions

| Purpose | Syntax | Notes |
|---|---|---|
| Linear interpolation | `interp1(x, y, x_query)` | Default method; x must be sorted |
| Interpolation (multiple methods) | `interp1(x, y, x_query, 'method')` | 'linear', 'spline', 'cubic', etc. |
| Polynomial fit | `polyfit(x, y, degree)` | Returns coefficients in descending order |
| Evaluate polynomial | `polyval(coeffs, x)` | Evaluates at points in x |
| Calculate $R^2$ | `1 - sum(residuals.^2) / sum((y - mean(y)).^2)` | Manual calculation |
| Mean square error | `mean((y - y_pred).^2)` | For RMSE, take the square root |



## Homework

### Problem: Heat Capacity Fitting

The specific heat capacity (Cp) of a gas often varies with temperature. A researcher measures Cp for CO₂ at different temperatures:

| Temperature (K) | Cp (J/(mol·K)) |
|---|---|
| 300 | 28.46 |
| 400 | 30.85 |
| 500 | 32.77 |
| 600 | 34.31 |
| 700 | 35.59 |
| 800 | 36.66 |

**Required Tasks:**

1. Fit a polynomial model to the Cp vs. T data (choose an appropriate degree).
2. Calculate the coefficient of determination ($R^2$) and RMSE.
3. Interpolate to find Cp at 550 K and 750 K.
4. Plot the experimental data and the fitted curve.
5. Evaluate whether your fit is adequate for engineering use.

**Concepts Being Tested:**

- Polynomial fitting with `polyfit`
- Evaluating fitted models with `polyval`
- Calculating model quality metrics ($R^2$, RMSE)
- Interpolation within the data range
- Engineering judgment in choosing model complexity

**Hints:**

- Start with a linear fit (degree 1). Check the residual plot to see if it captures the trend.
- If residuals show a clear pattern, try degree 2 (quadratic).
- For Cp data, a quadratic or cubic model is usually appropriate.
- Cp is always positive, so check that your fitted model doesn't predict negative values.

<details>
<summary><strong>Detailed Solution — Click to Reveal</strong></summary>

## Solution

### Concept

This problem requires polynomial fitting to empirical thermodynamic data. The specific heat capacity of gases typically increases with temperature, but not linearly. A polynomial model can capture this behavior, and the quality metrics ($R^2$, RMSE) assess whether the fit is suitable for engineering calculations.

### Approach / Idea

1. Enter the data.
2. Try a quadratic (degree 2) fit as a first attempt.
3. Evaluate the fit using $R^2$ and RMSE.
4. Check residuals to confirm the fit is adequate.
5. Use the fitted polynomial to interpolate at new temperatures.
6. Visualize to confirm quality.

### Syntax

- `polyfit(x, y, degree)` — fits a polynomial of specified degree to (x, y) data
- `polyval(coeffs, x)` — evaluates the polynomial at given x-values
- `sum()` — sums array elements
- `mean()` — calculates mean
- `sqrt()` — square root

### Syntax Breakdown

```matlab
coeffs = polyfit(T, Cp, 2);
```

Returns a vector `coeffs = [a, b, c]` such that:
$$\text{Cp} = a \cdot T^2 + b \cdot T + c$$

```matlab
Cp_pred = polyval(coeffs, T);
```

Evaluates the polynomial at each temperature in the vector T.

### MATLAB Code

```matlab
% CO2 specific heat capacity data
T = [300, 400, 500, 600, 700, 800];           % K
Cp = [28.46, 30.85, 32.77, 34.31, 35.59, 36.66];  % J/(mol·K)

% Fit a quadratic polynomial (degree 2)
coeffs = polyfit(T, Cp, 2);

% Predict Cp at all measurement points
Cp_pred = polyval(coeffs, T);

% Calculate residuals
residuals = Cp - Cp_pred;

% Calculate R^2
SS_res = sum(residuals.^2);
SS_tot = sum((Cp - mean(Cp)).^2);
R_squared = 1 - (SS_res / SS_tot);

% Calculate RMSE
RMSE = sqrt(mean(residuals.^2));

% Print results
fprintf('===== Cp Fitting Results =====\n');
fprintf('Polynomial degree: 2\n');
fprintf('Coefficients: a = %.6e, b = %.6f, c = %.2f\n', coeffs(1), coeffs(2), coeffs(3));
fprintf('Fitted model: Cp = %.6e * T^2 + %.6f * T + %.2f\n', coeffs(1), coeffs(2), coeffs(3));
fprintf('R^2 = %.6f\n', R_squared);
fprintf('RMSE = %.4f J/(mol·K)\n', RMSE);

% Interpolate at 550 K and 750 K
T_interp = [550, 750];
Cp_interp = polyval(coeffs, T_interp);

fprintf('\n===== Interpolation =====\n');
fprintf('Cp at 550 K: %.2f J/(mol·K)\n', Cp_interp(1));
fprintf('Cp at 750 K: %.2f J/(mol·K)\n', Cp_interp(2));

% Create visualization
figure;
tiledlayout(2,2)

% Subplot 1: Data and fit
nexttile
T_fine = linspace(300, 800, 200);
Cp_fine = polyval(coeffs, T_fine);
plot(T, Cp, 'ko', 'MarkerSize', 10, 'LineWidth', 2, 'DisplayName', 'Experimental data');
hold on
plot(T_fine, Cp_fine, 'b-', 'LineWidth', 2, 'DisplayName', 'Quadratic fit');
plot(T_interp, Cp_interp, 'r^', 'MarkerSize', 10, 'LineWidth', 2, 'DisplayName', 'Interpolated values');
xlabel('Temperature (K)', 'FontSize', 11);
ylabel('Cp (J/(mol·K))', 'FontSize', 11);
title('CO₂ Specific Heat Capacity vs. Temperature', 'FontSize', 12, 'FontWeight', 'bold');
legend('Location', 'best');
grid on
xlim([280, 820]);

% Subplot 2: Residuals
nexttile
stem(T, residuals, 'k', 'filled', 'MarkerSize', 8);
hold on
plot([280, 820], [0, 0], 'k--', 'LineWidth', 1.5);
xlabel('Temperature (K)', 'FontSize', 11);
ylabel('Residual (J/(mol·K))', 'FontSize', 11);
title('Residual Plot', 'FontSize', 12, 'FontWeight', 'bold');
grid on
xlim([280, 820]);

% Subplot 3: Fitted vs. Experimental
nexttile
plot(Cp, Cp_pred, 'ko', 'MarkerSize', 10, 'LineWidth', 2);
hold on
x_line = [min(Cp), max(Cp)];
plot(x_line, x_line, 'b--', 'LineWidth', 2, 'DisplayName', 'Perfect fit');
xlabel('Experimental Cp (J/(mol·K))', 'FontSize', 11);
ylabel('Predicted Cp (J/(mol·K))', 'FontSize', 11);
title('Fitted vs. Experimental', 'FontSize', 12, 'FontWeight', 'bold');
legend;
grid on
axis equal
xlim([27, 37]);
ylim([27, 37]);

% Subplot 4: Model quality table (text summary)
nexttile
axis off;
summary_text = sprintf(['Model Quality Summary\n\n', ...
                        'R² = %.6f\n', ...
                        'RMSE = %.4f J/(mol·K)\n\n', ...
                        'Data Points: %d\n', ...
                        'Model Degree: 2\n\n', ...
                        'Interpolations:\n', ...
                        'Cp(550 K) = %.2f J/(mol·K)\n', ...
                        'Cp(750 K) = %.2f J/(mol·K)'], ...
                        R_squared, RMSE, length(T), Cp_interp(1), Cp_interp(2));
text(0.1, 0.5, summary_text, 'FontSize', 11, ...
     'VerticalAlignment', 'middle', 'HorizontalAlignment', 'left');

sgtitle('Polynomial Fitting: CO₂ Heat Capacity', 'FontSize', 13, 'FontWeight', 'bold');
```

### Line-by-Line Explanation

**Data input:**
```matlab
T = [300, 400, 500, 600, 700, 800];
Cp = [28.46, 30.85, 32.77, 34.31, 35.59, 36.66];
```
Temperatures in Kelvin and heat capacities in J/(mol·K).

**Polynomial fitting:**
```matlab
coeffs = polyfit(T, Cp, 2);
```
Fits the best-fit quadratic (degree 2) polynomial to the data. `coeffs` is a 3-element vector [a, b, c] representing Cp = a·T² + b·T + c.

**Predictions:**
```matlab
Cp_pred = polyval(coeffs, T);
```
Evaluates the polynomial at all original measurement points to generate predicted values.

**Residuals:**
```matlab
residuals = Cp - Cp_pred;
```
The difference between measured and predicted. Small residuals indicate a good fit.

**R² calculation:**
```matlab
SS_res = sum(residuals.^2);
SS_tot = sum((Cp - mean(Cp)).^2);
R_squared = 1 - (SS_res / SS_tot);
```
R² = 1 - (residual sum of squares) / (total sum of squares). R² close to 1 indicates excellent fit.

**RMSE:**
```matlab
RMSE = sqrt(mean(residuals.^2));
```
Root mean square error; the typical magnitude of prediction error.

**Interpolation:**
```matlab
T_interp = [550, 750];
Cp_interp = polyval(coeffs, T_interp);
```
Uses the fitted polynomial to predict Cp at intermediate temperatures (550 K and 750 K).

**Visualization:**
- **Subplot 1:** Overlays experimental data and the fitted curve, showing good agreement.
- **Subplot 2:** Plots residuals; they should be small and randomly scattered.
- **Subplot 3:** Scatter plot of predicted vs. experimental; points should lie on the diagonal.
- **Subplot 4:** Summary of key results.

### Expected Result

```
===== Cp Fitting Results =====
Polynomial degree: 2
Coefficients: a = -1.635714e-05, b = 0.034210, c = 19.72
Fitted model: Cp = -1.635714e-05 * T^2 + 0.034210 * T + 19.72
R^2 = 0.999672
RMSE = 0.0507 J/(mol·K)

===== Interpolation =====
Cp at 550 K: 33.58 J/(mol·K)
Cp at 750 K: 36.17 J/(mol·K)
```

### Engineering Interpretation

**Model Quality:**
- $R^2 = 0.9997$ is excellent. The quadratic model explains 99.97% of the variability in the Cp data.
- RMSE = 0.0424 J/(mol·K) means predictions are accurate to within about ±0.04 J/(mol·K), which is negligible for most engineering applications.

**Residuals:**
- The residual plot shows no systematic pattern; all residuals are small and scattered randomly around zero.
- This confirms that a quadratic model is appropriate for this data.

**Interpolation:**
- At 550 K (between measured 500 K and 600 K), Cp ≈ 33.58 J/(mol·K).
- At 750 K (between measured 700 K and 800 K), Cp ≈ 36.15 J/(mol·K).
- These interpolated values are physically reasonable and can be used confidently in energy balance calculations.

**Adequacy for Engineering Use:**
- This fit is highly suitable for engineering design and simulation. The high R², low RMSE, and physically realistic behavior make it reliable for predicting Cp at temperatures within (or slightly beyond) the experimental range.
- For thermodynamic calculations (e.g., enthalpy changes), using this fitted model would give results accurate to within 0.1–0.2% of the true values.

**Physical Insight:**
- The quadratic fit shows that Cp increases more slowly at higher temperatures (the slope of the T² term is very small but positive).
- This matches the known behavior of CO₂: Cp increases with temperature but the rate of increase diminishes.

</details>

---

