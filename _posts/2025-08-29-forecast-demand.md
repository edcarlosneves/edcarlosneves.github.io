---

layout: post
title:  "How to Forecast Demand with Seasonal Adjustment in Python"
date:   2025-08-29 00:00:00 -0300
categories: posts python
---

Forecasting future demand is one of the most critical challenges for any business. When sales or production follow patterns that repeat throughout the year—like more ice cream sales in summer or coats in winter—we’re dealing with **seasonality**.

In this tutorial, we’ll explore the **seasonal adjustment method**, a powerful technique to forecast demand in scenarios like this. We’ll do it practically, step by step, using Python and the NumPy library.

**What we’ll do:**

1. Analyze a quarterly production time series.
2. Calculate seasonal factors to understand peaks and troughs in demand.
3. Use simple linear regression to forecast total demand for the next year.
4. Adjust this total forecast for each quarter by applying seasonal factors.

### **Step 1: Understand the Problem and the Data**

Imagine we have a company’s production data over three years, broken down by quarter.

|            | Quarter 1 | Quarter 2 | Quarter 3 | Quarter 4 |
| :--------- | :-------- | :-------- | :-------- | :-------- |
| **Year 1** | 60        | 890       | 250       | 120       |
| **Year 2** | 80        | 1600      | 290       | 150       |
| **Year 3** | 130       | 2300      | 500       | 260       |

Looking at the table, we notice a clear pattern: production (and consequently demand) is much higher in quarters 2 and 3. Our goal is to **forecast demand for each quarter of Year 4**.

### **Step 2: Setting Up the Python Environment**

To begin, we’ll only need the NumPy library, which is essential for numerical calculations in Python. If you don’t have it installed yet, you can do so with a simple command in your terminal:

```bash
pip install numpy
```

Now, let’s open our code editor or Jupyter Notebook and start coding.

1. **Import the library and define the data:**

   ```python
   import numpy as np

   # Quarterly production data for years 1, 2, and 3
   data = np.array([
   [60, 890, 250, 120],  # Year 1
   [80, 1600, 290, 150],  # Year 2
   [130, 2300, 500, 260]  # Year 3
   ])

   # Years corresponding to the data
   years = np.array([1, 2, 3])
   ```

### **Step 3: Calculate Annual Totals and Averages**

The first step in our analysis is to calculate the total production and the quarterly average for each year.

```python
# Calculate annual totals (sum of rows)
annual_totals = np.sum(data, axis=1)
print(f"Annual totals: {annual_totals}")
# Expected output: Annual totals: [1320 2120 3190]

# Calculate annual quarterly averages
annual_means = np.mean(data, axis=1)
print(f"Annual quarterly averages: {annual_means}")
# Expected output: Annual quarterly averages: [330.  530.  797.5]
```

### **Step 4: Find the Seasonal Coefficients**

This is the most important part of the method! The seasonal coefficient tells us how much a specific quarter deviates from that year’s average.

We calculate it by dividing the production of each quarter by its year’s average.

```python
# Create an empty matrix to store coefficients
seasonal_coefficients = np.zeros_like(data, dtype=float)

# Calculate the coefficients
for i in range(len(years)):
    for j in range(4): # 4 quarters
        seasonal_coefficients[i, j] = data[i, j] / annual_means[i]

print("Seasonal Coefficients by Year and Quarter:")
print(seasonal_coefficients)
```

Now, to obtain a single adjustment factor for each quarter, we calculate the mean of the coefficients in each column.

```python
# Calculate the average coefficient for each quarter (column means)
avg_seasonal_factors = np.mean(seasonal_coefficients, axis=0)

print(f"\nAverage Seasonal Coefficients (Adjustment Factors):")
print(avg_seasonal_factors)
# Expected output: [0.16525699 2.86661672 0.64390161 0.32422468]
```

These four numbers are our **adjustment factors**. A value greater than 1 (as in Q2 and Q3) indicates above-average demand, while a value less than 1 (Q1 and Q4) indicates below-average demand.

### **Step 5: Forecast Total Demand for Year 4**

Looking at the annual totals (`[1320 2120 3190]`), we see a growth trend. We’ll use **linear regression** to project the total for Year 4.

The `np.polyfit()` function does this for us, finding the coefficients of the line (`y = ax + b`) that best fits our data.

```python
# Fit a line to the annual totals
# The '1' indicates we want a degree-1 polynomial (a straight line)
line_coefficients = np.polyfit(years, annual_totals, 1)

# The prediction function (our line)
forecast_func = np.poly1d(line_coefficients)

# Forecast the total for Year 4
year_to_forecast = 4
forecast_total_year4 = forecast_func(year_to_forecast)

print(f"\nThe trend line equation is: y = {line_coefficients[0]:.2f}x + {line_coefficients[1]:.2f}")
# Expected output: The trend line equation is: y = 935.00x + 340.00
print(f"The total demand forecast for Year 4 is: {forecast_total_year4:.2f}")
# Expected output: The total demand forecast for Year 4 is: 4080.00
```

### **Step 6: Adjust the Forecast with Seasonal Factors**

We have the total forecast for Year 4 (4080 units), but how is it distributed across the quarters?

1. First, calculate the base quarterly demand as if there were no seasonality:

   ```python
   base_quarterly_demand_year4 = forecast_total_year4 / 4
   print(f"\nBase quarterly demand (no adjustment) for Year 4: {base_quarterly_demand_year4:.2f}")
   # Expected output: Base quarterly demand (no adjustment) for Year 4: 1020.00
   ```

2. Finally, multiply this base demand by our seasonal adjustment factors:

   ```python
   # Apply seasonal factors to get the final forecast
   adjusted_forecast_year4 = base_quarterly_demand_year4 * avg_seasonal_factors

   print("\n--- FINAL FORECAST FOR YEAR 4 ---")
   for i, forecast in enumerate(adjusted_forecast_year4):
       print(f"Quarter {i+1}: {forecast:.2f} units")
   ```

**Expected Final Result:**

```
--- FINAL FORECAST FOR YEAR 4 ---
Quarter 1: 168.56 units
Quarter 2: 2923.95 units
Quarter 3: 656.78 units
Quarter 4: 330.71 units
```

As we can see, our final forecast perfectly reflects the seasonal pattern observed in the historical data, with much higher demand in quarters 2 and 3.

### **Conclusion**

We’ve just implemented a robust demand forecasting method. With just a few lines of Python, we captured both the growth trend and the seasonal patterns to generate a forecast for the future. This method can be applied to many types of data, from monthly sales to daily production.
