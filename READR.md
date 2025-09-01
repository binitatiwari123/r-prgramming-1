# Chronological Steps for Data Analysis in R (with R Code)

## Step 1: Prepare and Import Your Data

* Ensure your raw data has **no extra headers or footers**—only actual data rows.
* Save your data in a **clean format** (CSV or Excel).
* Import the data into R. If there are extra headers, remove them during import.

```r
# Import CSV file
data <- read.csv("data.csv", header = TRUE)

# Import Excel file
library(readxl)
data <- read_excel("data.xlsx", skip = 1)  # skip extra headers if any
```

## Step 2: Clean and Prepare Variables

### Check Variable Types

* **Numeric** for continuous variables
* **Factor** for categorical variables
* Use `str(data)` to check variable types

```r
# Check structure of data
str(data)

# Convert variable to numeric
data$score <- as.numeric(data$score)

# Convert variable to factor
data$group <- as.factor(data$group)
```

### Summary: Factors in R

* Factors are for **categorical variables**
* Converting character or numeric codes to factors ensures correct handling in **summaries, plots, and models**

```r
# Character to factor
data$education <- factor(data$education, levels = c("Primary", "Secondary", "Tertiary"))

# Numeric to factor with labels
data$education <- factor(data$education, levels = c(1, 2, 3), labels = c("Primary", "Secondary", "Tertiary"))
```

### Rename Variables

```r
names(data)[names(data) == "oldName"] <- "newName"
```

### Recode or Define Categories

```r
data$group <- factor(data$group, labels = c("Control", "Intervention"))
```

## Step 3: Remove Duplicates and Handle Missing Data

### Step 3a: Identify and Remove Duplicates

```r
# Remove duplicate rows in entire dataset
data <- data[!duplicated(data), ]

# Check duplicates in a specific column (e.g., ID)
duplicated_ids <- data[duplicated(data$ID), ]
```

### Step 3b: Identify Missing Data

```r
# Check missing values for each column
colSums(is.na(data))

# Check missing values for each row
rowSums(is.na(data))
```

### Step 3c: Data Imputation

#### 1. Continuous (Numeric) Variables

```r
# Replace with mean
data$var[is.na(data$var)] <- mean(data$var, na.rm = TRUE)

# Replace with median
data$var[is.na(data$var)] <- median(data$var, na.rm = TRUE)
```

#### 2. Ordinal Variables

```r
# Replace with median
data$var[is.na(data$var)] <- median(data$var, na.rm = TRUE)

# Replace with mode
mode_val <- names(sort(table(data$var), decreasing = TRUE))[1]
data$var[is.na(data$var)] <- mode_val
```

#### 3. Nominal (Categorical) Variables

```r
# Replace with mode
mode_val <- names(sort(table(data$var), decreasing = TRUE))[1]
data$var[is.na(data$var)] <- mode_val
```

**Notes:**

* Small missing data → can remove rows instead of imputing
* Large datasets → consider multiple imputation (`mice` package)
* Always check distributions before choosing mean vs median

### Step 3d: Decide What to Do With Missing Data

```r
# Remove rows with any missing values
data <- na.omit(data)

# Remove columns with >30–40% missing
threshold <- 0.4 * nrow(data)
data <- data[, colSums(is.na(data)) <= threshold]
```

### Step 3e: After Imputation

```r
# Check for remaining missing values
colSums(is.na(data))

# Ensure correct data types
str(data)
```

## Step 4: Explore Your Data (Descriptive Statistics)

### Categorical Variables

```r
# Frequency table
table(data$group)

# Proportion table
prop.table(table(data$group)) * 100
```

### Continuous Variables

```r
mean(data$score, na.rm = TRUE)
median(data$score, na.rm = TRUE)
sd(data$score, na.rm = TRUE)
```

### Grouped Summaries

```r
aggregate(score ~ group, data = data, FUN = mean)
aggregate(score ~ group, data = data, FUN = median)
```

## Step 5: Visualize Your Data

```r
library(ggplot2)

# Boxplot for continuous variables by group
ggplot(data, aes(x = group, y = score)) +  
  geom_boxplot()

# Histogram
ggplot(data, aes(x = score)) +
  geom_histogram(binwidth = 5, fill = "blue", color = "black")

# Bar plot for categorical variable
ggplot(data, aes(x = group)) +
  geom_bar(fill = "lightgreen")
```

## Step 6: Check Normality (Before Statistical Tests)

```r
# Shapiro-Wilk test
shapiro.test(data$score)

# QQ plot
qqnorm(data$score); qqline(data$score)

# Histogram
hist(data$score)
```

* Interpretation:

  * `p > 0.05` → approximately normal → parametric tests
  * `p ≤ 0.05` → not normal → non-parametric tests

## Step 7: Choose and Perform Statistical Tests

### Paired Tests (Same Group Pre-Post)

```r
# Normal: paired t-test
t.test(pre, post, paired = TRUE)

# Non-normal: Wilcoxon signed-rank test
wilcox.test(pre, post, paired = TRUE)
```

### Independent Tests (Two Groups)

```r
# Normal: independent t-test
t.test(score ~ group, data = data)

# Non-normal: Mann-Whitney U test
wilcox.test(score ~ group, data = data)
```

### More Than Two Groups

```r
# Normal: ANOVA
anova_result <- aov(score ~ group, data = data)
summary(anova_result)

# Non-normal: Kruskal-Wallis test
kruskal.test(score ~ group, data = data)
```

### Step 7a: Decide Which Test to Use

**1. Outcome Type**

* **Categorical** (Yes/No, Male/Female) → use **Chi-square / Fisher**

```r
# Chi-square test
table_data <- table(data$understanding, data$location)
chisq.test(table_data)

# Fisher's exact test (if small sample)
fisher.test(table_data)
```

* **Numerical** (scores, BMI, age) → check normality

**2. Normality Check (if numerical)**

* Normal → parametric test (t-test, ANOVA)
* Not normal → non-parametric test (Mann–Whitney, Wilcoxon)

**Examples:**

* **Case 1: Categorical**

  * Outcome: Understanding improved (Yes/No) by School Location (City/Regional)
  * Test: **Chi-square**

* **Case 2: Numerical**

  * Outcome: Change in understanding score by School Location
  * Different group → Normal: t-test, Not normal: Mann–Whitney U
  * Same group → Normal: paired t-test, Not normal: Wilcoxon
