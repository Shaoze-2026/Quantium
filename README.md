# Chips Sales Data Analysis
## Backround
You are part of Quantium’s retail analytics team and have been approached by your client, the Category Manager for chips, who wants to better understand the types of customers who purchase chips and their purchasing behaviour within the region.

The insights from your analysis will feed into the supermarket’s strategic plan for the chip category in the next half year.
## Tool Used
R:Data Cleaning, Analysis, and Visualization
## Data File
You can find the original file here:[QVI_purchase_behaviour](QVI_purchase_behaviour.csv),  [QVI_transaction_data](QVI_transaction_data.csv)
## Data Processing
### 1、Library Packages
To extend a programming language's capabilities we need to library packages first.
```
library(tidyverse)
library(data.table)
library(ggplot2)
library(readr)
library(dplyr)
```
### 2、Data Cleaning
Import a file.
```
filePath <- "F:/QVI/"
transactionData <- fread(paste0(filePath,"QVI_transaction_data.csv"))
customerData <- fread(paste0(filePath,"QVI_purchase_behaviour.csv"))
```
To learn about types of different columns
```
str(transactionData)
str(customerData)
```
I noticed that the DATE column was not displayed in the correct data type.
![Type of data](Type_of_DATE.png)
So next step is to revise it. 
```
transactionData$DATE <- as.Date(transactionData$DATE, origin = "1899-12-30")
```
And then should check the right products by examining PROD_NAME.
```
summary(as.factor(transactionData$PROD_NAME))
```
I need to check that these are all chips. So I can do some basic text analysis by summarising the individual words in the product name.
```
productwords <- data.table(unlist(strsplit(unique(transactionData[,PROD_NAME])," ")))
setnames(productwords,"words")
clean_words <- productwords[!grepl("[0-9&/]",words),]
word_freq <- clean_words[, .N, by = words][order(-N)]
head(word_freq)
```
There are different sales products in the dataset. But I only need chips category so I need to remove them.
```
transactionData[, SALSA := grepl("salsa", tolower(PROD_NAME))]
transactionData <- transactionData[SALSA == FALSE,][, SALSA := NULL]
```
Next, I can use summary() to check summary statistics such as mean, min and max values for each
feature to see if there are any obvious outliers in the data and if there are any nulls in any of the columns.
```
summary(transactionData)
```
Filter the dataset to find the outlier
```
transactionData[PROD_QTY == 200,]
filter(transactionData, LYLTY_CARD_NBR == 226000)
transactionData <- filter(transactionData, LYLTY_CARD_NBR != "226000")
summary(transactionData)
```
Check if there are any obivious data issues.
```
transaction_by_date <- table(transactionData$DATE)
head(transaction_by_date)
```
There are only 364 rows which indicates a missing date. Let's found it.
```
complete_data <- transactionData %>%
count(DATE) %>%
complete(DATE = seq(as.Date("2018-07-01"), as.Date("2019-06-30"), by = "day"), fill = list(n = 0))
theme_set(theme_bw())
theme_update(plot.title = element_text(hjust = 0.5))
ggplot(complete_data, aes(x = DATE, y = n)) +
geom_line() +
labs(x = "Day", y = "Number of transactions", title = "Transactions over time") +
scale_x_date(breaks = "1 month") +
theme(axis.text.x = element_text(angle = 90, vjust = 0.5))
```
