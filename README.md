# Pivot Multiple Columns of a Data Frame

## Table of Contents
- [Introduction](#introduction)
- [Libraries](#libraries)
- [Import Data](#import-data)
- [Pivot Wide-to-Long](#pivot-wide-to-long)

## Introduction
This code was used to order survey data, where each respondent was presented with two scenarios each containing 11 questions. The order of the scenarios remained consistent for each participant (e.g., scenario 1 then scenario 2), but the questions within each group were randomized. The goal was to unscramble the data so that questions and the associated participant responses were in a consistent order (i.e., Scenario 1: Q1_1, Q1_2, Q1_3, etc.).  

## Libraries
```R
library(tidyverse)
```
## Import Data
`data` contains the variables `ResponseId` and two sets of questions and responses. 
For example, Q1_1 contains the first question in the scenario
seen by a participant, while Q2_3 is the third question in the second scenario 
seen by a participant. Responses are code the same way. For example, 
A1_1 is the response to Q1_1, A2_2 to Q2_2, etc. 

```R
data <- read.csv("data.csv")
```

## Pivot Wide-to-Long
The pivoting operation consists of several steps.

First, we identify the columns that we wish to pivot from wide-to-long format.
These are the columns that match the regular expression
"[A-Za-z][0-9]\[0-9]+", which means that we are looking for column names that
start with a letter (A-Za-z), are followed by a number (0-9), an underscore
(\), and then one or more numbers (0-9+). For example, the column names "Q1_1"
and "Q2_11" match this pattern, but the column name "ResponseId" does not.

Now we have two pieces of related information (or values) for each participant:
the question and response. These need to go into separate columns in the final
data frame. We supply the ".value" and "item" arguments to names_to, which
tells pivot_longer that part of the column name specifies the value being
measure --in this case the prefix 'Q1', 'A1', "Q2", "A2".
Then we use names_sep = "_" to identify how items belong to each group. For
example, Q1_1 is separated into the group "Q1" and is identified as item 1.
Similarly, A1_1 is separated into the group "A1" and is identified as item 1,
etc.

After the pivoting operation, we clean up the data frame a bit. We group by
ResponseId, and order each of the questions and responses alphabetically using
arrange. This means that the questions seen by each participant are now in the
same order. We then add item_num which numbers each question, this should be
the same for every participant. We remove the extraneous columns item and n,
and finally, ungroup the data frame (as is best practice).

```R
data_cleaned_simple <- data %>%
  pivot_longer(cols = matches("[A-Za-z][0-9]\\_[0-9]+"), # Select columns relating to the questions
               names_to = c(".value", "item"),
               names_sep = "_") %>% # How we know which items are related
  group_by(ResponseId) %>%
  arrange(ResponseId, Q1) %>% # Sort by ResponseId and Q1
  mutate(n = n(),# Number of rows in each group
         item_num = cumsum(n)/n) %>% # Add 
  select(-item, -n) %>% # Remove `item` and `n` columns
  ungroup()
```
