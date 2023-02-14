![cover_photo](./6_README_files/abnormal-ecg.png)

# ECG Anomaly Detection

_The sport of rock climbing has been steadily increasing in popularity. From 2012-2017, the IBISWorld estimates that from average annual growth for the indoor climbing wall industry was [3.9% in the USA](https://www.ibisworld.com/industry-trends/specialized-market-research-reports/consumer-goods-services/sports-recreation/indoor-climbing-walls.html). In 2015, it ranked 17th out of 111 out of the most popular sports in the United States ( Physical Activity Council and PHIT America). Yet, even with this growth in popularity, most of the international rock climbing websites still lack a rock climbing recommendation system. In this project, I will create a recommendation system for the 8a.nu website that will help climbers identify some unique international climbing objectives._

## 1. Data

The original dataset for "ECG5000" is a 20-hour long ECG downloaded from Physionet. The name is BIDMC Congestive Heart Failure Database(chfdb) and it is record "chf07". It was originally published in "Goldberger AL, Amaral LAN, Glass L, Hausdorff JM, Ivanov PCh, Mark RG, Mietus JE, Moody GB, Peng C-K, Stanley HE. PhysioBank, PhysioToolkit, and PhysioNet: Components of a New Research Resource for Complex Physiologic Signals. Circulation 101(23)". The data was pre-processed in two steps: (1) extract each heartbeat, (2) make each heartbeat equal length using interpolation. This dataset was originally used in paper "A general framework for never-ending learning from time series streams", DAMI 29(6). After that, 5,000 heartbeats were randomly selected. The patient has severe congestive heart failure and the class values were obtained by automated annotation. The data can be accessable using link below:

> - [time series classification website](http://www.timeseriesclassification.com/description.php?Dataset=ECG5000)

## 2. Method

There are six classification models used in this project:

1. **Content-based filter:** Recommending future items to the user that have similar innate features with previously "liked" items. Basically, content-based relies on similarities between features of the items & needs good item profiles to function properly.

2. **Collaborative-based filter:** Recommending products based on a similar user that has already rated the product. Collaborative filtering relies on information from similar users, and it is important to have a large explicit user rating base (doesn't work well for new customer bases).

3. **Hybrid Method:** Leverages both content & collaborative based filtering. Typically, when a new user comes into the recommender, the content-based recommendation takes place. Then after interacting with the items a couple of times, the collaborative/ user based recommendation system will be utilized.

![](./6_README_files/matrix_example.png)

**WINNER:User-based collaborative filtering system**

I choose to work with a user-based collaborative filtering system. This made the most sense because half of the 4 million user-entered climbs had an explicit rating of how many stars the user would rate the climb. Unfortunately, the data did not have very detailed "item features". Every rock climbing route had an area, a difficulty grade, and a style of climbing (roped or none). This would not have been enough data to provide an accurate content-based recommendation. In the future, I would love to experiment using a hybrid system to help solve the problem of the cold-start-threshold.

## 3. Data Wrangling

[Data Wrangling Report](https://github.com/azarbarzin/Springboard/blob/master/Capstone%20Project%202/Notebooks/02_data_wrangling.ipynb)

In this step, data and its features were examined. As the data used was pre-processed: 1)heartbeats were extracted, 2) the length of each heartbeat were set equal using interpolation, 3) There is no missing values, duplicate rows, duplicate columns in the data. The Target feature was examined and its distribution shows base on labels normal and abnormal, the data is considered as a balanced dataset.

## 4. EDA

[EDA Report](https://github.com/azarbarzin/Springboard/blob/master/Capstone%20Project%202/Notebooks/03_exploratory_data_analysis.ipynb)

- The star-rating distributions all checked out to be normal. It is very common with explicit ratings to see a diminished number of low ratings.

![](./6_README_files/star_counts.png)

## 5. Algorithms & Machine Learning

[ML Notebook](https://colab.research.google.com/drive/1kAlvwwJnGcdCAJD8oFokT3gtJF2UnyZP)

I chose to work with the Python [surprise library scikit](http://surpriselib.com/) for training my recommendation system. I tested all four different filtered datasets on the 11 different algorithms provided, and every time the Single Value Decomposition++ (SVD++) algorithm performed the best. It should be noted that this algorithm, although the most accurate is also the most computationally expensive, and that should be taken into account if this were to go into production.

![](./6_README_files/algo.png)

> **\*NOTE:** I choose RMSE as the accuracy metric over mean absolute error(MAE) because the errors are squared before they are averaged which gives the RMSE a higher weight to large errors. Thus, the RMSE is useful when large errors are undesirable. The smaller the RMSE, the more accurate the prediction because the RMSE takes the square root of the residual errors of the line of best fit.\*

**WINNER: SVD++ Algorithm**

This algorithm is an improved version of the SVD algorithm that Simon Funk popularized in the million dollar Netflix competition that also takes into account implicit ratings (_yj_). Using stochastic gradient descent (SGD), parameters are learned using the regularized squared error objective.

![](./6_README_files/forumla.png)

## 6. Which Dataset to choose?

[More details about this process...](https://colab.research.google.com/drive/1kAlvwwJnGcdCAJD8oFokT3gtJF2UnyZP)

After choosing the SVD++ algorithm, I tested the accuracy of all four different filtered datasets. The dataset which filtered out any route names occurring less than 6 times performed the most accurate predictions. Thus, it was chosen to be the dataset I trained on.

> - All of the dataframes displayed discrepancies with the 1 star ratings(This is to be expected due to the inherent skewed positive ratings). Also, the one star ratings are not imperative to this project's goal. It is more important that the 1 star ratings are different enough to be filtered out of the top ten routes recommended to users.
> - Notice the 3-star rating has a fat bulge at the top of the "violin" which indicates its predicting 3-star ratings for some of the true 3-star routes. This was not as prominent in the other dataframes
> - The 1-star rating also has a fatter tail than the other datasets displayed

![](./6_README_files/accuracy.png)

## 7. Coldstart Threshold

[More details about this process...](https://colab.research.google.com/drive/1kAlvwwJnGcdCAJD8oFokT3gtJF2UnyZP)

**Coldstart Threshold**: There is a problem when only using collaborative based filtering: _what to recommend to new users with very little or no prior data?_ Remember, we already set our cold start threshold for the routes by choosing the dataset that filtered out any route occurring less than 6 times. Now, let investigate where to put the threshold for users.

![](./6_README_files/20user_thresh.png)

_It is my hypothesis that the initial filtering of the routes is what affected the RMSE of the users_

> - Increasing the user threshold to 5 would increase the RMSE by .005 & would lose approximately 40% of the data.
> - Increasing the user threshold to 13 would increase the RMSE by .0075 & would lose approximately 60% of the data
> - If there were a larger increase in the RMSE (>= .01) I would trade my users' data for this improvement. However, these improvements are too minuscule to give up 40%-60% of my data to train on. Instead, I voted to keep some of these outliers to help the model train, and will focus on fine tuning my parameters using gridsearch to improve the RMSE

## 7. Predictions

[Final Predictions Notebook](https://colab.research.google.com/drive/1vLkoW_4SYessy_igmJxlVz_jEPlgJ06v)

In the final predictions notebook, the user can enter their user_id number and receive a list of top ten routes recommended to them:

![](./6_README_files/predictions.png)

## 8. Future Improvements

- In the future, I would love to spend more time creating a filtering system, wherein a climber could filter out the type, difficulty of climb, & country before receiving their top ten recommendation

- This recommendation system could also be improved by connecting to the 8a.nu website so that the user could input their actual online ID instead of just their user_id number

- Due to RAM constraints on google colab, I had to train a 65% sample of the original 6x dataset. Without resource limitations, I would love to train on the full dataset. Preliminary tests showed that the bigger the training size, the lower the RMSE. One test showed an increase in sample size could increase the RMSE by .03 (in contrast to the .005 improvement I received when increasing the coldstart threshold)

## 9. Credits

Thanks to Nicolas Hug for his superb surprise library scikit, Colin Brochard for his stellar advice from his Mountain Project recommendation system, and DJ Sarkar for being an amazing Springboard mentor.
