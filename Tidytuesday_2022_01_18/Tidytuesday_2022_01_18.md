---
title: "Chocolate Ratings"
author: "Henry Truong"
date: "04/03/2022"
output: 
  html_document:
    keep_md: yes
---



## **1. Introduction**

This work is inspired by **R Data Science: Tidy Tuesday**. This week's topic is about **Chocolate Ratings**. The data can be accessed via **[this link](https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2022/2022-01-18/chocolate.csv)**.

The work will answer three main questions related to:  

  1. How each continent in the world contribute to the origin of chocolate  
  2. What words and how they are used describe the chocolate taste  
  3. Whether we can build a predictive models using text data along with other features to predict chocolate ratings  
  
The content is listed below:  

  1. Introduction
  2. Getting the data
  3. Understanding the data
  4. Cleaning and wrangling the data
  5. Performing exploratory data analysis  
  6. Building models
  7. Conclusions

## **2. Getting the data**

Load packages


```r
library(tidyverse)
library(tidytext)
library(textrecipes)
library(tidymodels)
library(knitr)
```

Read in the data 


```r
chocolate_raw <- read_csv("./data/chocolate.csv")
country_continent <- read_csv("./data/country_continent.csv")
country_continent_map <- read_csv("./data/country_continent_map.csv")
```

Set a seed for reproducibility


```r
set.seed(12345)
```

## **3. Understanding the data**

#### **3.1. Overall structure**

Let's take a look at the data using `View(chocolate_raw)` and `glimpse(chocolate_raw)`.  

***Comment:*** The data set has 2530 rows and 10 columns.  

Let's see how many distinct entries each feature has.


```r
purrr::map_dbl(chocolate_raw, n_distinct)
```

```
                             ref             company_manufacturer 
                             630                              580 
                company_location                      review_date 
                              67                               16 
          country_of_bean_origin specific_bean_origin_or_bar_name 
                              62                             1605 
                   cocoa_percent                      ingredients 
                              46                               22 
  most_memorable_characteristics                           rating 
                            2487                               12 
```

Let's see how many distinct entries of `NA` each feature has.


```r
purrr::map_dbl(chocolate_raw, ~ sum(is.na(.)))
```

```
                             ref             company_manufacturer 
                               0                                0 
                company_location                      review_date 
                               0                                0 
          country_of_bean_origin specific_bean_origin_or_bar_name 
                               0                                0 
                   cocoa_percent                      ingredients 
                               0                               87 
  most_memorable_characteristics                           rating 
                               0                                0 
```

***Comment:*** `ingredients` is the only feature with entries of `NA`.  

#### **3.2. Column `company_location`**

We would like to know the count `n` and the mean rating `mean_rating` of each chocolate company location `company_location`. Here is a list of 5 chocolate company locations with the highest count.


```r
chocolate_by_company_location <- chocolate_raw %>%
  group_by(company_location) %>% 
  summarize(n = n(), 
            mean_rating = mean(rating),
            median_rating = median(rating),
            std_dev_rating = sd(rating))
chocolate_by_company_location %>% 
  slice_max(n, n = 5) %>% 
  kable()
```



|company_location |    n| mean_rating| median_rating| std_dev_rating|
|:----------------|----:|-----------:|-------------:|--------------:|
|U.S.A.           | 1136|    3.190801|          3.25|      0.4242966|
|Canada           |  177|    3.303672|          3.25|      0.4162202|
|France           |  176|    3.258523|          3.25|      0.5154040|
|U.K.             |  133|    3.069549|          3.00|      0.4660372|
|Italy            |   78|    3.230769|          3.25|      0.4677405|

***Comment:***  
  1. USA manufacturers accounts for 45% of observations.  
  2. American and European manufacturers account for 85% of observations.  

Then we can get a list of 5 chocolate company locations with the highest mean ratings.


```r
chocolate_by_company_location %>% 
  filter(n >= 20) %>% 
  slice_max(mean_rating, n = 5) %>% 
  kable()
```



|company_location |   n| mean_rating| median_rating| std_dev_rating|
|:----------------|---:|-----------:|-------------:|--------------:|
|Australia        |  53|    3.358491|          3.50|      0.4087664|
|Denmark          |  31|    3.338710|          3.25|      0.3738783|
|Switzerland      |  44|    3.318182|          3.25|      0.4490417|
|Canada           | 177|    3.303672|          3.25|      0.4162202|
|Brazil           |  25|    3.280000|          3.25|      0.3559026|

#### **3.3. Column `country_of_bean_origin`**

We would like to know the count `n` and the mean rating `mean_rating` of each country where beans originate `country_of_bean_origin`. Here is a list of 5 countries where beans originate with the highest count.


```r
chocolate_by_country_of_bean_origin <- chocolate_raw %>%
  group_by(country_of_bean_origin) %>% 
  summarize(n = n(), 
            mean_rating = mean(rating),
            median_rating = median(rating),
            std_dev_rating = sd(rating))
chocolate_by_country_of_bean_origin %>% 
  slice_max(n, n = 5) %>% 
  kable()
```



|country_of_bean_origin |   n| mean_rating| median_rating| std_dev_rating|
|:----------------------|---:|-----------:|-------------:|--------------:|
|Venezuela              | 253|    3.231225|          3.25|      0.4649344|
|Peru                   | 244|    3.197746|          3.25|      0.4854732|
|Dominican Republic     | 226|    3.215708|          3.25|      0.3854102|
|Ecuador                | 219|    3.164384|          3.25|      0.5122678|
|Madagascar             | 177|    3.266949|          3.25|      0.3876587|

***Comment:***  
  1. North and South American origin accounts for 70% of observations.  
  2. European origin accounts for 0% of observations.  

Then we can get a list of 5 countries where beans originate with the highest mean ratings.


```r
chocolate_by_country_of_bean_origin %>% 
  filter(n >= 20) %>% 
  slice_max(mean_rating, n = 5) %>% 
  kable()
```



|country_of_bean_origin |   n| mean_rating| median_rating| std_dev_rating|
|:----------------------|---:|-----------:|-------------:|--------------:|
|Vietnam                |  73|    3.287671|          3.25|      0.3136679|
|Papua New Guinea       |  50|    3.280000|          3.25|      0.3767476|
|Madagascar             | 177|    3.266949|          3.25|      0.3876587|
|Haiti                  |  30|    3.266667|          3.25|      0.4096536|
|Brazil                 |  78|    3.262821|          3.25|      0.4165751|

***Comment:*** Blend beans have the lowest rating.

#### **3.4. Column `cocoa_percent`**

It should be noted that `cocoa_percent` is of the `chr` type.


```r
chocolate_raw <- chocolate_raw %>% 
  mutate(cocoa_percent = parse_number(cocoa_percent))
```

We can get a 5-number summary of `cocoa_percent`.


```r
chocolate_raw$cocoa_percent %>% summary()
```

```
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
  42.00   70.00   70.00   71.64   74.00  100.00 
```

The following histogram of `cocoa_percent` shows its distribution.


```r
chocolate_raw %>% 
  ggplot(aes(cocoa_percent)) +
  geom_histogram(fill = "steelblue", color = "white", binwidth = 5) +
  labs(x = "Cocoa Percent", y = "Count")
```

<img src="Tidytuesday_2022_01_18_files/figure-html/unnamed-chunk-12-1.png" style="display: block; margin: auto;" />

***Comment:*** The bin of [67.5, 72.5] has the highest count.  

We can also have a look at the correlation between `cocoa_percent` and `rating`.


```r
cor(chocolate_raw$cocoa_percent, chocolate_raw$rating)
```

```
[1] -0.1466896
```

***Comment:*** This is a very weak negative correlation.

#### **3.5. Column `ingredients`**

We would like to know the count `n` and the mean rating `mean_rating` of each collection of chocolate ingredients `ingredients`. Here is a list of 5 collections of chocolate ingredients with the highest count.


```r
chocolate_by_ingredients <- chocolate_raw %>%
  group_by(ingredients) %>% 
  summarize(n = n(), 
            mean_rating = mean(rating),
            median_rating = median(rating),
            std_dev_rating = sd(rating))
chocolate_by_ingredients %>% 
  slice_max(n, n = 5) %>% 
  kable()
```



|ingredients  |   n| mean_rating| median_rating| std_dev_rating|
|:------------|---:|-----------:|-------------:|--------------:|
|3- B,S,C     | 999|    3.278529|          3.25|      0.3921026|
|2- B,S       | 718|    3.229457|          3.25|      0.3996328|
|4- B,S,C,L   | 286|    3.213287|          3.25|      0.4352435|
|5- B,S,C,V,L | 184|    3.089674|          3.00|      0.5470756|
|4- B,S,C,V   | 141|    2.975177|          3.00|      0.4573848|

***Comment:***  
  1. Right at the 6th place is the collection of ingredients with the value of `NA`.  
  2. `B` is the ingredient present in all chocolates.  

Because we do not have too many distinct collections of ingredients, we can get the whole list of collections of ingredients ordered by the mean rating.  


```r
chocolate_by_ingredients %>% 
  filter(n >= 20) %>% 
  arrange(desc(n)) %>% 
  kable()
```



|ingredients  |   n| mean_rating| median_rating| std_dev_rating|
|:------------|---:|-----------:|-------------:|--------------:|
|3- B,S,C     | 999|    3.278529|          3.25|      0.3921026|
|2- B,S       | 718|    3.229457|          3.25|      0.3996328|
|4- B,S,C,L   | 286|    3.213287|          3.25|      0.4352435|
|5- B,S,C,V,L | 184|    3.089674|          3.00|      0.5470756|
|4- B,S,C,V   | 141|    2.975177|          3.00|      0.4573848|
|NA           |  87|    2.810345|          3.00|      0.6819576|
|2- B,S*      |  31|    2.959677|          3.00|      0.4429860|
|4- B,S*,C,Sa |  20|    3.112500|          3.25|      0.5819376|

***Comment:***  
  1. `3- B,S,C` have the highest mean rating and number of reviews.  
  2. Top 5 of `n` is exactly the same as top 5 of `mean_rating`.  

#### **3.6. Column `most_memorable_characteristics`**

We would like to know the count `n` and the mean rating `mean_rating` of each word describing chocolate taste. which can be extracted from `most_memorable_characteristics`. Here is a list of 10 words describing chocolate taste with the highest count.


```r
chocolate_by_characteristic_word <- chocolate_raw %>% 
  select(most_memorable_characteristics, rating) %>% 
  unnest_tokens(characteristic_word, most_memorable_characteristics,
                token = "regex", pattern = ",") %>% 
  mutate(characteristic_word = characteristic_word %>% str_squish()) %>% 
  group_by(characteristic_word) %>% 
  summarize(n = n(),
            mean_rating = mean(rating),
            median_rating = median(rating),
            std_dev_rating = sd(rating))
chocolate_by_characteristic_word %>% 
  slice_max(n, n = 10) %>% 
  kable()
```



|characteristic_word |   n| mean_rating| median_rating| std_dev_rating|
|:-------------------|---:|-----------:|-------------:|--------------:|
|sweet               | 273|    3.053114|          3.00|      0.3514929|
|nutty               | 261|    3.292146|          3.25|      0.3843646|
|cocoa               | 252|    3.380952|          3.50|      0.4129554|
|roasty              | 214|    3.198598|          3.25|      0.3438567|
|earthy              | 190|    3.031579|          3.00|      0.3530706|
|creamy              | 189|    3.478836|          3.50|      0.4294067|
|sandy               | 170|    3.095588|          3.00|      0.3689838|
|fatty               | 166|    3.075301|          3.00|      0.3778627|
|floral              | 146|    3.243151|          3.25|      0.4141310|
|intense             | 141|    3.161348|          3.25|      0.4463490|

***Comment:*** We can see that in this list positive words describing chocolate taste have their mean rating higher than the overall mean rating and negative words describing chocolate taste have their mean rating lower than the overall mean rating.   

Here is a list of 5 words describing chocolate taste with the highest mean rating.


```r
chocolate_by_characteristic_word %>%
  filter(n >= 20) %>%
  slice_max(mean_rating, n = 5) %>% 
  kable()
```



|characteristic_word |   n| mean_rating| median_rating| std_dev_rating|
|:-------------------|---:|-----------:|-------------:|--------------:|
|complex             |  52|    3.567308|           3.5|      0.3096305|
|creamy              | 189|    3.478836|           3.5|      0.4294067|
|cherry              |  38|    3.473684|           3.5|      0.3277146|
|smooth              |  27|    3.472222|           3.5|      0.4564355|
|orange              |  20|    3.462500|           3.5|      0.3371221|

***Comment:***  
  1. The word `complex` which seems to be neutral turns out to be the most positive word to describe the chocolate taste.  
  2. Words related to fruit flavor is likely to be positive words to describe the chocolate taste.  

Here is a list of 5 words describing chocolate taste with the lowest mean rating.


```r
chocolate_by_characteristic_word %>%
  filter(n >= 20) %>%
  slice_min(mean_rating, n = 5) %>% 
  kable()
```



|characteristic_word |  n| mean_rating| median_rating| std_dev_rating|
|:-------------------|--:|-----------:|-------------:|--------------:|
|off notes           | 26|    2.576923|          2.50|      0.2526780|
|burnt               | 22|    2.715909|          2.75|      0.4103965|
|bitter              | 69|    2.721014|          2.75|      0.4746055|
|rubber              | 22|    2.727273|          2.75|      0.2302831|
|pungent             | 33|    2.742424|          2.75|      0.3092635|

***Comment:*** All the words in this list are clearly negative in terms of the chocolate taste.  

#### **3.7. Column `rating`**

We can get a 5-number summary of `rating`.


```r
chocolate_raw$rating %>% summary()
```

```
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
  1.000   3.000   3.250   3.196   3.500   4.000 
```

The following histogram of `rating` shows its distribution.


```r
chocolate_raw %>% 
  ggplot(aes(rating)) +
  geom_histogram(fill = "steelblue", color = "white", binwidth = 0.25) +
  labs(x = "Rating", y = "Count")
```

<img src="Tidytuesday_2022_01_18_files/figure-html/unnamed-chunk-20-1.png" style="display: block; margin: auto;" />

***Comment:*** The bin with 3.5 as its midpoint has the highest count.

## **4. Cleaning and wrangling the data**

We add 2 columns which contains the information on the continent of `company_location` and `country_of_bean_origin`. Next, we split `ingredients` into 2 columns: (1) a column with the number of ingredients `num_ingreds` and (2) a column with a collection of ingredients `ingreds`.


```r
chocolate_raw <- chocolate_raw %>% 
  left_join(country_continent, c("company_location" = "country")) %>% 
  rename(comp_loc_continent = continent) %>% 
  left_join(country_continent, c("country_of_bean_origin" = "country")) %>% 
  rename(continent_of_bean_origin = continent) %>% 
  mutate(ingredients = str_replace_all(ingredients, "S\\*", "OS")) %>% 
  separate(ingredients, c("num_ingreds", "ingreds"), sep = "-")
```

***Note:*** `country_continent` was read in from the section of ***"Getting the data"***.

## **5. Performing exploratory data analysis**

### **5.1. Question 1:**

We would like to know how each continent in the world contribute to the origin of chocolate collected in this data set.  

***Note:*** `country_continent_map` was read in from the section of ***"Getting the data"***. 


```r
# Create a data frame with information on continents and their geographic location
world <- map_data("world") %>% 
  as_tibble() %>% 
  left_join(country_continent_map, by = c("region" = "country")) %>% 
  filter(!is.na(continent))
# Create a data frame with information on continents and their contribution on the chocolate origin
results <- chocolate_raw %>% 
  count(continent_of_bean_origin)
# Join two data frames into one
results_world <- left_join(world, results,
                            by = c("continent" = "continent_of_bean_origin")) %>% 
  mutate(n = replace_na(n, 0))
# Create a data frame with info on labels and their location on the next visualization
continent_label <- tibble(continent = c("Africa", "Asia", "Europe", "North America", "Oceania", "South America"),
                          long = c(25, 90, 90, -92.5, 135, -60),
                          lat = c(4, 35, 62.5, 50, -25, -20))
# Create a theme for the next visualization
plain <- theme(
  axis.text = element_blank(),
  axis.line = element_blank(),
  axis.ticks = element_blank(),
  panel.border = element_blank(),
  panel.grid = element_blank(),
  axis.title = element_blank(),
  plot.background = element_rect(fill = "gray"),
  legend.background = element_rect(fill = "gray"),
  panel.background = element_rect(fill = "gray"),
  strip.background = element_rect(fill = "gray")
)
# Create a visualization
ggplot(data = results_world,
       aes(x = long, y = lat)) +
  coord_fixed(1.5) +
  geom_polygon(aes(fill = n, group = group)) +
  geom_text(aes(label = continent), data = continent_label, 
            fontface = "bold") +
  scale_fill_distiller(NULL, palette = "Spectral") +
  labs(subtitle = "") +
  plain
```

<img src="Tidytuesday_2022_01_18_files/figure-html/unnamed-chunk-22-1.png" style="display: block; margin: auto;" />

***Comment:*** According to this data set,  
  1. South American and North America are at the 1st and 2nd place respectively.  
  2. It is surprising that Africa which is famous for chocolate beans is at the 3rd place.  
  3. Europe has no contribution to the origin of chocolate beans.  
  
### **5.2. Question 2:**

We would like to know how words which describe the chocolate taste are used in this data set.  

We can create a visualization which shows words that positively and negatively describe the chocolate taste on the basis of their frequency.  

***Note:*** If the mean rating of a word is higher than the overall mean rating, the word gives a positive (or good) description. If the mean rating of a word is lower than the overall mean rating, the word gives a negative (or bad) description.


```r
mean_rating <- chocolate_raw$rating %>% mean()
chocolate_raw %>% 
  select(most_memorable_characteristics, rating) %>% 
  unnest_tokens(characteristic_word, most_memorable_characteristics,
                token = "regex", pattern = ",") %>% 
  mutate(characteristic_word = characteristic_word %>% str_squish()) %>% 
  mutate(rating_cat = case_when(rating > mean_rating ~ "Good",
                                rating <= mean_rating ~ "Bad")) %>% 
  count(rating_cat, characteristic_word, sort = TRUE) %>% 
  reshape2::acast(characteristic_word ~ rating_cat,
                  value.var = "n", fill = 0) %>% 
  wordcloud::comparison.cloud(max.words = 100, random.order = FALSE,
                              colors = c("darkred", "darkgreen"),
                              title.bg.colors = "white",
                              rot.per = 0.25)
```

<img src="Tidytuesday_2022_01_18_files/figure-html/unnamed-chunk-23-1.png" style="display: block; margin: auto;" />

***Comment:***  
  1. It seems that words related to fruit flavors give chocolate good ratings.  
  2. Words which are clearly negative in terms of the chocolate taste are used frequently to give chocolates bad ratings.  
  3. This leads to the next question of whether we can use these words along with other features to predict the ratings of chocolates.  

### **5.3. Question 3:**

We would like to build a predictive models using text data along with other features to predict chocolate rating.  

## **6. Building models**

First, we need to split the data into training and test sets. To compare different models' performance, we apply cross validation on the training set.  

#### **Get training, cross validation and test sets**


```r
# Create training and test sets
choco_split <- initial_split(chocolate_raw)
choco_train <- training(choco_split)
choco_test <- testing(choco_split)
# Create a 10-fold cross validation set from the training set
choco_folds <- vfold_cv(choco_train)
```

### **6.1. First models**

We will build two very first models which are SVM and random forest to predict chocolate ratings using unigram extracted from the `most_memorable_characteristics` feature.

#### **Set up a workflow to train a predictive model**


```r
# Set up a recipe
choco_rec <- recipe(rating ~ most_memorable_characteristics, 
                    data = choco_train) %>%
  step_tokenize(most_memorable_characteristics) %>%
  step_tokenfilter(most_memorable_characteristics, 
                   max_tokens = 200) %>%
  step_tf(most_memorable_characteristics)
# Set up model specification
## Random forest 
rf_spec <- rand_forest(trees = 500) %>%
  set_mode("regression")
## Support vector machine
svm_spec <- svm_linear() %>%
  set_mode("regression")
# Create a workflow
## Support vector machine
svm_wf <- workflow(choco_rec, svm_spec)
## Random forest
rf_wf <- workflow(choco_rec, rf_spec)
```

Now we have specified the workflow for fitting 2 models.  

#### **Fit the model**


```r
# Set up control options of fitting 
contrl_preds <- control_resamples(save_pred = TRUE)
# Fit the model using the cross validation set
## Support vector machine
svm_rs <- fit_resamples(
  svm_wf,
  resamples = choco_folds,
  control = contrl_preds
)
## Random forest
ranger_rs <- fit_resamples(
  rf_wf,
  resamples = choco_folds,
  control = contrl_preds
)
```

Now we have 2 fitted models of SVM and random forest.  

#### **Collect the performance metrics of the trained model**


```r
# Support vector machine
collect_metrics(svm_rs)
```

```
# A tibble: 2 x 6
  .metric .estimator  mean     n std_err .config             
  <chr>   <chr>      <dbl> <int>   <dbl> <chr>               
1 rmse    standard   0.340    10 0.00620 Preprocessor1_Model1
2 rsq     standard   0.406    10 0.0166  Preprocessor1_Model1
```

```r
# Random forest
collect_metrics(ranger_rs)
```

```
# A tibble: 2 x 6
  .metric .estimator  mean     n std_err .config             
  <chr>   <chr>      <dbl> <int>   <dbl> <chr>               
1 rmse    standard   0.341    10 0.00641 Preprocessor1_Model1
2 rsq     standard   0.406    10 0.0197  Preprocessor1_Model1
```

***Comment:***  
  1. SVM model is slightly better than random forest model in terms of both RMSE (root mean squared error) and RSQ (R squared).  
  2. The value of R squared of both models is not really high.  

#### **Visualize the predicted results**

We can visualize how well and differently these two models predict chocolate ratings in the test set.  


```r
bind_rows(
  collect_predictions(svm_rs) %>%
    mutate(mod = "SVM"),
  collect_predictions(ranger_rs) %>%
    mutate(mod = "ranger")
) %>%
  ggplot(aes(rating, .pred, color = id)) +
  geom_abline(lty = 2, color = "gray50", size = 1.2) +
  geom_jitter(width = 0.5, alpha = 0.5, show.legend = FALSE) +
  facet_wrap(vars(mod), ncol = 1) +
  coord_fixed() +
  labs(x = "True rating", y = "Predicted rating")
```

<img src="Tidytuesday_2022_01_18_files/figure-html/unnamed-chunk-28-1.png" style="display: block; margin: auto;" />

***Comment:*** It is now concrete that   
  1. Both models do not have an excellent performance on rating prediction.  
  2. SVM model is slightly better than random forest model

From now on, we will use this SVM model as a baseline model. Now we want to compare it with two other models:  
  1. Model using compound words, rather than unigram, as a predictor  
  2. Model using a combination of text and other features as predictors

### **6.2. Model using compound words, rather than unigrams, as a predictor**

#### **Set up a workflow to train a predictive model**


```r
# Create a function extracting compound words as tokens
split_comma <- function(x) {
  str_split(x, ",") %>% map(str_squish)
}
# Set up a recipe
choco_rec_compound_words <- recipe(rating ~ most_memorable_characteristics,
                                   data = choco_train) %>% 
  step_tokenize(most_memorable_characteristics, 
                custom_token = split_comma) %>% 
  step_tokenfilter(most_memorable_characteristics, max_tokens = 200) %>% 
  step_tf(most_memorable_characteristics)
# Set up model specification
svm_spec <- svm_linear() %>%
  set_mode("regression")
# Create a workflow
svm_compound_words_wf <- workflow(choco_rec_compound_words, svm_spec)
```

#### **Fit the model**


```r
# Set up control options of fitting
contrl_preds <- control_resamples(save_pred = TRUE)
# Fit the model
svm_compound_words_rs <- fit_resamples(
  svm_compound_words_wf,
  resamples = choco_folds,
  control = contrl_preds
)
```

#### **Collect the performance metrics of the trained model**


```r
collect_metrics(svm_compound_words_rs)
```

```
# A tibble: 2 x 6
  .metric .estimator  mean     n std_err .config             
  <chr>   <chr>      <dbl> <int>   <dbl> <chr>               
1 rmse    standard   0.363    10  0.0102 Preprocessor1_Model1
2 rsq     standard   0.325    10  0.0149 Preprocessor1_Model1
```

***Comment:*** With the same cross validation set, the performance of the model using word compounds as tokens `rmqe = 0.363` is lower than the one of the baseline model `rmqe = 0.340`.  

Therefore, in the model using a combination of text and other features as predictors, we will use unigrams as tokens for text data. 

### **6.3. Model using a combination of text and other features as predictors**

In this model, we are going to use the following as predictors:  
  1. unigrams of `most_memorable_characteristics`  
  2. continent of the chocolate company `comp_loc_continent`  
  3. each ingredient of `ingreds`  
  4. cocoa percent `cocoa_percent`  
  5. continent of chocolate beans `continent_of_bean_origin`  

#### **Set up a workflow to train a predictive model**


```r
# Set up a recipe
choco_rec_multi <- recipe(rating ~ comp_loc_continent + 
                            continent_of_bean_origin +
                            ingreds + cocoa_percent +
                            most_memorable_characteristics,
                          data = choco_train) %>% 
  step_dummy(comp_loc_continent) %>% 
  step_dummy(continent_of_bean_origin) %>% 
  step_unknown(ingreds) %>% 
  step_tokenize(ingreds) %>% 
  step_tf(ingreds) %>% 
  step_tokenize(most_memorable_characteristics) %>%
  step_tokenfilter(most_memorable_characteristics, 
                   max_tokens = 200) %>%
  step_tf(most_memorable_characteristics) %>% 
  step_normalize(all_predictors())
# Set up model specification
svm_spec <- svm_linear() %>%
  set_mode("regression")
# Create a workflow
svm_multi_wf <- workflow(choco_rec_multi, svm_spec)
```

#### **Fit the model**


```r
# Set up control options of fitting
contrl_preds <- control_resamples(save_pred = TRUE)
# Fit the model using the cross validation set
svm_multi_rs <- fit_resamples(
  svm_multi_wf,
  resamples = choco_folds,
  control = contrl_preds
)
```

#### **Collect the performance metrics of the trained model**


```r
collect_metrics(svm_multi_rs)
```

```
# A tibble: 2 x 6
  .metric .estimator  mean     n std_err .config             
  <chr>   <chr>      <dbl> <int>   <dbl> <chr>               
1 rmse    standard   0.329    10 0.00408 Preprocessor1_Model1
2 rsq     standard   0.455    10 0.0164  Preprocessor1_Model1
```

***Comment:*** The performance of this model `rmse = 0.329` is better than the baseline model's performance `rmse = 0.340`.  

Therefore, we will use this model as the final one.

### **6.4. Final model**

After selecting the SVM model with a combination of text and mentioned features as predictors, we finally fit the model using the training set.


```r
# Fit the final model using the training set
multi_final_fitted <- last_fit(svm_multi_wf, choco_split)
# Collect the performance metrics
collect_metrics(multi_final_fitted)
```

```
# A tibble: 2 x 4
  .metric .estimator .estimate .config             
  <chr>   <chr>          <dbl> <chr>               
1 rmse    standard       0.339 Preprocessor1_Model1
2 rsq     standard       0.454 Preprocessor1_Model1
```

***Comment:*** RMSE in the case of the test set, `0.339`, is somewhat similar to RMSE in the case of the cross validate set, `0.329`. Therefore, the model is not overfit.


```r
# Extract the final model's parameters
multi_final_wf <- extract_workflow(multi_final_fitted)
# Save the final model for future uses
saveRDS(multi_final_wf, "multi_final_wf.rds")
# Make a prediction
tibble(
  prediction = pull(predict(multi_final_wf, choco_test[55, ]), .pred),
  truth = choco_test$rating[55]
)
```

```
# A tibble: 1 x 2
  prediction truth
       <dbl> <dbl>
1       3.03     3
```

We can have a look at features with the most contribution to the chocolate ratings.  


```r
multi_final_wf %>% 
  tidy() %>%
  filter(term != "Bias") %>%
  group_by(estimate > 0) %>%
  slice_max(abs(estimate), n = 10) %>%
  ungroup() %>%
  mutate(term = str_remove(term, "tf_most_memorable_characteristics_")) %>%
  ggplot(aes(estimate, fct_reorder(term, estimate), fill = estimate < 0)) +
  geom_col(alpha = 0.8) +
  scale_fill_manual(labels = c("High ratings", "Low ratings"),
                    values = c("darkgreen", "darkred")) +
  labs(y = NULL, fill = "Contribution to:")
```

<img src="Tidytuesday_2022_01_18_files/figure-html/unnamed-chunk-37-1.png" style="display: block; margin: auto;" />

***Comment:***  
  1. In the top 20 variables which most contribute to chocolate ratings, the vanilla ingredient `tf_ingreds_v` is the only one not to be words describing the chocolate taste.  
  2. We can also see the presence of `vanilla` term describing the chocolate taste in the group of lowering the ratings.  
  3. Text features have more contribution than the others do.  

## **7. Conclusions**

**Question 1:**  
  1. South American and North America are at the 1st and 2nd place respectively.  
  2. It is surprising that Africa which is famous for chocolate beans is at the 3rd place.  
  3. Europe has no contribution to the origin of chocolate beans.  

**Question 2:**  
  1. It seems that words related to fruit flavors give chocolate good ratings.  
  2. Words which are clearly negative in terms of the chocolate taste are used frequently to give chocolates bad ratings.  

**Question 3:**  
  1. We can build a SVM model with a combination of text and mentioned features as predictors to predict chocolate ratings with RMSE of `0.329` on the cross validation set and `0.339` on the test set.  
  2. In the top 20 variables which most contribute to chocolate ratings, the vanilla ingredient `tf_ingreds_v` is the only one not to be words describing the chocolate taste.  
  3. We can also see the presence of `vanilla` term describing the chocolate taste in the group of lowering the ratings.  
  4. Text features have more contribution than the others do.
