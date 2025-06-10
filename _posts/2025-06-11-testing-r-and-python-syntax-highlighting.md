---
layout: post
title: Testing R and Python Syntax Highlighting
date: 2025-06-10 19:45:00 +0100
categories: [R, Python, Code]
tags:
- syntax highlighting
- code examples
- R
- Python
---

# Testing Syntax Highlighting for R and Python

This post demonstrates the syntax highlighting capabilities for R and Python code in this blog.

## Full Disclosure
I got Cline to help me with this post. It really is magical how you can just ask it to do things and it does them. I am not sure how I lived without it before. I salute the AI Overlords.

## R Code Examples

Here's a simple R example:

```r
# Basic R operations
x <- 1:10
y <- x^2
mean_y <- mean(y)
sd_y <- sd(y)

# Print results
cat("Mean:", mean_y, "\n")
cat("Standard Deviation:", sd_y, "\n")
```

Here's a more complex R example using dplyr:

```r
# Load libraries
library(dplyr)
library(ggplot2)

# Create a sample dataset
data <- tibble(
  id = 1:100,
  value = rnorm(100, mean = 10, sd = 2),
  group = sample(LETTERS[1:5], 100, replace = TRUE)
)

# Data manipulation with dplyr
results <- data %>%
  group_by(group) %>%
  summarize(
    count = n(),
    avg_value = mean(value),
    median_value = median(value),
    sd_value = sd(value)
  ) %>%
  arrange(desc(avg_value))

# Print the results
print(results)
```

## Python Code Examples

Here's a basic Python example:

```python
# Basic Python operations
import numpy as np

# Create arrays
x = np.arange(1, 11)
y = x**2

# Calculate statistics
mean_y = np.mean(y)
std_y = np.std(y)

# Print results
print(f"Mean: {mean_y}")
print(f"Standard Deviation: {std_y}")
```

Here's a more complex Python example using pandas:

```python
# Data manipulation with pandas
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

# Create a sample dataset
np.random.seed(42)
data = pd.DataFrame({
    'id': range(1, 101),
    'value': np.random.normal(10, 2, 100),
    'group': np.random.choice(list('ABCDE'), 100)
})

# Group by and summarize
results = data.groupby('group').agg({
    'id': 'count',
    'value': ['mean', 'median', 'std']
}).reset_index()

# Rename columns
results.columns = ['group', 'count', 'avg_value', 'median_value', 'std_value']

# Sort by average value
results = results.sort_values('avg_value', ascending=False)

# Print the results
print(results)
```

## Conclusion

With highlight.js properly configured, the code blocks above should display with appropriate syntax highlighting for both R and Python.
