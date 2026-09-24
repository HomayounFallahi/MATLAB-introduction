# File Handling and Input/Output

## Learning Objectives

By the end of this lecture, you should be able to:

- Understand file paths, directories, and how MATLAB manages file locations
- Use `load`, `save`, `importdata` for MATLAB native MAT-files and simple text data
- Read and write text files with `fopen`, `fclose`, `fscanf`, `fprintf`, `fgetl`
- Import and export Excel and CSV files with `readtable`, `writetable`, `readmatrix`, `writematrix`, `readcell`
- Choose the appropriate import function for chemical engineering data: lab notebooks, DCS historian exports, Aspen results
- Organize experimental and process data into typed tables, clean missing values, outliers, and inconsistent units
- Implement basic data cleaning: `ismissing`, `fillmissing`, `rmmissing`, `standardizeMissing`, logical filtering
- Save processed data and generate reports for reuse in later numerical methods and reactor models
- Avoid common pitfalls: path issues, delimiter mismatches, header handling, encoding
- Navigate within files using file pointers (`ftell`, `fseek`)
- Detect end-of-file conditions with `feof`
- Handle file errors using `ferror`


## Why This Topic Matters

As a chemical engineer, you will frequently need to:

- **Read experimental data** from instruments, sensors, and lab reports
- **Save calculated results** for reports, presentations, and further analysis
- **Exchange data** with colleagues, other programs, and databases
- **Log process data** from simulations or pilot plants
- **Manage datasets** from multiple experiments or process runs

MATLAB's file handling capabilities allow you to automate data import, process large experimental datasets, and export results in formats compatible with Excel, Python, and other tools. Without file I/O, you would be limited to manually entering data and manually copying results—both time-consuming and error-prone in engineering practice.

---

## File Management

### Concept

**File management** involves understanding where files are stored, how to specify file locations (paths), and how to open and close files in MATLAB.

MATLAB works with:
- **Current working directory** — the folder where MATLAB looks for files by default
- **Absolute paths** — complete file locations from the root directory
- **Relative paths** — locations relative to the current working directory

### File Paths and Directories

#### Understanding Paths

**Absolute path** (complete location):
```matlab
'/home/user/data/temp_log.txt'          % Linux/macOS
'C:\Users\user\data\temp_log.txt'       % Windows
```

**Relative path** (relative to current directory):
```matlab
'data/temp_log.txt'                     % From current directory
'../data/temp_log.txt'                  % Go up one level
```

#### Important MATLAB Functions for File Management

| Function | Purpose |
|---|---|
| `pwd` | Print working directory (show current folder) |
| `cd(path)` | Change directory |
| `mkdir(dirname)` | Create a new directory |
| `dir(path)` | List files in a directory |
| `exist(filename, filetype)` | Check if a file/variable exists |

#### File Paths and Directories

```matlab
pwd                         % current working directory
cd('C:\Data\ReactorLab')    % change directory (Windows example)
cd('/home/user/data')       % Linux / macOS example

ls                          % list files (or dir)
mkdir('results')            % create a folder
rmdir('results')            % remove an empty folder
```

### MATLAB Syntax

#### Check Current Directory
```matlab
pwd
```

Returns the current working directory as text.

#### Change Directory
```matlab
cd('/home/user/matlab_projects')
```

Changes MATLAB's working directory to the specified folder.

#### List Files
```matlab
dir
dir('data')
dir('*.txt')
```

Lists files in the current directory, in a specific folder, or matching a pattern.

#### Check File Existence
```matlab
exist('sample_data\temp_log.txt', 'file')
```

Returns 2 if the file exists, 0 if it does not.

### Syntax Breakdown

```matlab
status = exist('sample_data\temp_log.txt', 'file');
```

- `status` — Output: 2 if file exists, 0 if not, or other values for variables/directories
- `exist` — MATLAB function
- `'sample_data\temp_log.txt'` — Filename to check (can include path)
- `'file'` — Query type (we specify 'file' to check for files specifically)

### Simple Example: File Management

```matlab
% Check current working directory
current_dir = pwd;
disp(['Current directory: ' current_dir])

% Create a new directory for our data
mkdir('experimental_data')

% Change to that directory
cd('experimental_data')

% Check if a file exists
if exist('reactor_output.txt', 'file') == 2
    disp('File found!')
else
    disp('File not found. This will be our first run.')
end

% Return to the original directory
cd('..')
```

### Example Walkthrough

1. `pwd` — Shows something like `/home/user/MATLAB`
2. `mkdir('experimental_data')` — Creates a new folder called `experimental_data`
3. `cd('experimental_data')` — Moves into that folder; now `pwd` shows `/home/user/MATLAB/experimental_data`
4. `exist('reactor_output.txt', 'file')` — Returns 0 because we haven't created the file yet
5. `cd('..')` — Moves back up to the parent directory

### Chemical Engineering Context

In a chemical engineering lab, you might organize your MATLAB work like this:

```
MATLAB_Projects/
  ├── Experiment_1/
  │   ├── raw_data.txt
  │   └── analysis.m
  ├── Experiment_2/
  │   ├── raw_data.txt
  │   └── analysis.m
  └── process_simulation/
      ├── reactor_model.m
      └── results/
```

Each experiment or process has its own folder, keeping data organized.

---

## Opening and Closing Files

### Concept

Before reading from or writing to a file, you must **open** it, which creates a connection between MATLAB and the file on disk. This connection is represented by a **file identifier** (or file handle). After you are finished, you must **close** the file to release the connection.

### MATLAB Syntax

#### Open a File
```matlab
fid = fopen(filename, permission);
```

Returns a file identifier `fid` (usually a positive integer).

#### Close a File
```matlab
fclose(fid);
```

Closes the file associated with file identifier `fid`.

### Syntax Breakdown

```matlab
fid = fopen('sample_data\temp_log.txt', 'r');
```

- `fid` — File identifier (a number like 3, used to reference this file in later operations)
- `fopen` — MATLAB function to open a file
- `'sample_data\temp_log.txt'` — Filename (can include path)
- `'r'` — Permission mode:
  - `'r'` — Read (file must exist)
  - `'w'` — Write (creates file if it doesn't exist; overwrites if it does)
  - `'a'` — Append (add to the end of the file)

**File Permissions Table:**

| Mode | Purpose | Overwrites? |
|---|---|---|
| `'r'` | Read from file | N/A (file must exist) |
| `'w'` | Write to file | Yes |
| `'a'` | Append to file | No (adds to end) |
| `'r+'` | Read and write | No |
| `'w+'` | Write and read | Yes |

### Simple Example: Opening and Closing Files

```matlab
% Open a file for reading
fid_in = fopen('sample_data\temp_log.txt', 'r');

% Check if file opened successfully
if fid_in == -1
    error('Could not open file for reading')
end

% Read some data (we'll cover fscanf next)
data = fscanf(fid_in, '%f');

% Always close the file when done
fclose(fid_in);

disp('File closed successfully')
```

### Example Walkthrough

1. `fopen('sample_data\temp_log.txt', 'r')` — Attempts to open the file for reading
2. If successful, `fid_in` contains a positive number (e.g., 3)
3. If unsuccessful, `fid_in` equals -1 (file not found, cannot be read, etc.)
4. We check `if fid_in == -1` to catch errors
5. After we finish reading, `fclose(fid_in)` closes the file

### Chemical Engineering Example: Logging Process Data

Imagine you're logging temperature data from a chemical reactor over time. You might:

1. Open a file for appending (adding new data)
2. Take temperature measurements every second
3. Write each measurement to the file
4. After the experiment, close the file

```matlab
% Create/open a file for appending
fid = fopen('sample_data\process_log.txt', 'a');

% Simulate 10 seconds of temperature data
for t = 1:10
    T = 350 + 5*sin(t/2);  % Simulated temperature oscillation (K)
    fprintf(fid, 'Time: %d s, Temperature: %.2f K\n', t, T);
    pause(1);  % Wait 1 second
end

% Close the file
fclose(fid);

disp('Reactor logging complete')
```

### Common Mistakes

**Mistake 1:** Not checking if the file opened successfully
```matlab
% WRONG — will crash if file doesn't open
fid = fopen('data.txt', 'r');
data = fscanf(fid, '%f');  % fid might be -1!

% CORRECT
fid = fopen('data.txt', 'r');
if fid == -1
    error('Failed to open file')
end
data = fscanf(fid, '%f');
fclose(fid);
```

**Mistake 2:** Forgetting to close files
```matlab
% WRONG — leaves file open, preventing other programs from accessing it
fid = fopen('data.txt', 'r');
data = fscanf(fid, '%f');
% No fclose!  File remains open.

% CORRECT
fid = fopen('data.txt', 'r');
data = fscanf(fid, '%f');
fclose(fid);  % Always close
```

**Mistake 3:** Using the wrong file mode
```matlab
% WRONG — file doesn't exist, and 'r' mode requires it to exist
fid = fopen('new_data.txt', 'r');  % Will return -1

% CORRECT — use 'w' to create a new file
fid = fopen('new_data.txt', 'w');
fprintf(fid, 'Data\n');
fclose(fid);
```

### Key Takeaways

- Always check `if fid == -1` after opening a file
- Use `'r'` to read from an existing file
- Use `'w'` to create a new file or overwrite an existing one
- Use `'a'` to append data to the end of an existing file
- Always call `fclose(fid)` when you're finished with a file
- Forgetting to close files can cause problems for other programs trying to access them

---

## Reading and Writing Data

### Concept

Once a file is open, you can **read data from it** (if opened in read mode) or **write data to it** (if opened in write or append mode). The most common formatted I/O functions are `fprintf` (write) and `fscanf` (read).

### Writing Data with fprintf

#### MATLAB Syntax

```matlab
fprintf(fid, format, variables);
```

Writes formatted data to a file.

#### Syntax Breakdown

```matlab
fprintf(fid, 'Temperature: %.2f K, Pressure: %.1f bar\n', T, P);
```

- `fid` — File identifier (from `fopen`)
- `'Temperature: %.2f K, Pressure: %.1f bar\n'` — Format string:
  - `%.2f` — Float with 2 decimal places
  - `%.1f` — Float with 1 decimal place
  - `\n` — Newline character
  - `%d` — Integer
  - `%s` — String
- `T, P` — Variables to write (match the format specifiers)

#### Format Specifiers Table

| Specifier | Type | Example |
|---|---|---|
| `%d` | Integer | `%d` → `42` |
| `%f` | Floating-point | `%f` → `3.141593` |
| `%.2f` | Float, 2 decimals | `%.2f` → `3.14` |
| `%e` | Scientific notation | `%e` → `3.141593e+00` |
| `%s` | String | `%s` → `reactor` |
| `\n` | Newline | Starts a new line |
| `\t` | Tab | Horizontal spacing |

### Simple Example: Writing to a File

```matlab
% Open file for writing
fid = fopen('sample_data\experiment_results.txt', 'w');

% Write header
fprintf(fid, 'Experiment Results\n');
fprintf(fid, '==================\n\n');

% Write data
T = 350;      % K
P = 5;        % bar
F = 2.5;      % mol/s
X = 0.65;     % Conversion (fraction)

fprintf(fid, 'Temperature: %.1f K\n', T);
fprintf(fid, 'Pressure: %.2f bar\n', P);
fprintf(fid, 'Feed Rate: %.3f mol/s\n', F);
fprintf(fid, 'Conversion: %.1f %%\n', X*100);  % %% prints a single %

% Close file
fclose(fid);

disp('Results written to file')
```

### Reading Data with fscanf

#### MATLAB Syntax

```matlab
data = fscanf(fid, format);
```

Reads formatted data from a file and returns it as an array.

#### Syntax Breakdown

```matlab
temperatures = fscanf(fid, '%f');
```

- `temperatures` — Output array (contains the data read)
- `fscanf` — MATLAB function for formatted read
- `fid` — File identifier
- `'%f'` — Format specifier (read floating-point numbers)

### Simple Example: Reading from a File

Suppose `temp_log.txt` contains:

```
350.5
351.2
349.8
352.1
350.9
```

We can read it:

```matlab
% Open file for reading
fid = fopen('sample_data\temp_log.txt', 'r');

% Read all floating-point numbers
temperatures = fscanf(fid, '%f');

% Close file
fclose(fid);

% Display results
disp('Temperatures (K):')
disp(temperatures)

% Calculate statistics
avg_T = mean(temperatures);
fprintf('Average temperature: %.2f K\n', avg_T)
```

### Reading Data with Format Control

For more complex file structures, you can control the format more precisely:

```matlab
% Open file for reading
fid = fopen('sample_data\temp_log.txt', 'r');

% Read multiple columns: time (integer) and temperature (float)
data = fscanf(fid, '%d %f', [2, Inf]);
% [2, Inf] means: 2 rows, read until end of file

% Transpose to get rows of data
data = data';  % Now each row is: time, temperature

% Close file
fclose(fid);

% Extract columns
time = data(:, 1);
temperature = data(:, 2);
```

### Example Walkthrough: Two-Column Data

If `process_log.txt` contains:

```
1 350.5
2 351.2
3 349.8
4 352.1
5 350.9
```

```matlab
fid = fopen('sample_data\process_log.txt', 'r');
data = fscanf(fid, '%d %f', [2, Inf]);
fclose(fid);

% data is now a 2×5 matrix:
% [1   2   3   4   5 ]
% [350.5 351.2 349.8 352.1 350.9]

% Transpose to get 5 rows × 2 columns
data = data';

% Extract columns
time = data(:, 1);          % [1; 2; 3; 4; 5]
temperature = data(:, 2);  % [350.5; 351.2; 349.8; 352.1; 350.9]
```

### CSV Files (Comma-Separated Values)

CSV is a common format for storing tabular data. MATLAB can read and write CSV easily.

Modern MATLAB high-level functions:

- `readtable` — reads Excel/CSV into table, preserves column names and mixed types, best for lab data
- `writetable` — writes table to Excel/CSV
- `readmatrix` — reads numeric matrix (ignores text or converts)
- `writematrix` — writes matrix
- `readcell` — reads as cell array, mixed types
- `readtimetable` — for time-stamped data

```matlab
% Simulate CSV file: lab_data.csv
% Time_h,T_K,C_A_mol_L,Valid
% 0,320,1.0,true
% 0.5,340,0.8,true
% ...

% Read CSV into table
T = readtable('lab_data.csv');  

% Read Excel
% T_excel = readtable('plant_data.xlsx', 'Sheet', 'Run1');
% T_excel = readtable('plant_data.xlsx', 'Range', 'A1:D100');

% Write table to Excel
writetable(T, 'cleaned_data.xlsx')
writetable(T, 'cleaned_data.csv')

% Read numeric matrix only
M = readmatrix('numeric_data.csv')
writematrix(M, 'output_matrix.csv')

% Read as cell for raw mixed
C = readcell('mixed_data.csv')
```

**Import options:** For robust imports, use `detectImportOptions`.

```matlab
opts = detectImportOptions('lab_data.csv');
opts.Delimiter = ',';
opts.HeaderLines = 1;
opts.VariableNames = {'Time_h','T_K','C_A','IsValid'};
opts.VariableTypes = {'double','double','double','logical'};
opts = setvaropts(opts, 'IsValid', 'Type', 'logical');
data = readtable('lab_data.csv', opts);
```

#### Writing CSV

```matlab
% Sample data
time = [1; 2; 3; 4; 5];
temperature = [350.5; 351.2; 349.8; 352.1; 350.9];
pressure = [5.0; 5.1; 4.9; 5.2; 5.0];

% Open file for writing
fid = fopen('sample_data\temp_log.csv', 'w');

% Write header
fprintf(fid, 'Time (s),Temperature (K),Pressure (bar)\n');

% Write data rows
for i = 1:length(time)
    fprintf(fid, '%d,%.1f,%.2f\n', time(i), temperature(i), pressure(i));
end

% Close file
fclose(fid);
```

This creates a file that looks like:

```
Time (s),Temperature (K),Pressure (bar)
1,350.5,5.00
2,351.2,5.10
3,349.8,4.90
4,352.1,5.20
5,350.9,5.00
```

#### Reading CSV

MATLAB provides the `readtable` function for easy CSV reading:

```matlab
% Simple way: use readtable (recommended for CSV)
data_table = readtable('sample_data\temp_log.csv');

% Access columns by name
time = data_table.Time_s;
temperature = data_table.Temperature_K;
pressure = data_table.Pressure_bar;

% warning('off', 'MATLAB:table:ModifiedAndSavedVariableNames');
% or: 
% data = readtable('sample_data\temp_log.csv', 'VariableNamingRule', 'preserve');
```
This warning occurs because MATLAB's `readtable` automatically changes column headers that contain spaces, special characters, or numbers at the start so they become valid MATLAB variable names (e.g., changing "Age (years)" to "Age_years_").

Or, using lower-level I/O:

```matlab
% Alternative: use fprintf/fscanf
fid = fopen('sample_data\temp_log.csv', 'r');

% Skip header line
fgetl(fid);  % Read and discard first line

% Read data
data = fscanf(fid, '%d,%f,%f', [3, Inf]);
fclose(fid);

% Extract columns
time = data(1, :)';
temperature = data(2, :)';
pressure = data(3, :)';
```

### Chemical Engineering Example: Saving Reactor Data

```matlab
% Simulate a batch reactor experiment
t = 0:0.5:10;                  % Time points (s)
T = 350 + 10*sin(t/2);         % Temperature (K)
P = 5 + 0.1*t;                 % Pressure (bar)
X = 0.1*t.^2 ./ (1 + 0.1*t);   % Conversion

% Create output file
fid = fopen('sample_data\batch_reactor_results.csv', 'w');

% Write header
fprintf(fid, 'Time (s),Temperature (K),Pressure (bar),Conversion\n');

% Write data
for i = 1:length(t)
    fprintf(fid, '%.1f,%.2f,%.3f,%.4f\n', t(i), T(i), P(i), X(i));
end

% Close file
fclose(fid);

% Now read it back
fid = fopen('sample_data\batch_reactor_results.csv', 'r');
header = fgetl(fid);  % Read header
data = fscanf(fid, '%f,%f,%f,%f', [4, Inf]);
fclose(fid);

% Plot results
data = data';
plot(data(:,1), data(:,2), 'b-', 'LineWidth', 2)
xlabel('Time (s)')
ylabel('Temperature (K)')
title('Batch Reactor Temperature Profile')
grid on
```

### Common Mistakes

**Mistake 1:** Incorrect format specifiers
```matlab
% WRONG — %d is for integers, not floats
fprintf(fid, 'Temperature: %d K\n', 350.5);  % Prints: 350, not 350.5

% CORRECT
fprintf(fid, 'Temperature: %.1f K\n', 350.5);  % Prints: 350.5
```

**Mistake 2:** Forgetting to escape the `%` symbol when printing percentages
```matlab
% WRONG — will interpret %% as format specifier
fprintf(fid, 'Conversion: 65 %\n');  % Error or unexpected output

% CORRECT — use %% to print a single %
fprintf(fid, 'Conversion: 65 %%\n');  % Prints: Conversion: 65 %
```

**Mistake 3:** Incorrect dimensions when reading multiple columns
```matlab
% File has 3 columns, 5 rows
% WRONG — doesn't specify dimensions correctly
data = fscanf(fid, '%f %f %f');  % Returns as a column vector

% CORRECT — specify dimensions as [rows, cols] or [rows, Inf]
data = fscanf(fid, '%f %f %f', [3, 5]);  % 3 columns, 5 rows
data = data';  % Transpose to get rows × columns format
```

**Mistake 4:** Mismatched format specifiers and variables
```matlab
% WRONG — 3 format specifiers but only 2 variables
fprintf(fid, '%d %f %f\n', time, temp);

% CORRECT — match specifiers to variables
fprintf(fid, '%d %f\n', time, temp);
```
---

## Reading csv and Excel Files

### The readtable() Function

The most straightforward way to read structured data:

```matlab
% Read CSV or text file into table
data = readtable("sample_data\reactor_dynamics.csv");

% Display info
disp(data)

% Access specific column
T = data.Temperature_K;
C = data.Conversion;
```

---

## Writing csv and Excel Files

### The writetable() Function

Save a table to a file:

```matlab
% Create table
Time = (0:10)' ;
Temperature = [20, 22, 25, 28, 32, 35, 37, 38, 38, 38, 38]';
Pressure = [1.0, 1.05, 1.1, 1.15, 1.2, 1.22, 1.23, 1.23, 1.23, 1.23, 1.23]';

data_table = table(Time, Temperature, Pressure);

% Write to CSV
writetable(data_table, "output_results.csv")

% Write to Excel
writetable(data_table, "output_results.xlsx")
```

### Appending to Files

```matlab
% Append to existing file (no header)
writetable(new_data, "log.csv", "WriteMode", "append")
```
## Excel Files

### Reading from Excel

```matlab
% Read entire sheet
data = readtable("sample_data\lab_experiment.xlsx", "Sheet", 1);

% Specify range
data = readtable("sample_data\lab_experiment.xlsx", "Sheet", "Summary", "Range", "A1:C4");

% Read cell value
cell_value = readcell("sample_data\lab_experiment.xlsx", "Sheet", 3, "Range", "B3:B3");
```

### Writing to Excel

```matlab
% Write to multiple sheets
writetable(reactor_data, "process_data.xlsx", "Sheet", 1, "Range", "A1")
writetable(product_data, "process_data.xlsx", "Sheet", 2, "Range", "A1")

% Write with formatting (requires xlswrite, older approach)
xlswrite("results.xlsx", data)
```
### Engineering Example: Equipment Specifications

```matlab
% Read equipment database
equipment = readtable("sample_data\pump_specifications.xlsx", "Sheet", "Pumps");

% Filter for specific performance
flow_min = 50;  % L/min
flow_max = 200; % L/min

COND = equipment.MaxFlow >= flow_min & equipment.MaxFlow <= flow_max;
suitable = equipment(COND, :);

% Display suitable pumps
disp(suitable(:, {'Model', 'MaxFlow', 'HeadMax', 'Power'}))

% Write selection to report
writetable(suitable, "sample_data\pump_selection_report.xlsx")
```

---

## MATLAB Data Files (.mat)

### Saving Variables

Save all or selected variables to a .mat file:

```matlab
% Save all variables in workspace
save("my_simulation.mat")

% Save specific variables
T_data = linspace(300, 400, 100);
P_data = [1, 2, 5, 10, 20];
save("thermodynamic_data.mat", "T_data", "P_data")

% Append to existing file
save("simulation_log.mat", "new_var", "-append")
```

### Loading Variables

```matlab
% Load entire file
load("my_simulation.mat")

% Load specific variable
data = load("thermodynamic_data.mat", "T_data");
T = data.T_data;
```

---

## `importdata` — Simple Text and MAT

`importdata` is a quick importer that guesses format.

```matlab
% Text file with numeric matrix
% File: conc.txt contains:
% 1.0 0.8 0.6
% 0.9 0.7 0.5
A = importdata('sample_data\conc.txt');  % returns double matrix

% File with header
% File: data_with_header.txt
% Temperature Pressure
% 300 1.0
% 350 2.5
S = importdata('sample_data\antoine_coefficients.txt');  % returns struct with .data and .textdata
num_data = S.data;      % numeric part
txt_data = S.textdata; % header

% CSV
M = importdata('sample_data\pbr_simulation_results.csv');  
```

**Limitations:** `importdata` is convenient but less robust than `readtable`/`readmatrix` for mixed types. Prefer `readtable` for heterogeneous data.


## Structured Data: JSON and XML

### Reading JSON (requires jsonencode/jsondecode)

```matlab
% Modern MATLAB (R2016b+): jsondecode
json_text = fileread("process_config.json");
config = jsondecode(json_text);

% Access nested values
reactor_type = config.reactor.type;
set_point = config.parameters.setpoint;
```

### Creating and Saving JSON

```matlab
% Create structure
config.reactor.type = "CSTR";
config.reactor.volume = 10;
config.parameters.temperature = 350;
config.parameters.pressure = 5;

% Convert to JSON and save
json_string = jsonencode(config);
fid = fopen("config.json", "w");
fprintf(fid, json_string);
fclose(fid);
```

## Error Handling in File Operations

### Checking File Existence

```matlab
filename = "data.csv";

if isfile(filename)
    data = readtable(filename);
else
    fprintf("Error: File %s not found\n", filename)
end
```

### Try-Catch Error Handling

```matlab
try
    data = readtable("sensor_data.csv");
    disp("File loaded successfully")
    
catch ME
    fprintf("Error reading file: %s\n", ME.message)
    % Create empty table or use default data
    data = table();
end
```

### Validating Data After Reading

```matlab
% Read file
data = readtable("measurements.csv");

% Check for empty table
if height(data) == 0
    error("Data file is empty")
end
```

---

### Common Mistakes

**Mistake 1:** Assuming all experimental data is valid
```matlab
% WRONG — doesn't check for invalid data
fid = fopen('data.txt', 'r');
data = fscanf(fid, '%f');
avg = mean(data);  % May give incorrect result if data contains NaN or Inf

% CORRECT — validate data
fid = fopen('data.txt', 'r');
data = fscanf(fid, '%f');
data = data(isfinite(data));  % Remove NaN and Inf
if isempty(data)
    error('No valid data found')
end
avg = mean(data);
```

**Mistake 2:** Not preserving precision when reading/writing
```matlab
% WRONG — low precision for scientific data
fprintf(fid, '%.1f', value);  % Only 1 decimal

% CORRECT — use appropriate precision
fprintf(fid, '%.6f', value);  % 6 decimals for reasonable precision
```

**Mistake 3:** Hardcoding file sizes instead of using dynamic allocation
```matlab
% WRONG — assumes exactly 100 data points
data = zeros(100, 1);
data = fscanf(fid, '%f');  % Fails if file has different size

% CORRECT — let fscanf determine size
data = fscanf(fid, '%f');  % Reads all data
```


## Data Cleaning Basics

**1. Missing values:**

```matlab
% Detect missing
TF = ismissing(T)  % table of logicals, true where missing
numMissing = sum(ismissing(T))  % per column count

% Standardize custom missing markers
T = standardizeMissing(T, {'NA', '-', '999', ''});

% Remove rows with any missing
T_clean = rmmissing(T);

% Remove rows where specific column missing
T_clean = rmmissing(T, 'DataVariables', {'C_A'});

% Fill missing: previous value, linear interpolation, constant
T_filled = fillmissing(T, 'previous');  % fill with previous
T_filled = fillmissing(T, 'linear');    % linear interpolation
T_filled = fillmissing(T, 'constant', 0); % fill with 0

% For timetable, more options
% TT_filled = fillmissing(TT, 'linear', 'DataVariables', {'T_K'});
```

**2. Outliers:**

```matlab
% Logical filtering (from Lecture 4)
isOutlier = (T.T_K > 500) | (T.T_K < 200);  % physical impossible
T_valid = T(~isOutlier, :);

% Or using isoutlier function
TF_out = isoutlier(T.T_K, 'median');  % median-based outlier detection
T_clean2 = T(~TF_out, :);

% Cap instead of remove
T.T_K(T.T_K > 500) = 500;  % cap at 500 K
```

**3. Unit consistency:**

```matlab
% Ensure all pressures in bar
% Suppose some data in Pa (large values >1000)
isPa = T.P_bar > 1000;  % likely Pa
T.P_bar(isPa) = T.P_bar(isPa) / 1e5;  % Pa to bar (1 bar = 1e5 Pa)
fprintf('Converted %d points from Pa to bar\n', sum(isPa));

% Temperature C to K
isC = T.T_K < 200;  % if T_K <200, likely C
T.T_K(isC) = T.T_K(isC) + 273.15;
```

**4. Duplicate and sorting:**

```matlab
% Remove duplicates based on timestamp
T_unique = unique(T, 'Rows');  % removes duplicate rows
% Or based on time variable
[~, idx] = unique(T.Timestamp);
T_unique = T(idx, :);

% Sort
T_sorted = sortrows(T, 'Timestamp');
```


## Common Mistakes Summary

| Mistake | Example | Fix |
|---|---|---|
| Forgetting to check `fid == -1` | `fid = fopen(...); data = fscanf(...)` | Always check: `if fid == -1, error(...), end` |
| Forgetting to close files | `fid = fopen(...); data = fscanf(...);` | Always close: `fclose(fid);` |
| Wrong file mode | `fopen('new_file.txt', 'r')` on non-existent file | Use `'w'` to create, `'a'` to append |
| Mismatched format specifiers | `fprintf(fid, '%d %f', var1, var2, var3)` | Match specifiers to variables: `'%d %f %d'` |
| Not escaping backslashes (Windows) | `'C:\data'` | Use `'C:\\data'` or `'C:/data'` |
| Not detecting EOF | While loop with unchecked `feof` | Use `while ~feof(fid)` and check `ischar(line)` |
| Incorrect dimensions in fscanf | `data = fscanf(fid, '%f %f')` | Specify: `fscanf(fid, '%f %f', [2, Inf])` |
| Ignoring ferror silently | Missing error check after operations | Use `[msg, err] = ferror(fid)` and check result |
| Low precision in exports | `fprintf(fid, '%.1f', value)` | Use `%.6f` or higher for scientific data |
| Not validating imported data | Direct use of `fscanf` output | Check for NaN, Inf, outliers after reading |

---

## Key MATLAB Syntax Reference

| Purpose | Syntax |
|---|---|
| Save/Load MAT | `save('file.mat','var1','var2')`, `load('file.mat')`, `save('file.mat','-v7.3')` |
| Quick import | `importdata('file.txt')` |
| Table import | `readtable('file.csv')`, `readtable('file.xlsx','Sheet','Sheet1')`, `readtable('file.csv',opts)` |
| Matrix import | `readmatrix('file.csv')`, `readcell('file.csv')`, `readtimetable('file.csv')` |
| Import options | `opts=detectImportOptions(file)`, `setvaropts(opts,var,'TreatAsMissing',{'NA'})` |
| Write | `writetable(T,'file.xlsx')`, `writematrix(M,'file.csv')`, `writetimetable(TT,'file.xlsx')` |
| Low-level open/close | `fid=fopen(file,'r')`, `fid=fopen(file,'w')`, `fclose(fid)`, check `fid==-1` |
| Low-level write/read | `fprintf(fid,'%.2f %f\n',x,y)`, `fscanf(fid,'%f %f',[2 Inf])`, `fgetl(fid)`, `textscan(fid,'%s %f %f')` |
| Summary | `summary(T)`, `varfun(@class,T)` |
| Missing | `ismissing(T)`, `standardizeMissing(T,{'NA','-',''})`, `rmmissing(T)`, `fillmissing(T,'linear')`, `fillmissing(T,'previous')` |
| Outliers | `isoutlier(T.Var)`, `(T.Var>thr)` logical, `T.Var(T.Var>thr)=NaN` |
| Organize | `sortrows(T,'Var')`, `unique(T)`, `unique(T,'Rows')`, `groupsummary(T,'GroupVar','mean','DataVar')` |
| Timetable | `table2timetable(T)`, `retime(TT,'hourly','mean')`, `synchronize(TT1,TT2)` |

Common pitfalls: forgetting `fclose`, wrong Current Folder, delimiter mismatch, not handling header lines, overwriting with `'w'`, not standardizing missing markers like `'NA'` or `999`, mixing units without conversion.



## Homework

### Problem

A laboratory experiment produced the following CSV file named `cstr_data.csv` (you may create it yourself for the exercise):

```text
Time_min,CA_molL,T_C
0,2.00,65.0
5,1.65,66.2
10,1.38,67.1
15,1.15,67.8
20,0.97,68.3
25,0.82,68.6
30,0.70,68.9
```

**Required Tasks**

1. Write a short MATLAB script (or series of commands) that creates the file `cstr_data.csv` with the content shown above (use `writetable` or `fprintf`).
2. Read the file back into a MATLAB table.
3. Convert temperature from °C to K and compute the conversion \(X = 1 - C_A/C_{A0}\) (take \(C_{A0}\) as the first concentration value).
4. Add the two new columns (`T_K` and `X`) to the table.
5. Write the enriched table to a new file `cstr_processed.csv`.
6. Also save the numeric vectors `t`, `CA`, `T_K` and `X` in a MATLAB binary file `cstr_processed.mat`.
7. Display the final table in the Command Window.

### Concepts Being Tested

- Creating a delimited text file
- Importing a CSV into a table
- Unit conversion and simple derived quantities
- Exporting both text and binary results
- Basic table manipulation

### Hints

- You can build the original table with `table(...)` and then `writetable`.
- Conversion: `T_K = T_C + 273.15;`
- After adding columns, `writetable` will include the new variable names automatically.

<details>
<summary><strong>Detailed Solution — Click to Reveal</strong></summary>

### Concept

The exercise walks through a complete, realistic data-handling pipeline: write a raw data file, read it, augment it with calculated quantities, and export both a human-readable CSV and a binary MATLAB file for later use.

### Approach / Idea

1. Construct a table that matches the given CSV and write it to disk.
2. Read the file with `readtable`.
3. Perform the unit conversion and conversion calculation.
4. Append the new columns to the table.
5. Export the enriched table as CSV and the selected vectors as a `.mat` file.
6. Display the result.

### Syntax

```matlab
writetable(raw, 'cstr_data.csv');
data = readtable('cstr_data.csv');
data.T_K = data.T_C + 273.15;
data.X   = 1 - data.CA_molL / data.CA_molL(1);
writetable(data, 'cstr_processed.csv');
save('cstr_processed.mat', 't', 'CA', 'T_K', 'X');
```

### Syntax Breakdown

- `table` + `writetable` creates a properly formatted CSV with a header row.
- Dot notation adds new variables to an existing table.
- `save` with an explicit list of variable names stores only what is needed.
- Displaying the table confirms that all columns are present and correctly named.

### MATLAB Code

```matlab
% ---- 1. Create the original CSV ----
Time_min = [0; 5; 10; 15; 20; 25; 30];
CA_molL  = [2.00; 1.65; 1.38; 1.15; 0.97; 0.82; 0.70];
T_C      = [65.0; 66.2; 67.1; 67.8; 68.3; 68.6; 68.9];

raw = table(Time_min, CA_molL, T_C);
writetable(raw, 'sample_data\cstr_data.csv');
fprintf('Created cstr_data.csv\n');

% ---- 2. Read it back ----
data = readtable('sample_data\cstr_data.csv');

% ---- 3 & 4. Convert and compute conversion ----
data.T_K = data.T_C + 273.15;
CA0 = data.CA_molL(1);
data.X = 1 - data.CA_molL / CA0;

% ---- 5. Export enriched CSV ----
writetable(data, 'sample_data\cstr_processed.csv');
fprintf('Wrote cstr_processed.csv\n');

% ---- 6. Save binary MATLAB file ----
t   = data.Time_min;
CA  = data.CA_molL;
T_K = data.T_K;
X   = data.X;
save('sample_data\cstr_processed.mat', 't', 'CA', 'T_K', 'X');
fprintf('Wrote cstr_processed.mat\n');

% ---- 7. Display ----
disp(data);
```

### Line-by-Line Explanation

- The first block builds the exact content of the laboratory file and writes it.
- `readtable` reconstructs the table, preserving column names.
- Two new columns are calculated and attached with dot assignment.
- `writetable` produces the processed CSV; `save` stores the numeric vectors for fast later loading.
- `disp` shows the final seven-row table with five columns.

### Expected Result

```
Created cstr_data.csv
Wrote cstr_processed.csv
Wrote cstr_processed.mat

Time_min    CA_molL    T_C     T_K       X  
________    _______    ____    ______    _______
    0         2        65      338.15         0
    5        1.65     66.2     339.35      0.175
   10        1.38     67.1     340.25       0.31
   15        1.15     67.8     340.95      0.425
   20        0.97     68.3     341.45      0.515
   25        0.82     68.6     341.75       0.59
   30         0.7     68.9     342.05       0.65
```

### Engineering Interpretation

The script has taken a raw laboratory file, added the thermodynamically required absolute temperature and the derived conversion, and produced both a shareable CSV and a compact binary file. The conversion rises smoothly from 0 to 0.65 over 30 min, consistent with a typical batch or long-residence-time CSTR experiment. The same pattern can be reused for any subsequent laboratory run by simply changing the input file name.

</details>


---