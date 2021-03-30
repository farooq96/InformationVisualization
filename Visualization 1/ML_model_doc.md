---
Title       : Documentation for Machine Learning Model  
Job         : Predict whether a given set of strings (a,b) are match or mismatch. In the given context a--> Product on Receipt and b--> Product matched by Elastic Search 
Output      : Binary output; 0 signifies a mismatch prediction and 1 signifies a match prediction
Model Used  : XGBoost 
Features    : Trained on 16 Numerical features
Language    : Python
---

## Background  

The model is built to leverage the power of Ensemble learning wherein we have used 16 different similarity metrics to predict whether the two input strings are a match or mismatch.
In recent years, there has been development in the concepts of Ensemble learning and Machine learning algorithm that make use of multiple metrics to capture a wide variety of features from the string which cannot be captured by any one similarity metric. In other words, Levenshtein captures the number of edits required to transform string but it ignores the actual distance between the two strings. On the other hand, a metric like longest common subsequence captures the features as to how many characters are the same in two given string, while cosine similarity adopts a more holistic approach by converting the strings to vectors and then measures the cosine of the angle between two vectors projected in a multi-dimensional space. The smaller the angle, the higher the cosine similarity. The idea that I am trying to put across is that every similarity measure captures one particularly unique feature. What if we could find a way to combine various similarity measures and design a system that is not over-reliant on the cons of one particular similarity metric? it would mean the system is robust and deal with a wide variety of string matching without compromising on the False positive rate ( which we will address in this document as we proceed further) This method is a proved and established method to improve accuracy and performance. 


---

## A. MODEL SUMMARY

* Training Method: 

A collection of about 1000 receipts were run through the existing algorithm and the the results of the product on receipt hereafter refered to as input search string and the product matched were saved in a CSV file. The resultant training dataset contained arounf 8k rows signifying 8k products and their corresponsding matches found by the preexisting algorithm. After this, all the product and the their matches were manually labelleed as 0 (mismatch) or 1 (match) to form the training dataset required for supervised machine learning algorithm. 

Another crucial step in preparation of the training data was calculating the 16 similarity metrics for each record in our previous genrated CSV file. The model we are building is a classification model which makes a prediction or the classification based on a set of features and for the model needs to learn what set of features are needed for it to know that the pair of strings are the same or the pair of strings are different.

* Description of the 16 Features used for training are:

  - Prefix Similarity: Checks if the prefixes are same for the given strings
  - Postfix Similarity: checks if the postfix matches for the given strings
  - get_similarity (custom built function) : Returns simple cosine similarity for the given strings
  - Dice Coefficient (Tri-gram) : Dice Coefficient is 2 * the number of characters common between the two strings divided by the total number of characters in both string
  - Dice Coefficient (Bi-gram)
  - strCmp95: Originally written in C.The strcmp95 function returns a double precision value from 0.0 (total disagreement) to 1.0 (character-by-character agreement)
  - string_similarity : Perform bigram comparison between two strings and return a percentage match in decimal form 
  - Jaro-Winkler : Jaro-Winkler modifies the standard Jaro distance metric by putting extra weight on string differences at the start of the strings to be compared. The metric is scaled between 0 (not similar at all) and 1 (exact match).
  - cosine_similarity_ngrams: Custom implementation of cosine similarity for ngrams where is it takes n as the number of grams or smallest tokens that make up the string
  - cosine_similarity_ngrams : n = 2 and hence bi-gram cosine similarity
  - cosine_similarity_ngrams : n = 3 and hence Tri-gram cosine similarity
  - jaccard_distance: it is defined as the size of the intersection divided by the size of the union of two sets
  - fuzz_token_sort_ratio (bi-gram) : Fuzzy Wuzzy token sort ratio raw raw_score is a measure of the strings similarity as an int in the range [0, 100]. For two strings X and Y, the score is obtained by splitting the two strings into tokens and then sorting the tokens. The score is then the fuzzy wuzzy ratio raw score of the transformed strings. Fuzzy Wuzzy token sort sim score is a float in the range [0, 1] and is obtained by dividing the raw score by 100.
  - fuzz_token_sort_ratio (tri-gram)

---

## DON'T: Point And Click

* Many data processing / statistical analysis packages have graphical
  user interfaces (GUIs)

* GUIs are convenient / intuitive but the actions you take with a GUI
  can be difficult for others to reproduce

* Some GUIs produce a log file or script which includes equivalent
  commands; these can be saved for later examination

* In general, be careful with data analysis software that is highly
  *interactive*; ease of use can sometimes lead to non-reproducible
  analyses

* Other interactive software, such as text editors, are usually fine

---

## DO: Teach a Computer

* If something needs to be done as part of your analysis /
  investigation, try to teach your computer to do it (even if you only
  need to do it once)

* In order to give your computer instructions, you need to write down
  exactly what you mean to do and how it should be done

* Teaching a computer almost guarantees reproducibilty

For example, by hand, you can 

  1. Go to the UCI Machine Learning Repository at
  http://archive.ics.uci.edu/ml/

  2. Download the [Bike Sharing
  Dataset](http://archive.ics.uci.edu/ml/datasets/Bike+Sharing+Dataset)
  by clicking on the link to the Data Folder, then clicking on the
  link to the zip file of dataset, and choosing "Save Linked File
  As..." and then saving it to a folder on your computer

---

## DO: Teach a Computer


Or You can teach your computer to do the same thing using R:

```r
download.file("http://archive.ics.uci.edu/ml/machine-learning-databases/00275/
               Bike-Sharing-Dataset.zip", "ProjectData/Bike-Sharing-Dataset.zip")
```

Notice here that

* The full URL to the dataset file is specified (no clicking through a
  series of links)

* The name of the file saved to your local computer is specified

* The directory in which the file was saved is specified ("ProjectData")

* Code can always be executed in R (as long as link is available)


---

## DO: Use Some Version Control

* Slow things down

* Add changes in small chunks (don't just do one massive commit)

* Track / tag snapshots; revert to old versions

* Software like GitHub / BitBucket / SourceForge make it easy to
  publish results

---
## DO: Keep Track of Your Software Environment

* If you work on a complex project involving many tools / datasets,
  the software and computing environment can be critical for
  reproducing your analysis

* **Computer architecture**: CPU (Intel, AMD, ARM), GPUs, 

* **Operating system**: Windows, Mac OS, Linux / Unix

* **Software toolchain**: Compilers, interpreters, command shell,
  programming languages (C, Perl, Python, etc.), database backends,
  data analysis software

* **Supporting software / infrastructure**: Libraries, R packages,
  dependencies

* **External dependencies**: Web sites, data repositories, remote
  databases, software repositories

* **Version numbers**: Ideally, for everything (if available)
  

---

## DO: Keep Track of Your Software Environment



```r
sessionInfo()
```

```
## R version 3.0.2 Patched (2014-01-20 r64849)
## Platform: x86_64-apple-darwin13.0.0 (64-bit)
## 
## locale:
## [1] en_US.UTF-8/en_US.UTF-8/en_US.UTF-8/C/en_US.UTF-8/en_US.UTF-8
## 
## attached base packages:
## [1] stats     graphics  grDevices utils     datasets  base     
## 
## other attached packages:
## [1] slidify_0.3.3
## 
## loaded via a namespace (and not attached):
## [1] evaluate_0.5.1 formatR_0.10   knitr_1.5      markdown_0.6.3
## [5] stringr_0.6.2  tools_3.0.2    whisker_0.3-2  yaml_2.1.8
```


---

## DON'T: Save Output

* Avoid saving data analysis output (tables, figures, summaries,
  processed data, etc.), except perhaps temporarily for efficiency
  purposes.

* If a stray output file cannot be easily connected with the means by
  which it was created, then it is not reproducible.

* Save the data + code that generated the output, rather than the
  output itself

* Intermediate files are okay as long as there is clear documentation
  of how they were created


---

## DO: Set Your Seed

* Random number generators generate pseudo-random numbers based on an
  initial seed (usually a number or set of numbers)

  - In R you can use the `set.seed()` function to set the seed and to
    specify the random number generator to use

* Setting the seed allows for the stream of random numbers to be
  exactly reproducible

* Whenever you generate random numbers for a non-trivial purpose,
  **always set the seed**


---

## DO: Think About the Entire Pipeline

* Data analysis is a lengthy process; it is not just tables / figures
  / reports

* Raw data &rarr; processed data &rarr; analysis &rarr; report

* How you got the end is just as important as the end itself

* The more of the data analysis pipeline you can make reproducible,
  the better for everyone



---

## Summary: Checklist

* Are we doing good science?

* Was any part of this analysis done by hand?
  - If so, are those parts *precisely* document?
  - Does the documentation match reality?

* Have we taught a computer to do as much as possible (i.e. coded)?

* Are we using a version control system?

* Have we documented our software environment?

* Have we saved any output that we cannot reconstruct from original
  data + code?

* How far back in the analysis pipeline can we go before our results
  are no longer (automatically) reproducible?
