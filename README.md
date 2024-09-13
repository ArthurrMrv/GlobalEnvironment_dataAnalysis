Global Temperature Anomaly Visualization
Goal
The goal of this homework is to create a visual representation of the evolution of global surface temperatures over time using a set of colored stripes. The visualization will illustrate the temperature anomalies from 1850 to 2023, with cooler temperatures represented in shades of blue and warmer temperatures represented in shades of red.

Example


Administrativia
Groups composition: To be provided no later than Thursday, Sep. 12, 8 PM (by email).
Homework submission: Friday, Sep. 20, 10 PM (procedure TBA).
Data Set
The data consists of two CSV files:

NoMon.csv: Northern Hemisphere monthly temperature anomalies.
SoMon.csv: Southern Hemisphere monthly temperature anomalies.
CSV File Description
Column A: Month
Column B: Temperature anomaly in °C (difference from the 1961-1990 average)
Columns C and D: Lower and upper boundaries of the confidence interval (not used in this analysis)
Code Walkthrough
Load and Check Data
python
Copy code
import matplotlib.pyplot as plt
import pandas as pd
import csv

# Load the data
df_no = pd.read_csv('NoMon.csv')
df_so = pd.read_csv('SoMon.csv')

# Check for missing values
print("NoMon Dataset Info: ")
print(df_no.info())
print("SoMon Dataset Info: ")
print(df_so.info())

print("Number of missing values in NoMon data: \n{}".format(df_no.isnull().sum()))
print("Number of missing values in SoMon data: \n{}".format(df_so.isnull().sum()))
Data Preparation
python
Copy code
# Merge datasets by year
df_merged = pd.merge(df_no, df_so, on=["Time"], suffixes=('_no', '_so'))

# Calculate global average temperature anomaly
df_gl = pd.DataFrame()
df_gl['Time'] = df_merged['Time']

for c in df_so.columns:
    if c == 'Time':
        continue
    df_gl[c] = df_merged[[c+'_no', c+'_so']].mean(axis=1)
Color Palette
python
Copy code
import matplotlib.colors as mcolors
import numpy as np

# Extract colors
colors = mcolors.CSS4_COLORS
blue_colors_keys =  list(filter(lambda c: 'blue' in c, list(mcolors.CSS4_COLORS.keys())))
red_colors_keys =  list(filter(lambda c: sum([a in c for a in ["red", "pink", "salmon"]]), list(mcolors.CSS4_COLORS.keys())))

df_col = pd.DataFrame(mcolors.CSS4_COLORS, index=[0])

# Select and concatenate colors
colors = np.concatenate([df_col[k].sample(10, axis=1).iloc[0].values for k in [blue_colors_keys, red_colors_keys]])
palette = list(colors)
Temperature to Color Mapping
python
Copy code
# Calculate temperature range
min_temp = df_gl['Anomaly (deg C)'].min()
max_temp = df_gl['Anomaly (deg C)'].max()
delta = max_temp - min_temp

# Create a color mapper
incr = delta / len(palette)
mapper = [(min_temp + i*incr, min_temp + (i+1)*incr, palette[i]) for i in range(len(palette))]

# Function to get color based on temperature
def get_color(temp):
    for m in mapper:
        if m[0] <= temp < m[1]:
            return m[2]
    return mapper[-1][2]
Visualization
python
Copy code
# Prepare dataset for plotting
df_gl["Year"] = df_gl["Time"].str[:4].astype(int)
df_yearly_gl = df_gl.drop(columns=['Time']).groupby('Year').mean()

# Get temperature anomalies and convert to colors
colors = [get_color(temp) for temp in df_yearly_gl['Anomaly (deg C)']]

# Plot the stripes
fig, ax = plt.subplots(figsize=(15, 2))

for i, year in enumerate(df_yearly_gl.index):
    ax.bar(year, 1, color=colors[i], edgecolor=colors[i], width=1.0)

# Set axis labels and title
ax.set_xlabel('Year')
ax.set_ylabel('Anomaly')
ax.set_title('Global Temperature Anomaly (Stripes Representation)')

ax.set_xlim([df_yearly_gl.index.min(), df_yearly_gl.index.max()])
ax.set_yticks([])

plt.show()
Conclusion
The visualization provides a clear representation of how global surface temperatures have evolved over time, highlighting periods of warming and cooling with distinct color gradients.
