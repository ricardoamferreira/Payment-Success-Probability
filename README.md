# TrueLayer - Data Scientist - Take-Home Test

In this code test, I approached the problem of developing an ML model that can provide a probability of success for each payment.

I chose to display the three stages in a Jupyter Notebook to make it easier to show the code and results simultaneously, along with comments or explanations of the thought process at each stage.

To solve this problem, I approached the issue in three stages:

## Exploratory Data Analysis (EDA) - [Link](https://github.com/ricardoamferreira/TrueLayer/blob/224cea19b76481bc34327fcb7797a2fa34ed2dfe/EDA.ipynb)

The goal of this first stage was to explore the dataset, identify present labels, and determine which features I could use to train a model.

In this stage, I examined the distribution of the features, assessed how many values were missing, identified their types, and fixed an issue with the labels. I also filtered the dataset down to the two labels that are important (executed and failed) based on TrueLayer's documentation, as these are the only ones that signal that the payment flow is complete.

## Feature Engineering - [Link](https://github.com/ricardoamferreira/TrueLayer/blob/224cea19b76481bc34327fcb7797a2fa34ed2dfe/Feature_Engineering.ipynb)

The goal of this crucial stage was to create additional features based on the existing ones that provide as much information as possible to a model (without any leakage), which is critical for good model performance in a real-life scenario.

Here, I dropped some features which are connected to the target label and would not be present at the moment of payment creation. 

In total, and as an example of what is possible to do, I created six features:
- Count of previous payments per customer (Total, Successful, Failed)
- Payment failure rate
- Time since last payment
- Time since first payment

All of these were created based on information known at the time of payment creation.

Many more features could be created based on what we have in the dataset, but these are a good example and should significantly increase the model's performance.

## Model Training - [Link](https://github.com/ricardoamferreira/TrueLayer/blob/224cea19b76481bc34327fcb7797a2fa34ed2dfe/Model_Training.ipynb)

In the final section, I aimed to train a model that performs well and would be able to reliably predict how likely a payment is to succeed. 
    
The model used was XGBoost which, based on my previous experience with similar problems, is able to perform well relatively quickly and without too much tuning. It can also directly use categorical features, which is a significant advantage for this dataset.

Before training the model, I dropped five features:
- **customer_id** - Using customer_id as part of the model would result in overfitting
- **createdat_ts** - Could result in leakage and overfitting
- **is_failure/is_success** - Related to the target column
- **api_version** - After an initial run, we found that API Version dominated the feature importance chart, which still resulted in good performance in this dataset but would probably not correlate with good performance in any future transactions, especially as new API versions are released. As such, it is preferable not to use it at this stage.

For hyperparameter optimisation, I conducted multiple runs of a random search algorithm, slightly tuning the search space based on the previous runs until I did one final search which is present in the notebook.

I used Stratified K-fold for cross-validation during the search and ROC AUC as the scoring target since it performs well with unbalanced datasets like ours. It also has the advantage of providing a measure of the performance across all probability thresholds, which is ideal since we don't use a threshold to define an outcome at this stage.

After tuning and training, I ran a series of model evaluation tests and saved the results in the notebook.
The final performance is decent, having achieved an F1 score of 0.71 and a ROC AUC score of 0.815, but as this is an unbalanced dataset, a view into the remaining metrics is essential to understand its performance. Especially when looking at the predictive fails, we ended up with 20% precision and 92% recall. While this performance is relative, these results are definitely a solid starting point, and further tuning could be made to increase precision at the cost of some of that recall. 

In terms of feature importance, the approach to feature engineering was validated since three out of the top five features were created during this stage, including the top feature, Fail Rate.

I also included some examples of predictions in the notebook.

This model could then be used not only to directly give a likelihood of success but also to, for example, send different payments into different payment flows based on a probability threshold.
