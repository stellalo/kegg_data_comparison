<h1>KEGG Data Comparison</h1>

<h2>👩🏻‍💻 Description</h2>
This repository contains scripts that cross-compare data gathered from RF scripts. Comparison items contains KEGG expression, OOB, MeanDecreaseAccuracy, MeanDecreaseGini, and misclassification rate.
<br />

<h2>🪐 Languages and Utilities Used</h2>

- <b>R Markdown</b>

<h2>🦠 Comparison_KEGG.Rmd Walk-Through:</h2>
This script counts the number of times a KEGG (out of 3176 KEGGs) was ranked top 350 by MeanDecreaseAccuracy (MDA) and MeanDecreaseGini (MDG) values for 10 different characteristics. These 10 characteristics includes temp_at_37 and 9 other antifungal treatment resistance. The final result is ranked. 

<br/>
<br />

- After reading all relevant data files, store them in a new data frame MDA. Note that in the example below, the data stored in MDA is formatted in decreasing order. Keep only the top 350 KEGGs. 

<br/>

```ruby
#Create data frame for 3176 KEGGs and 10 characteristics
MDA <- data.frame(matrix(NA, nrow = 3176, ncol = 10))
#Top 350 Keggs
MDA <- MDA[1:350,]
```

<br/>

- Now, create a new data frame that includes all 3176 KEGGs and 10 characteristic, with an extra column to store total count. Set the row names of that data frame (comparison) to be all 3176 KEGG values. Note that data frame temp_37 stores all 3176 KEGG values.

<br/> 

 ```ruby
#create comparison data frame
comparison <- data.frame(matrix(NA, nrow = 3176, ncol = 11))
rows <- rownames(temp_37)
rownames(comparison) <- rows
```

<br/>

- Loop through the top 350 KEGGs of the 10 characteristics. Use the match() function to find the row index of the KEGG value in variable <b>rows</b>, using that index, set comparison[index,characteristic] = TRUE.
- Remeber that <b>MDA</b> (or MDG) stores only the top 250 KEGGs of each 10 characteristics. Whereas <b>comparison</b> stores all 3176 KEGGs.

<br/>

```ruby
#start comparing!
for (i in 1:10){
  for (j in 1:350){
    index <- match(MDA[j,i], rows)
    comparison[index,i] <- TRUE
  }
}
```

<br/>

- Count the number of TRUEs in each row, this essentially counts the number of times a KEGG was ranked top 350 for the 10 characteristics. Sort by decreasing order. 

<br/>

```ruby
#count the number of Trues for each row
comparison$count <- rowSums(comparison,na.rm = TRUE)
#sort by count
comparison <- comparison[order(-comparison$count),]
write.table(comparison, file="top_KEGG_MDA.txt")
```

<h2>🦠 Comparison_species.Rmd Walk-Through:</h2>
This script counts the number of times a yeast species (out of 835 yeast species) was misclassified for 10 chosen characteristics. The result is again ranked. 

<br/>
<br/>

- Note that the misclassified count for MeanDecreaseAccuracy and MeanDecreaseGini were kept separate in this script, whereas the misclassified count for training and testing OOB were combined. 

<br/>
<br/>

- Read testing and training misclassified species data frame. Find the matching indices of the two data frames and bind them together using the matching indices. Then combine the number of times a species was misclassified for both testing and training.

<br/>

```ruby
#read data, both testing and training
temp_37_train <- read.delim("RF Loop Results/Misclassify_train.txt", sep = " ", header = T)
temp_37_test <- read.delim("RF Loop Results/Misclassify_test.txt", sep = " ", header = T)
match_indices <- match(row.names(temp_37_train), row.names(temp_37_test))
#bind the 2 data frames
temp_37 <- cbind(temp_37_train,temp_37_test[match_indices,-1])
#combine data of train and test
temp_37$misclassify <- temp_37$misclassify + temp_37$`temp_37_test[match_indices, -1]`
```

<br/>

- Create a new data frame that stores all 835 species for all 10 characteristics, plus an extra column to store the total sum a species was misclassified. Store the misclassification data to the correct row of all 10 characteristics one by one. 

<br/>

```ruby
comparison <- data.frame(matrix(NA, nrow = 835, ncol = 11))
#Set the row names to be all 835 species
rows <- rownames(temp_37)
rownames(comparison) <- rows
#first col
comparison$temp_37 <- temp_37$misclassify
#second row
index <- match(row.names(comparison),row.names(X_FC_resistance))
comparison$X_FC_resistance <- X_FC_resistance$misclassify[index]
```
<br/>

- Finally, combine the misclassification count of all 10 characteristics and sort in decreasing order.

<br/>

```ruby
#combine all values and store in misclassify_count
comparison$misclassify_count <- rowSums(comparison,na.rm = TRUE)
#sort data frame
comparison <- comparison[order(-comparison$misclassify_count),]
#write file
write.table(comparison, file="top_misclassifies_species.txt")
```
