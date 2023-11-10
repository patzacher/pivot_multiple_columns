# Project Title

Brief project description.

## Table of Contents
- [Introduction](#introduction)
- [Libraries](#libraries)
- [Import Data](#import-data)
- [Pivot Wide-to-Long](#pivot-wide-to-long)

## Introduction
Briefly describe the purpose and context of the project.

## Libraries
```R
library(tidyverse)
```
## Import Data
Both of the data sets here have been edited prior to importing. data_raw
contains a lightly edited version of the original data set, in that it contains
all the original data but only two (of six) sets of questions and responses.
data_simple is a heavily edited version of the original data set only
containing ResponseId and two sets of questions and responses (useful for
troubleshooting the pivoting operation in the chunk below).

```R
data_raw <- read.csv("NLaw Student Data Choice Text Codes - edited.csv")
data_simple <- read.csv("ts_data_subset.csv")
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
data_cleaned_simple <- data_raw %>%
  pivot_longer(cols = matches("[A-Za-z][0-9]\\_[0-9]+"), # Select columns relating to the questions
               names_to = c(".value", "item"),
               names_sep = "_") %>% # How we know which items are related
  group_by(ResponseId) %>%
  arrange(ResponseId, Q1) %>% # Sort by ResponseId and Q1
  mutate(n = n(),# Number of rows in each group
         item_num = cumsum(n)/n) %>% # Add 
  select(-item, -n) %>% # Remove `item` and `n` columns
  ungroup()
```R
