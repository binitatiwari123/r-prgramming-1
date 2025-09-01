Teachers \<- Teachers\[-1, \]

library(readxl)

str(Teachers) head(Teachers) summary(Teachers)

# Rename by position

colnames(Teachers)\[2\] \<- "School_Location" colnames(Teachers)\[3\]
\<- "School_Name" colnames(Teachers)\[4\] \<- "Subject_Type"
colnames(Teachers)\[8\] \<- "Qualitativenotes"

#Remove first column Teachers \<- Teachers\[, -1\] library(dplyr) #hide
unwanted column \# Use all columns except this one Teachers \<- Teachers
%\>% select(-Qualitativenotes)

# Now perform analysis on Teachers_numeric, while the original dataset remains intact

colnames(Teachers) colnames(Teachers)\[4\] \<-
"CollectedMarchUnderstanding" colnames(Teachers)\[5\] \<-
"CollectedMarchConfidence" colnames(Teachers)\[6\] \<-
"CollectedSeptemberUnderstanding" colnames(Teachers)\[7\] \<-
"CollectedSeptemberConfidence"

# Convert categorical variables into factors

Teachers$School_Location <- as.factor(Teachers$School_Location)
Teachers$School_Name <- as.factor(Teachers$School_Name)
Teachers$Subject_Type <- as.factor(Teachers$Subject_Type)

str(Teachers) #check the result of factor

# Columns to convert to numeric

cols \<- c("CollectedMarchUnderstanding",
"CollectedSeptemberUnderstanding")

# Extract numeric value at the start

Teachers\[cols\] \<- lapply(Teachers\[cols\], function(x) {
as.numeric(sub("\^(`\d+`{=tex}).\*","\\1", x)) \# keeps only the leading
number })

# Check result

head(Teachers\[cols\])

# Create a new variable for change in understanding

Teachers$UnderstandingChange <- Teachers$CollectedSeptemberUnderstanding -
Teachers\$CollectedMarchUnderstanding

# View first few rows

head(Teachers\[, c("CollectedMarchUnderstanding",
"CollectedSeptemberUnderstanding", "UnderstandingChange")\])

str(Teachers\[, c("CollectedMarchConfidence",
"CollectedSeptemberConfidence")\]) #to check the type of variable
#although it looks numeric but R is treating it as chr, now changeit to
numeric

Teachers$CollectedMarchConfidence <- as.numeric(Teachers$CollectedMarchConfidence)
Teachers$CollectedSeptemberConfidence <- as.numeric(Teachers$CollectedSeptemberConfidence)

# Create a new variable for change in confidence

Teachers$ConfidenceChange <- Teachers$CollectedSeptemberConfidence -
Teachers\$CollectedMarchConfidence

# Check result

head(Teachers\[, c("CollectedMarchConfidence",
"CollectedSeptemberConfidence", "ConfidenceChange")\])

#check for duplication sum(duplicated(Teachers))

#check for missing/NA colSums(is.na(Teachers))

#check missing in rows as well Teachers\[!complete.cases(Teachers), \]

#Descriptive Statistics

# Frequency counts

table(Teachers$School_Location) table(Teachers$School_Name)
table(Teachers\$Subject_Type)

# Percentage of School Location

prop.table(table(Teachers\$School_Location)) \* 100

# Percentage of School Name

prop.table(table(Teachers\$School_Name)) \* 100

# Percentage of Subject Type

prop.table(table(Teachers\$Subject_Type)) \* 100

#Together both count and percentage \# Frequency table freq \<-
table(Teachers\$School_Location)

# Percentage table

perc \<- prop.table(freq) \* 100

# Combine into a data frame

SchoolLocation_summary \<- data.frame( School_Location = names(freq),
Frequency = as.vector(freq), Percentage = round(as.vector(perc), 1) \#
rounded to 1 decimal )

SchoolLocation_summary

# School Name

freq \<- table(Teachers\$School_Name) perc \<- prop.table(freq) \* 100
SchoolName_summary \<- data.frame( School_Name = names(freq), Frequency
= as.vector(freq), Percentage = round(as.vector(perc), 1) )
SchoolName_summary

# Subject Type

freq \<- table(Teachers\$Subject_Type) perc \<- prop.table(freq) \* 100
Subject_summary \<- data.frame( Subject_Type = names(freq), Frequency =
as.vector(freq), Percentage = round(as.vector(perc), 1) )

Subject_summary

#For Numeric variable,, to find out mean, median and mode
summary(Teachers\[, c("CollectedMarchUnderstanding",
"CollectedSeptemberUnderstanding", "UnderstandingChange",
"CollectedMarchConfidence", "CollectedSeptemberConfidence",
"ConfidenceChange")\])

#labeling understanding change to increased and decreased/no change
Teachers$UnderstandingChange_cat <- ifelse(  Teachers$UnderstandingChange
\> 0, "Increased", "No change / Decreased" )
Teachers$UnderstandingChange_cat <- factor(  Teachers$UnderstandingChange_cat,
levels = c("No change / Decreased", "Increased") \# for consistent order
)

# Create categorical variable for Confidence Change

Teachers$ConfidenceChange_cat <- ifelse(  Teachers$ConfidenceChange \>
0, "Increased", "No change / Decreased" )

# Make it a factor with consistent order

Teachers$ConfidenceChange_cat <- factor(  Teachers$ConfidenceChange_cat,
levels = c("No change / Decreased", "Increased") )

library(dplyr)

# Understanding

understanding_counts \<-
table(Teachers$School_Location, Teachers$UnderstandingChange_cat)
understanding_perc \<- round(prop.table(understanding_counts, 1) \* 100,
1)

# Confidence

confidence_counts \<-
table(Teachers$School_Location, Teachers$ConfidenceChange_cat)
confidence_perc \<- round(prop.table(confidence_counts, 1) \* 100, 1)

# Combine into one data frame

combined_crosstab \<- data.frame( School_Location =
rep(rownames(understanding_counts), 2), Measure = rep(c("Understanding",
"Confidence"), each = nrow(understanding_counts)), NoChange_Decreased_n
= c(understanding_counts\[, "No change / Decreased"\],
confidence_counts\[, "No change / Decreased"\]), Increased_n =
c(understanding_counts\[, "Increased"\], confidence_counts\[,
"Increased"\]), NoChange_Decreased_pct = c(understanding_perc\[, "No
change / Decreased"\], confidence_perc\[, "No change / Decreased"\]),
Increased_pct = c(understanding_perc\[, "Increased"\],
confidence_perc\[, "Increased"\]) )

combined_crosstab

library(ggplot2) library(dplyr)

# Prepare data for plotting

plot_data \<- Teachers %\>% group_by(School_Location) %\>% summarise(
Understanding_Increased = mean(UnderstandingChange_cat == "Increased")
\* 100, Confidence_Increased = mean(ConfidenceChange_cat == "Increased")
\* 100 ) %\>% tidyr::pivot_longer( cols = c(Understanding_Increased,
Confidence_Increased), names_to = "Measure", values_to = "Percentage" )
%\>% mutate( Measure = ifelse(Measure == "Understanding_Increased",
"Understanding", "Confidence") )

# Plot

ggplot(plot_data, aes(x = School_Location, y = Percentage, fill =
Measure)) + geom_bar(stat = "identity", position = "dodge") + labs(
title = "Percentage of Teachers Reporting Increased Understanding &
Confidence", x = "School Location", y = "Percentage (%)" ) +
scale_fill_brewer(palette = "Set1") + theme_minimal(base_size = 14)

#Test normality before choosing what statistical test to carry out \#
For Understanding change shapiro.test(Teachers\$UnderstandingChange)

# For Confidence change

shapiro.test(Teachers\$ConfidenceChange)

#because p value is less than 0.05, this is not normally distributed, so
can't choose T-test and #have to carry out the alternative option
wilcoxan signed-rank test

# Wilcoxon signed-rank test for Understanding (September vs March)

wilcox.test(Teachers$CollectedSeptemberUnderstanding,  Teachers$CollectedMarchUnderstanding,
paired = TRUE)

# Wilcoxon signed-rank test for Confidence (September vs March)

wilcox.test(Teachers$CollectedSeptemberConfidence,  Teachers$CollectedMarchConfidence,
paired = TRUE)

#since pvalue is less than 0.05, we reject null hypothesie and #This
means there is a statistically significant difference between March and
September scores for Understanding and confidence.

# Already calculated change scores

Teachers$UnderstandingChange <- Teachers$CollectedSeptemberUnderstanding -
Teachers$CollectedMarchUnderstanding Teachers$ConfidenceChange \<-
Teachers$CollectedSeptemberConfidence - Teachers$CollectedMarchConfidence

library(dplyr)

change_summary \<- Teachers %\>% group_by(School_Location) %\>%
summarise( MeanUnderstandingChange = mean(UnderstandingChange, na.rm =
TRUE), MeanConfidenceChange = mean(ConfidenceChange, na.rm = TRUE) )

change_summary

library(tidyr) library(ggplot2)

# Convert to long format for plotting

plot_data \<- change_summary %\>% pivot_longer(cols =
c(MeanUnderstandingChange, MeanConfidenceChange), names_to = "Measure",
values_to = "MeanChange") %\>% mutate(Measure = ifelse(Measure ==
"MeanUnderstandingChange", "Understanding", "Confidence"))

# Plot

ggplot(plot_data, aes(x = School_Location, y = MeanChange, fill =
Measure)) + geom_bar(stat = "identity", position = "dodge") + labs(title
= "Average Increment in Understanding and Confidence by School
Location", x = "School Location", y = "Average Change") +
scale_fill_brewer(palette = "Set1") + theme_minimal()
