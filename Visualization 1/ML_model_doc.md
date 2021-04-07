---
Title       : Documentation for Machine Learning Model  
Job         : Predict whether a given set of strings (a,b) are match or mismatch. In the given context a--> Product on Receipt and b--> Product matched by Elastic Search 
Output      : Binary output; 0 signifies a mismatch prediction and 1 signifies a match prediction
Model Used  : XGBoost 
Features    : Trained on 16 Numerical features
Model file  : Written in Python and saved in .sav file (SPSS System Data File Format Family) using the Pickle library 
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
  - Jaro-Winkler : Jaro-Winkler modifies the standard Jaro distance metric by putting extra weight on string differences at the start of the strings to be compared. The metric is scaled between 0 (not similar at all) and 1 (exact match)
  - NeedlemanWunsch : Similarity based on pairewise sequence alignment.Sequence alignment is a method of arranging sequences of DNA, RNA, or protein to identify regions of similarity. 
  - Longest common Subsequence(LCS) : A longest common subsequence (LCS) is a common subsequence of two strings of maximum length
  - cosine_similarity_ngrams: Custom implementation of cosine similarity for ngrams where is it takes n as the number of grams or smallest tokens that make up the string
  - cosine_similarity_ngrams : n = 2 and hence bi-gram cosine similarity
  - cosine_similarity_ngrams : n = 3 and hence Tri-gram cosine similarity
  - jaccard_distance: it is defined as the size of the intersection divided by the size of the union of two sets
  - fuzz_token_sort_ratio (bi-gram) : Fuzzy Wuzzy token sort ratio raw raw_score is a measure of the strings similarity as an int in the range [0, 100]. For two strings X and Y, the score is obtained by splitting the two strings into tokens and then sorting the tokens. The score is then the fuzzy wuzzy ratio raw score of the transformed strings. Fuzzy Wuzzy token sort sim score is a float in the range [0, 1] and is obtained by dividing the raw score by 100.
  - fuzz_token_sort_ratio (tri-gram)


* Feature Importance:

The following Bar chart shows the ranked featured that the model has learned the most from.

![Feature importance Plot](https://github.com/farooq96/Vega-lite_visualizations/blob/master/Visualization%201/FEature%20inportance.jpg?raw=true)

---

## System Flow:

* Receipt read by the OCR 
* Line by line OCR output is passed to match_product function
* For each input line, we remove numbers, special characters using regex to ensure only text remains
* We remove any stopwords that may be present in the text
* The input text or the search string is then matched with both the Alias table and the Products table using Elastic Search 
* we do nott use the scores returned by Elastic search because it is machine dependent and are inconsistent across systems
* We reject any products table match that does not yeild a cosine trigram similarity score of 58% and fuzzy token sort ratio of 56% 
* similary, we reject any alias table match that does not yeild cosine trigram scorre of 56% or the fuzzy token sort ratio of 85% 
* From the product table match and the aias table match that go through, we select the best match by giving a priority to the products table match followed by alias table match and we also comapre the fuzzy token sort score for both in case we get matches from product as well as alias table. Whichever has the highest score becomes our matched product
* Once we have the best matched product, we pass it to the xg_boost_model function
* The xg_boost_model functions accepts two input parameters: search string and the matched product and returns 1 if the prediction is that both are a match else it returns 0 
* The function calculates all the 16 metrics for the given two inputs, which are then paased to the model as inputs to make the prediction. 
* The model is trained to penalize false positives and therefore may miss few true positives or correct matches. To compensate the loss, whenever the model predicts as mismatch we compare the fuzzy token sort ratio and cosine bigram similarity after removing all the stop words from the comparing strings. If both the scores are above 80 then we change the class label from mismatch (0) to match (1)

---

## Performance and testing Methodology

* The Raw performance Metrics of the Model alone without any FP reducing Filters is summarized in the follwoing :
* ![Confusion Matrix](./confusion_matrix.jpg?raw=true)

* Algorithm was tested on 30 receipts from Irish and UK market and found 473 correct matches & FP rate --> 5/472*100 = 1.05%

