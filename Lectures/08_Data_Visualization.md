# Data Visualization

## Learning Objectives


By the end of this lecture, you will be able to:

- Create 2D plots with `plot`, `scatter`, `bar`, `histogram`, `semilogx`, `semilogy`, `loglog`
- Add labels, titles, legends, grids, and axis limits with `xlabel`, `ylabel`, `title`, `legend`, `grid`, `xlim`, `ylim`
- Create multiple plots in one figure with `hold on`, `subplot`, `tiledlayout`, `yyaxis`
- Customize line styles, colors, markers, linewidth, and fonts for engineering-style publication plots
- Annotate plots with `text`, `annotation`, `xline`, `yline` for operating limits and safety thresholds
- Create 3D and advanced visualizations: `plot3`, `surf`, `mesh`, `contour`, `contourf`, `pcolor` for temperature fields, reaction surfaces, and VLE diagrams
- Visualize chemical engineering data: concentration vs time, temperature profiles, CSTR conversion vs residence time, Arrhenius plots, and histogram of process variability
- Export publication-quality figures with `exportgraphics`, `print`, and proper resolution

---

## Why This Topic Matters

**Visualizations** communicate data insights:

- **Reveal patterns** — trends are easier to see in plots than tables
- **Compare data** — side-by-side plots show differences clearly
- **Build intuition** — visual understanding aids design decisions
- **Publish results** — figures are essential in technical reports and papers
- **Debug code** — plots reveal if calculations are sensible

Professional presentations and publications require high-quality, publication-ready plots.

---

## Basic Line Plot

### Creating and Formatting

```matlab
% Data
x = 0:0.1:10;
y = sin(x);

% Create line plot
figure;
plot(x, y, "b-", "LineWidth", 2)

% Labels and title
xlabel("Angle (radians)", "FontSize", 12)
ylabel("Sine Value", "FontSize", 12)
title("Sine Wave", "FontSize", 14)

% Grid and layout
grid on
grid minor
```

### Syntax Breakdown

```matlab
plot(x, y, 'color-linestyle-marker', 'PropertyName', PropertyValue, ...)
```

- `x, y` — data vectors
- `'b-'` — blue line (no markers)
- Color codes: `'b'` (blue), `'r'` (red), `'g'` (green), `'k'` (black)
- Line styles: `'-'` (solid), `'--'` (dashed), `':'` (dotted)
- Markers: `'o'` (circle), `'s'` (square), `'^'` (triangle)
- `'LineWidth'`, 2 — line thickness
- `'MarkerSize'`, 8 — marker size

| Component | Options | Example |
|---|---|---|
| Color | `r` red, `g` green, `b` blue, `k` black, `m` magenta, `c` cyan, `y` yellow, `w` white | `'r'` |
| Line style | `-` solid, `--` dashed, `:` dotted, `-.` dash-dot, `none` no line | `'--'` |
| Marker | `o` circle, `s` square, `^` triangle, `*` star, `.` point, `+` plus, `x` x | `'o'` |
| Combined | `'r--o'` | red dashed with circles |


### Labels, Titles, Legends, Grids

```matlab
xlabel('Time (h)', 'FontSize', 12)
ylabel('Concentration (mol/L)', 'FontSize', 12)
title('Batch Reactor: C_A vs Time', 'FontSize', 14, 'FontWeight', 'bold')
legend('Experimental', 'Model', 'Location', 'best')
grid on
grid minor
xlim([0 10]); ylim([0 1.2])
```

- `xlabel`, `ylabel`, `title` — add text, include units in parentheses, standard for chemical engineering
- `legend` — labels each line, `'Location','best'` auto-places, or `'northeast'`, `'southwest'`, etc.
- `grid on` — major grid, `grid minor` — minor grid, helps reading values
- `xlim`, `ylim` — set axis limits, e.g., `xlim([0 10])`
- `xticks`, `yticks` — customize tick locations

### Multiple Lines

```matlab
x = linspace(0, 2*pi, 100);

figure
plot(x, sin(x), "b-", "LineWidth", 2, "DisplayName", "sin(x)")
hold on
plot(x, cos(x), "r--", "LineWidth", 2, "DisplayName", "cos(x)")
plot(x, tan(x), "g:", "LineWidth", 2, "DisplayName", "tan(x)")

xlabel("x", "FontSize", 12)
ylabel("Function Value", "FontSize", 12)
title("Trigonometric Functions", "FontSize", 14)
legend("FontSize", 11)
grid on
```


### Multiple Plots and Subplot

**Hold on:**

```matlab
figure;
plot(t, C_A, 'b-o'); hold on
plot(t, C_B, 'r--s'); hold off
xlabel('Time (h)'); ylabel('Concentration (mol/L)');
legend('A','B'); grid on
```

`hold on` keeps current plot, next `plot` adds to same axes; `hold off` releases.

**Subplot:**

```matlab
figure;
subplot(2,2,1); plot(t, C_A); title('C_A')
subplot(2,2,2); plot(t, C_B); title('C_B')
subplot(2,2,3); plot(t, T); title('T')
subplot(2,2,4); plot(t, P); title('P')
```

`subplot(rows, cols, index)` — divides figure into grid.

**Modern alternative `tiledlayout`:**

```matlab
tiledlayout(2,2);
nexttile; plot(t, C_A); title('C_A')
nexttile; plot(t, C_B); title('C_B')
nexttile; plot(t, T); title('T')
nexttile; plot(t, P); title('P')
```

More flexible than `subplot`.

**Dual y-axis:**

```matlab
yyaxis left; plot(t, T); ylabel('T (K)')
yyaxis right; plot(t, P); ylabel('P (bar)')
```

Useful when $T$ and $P$ have different scales but same time axis.


### Chemical Engineering Example: Reactor Conversion

```matlab
% Plot conversion vs. temperature for exothermic reaction
T = linspace(300, 400, 100);     % Temperature (K)
k = 1e8 * exp(-50000 ./ (8.314 * T));  % Arrhenius: rate constant
X = k .* 10 ./ (1 + k .* 10);    % Conversion (simplified)

figure
plot(T, X*100, "b-", "LineWidth", 2.5)
hold on

xlabel("Temperature (K)", "FontSize", 12)
ylabel("Conversion (%)", "FontSize", 12)
title("Reactor Conversion vs. Temperature", "FontSize", 14)
grid on

% Add reference temperature
T_design = 350;
X_design = interp1(T, X*100, T_design);
plot(T_design, X_design, "ro", "MarkerSize", 10)
legend("Conversion", "Design Point", "FontSize", 11)
```
### Chemical Engineering Example — CSTR Conversion vs Residence Time

```matlab
% First-order CSTR: X = k*tau/(1+k*tau)
k = 0.1; % 1/min
tau = linspace(0,50,100); % min
X = k*tau./(1+k*tau);

figure;
plot(tau, X, 'r-', 'LineWidth', 2)
xlabel('\tau (min)'); ylabel('Conversion X');
title('CSTR: Conversion vs Residence Time (First-Order, k=0.1 1/min)');
grid on; ylim([0 1]); xlim([0 50])
yline(0.9, 'k--', '90% conversion', 'LabelHorizontalAlignment','left')
xline(40, 'b--', '\tau for 40% X') 
```

---

## Scatter Plots

### Basic Scatter

```matlab
% Experimental data
x = randn(1, 50) * 10 + 50;    % Temperature, ±10°C around 50°C
y = 2*x + randn(1, 50)*5 + 100; % Linear relationship with noise

figure("Name","Scatter plot Example")
scatter(x, y, 100, "b", "filled")

xlabel("Input (°C)", "FontSize", 12)
ylabel("Output (Response)", "FontSize", 12)
title("Experimental Data", "FontSize", 14)
grid on
```

### Syntax Breakdown

```matlab
scatter(x, y, size, color, 'filled');
```

- `x, y` — data points
- `size` — marker size (default 36)
- `color` — marker color
- `'filled'` — fill markers

---

## Bar Charts

### Basic Bar Plot

```matlab
% Production by plant
plants = {"Plant A", "Plant B", "Plant C", "Plant D"};
production = [450, 380, 520, 290];  % tons/day

figure
bar(production, 0.7, "FaceColor", [0.2 0.6 0.8])

set(gca, "XTickLabel", plants, "FontSize", 11)
ylabel("Production (tons/day)", "FontSize", 12)
title("Daily Production by Plant", "FontSize", 14)
grid on

% Add values on bars
for i = 1:length(production)
    text(i, production(i)+10, num2str(production(i)), ...
        "HorizontalAlignment", "center", "FontSize", 10)
end
```

### Grouped Bar Charts

```matlab
% Product yield by reactor type and temperature
ReactorTypes = {"CSTR", "PFR", "Batch"};
T_low = [65, 72, 58];      % Yield at 300K
T_high = [71, 78, 62];     % Yield at 350K

figure
x = 1:length(ReactorTypes);
width = 0.35;

bar(x - width/2, T_low, width, "FaceColor", [0.8 0.4 0.4], "DisplayName", "T = 300 K")
hold on
bar(x + width/2, T_high, width, "FaceColor", [0.4 0.4 0.8], "DisplayName", "T = 350 K")

set(gca, "XTick", x, "XTickLabel", ReactorTypes, "FontSize", 11)
ylabel("Yield (%)", "FontSize", 12)
title("Product Yield Comparison", "FontSize", 14)
legend("FontSize", 11)
grid on
```
---
## Histograms and Box Plots

```matlab
histogram(data, 15)             % 15 bins
boxplot(data)                   % distribution summary
```
**Example:**

```matlab
% Histogram: pressure variability
P_data = 5 + 0.3*randn(1000,1); % bar, normal distribution
figure; histogram(P_data, 20); xlabel('P (bar)'); ylabel('Count'); title('Pressure Distribution');
xline(5, 'r--', 'Mean');  % annotation line at mean
```

---
### Error Bars

```matlab
errorbar(x, y, yerr)                % symmetric error bars
errorbar(x, y, yerr_lower, yerr_upper)  % asymmetric
```

### Chemical Engineering Example — Conversion with Uncertainty

```matlab
T = [320 340 360 380];
X = [0.45 0.62 0.78 0.88];
Xerr = [0.03 0.04 0.03 0.02];       % standard error

figure
errorbar(T, X, Xerr, 'o', 'MarkerSize', 8, ...
    'MarkerFaceColor', 'b', 'LineWidth', 1.2)
xlabel('Temperature (K)')
ylabel('Conversion')
title('Measured conversion with standard error')
grid on
ylim([0 1])
```
---

## Subplots

### Layout Multiple Plots

```matlab
figure("Name", "Subplots")

% Top left: line plot
subplot(2, 3, 1)
x = linspace(0, 10, 100);
plot(x, sin(x), "b-", "LineWidth", 2)
ylabel("sin(x)", "FontSize", 11)
title("Sine", "FontSize", 12)
grid on

% Top middle: scatter
subplot(2, 3, 2)
scatter(randn(1,50), randn(1,50), 80, "r", "filled")
title("Random Data", "FontSize", 12)
grid on

% Top right: bar
subplot(2, 3, 3)
bar([10, 20, 15, 25], "FaceColor", [0.5 0.8 0.5])
title("Bar Chart", "FontSize", 12)
grid on

% Bottom left: histogram
subplot(2, 3, 4)
histogram(randn(1, 1000), 40, "FaceColor", [1 0.7 0.5])
title("Histogram", "FontSize", 12)
xlabel("Value")

% Bottom middle: loglog
subplot(2, 3, 5)
x = logspace(0, 3, 50);
loglog(x, x.^2, "g-", "LineWidth", 2)
title("Log-Log Plot", "FontSize", 12)
grid on

% Bottom right: semilogy
subplot(2, 3, 6)
x = 1:10;
semilogy(x, 10.^x, "m-", "LineWidth", 2)
title("Semilog Plot", "FontSize", 12)
grid on
```
Or:

```matlab
figure("Name", "Tiledlayouts")

tiledlayout(2,3);
% Top left: line plot
nexttile;
x = linspace(0, 10, 100);
plot(x, sin(x), "b-", "LineWidth", 2)
ylabel("sin(x)", "FontSize", 11)
title("Sine", "FontSize", 12)
grid on

% Top middle: scatter
nexttile;
scatter(randn(1,50), randn(1,50), 80, "r", "filled")
title("Random Data", "FontSize", 12)
grid on

% Top right: bar
nexttile;
bar([10, 20, 15, 25], "FaceColor", [0.5 0.8 0.5])
title("Bar Chart", "FontSize", 12)
grid on

% Bottom left: histogram
nexttile;
histogram(randn(1, 1000), 40, "FaceColor", [1 0.7 0.5])
title("Histogram", "FontSize", 12)
xlabel("Value")

% Bottom middle: loglog
nexttile;
x = logspace(0, 3, 50);
loglog(x, x.^2, "g-", "LineWidth", 2)
title("Log-Log Plot", "FontSize", 12)
grid on

% Bottom right: semilogy
nexttile;
x = 1:10;
semilogy(x, 10.^x, "m-", "LineWidth", 2)
title("Semilog Plot", "FontSize", 12)
grid on
```

---

## Advanced Axis Options

### Concept

Many engineering relationships are best examined on logarithmic scales (reaction rates, particle-size distributions, frequency responses, etc.).

### Logarithmic Axes

```matlab
semilogy(x, y)          % log scale on y-axis only
semilogx(x, y)          % log scale on x-axis only
loglog(x, y)            % log scale on both axes
```

You can also change an existing axes object:

```matlab
set(gca, 'YScale', 'log')
set(gca, 'XScale', 'log')
```

### Axis Scaling and Limits

```matlab
xlim([0 10])
ylim([1e-3 1])
axis tight              % fit axes to the data
axis equal              % same scale on both axes (for geometric plots)
grid on
grid minor
```


### Logarithmic Axes

```matlab
% Pressure drop in pipe vs. diameter (laminar flow)
D = logspace(-3, -1, 50);  % Diameter: 0.001 to 0.1 m

% Laminar: dP ∝ D^-4
dP = 1000 ./ D.^4;

figure;

% Linear axes (hard to see)
subplot(1, 2, 1)
plot(D, dP, "b-", "LineWidth", 2)
xlabel("Diameter (m)")
ylabel("Pressure Drop (Pa)")
title("Linear Axes")
grid on

% Log-log axes (clear power law)
subplot(1, 2, 2)
loglog(D, dP, "r-", "LineWidth", 2)
xlabel("Diameter (m)")
ylabel("Pressure Drop (Pa)")
title("Log-Log Axes (Shows Power Law)")
grid on
```
---
## Plot Customization

### Concept

Customization turns a quick plot into a publication-quality engineering figure. Control color, line style, marker, linewidth, font, axis limits, and annotations.

### Line Styles, Colors, Markers

**Full syntax with name-value pairs:**

```matlab
plot(x, y, 'Color', [0 0.447 0.741], 'LineStyle', '--', 'LineWidth', 2, ...
     'Marker', 'o', 'MarkerSize', 8, 'MarkerFaceColor', 'r', 'MarkerEdgeColor', 'k')
```

- `Color` — RGB vector `[R G B]` 0-1, or name `'r'`, `'blue'`, or hex `'#0072BD'`
- `LineStyle` — `'-'`, `'--'`, `':'`, `'-.'`, `'none'`
- `LineWidth` — points, 0.5 thin, 2 thick
- `Marker` — `'o'`, `'s'`, `'^'`, `'d'`, `'*'`, etc.
- `MarkerSize` — points
- `MarkerFaceColor` — fill color, `'auto'` or color
- `MarkerEdgeColor` — border color

**Color order for multiple lines:**

MATLAB cycles through 7 colors automatically. Customize:

```matlab
figure;
plot(t, C_A, 'LineWidth',2); hold on
plot(t, C_B, 'LineWidth',2);
plot(t, C_C, 'LineWidth',2);

% Force the next plot to start back at color #1
ax = gca;
ax.ColorOrderIndex = 1;
```

**Best practice for chemical engineering:**

- Use thick lines `LineWidth` 1.5-2 for visibility in reports
- Use distinct markers for experimental data, solid lines for models
- Use colorblind-friendly palette: blue, orange, yellow, purple, green, light blue
- Consistent colors: A=blue, B=red, C=green across all figures in report


### Annotations and Engineering-Style Plots

Annotations highlight operating limits, safety thresholds, design points.

```matlab
% Lines for limits
xline(400, 'r--', 'T_{max}=400 K', 'LineWidth',1.5)  % vertical line at x=400
yline(0.9, 'k:', '90% conversion', 'LineWidth',1.5)   % horizontal

% Text
text(x, y, 'string', 'FontSize',12, 'Color','r')
text(2, 0.8, 'C_{A0}=1.0 mol/L\nFirst-order', 'Interpreter','tex')

% Annotation (figure coordinates 0-1)
annotation('textbox', [0.2 0.5 0.3 0.3], 'String','Reactor Design', 'FitBoxToText','on')

% Arrow
annotation('arrow', [0.3 0.5], [0.6 0.7])

% LaTeX interpreter for equations
title('$\frac{dC_A}{dt} = -k C_A$', 'Interpreter','latex')
xlabel('$T$ (K)', 'Interpreter','latex')
```
**Engineering-style example — Operating window:**

```matlab
t = linspace(0,10,100);
T = 300 + 15*t;  % K
P = 5 + 0.2*t;   % bar

figure;
yyaxis left
plot(t, T, 'b-', 'LineWidth',2); ylabel('T (K)'); ylim([300 500])
yline(400, 'r--', 'T_{alarm}=400 K')
yyaxis right
plot(t, P, 'r--', 'LineWidth',2); ylabel('P (bar)'); ylim([4 8])
yline(7, 'k--', 'P_{alarm}=7 bar')
xlabel('Time (h)'); title('Reactor Monitoring with Alarms');
grid on; legend('T','P','Location','best')
```
---
#### Common Mistakes

- Plotting a column vector against a row vector of different length (dimension mismatch).
- Forgetting `hold on` when adding a second curve.
- Using too many different colours or marker styles, making the figure hard to read.
- `subplot` index out of range: `subplot(2,2,5)` errors — index must ≤ rows*cols
- Not creating new `figure` → previous figure overwritten

#### Key Takeaways — Basic Plots

- `plot` for continuous or densely sampled curves.
- `scatter` for discrete experimental points.
- `histogram` and `boxplot` for distributions.
- Always label axes and include units.

---

## Additional 2D Plots

### Pie Chart

Use a pie chart to show how a whole is divided among a small number of categories.

```matlab
productNames = {'Product A', 'Product B', 'Product C', 'Waste'};
productMass = [42, 28, 20, 10];  % percent of total mass

figure
pie(productMass, productNames) % Legacy pie chart
title('Batch Product Distribution')


% Pie chartSince R2023b. Recommended over pie:
data = [1 2 3];
names = ["Blueberry","Pumpkin","Lemon"];
p = piechart(data,names);
% p.LabelStyle = "namedata"; %optional
```

### Stacked Bar Chart

Use a stacked bar chart to compare totals while also showing each component.

```matlab
units = {"Reactor 1", "Reactor 2", "Separation"};
rawMaterial = [40, 35, 20];
solvent = [15, 20, 10];
water = [25, 30, 35];

figure
bar([rawMaterial; solvent; water]', "stacked")
set(gca, "XTick", 1:numel(units), "XTickLabel", units)
xlabel("Process Unit")
ylabel("Usage (kg/day)")
title("Daily Material Usage by Process Unit")
legend("Raw Material", "Solvent", "Water", "Location", "northwest")
grid on
```

### Donut Chart

MATLAB provided a separate `donut` function Since R2023b.

```matlab
Bakers = ["Betty","Abby","Afiq","Ravi","Dave"];
Sales = [20 51.55 49.37 20.35 48.25];
d = donutchart(Sales,Bakers);
```

### Polar Plot

Polar axes are useful for periodic measurements such as mixing intensity, flow direction, or angular response.

```matlab
theta = linspace(0, 2*pi, 360);
mixingIntensity = 1 + 0.25*cos(3*theta);

figure
polarplot(theta, mixingIntensity, "LineWidth", 2)
title("Periodic Mixing Intensity")
```

### Heat Map

Use `heatmap` for a color-coded table of values, such as average temperatures by unit and shift.

```matlab
units = {"Reactor 1", "Reactor 2", "Separator"};
shifts = {"Night", "Day", "Evening"};
averageTemperature = [355 362 358; 370 375 372; 340 345 343];

figure
h = heatmap(shifts, units, averageTemperature, ...
    "Colormap", parula, "ColorbarVisible", "on");
h.Title = "Average Temperature by Unit and Shift";
h.XLabel = "Shift";
h.YLabel = "Process Unit";
h.ColorLimits = [330 380];
```

---

## 3D Plots

### Surface Plot

### Concept

3D plots visualize data with three dimensions: $z(x,y)$, e.g., temperature field $T(x,y)$ in packed bed, concentration surface $C_A(t,T)$, or $VLE$ surface $P(x,y)$.

### MATLAB Syntax — 3D Plots

```matlab
% 3D line
plot3(x, y, z)  % x,y,z vectors

% Surface: need meshgrid
[X, Y] = meshgrid(x, y);  % X,Y matrices of coordinates
Z = f(X, Y);  % Z matrix same size as X,Y
surf(X, Y, Z)  % colored surface
mesh(X, Y, Z)  % wireframe
contour(X, Y, Z)  % contour lines 2D
contourf(X, Y, Z) % filled contours
pcolor(X, Y, Z)   % pseudocolor, 2D colored

% Formatting
xlabel('X'); ylabel('Y'); zlabel('Z')
colorbar; colormap jet; colormap parula (default)
view(az, el)  % view angle azimuth, elevation
shading interp; shading flat
```

**Meshgrid explained:**

```matlab
% Temperature distribution in a rectangular domain
[X, Y] = meshgrid(-5:0.5:5, -5:0.5:5);
Z = exp(-(X.^2 + Y.^2));  % Gaussian

figure;
surf(X, Y, Z)

xlabel("X Position (m)", "FontSize", 11)
ylabel("Y Position (m)", "FontSize", 11)
zlabel("Temperature (°C)", "FontSize", 11)
title("2D Temperature Distribution", "FontSize", 12)
colorbar
```

### Contour Plot

```matlab
[X, Y] = meshgrid(-5:0.2:5, -5:0.2:5);
Z = exp(-(X.^2 + Y.^2));

figure;
contourf(X, Y, Z, 20, "ShowText", "on")  % 20 contour levels

xlabel("X Position (m)", "FontSize", 11)
ylabel("Y Position (m)", "FontSize", 11)
title("Contour Plot with Values", "FontSize", 12)
colorbar;
```
### Simple Example — Temperature Field

```matlab
% 2D packed bed temperature T(x,y) = 300 + 50*exp(-(x^2+y^2)/10)
x = linspace(-5,5,50); y = linspace(-5,5,50);
[X, Y] = meshgrid(x, y);
Z = 300 + 50*exp(-(X.^2 + Y.^2)/10);  % K, hot spot at center

figure;
surf(X, Y, Z); shading interp; colorbar
xlabel('x (m)'); ylabel('y (m)'); zlabel('T (K)');
title('Packed Bed Temperature Field: Hot Spot at Center');
colormap jet; view(45,30)

% Contour view
figure;
contourf(X, Y, Z, 20); colorbar
xlabel('x (m)'); ylabel('y (m)'); title('Temperature Contours');
```

### Chemical Engineering Examples — Advanced

**1. Reaction Surface $r(T, C_A)$:**

```matlab
% Rate r = k(T)*C_A, k = A*exp(-Ea/(R*T))
A=1e7; Ea=60000; R=8.314;
T_vec = linspace(300,500,50);  % K
C_vec = linspace(0.1,1.0,50);  % mol/L
[T_mesh, C_mesh] = meshgrid(T_vec, C_vec);
k_mesh = A*exp(-Ea./(R*T_mesh));
r_mesh = k_mesh.*C_mesh;  % mol/L/s

figure;
surf(T_mesh, C_mesh, r_mesh); shading interp; colorbar
xlabel('T (K)'); ylabel('C_A (mol/L)'); zlabel('r (mol/L/s)');
title('Reaction Rate Surface: r(T,C_A) = k(T)*C_A');
```

Shows rate increases with both T and C_A — useful for reactor optimization.


**2. 3D PFR Profile — $C_A(z,t)$:**

```matlab
% PFR with first-order, analytical solution C_A(z,t) = C0*exp(-k*z/v) for steady state
% For dynamic, use mesh
z = linspace(0,10,30); t = linspace(0,5,30);
[Z, T] = meshgrid(z, t);
C0=1; k=0.2; v=1; % m/s
C = C0*exp(-k*Z/v) .* (T>Z/v);  % wave front

figure;
surf(Z, T, C); shading interp; colorbar
xlabel('z (m)'); ylabel('t (min)'); zlabel('C_A (mol/L)');
title('PFR Dynamic: C_A(z,t) Wave Front');
view(45,30)
```

### Mesh 3D Plot

`mesh` shows the surface as a wireframe, which makes the underlying grid and shape easy to inspect.

```matlab
[X, Y] = meshgrid(linspace(-4, 4, 35));
Z = sin(sqrt(X.^2 + Y.^2)) ./ (sqrt(X.^2 + Y.^2) + 0.5);

figure
mesh(X, Y, Z)
xlabel("X"); ylabel("Y"); zlabel("Z")
title("3D Wireframe of a Response Surface")
grid on
```

### 3D Bar Chart

`bar3` compares a matrix of values across two categorical dimensions.

```matlab
conversion = [72 78 81; 65 71 76; 58 64 70];

figure
bar3(conversion)
set(gca, "XTick", 1:3, "XTickLabel", {"300 K", "325 K", "350 K"})
set(gca, "YTick", 1:3, "YTickLabel", {"CSTR", "PFR", "Batch"})
xlabel("Temperature")
ylabel("Reactor Type")
zlabel("Conversion (%)")
title("Conversion by Reactor Type and Temperature")
```

### 3D Scatter Plot

Use `scatter3` when each observation has three measured coordinates. Marker color can represent a fourth variable.

```matlab
temperature = 300 + 100*rand(60, 1);  % K
concentration = 0.1 + 0.9*rand(60, 1); % mol/L
pressure = 2 + 4*rand(60, 1);          % bar
conversion = 100*(1 - exp(-0.02*temperature.*concentration));

figure
scatter3(temperature, concentration, pressure, 45, conversion, "filled")
xlabel("Temperature (K)")
ylabel("C_A (mol/L)")
zlabel("Pressure (bar)")
title("Operating Data in 3D")
colorbar.Label.String = "Conversion (%)";
grid on
view(45, 25)
```

### 3D Pie Chart

`pie3` gives a pie chart with an extruded 3D appearance. Use it only when the extra perspective is useful; it is less precise than a standard pie chart for comparisons.

```matlab
componentMass = [45, 30, 15, 10];
componentNames = {"A", "B", "C", "Byproducts"};

figure
pie3(componentMass)
legend(componentNames, "Location", "eastoutside")
title("3D Composition of Product Stream")
```

---

## Common Mistakes

### Mistake 1: Poor Label Choices

```matlab
% WRONG
plot(x, y)
xlabel("x")
ylabel("y")

% CORRECT
plot(T, conversion*100)
xlabel("Temperature (K)", "FontSize", 12)
ylabel("Conversion (%)", "FontSize", 12)
```

### Mistake 2: Not Using Grid and Legend

```matlab
% WRONG (hard to read)
plot(x, y1, x, y2)

% CORRECT
plot(x, y1, "DisplayName", "Model 1")
hold on
plot(x, y2, "DisplayName", "Model 2")
legend
grid on
```

### Mistake 3: Inconsistent Scales

```matlab
% Multiple plots with different y-axis scales (confusing)
% BETTER: Use yyaxis for secondary axis or normalize data
figure
yyaxis left
plot(t, T)
ylabel("Temperature (K)")
yyaxis right
plot(t, P)
ylabel("Pressure (bar)")
```

---
## Worked Example

**Scenario:** A shell-and-tube heat exchanger is being analyzed. You have temperature measurements for the hot and cold streams along the exchanger length, and separately, pressure-drop data across the shell side at several flow rates. Produce a single, two-panel figure suitable for a report: the temperature profile on top, and the pressure-drop behavior (on log-log axes, since it follows a power law) on the bottom.

```matlab
% --- Data ---
L = [0 1 2 3 4 5];                    % position along exchanger, m
T_hot  = [420 405 392 380 370 362];   % K
T_cold = [300 315 328 340 349 356];   % K

Q  = [2 4 8 16 32];                   % shell-side flow rate, m3/h
dP = [3.1 10.8 38.5 140.2 505.0];     % pressure drop, kPa

% --- Figure with two panels ---
figure

subplot(2,1,1)
plot(L, T_hot, 'r-o', L, T_cold, 'b-s')
xlabel('Position Along Exchanger (m)')
ylabel('Temperature (K)')
title('Heat Exchanger Temperature Profile')
legend('Hot Stream', 'Cold Stream', 'Location', 'east')
grid on

subplot(2,1,2)
loglog(Q, dP, 'ko-')
xlabel('Shell-Side Flow Rate (m^3/h)')
ylabel('Pressure Drop (kPa)')
title('Shell-Side Pressure Drop vs. Flow Rate')
grid on

exportgraphics(gcf, 'heat_exchanger_report_figure.png', 'Resolution', 300)
```

## Worked Example: Batch Reactor Analysis

```matlab
% Simulate batch reactor and create comprehensive visualization
clear; clc; close;

% Reaction: A -> B, first-order kinetics
k = 0.05;           % Rate constant (1/min)
C_A0 = 2.0;         % Initial concentration of A (mol/L)
t = 0:1:100;        % Time (minutes)

% Analytical solutions
C_A = C_A0 * exp(-k * t);
C_B = C_A0 * (1 - exp(-k * t));
X = 1 - exp(-k * t);  % Conversion

% Create figure with multiple subplots
figure;

% Plot 1: Concentrations
subplot(2, 3, 1)
plot(t, C_A, "b-", "LineWidth", 2.5, "DisplayName", "A")
hold on
plot(t, C_B, "r-", "LineWidth", 2.5, "DisplayName", "B")
xlabel("Time (min)", "FontSize", 11)
ylabel("Concentration (mol/L)", "FontSize", 11)
title("Species Concentration Profiles", "FontSize", 12)
legend("FontSize", 10)
grid on

% Plot 2: Conversion
subplot(2, 3, 2)
plot(t, X*100, "g-", "LineWidth", 2.5)
xlabel("Time (min)", "FontSize", 11)
ylabel("Conversion (%)", "FontSize", 11)
title("Reactant Conversion", "FontSize", 12)
grid on

% Plot 3: Semi-log (for rate constant determination)
subplot(2, 3, 3)
semilogy(t, C_A, "b-", "LineWidth", 2.5)
xlabel("Time (min)", "FontSize", 11)
ylabel("C_A (mol/L)", "FontSize", 11)
title("Semi-Log Plot (Determines k)", "FontSize", 12)
grid on

% Plot 4: Rate of reaction
r_A = k * C_A;  % Rate of A consumption
subplot(2, 3, 4)
plot(t, r_A, "m-", "LineWidth", 2.5)
xlabel("Time (min)", "FontSize", 11)
ylabel("Reaction Rate (mol/L·min)", "FontSize", 11)
title("Reaction Rate vs. Time", "FontSize", 12)
grid on

% Plot 5: Phase plane (C_A vs. C_B)
subplot(2, 3, 5)
plot(C_A, C_B, "b-", "LineWidth", 2.5)
hold on
plot(C_A(1), C_B(1), "go", "MarkerSize", 10, "DisplayName", "Start")
plot(C_A(end), C_B(end), "ro", "MarkerSize", 10, "DisplayName", "End")
xlabel("C_A (mol/L)", "FontSize", 11)
ylabel("C_B (mol/L)", "FontSize", 11)
title("Phase Plane Trajectory", "FontSize", 12)
legend("FontSize", 10)
grid on

% Plot 6: Yield (assuming stoichiometry A -> B)
Y = C_B / C_A0 * 100;  % Percent yield
subplot(2, 3, 6)
plot(t, Y, "c-", "LineWidth", 2.5)
xlabel("Time (min)", "FontSize", 11)
ylabel("Yield (%)", "FontSize", 11)
title("Product B Yield", "FontSize", 12)
grid on

% Summary text box
fprintf("===== BATCH REACTOR ANALYSIS =====\n")
fprintf("Rate constant: %.4f min^-1\n", k)
fprintf("Time for 50%% conversion: %.2f min\n", log(2)/k)
fprintf("Final conversion: %.1f%%\n", X(end)*100)
fprintf("Final yield of B: %.2f mol/L (%.1f%%)\n", C_B(end), Y(end))
```

---

## Export and Publication Quality

```matlab
% Save figure in diffrent formats
saveas(gcf, "my_figure.jpeg")  
saveas(gcf, "my_figure.png")
saveas(gcf, "my_figure.pdf")   
```
Alternatively, use the graphical “Export” menu or `exportgraphics` (recent MATLAB):

```matlab
exportgraphics(gcf, 'myFigure.png', 'Resolution', 300)

% If you want the export folder to be relative to your MATLAB script/current project, use fullfile:

folder = fullfile("images");
exportgraphics(gcf, fullfile(folder, "my_figure.png"));
```
---

## Worked Example — Complete Reactor Data Visualization

Combine 2D, subplot, customization, and 3D.

```matlab
% reactor_visualization.m

clear; clc;

% --- Synthetic batch data ---
t = linspace(0,10,50); % h
C_A = exp(-0.3*t);  % mol/L first-order
T = 300 + 20*t + 5*sin(t);  % K, ramp with noise
k = 1e7*exp(-60000./(8.314*T));  % 1/s
r = k.*C_A;  % mol/L/s

% --- Figure 1: 2x2 subplot with customization ---
figure('Position',[100 100 800 600]);

subplot(2,2,1)
plot(t, C_A, 'b-o', 'LineWidth',1.5, 'MarkerSize',4, 'MarkerIndices',1:5:length(t))
xlabel('Time (h)'); ylabel('C_A (mol/L)'); title('Concentration vs Time');
grid on; xlim([0 10]); ylim([0 1.1])

subplot(2,2,2)
plot(t, T, 'r-', 'LineWidth',2)
xlabel('Time (h)'); ylabel('T (K)'); title('Temperature Profile');
grid on; yline(350, 'k--', 'T_{ref}'); ylim([300 550])

subplot(2,2,3)
semilogy(t, k, 'g--s', 'LineWidth',1.5)
xlabel('Time (h)'); ylabel('k (1/s)'); title('Rate Constant (log scale)');
grid on

subplot(2,2,4)
plot(t, r, 'm-', 'LineWidth',2)
xlabel('Time (h)'); ylabel('r (mol/L/s)'); title('Reaction Rate');
grid on

sgtitle('Batch Reactor Data Overview', 'FontWeight','bold', 'FontSize',14)

% Save
exportgraphics(gcf, 'batch_overview.png', 'Resolution',300)

% --- Figure 2: Arrhenius plot with fit ---
figure;
invT = 1./T;
lnk = log(k);
scatter(invT, lnk, 50, t, 'filled'); colorbar; colormap jet
xlabel('1/T (K^{-1})'); ylabel('ln(k)'); title('Arrhenius Plot Colored by Time');
grid on; hold on
p = polyfit(invT, lnk, 1);
plot(invT, polyval(p, invT), 'k--', 'LineWidth',1.5)
legend('Data (color=time)','Linear fit','Location','best')
text(0.0022, -5, sprintf('E_a = %.0f J/mol', -p(1)*8.314))

% --- Figure 3: 3D surface r(T,C_A) ---
T_grid = linspace(300,500,40);
C_grid = linspace(0,1,40);
[Tm, Cm] = meshgrid(T_grid, C_grid);
km = 1e7*exp(-60000./(8.314*Tm));
rm = km.*Cm;

figure;
surf(Tm, Cm, rm, 'EdgeColor','none'); shading interp; colorbar
xlabel('T (K)'); ylabel('C_A (mol/L)'); zlabel('r (mol/L/s)');
title('Reaction Rate Surface r(T,C_A)');
colormap parula; view(45,30)

% Contour projection
figure;
contourf(Tm, Cm, rm, 20); colorbar
xlabel('T (K)'); ylabel('C_A (mol/L)'); title('r(T,C_A) Contours');
```

This example shows how to turn raw vectors into a multi-panel report, Arrhenius analysis, and 3D surface — all essential for lab reports.

---

## Chemical Engineering Application — Process Monitoring Dashboard

Create a monitoring dashboard with alarms, dual y-axes, and histogram.

```matlab
% monitoring_dashboard.m

clear; clc;

% Simulate historian data 24 hours, 5 min interval
t_hours = (0:5/60:24)';  % h, column 289 points
T = 350 + 10*sin(2*pi*t_hours/24) + 2*randn(size(t_hours)); % K
P = 5 + 0.3*randn(size(t_hours)) + 0.5*(T-350)/10; % bar correlated with T
F = 100 + 5*randn(size(t_hours)); % mol/s

% Inject upset at 14:00-15:00
upsetIdx = (t_hours>=14) & (t_hours<=15);
T(upsetIdx) = T(upsetIdx) + 30;
P(upsetIdx) = P(upsetIdx) + 2;

% --- Dashboard figure ---
figure('Position',[100 100 1000 600]);
tiledlayout(2,2);

% Panel 1: T and P vs time with alarms
nexttile
yyaxis left
plot(t_hours, T, 'b-', 'LineWidth',1); ylabel('T (K)'); ylim([320 400])
yline(380, 'r--', 'T alarm 380 K', 'LineWidth',1.2)
yyaxis right
plot(t_hours, P, 'r--', 'LineWidth',1); ylabel('P (bar)'); ylim([3 8])
yline(7, 'k--', 'P alarm 7 bar')
xlabel('Time (h)'); title('Reactor T & P Monitoring'); grid on
legend('T','P','Location','best')

% Panel 2: Flow vs time
nexttile
plot(t_hours, F, 'g-', 'LineWidth',1); ylabel('F (mol/s)'); xlabel('Time (h)');
title('Feed Flow'); grid on; ylim([80 120])
yline(90, 'r--', 'F low alarm')

% Panel 3: Scatter T vs P colored by time
nexttile
scatter(T, P, 20, t_hours, 'filled'); colorbar; colormap jet
xlabel('T (K)'); ylabel('P (bar)'); title('T vs P Colored by Time');
grid on; c = colorbar; c.Label.String = 'Time (h)';

% Panel 4: Histogram of T
nexttile
histogram(T, 20, 'FaceColor','b', 'EdgeColor','k'); xlabel('T (K)'); ylabel('Count');
title('Temperature Distribution'); grid on
xline(mean(T), 'r--', sprintf('Mean %.1f K', mean(T)))

sgtitle('Process Monitoring Dashboard - 24h', 'FontWeight','bold')

exportgraphics(gcf, 'dashboard.png', 'Resolution',300)

% Additional: 3D contour of operating envelope
figure;
% Create 2D histogram or contour of T-P density
% Using histogram2
histogram2(T, P, 20, 'DisplayStyle','tile', 'ShowEmptyBins','on'); colorbar
xlabel('T (K)'); ylabel('P (bar)'); title('T-P Operating Density');
```

Dashboard combines all 2D techniques: dual y-axis, alarms with `yline`, scatter colored by time, histogram with mean line — exactly what DCS engineers view.


---

### Important MATLAB Plotting

| Purpose | Syntax | Example |
|---|---|---|
| Line plot | `plot(x, y)` | `plot(t, T, "b-", "LineWidth", 2)` |
| Scatter | `scatter(x, y)` | `scatter(Q, dP, 100, "b", "filled")` |
| Bar chart | `bar(data)` | `bar(production, 0.7)` |
| Histogram | `histogram(data)` | `histogram(x, 50)` |
| Log scales | `loglog(x, y)` | `loglog(D, dP)` |
| Subplots | `subplot(r, c, i)` | `subplot(2, 2, 1)` |
| Labels | `xlabel, ylabel` | `xlabel("Time (s)", "FontSize", 12)` |
| Legend | `legend()` | `legend("A", "B", "FontSize", 11)` |
| 3D surface | `surf(X, Y, Z)` | `surf(X, Y, Z)` |
| Contour | `contourf(X, Y, Z)` | `contourf(X, Y, Z, 20)` |

---

## Homework

### Problem

A pilot plant tested a new heat exchanger design. Two datasets were collected:

1. **Temperature profile** along the exchanger length for both the hot and cold streams.
2. **Pressure drop** across the shell side at five different flow rates (expected to follow a power-law trend).

```matlab
L = [0 1 2 3 4 5 6];                          % position, m
T_hot  = [450 430 414 400 388 378 370];       % K
T_cold = [310 322 333 343 351 358 364];       % K

Q  = [3 6 12 24 48];                          % flow rate, m3/h
dP = [2.0 7.2 26.5 97.0 355.0];               % pressure drop, kPa
```

### Required Tasks

1. Create a figure with **two subplots**:
   - Top panel: hot and cold stream temperatures vs. position, on linear axes, fully labeled with a legend distinguishing the two streams.
   - Bottom panel: pressure drop vs. flow rate, on the appropriate logarithmic axes so that a power-law relationship would appear as a straight line.
2. Add a title to each panel, and axis labels that include units.
3. Add gridlines to both panels.
4. Export the completed figure as `hx_pilot_report.png` at 300 dpi.

### Concepts Being Tested

- `subplot` for multi-panel figures
- `hold on` and multi-series `plot` calls with a `legend`
- Choosing between linear and logarithmic axes appropriately
- Full plot labeling (title, axis labels with units, legend, grid)
- Exporting a figure with `exportgraphics`

### Hints

- Both `T_hot` and `T_cold` change roughly linearly with `L`, so linear axes are appropriate for the top panel.
- Both `Q` and `dP` span more than one order of magnitude, and you are checking for a power-law trend — which single MATLAB function plots *both* axes on a log scale?

<details>
<summary><strong>Detailed Solution — Click to Reveal</strong></summary>

## Solution

### Concept

This problem combines three ideas from this lecture: multi-panel figures (`subplot`), choosing the correct axis scale for the physical relationship being examined, and complete, report-ready plot formatting.

### Approach / Idea

1. Open a new figure and use `subplot(2,1,1)` for the temperature profile, `subplot(2,1,2)` for the pressure-drop trend.
2. For the temperature profile, since neither variable spans orders of magnitude, use a standard linear `plot` with `hold on` (or a single `plot` call with both series) and a `legend`.
3. For the pressure-drop trend, since both `Q` and `dP` span more than one order of magnitude and a power law is expected, use `loglog` — a power law appears as a straight line on log-log axes, which is exactly what makes this plot useful for checking the relationship.
4. Finish both panels with titles, labeled axes (with units), and `grid on`.
5. Export the finished figure with `exportgraphics`, specifying the filename and resolution.

### Syntax

```matlab
subplot(rows, cols, index)
plot(x, y1, x, y2)
legend('Series 1', 'Series 2', 'Location', 'best')
loglog(x, y)
exportgraphics(gcf, 'filename.png', 'Resolution', 300)
```

### Syntax Breakdown

- `subplot(2,1,1)` — a grid with 2 rows and 1 column; index 1 selects the top panel.
- `subplot(2,1,2)` — the same grid; index 2 selects the bottom panel.
- `loglog(Q, dP)` — plots `dP` against `Q` with both axes on a logarithmic scale, so that `dP = a*Q^b` appears as a straight line.

### MATLAB Code

```matlab
% --- Data ---
L = [0 1 2 3 4 5 6];
T_hot  = [450 430 414 400 388 378 370];
T_cold = [310 322 333 343 351 358 364];

Q  = [3 6 12 24 48];
dP = [2.0 7.2 26.5 97.0 355.0];

% --- Figure ---
figure

subplot(2,1,1)
plot(L, T_hot, 'r-o', L, T_cold, 'b-s')
xlabel('Position Along Exchanger (m)')
ylabel('Temperature (K)')
title('Pilot Heat Exchanger: Temperature Profile')
legend('Hot Stream', 'Cold Stream', 'Location', 'east')
grid on

subplot(2,1,2)
loglog(Q, dP, 'ko-')
xlabel('Shell-Side Flow Rate (m^3/h)')
ylabel('Pressure Drop (kPa)')
title('Pilot Heat Exchanger: Pressure Drop vs. Flow Rate')
grid on

exportgraphics(gcf, 'hx_pilot_report.png', 'Resolution', 300)
```

### Line-by-Line Explanation

- `figure` opens a new, blank figure window so the two panels are not drawn on top of an old plot.
- `subplot(2,1,1)` activates the top half of a 2-row, 1-column layout.
- `plot(L, T_hot, 'r-o', L, T_cold, 'b-s')` draws both temperature curves in one call, with distinct colors and markers so they remain distinguishable even in black-and-white printing.
- `legend(..., 'Location', 'east')` places the legend on the right side of the panel, away from the data.
- `subplot(2,1,2)` switches to the bottom panel.
- `loglog(Q, dP, 'ko-')` plots the pressure-drop data with both axes logarithmic.
- `exportgraphics(gcf, 'hx_pilot_report.png', 'Resolution', 300)` saves everything currently in the figure window — both panels — to a single PNG file at print resolution.

### Expected Result

The top panel shows two roughly linear, converging trends: the hot stream temperature decreasing from 450 K to 370 K, and the cold stream increasing from 310 K to 364 K, with a clear legend distinguishing the two. The bottom panel shows five points that lie close to a straight line on the log-log axes, consistent with a power-law relationship between pressure drop and flow rate. A file named `hx_pilot_report.png` is created in the current MATLAB folder.

### Engineering Interpretation

The near-linear temperature profiles indicate a fairly uniform heat-transfer rate along the exchanger length, with no abrupt phase changes or fouling "hot spots." The straight-line behavior on the log-log pressure-drop plot confirms that the shell side is operating in the turbulent regime, where pressure drop scales as a power of flow rate (typically an exponent between about 1.7 and 2 for turbulent flow) — useful confirmation before sizing a pump or blower for this exchanger at a different design flow rate.

</details>

---

