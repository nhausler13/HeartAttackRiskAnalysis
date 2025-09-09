# -*- coding: utf-8 -*-
"""
Spyder Editor

This is a temporary script file.
"""
# CREATING HEATMAP FOR MORTALITY RATE BY COMBORDITY AND RISK FACTOR COUNT 

import pandas as pd;
import seaborn as sns;
import matplotlib.pyplot as plt;
import numpy as np
from sklearn.preprocessing import StandardScaler
from matplotlib.patches import Patch


df = pd.read_csv("C:/Users/hausl/Documents/SQL RESULTS/Comorbidity  Mortality Rate_1.csv")
heatmap_data = df.pivot(index="comorbidity", columns="risk_factor_count", values="mortality_rate")

heatmap_data = df.pivot(index="comorbidity", columns="risk_factor_count", values="mortality_rate")

# Pivot patient_count for annotation
count_data = df.pivot(index="comorbidity", columns="risk_factor_count", values="patient_count")

# Create annotated labels combining mortality_rate and patient_count
annot_labels = heatmap_data.copy().astype(str)
for i in heatmap_data.index:
    for j in heatmap_data.columns:
        annot_labels.loc[i, j] = f"{heatmap_data.loc[i,j]*100:.1f}%\n(n={count_data.loc[i,j]})"

# Plot heatmap
plt.figure(figsize=(9,6))
sns.heatmap(
    data=heatmap_data,
    annot=annot_labels,           # show the mortality rates on the cells
    fmt="",            # format as percentage
    cmap="RdYlGn_r",      # Red → Yellow → Green reversed, so high = red, low = green
    cbar_kws={'label': 'Mortality Rate'},
    annot_kws={"size": 14, "weight": "bold"}  # Bigger and bold numbers
)
plt.title("Mortality Rate by Comorbidity and Risk Factor Count")
plt.xlabel("Risk Factor Count")
plt.ylabel("Comorbidity")
plt.show()

#
df = pd.read_csv("C:/Users/hausl/Documents/SQL RESULTS/Risk_category.csv")

df['mortality_rate_pct'] = df['mortality_rate'] * 100

# Bar plot
plt.figure(figsize=(8,5))
palette_colors = ['lightgreen', 'gold', 'tomato']
sns.barplot(x='risk_category', y='mortality_rate_pct', data=df, palette=palette_colors)

plt.ylabel('Mortality Rate (%)')
plt.xlabel('Risk Category')
plt.title('Mortality by Risk Category')

# Annotate bars
for i, v in enumerate(df['mortality_rate_pct']):
    plt.text(i, v + 1, f"{v:.1f}% ({df['patient_count'][i]})", ha='center')

# Simplified legend
legend_elements = [
    Patch(facecolor='lightgreen', label='Low: None of the labs abnormal'),
    Patch(facecolor='gold', label='Medium: At least one lab abnormal'),
    Patch(facecolor='tomato', label='High: More than one lab abnormal / all other patients')
]

# Adjust legend to be vertical and narrower
plt.legend(handles=legend_elements, 
           title='Risk Category', 
           loc='upper left', 
           bbox_to_anchor=(1.02, 1),  # Slightly to the right
           labelspacing=1.2,
           handlelength=1.5)  # Shorter handle length to reduce width

# Add abnormal labs note under the legend
abnormal_labs_text = (
    "Abnormal labs thresholds:\n"
    "• Serum Creatinine > 1.3 mg/dL\n"
    "• Ejection Fraction < 40%\n"
    "• Serum Sodium > 145 mEq/L"
)
plt.text(.50, 0.65, abnormal_labs_text, transform=plt.gcf().transFigure,
         fontsize=10, ha='left', va='top')  # Position under legend

plt.tight_layout()
plt.show()

df = pd.read_csv("C:/Users/hausl/Documents/SQL RESULTS/Creatine vs Platelets.csv")

# Set figure

plt.figure(figsize=(12,8))

# Non-linear scaling for bubble size (square root for better differentiation)
bubble_sizes = (df['mortality_rate'] * np.sqrt(df['patient_count']))**1.5 * 1000
 # adjust multiplier for visibility

# Scatter plot
scatter = plt.scatter(
    x=df['creatine_ef_label'],
    y=df['platelets_label'],
    s=bubble_sizes,
    c=df['total_risk_score'],
    cmap='Blues',
    alpha=0.7,
    edgecolors='black',
    linewidth=0.8
)

# Colorbar for risk score
cbar = plt.colorbar(scatter)
cbar.set_label('Total Risk Score', rotation=270, labelpad=15)

# Labels and title
plt.xlabel('Creatinine-to-EF Quartile')
plt.ylabel('Platelets per Risk Factor Quartile')
plt.title('High-Risk Patients: Mortality Rate (Bubble Size) vs Total Risk Score (Color)')

# Optional: Rotate ticks for clarity
plt.xticks(rotation=30)
plt.yticks(rotation=0)

# Add explanation below plot
lab_explanation = (
    "Bubble size = Mortality rate (non-linear scaling)\n"
    "Color = Total risk score (0-6)\n"
    "X-axis = CPK/EF quartile\n"
    "Y-axis = Platelets per risk factor quartile"
)
plt.gcf().text(0.02, -0.1, lab_explanation, fontsize=10, ha='left')
plt.margins(x=0.15, y=0.15)  # 15% extra space around the axes
plt.tight_layout()
plt.show()

df = pd.read_csv("C:/Users/hausl/Documents/SQL RESULTS/Age Category.csv")

# --- Convert mortality_rate to percentages ---
df["mortality_rate_pct"] = df["mortality_rate"] * 100

# --- Create annotation column with mortality rate + patient count ---
df["label"] = df["mortality_rate_pct"].round(1).astype(str) + "%\n(n=" + df["patient_count"].astype(str) + ")"

# --- Pivot tables for values & annotations ---
pivot_values = df.pivot(index="risk_factor_count", columns="age_category", values="mortality_rate_pct")
pivot_labels = df.pivot(index="risk_factor_count", columns="age_category", values="label")

# --- Heatmap ---
plt.figure(figsize=(9,6))
sns.heatmap(
    pivot_values,
    annot=pivot_labels,
    fmt="",                # show custom text
    cmap="RdYlGn_r",       # green=low, red=high
    cbar_kws={'label': 'Mortality Rate (%)'},
    vmin=0, vmax=100       # keep scale consistent
)
plt.title("Mortality Rate by Age Category & Risk Factor Count", fontsize=14, pad=15)
plt.ylabel("Risk Factor Count")
plt.xlabel("Age Category")

# Make y-axis labels horizontal
plt.yticks(rotation=0)
plt.show()

# --- Grouped Bar Chart ---
plt.figure(figsize=(10,6))
sns.barplot(
    data=df,
    x="age_category",
    y="mortality_rate_pct",
    hue="risk_factor_count",
    palette="Set2"
)
plt.title("Mortality Rate by Age Category & Risk Factor Count", fontsize=14, pad=15)
plt.ylabel("Mortality Rate (%)")
plt.xlabel("Age Category")
plt.legend(title="Risk Factor Count")
plt.show()
# --- Grouped Bar Chart ---
plt.figure(figsize=(10,6))
sns.barplot(
    data=df,
    x="age_category",
    y="mortality_rate",
    hue="risk_factor_count",
    palette="Set2"
)
plt.title("Mortality Rate by Age Category & Risk Factor Count", fontsize=14, pad=15)
plt.ylabel("Mortality Rate (%)")
plt.xlabel("Age Category")
plt.legend(title="Risk Factor Count")
plt.show()
