# Predicting crop usage from historical data
This is the final report for capstone project of IBM [Advanced Data Science with IBM Specialization](https://www.coursera.org/specializations/advanced-data-science-ibm).

***This is a pet project developed during a training course and should not be taken as finished work or extract conclusions from it.*** 

Contact me if you are interested in the actual state of this work.

# Use Case
I work in a goberment body in a region of Spain that provides services and IT assistance to farmers. We are specially focused on developing decission support tools for different agricultural purposes. In one of this tools, it would be in a great help to have a model that could predict the next year land usage for a given area base on historical data.

The idea behind the project is that the farmer uses his land in a cyclical way, using patterns, if we can recognize these patterns, we could know which crop is the most likely to be used the following year, or at least the 2-3 crops with the most probability.
In this cyclical use of crops, is a common practice, for example, to leave the land uncultivated one year out of three, to avoid problems with pests, fertilizers, etc. In other cases, there are static crops that generally do not vary (trees, olive, vineyard).
The model does not intend to train a specific case of a farmer, we try to see if, taking a very high number of use cases, we can extract the generality of the patterns of crop usage and given a sequence of years, we can know the most likely crop for the following year.

So this is a **categorical time series problem**, where each step in timeline is represented with a value of a categorical variable. 
As a first attempt, we gathered the last 9 years history for around 2M points tagging for each year the crop code, hoping that we can apply some **LSTM network** to predict the next year crop code. 

# Data Set, ETL and Feature Creation
The data consist in around 2 million points scatterred over all the region territory, the original data file was a shape file with point features, having each point an attribute for each year with the crop code used.
We have 27 different crops: WHEAT, CORN, BARLEY, FLOOR, SUNFLOWER, RAPE, GREEN PEAS, ALFALFA, FORAGE, BEET, VINEYARD, OLIVE, HORTICULTURAL, AROMATIC, FRUITS, DIFFERENT KINDS OF LEAFY TREES, etc

The first feature extraction technique applied was the conversion of the crop codes so the can be used in the LSTM model. Crop codes is a categorical variable and must be converted to numerical values in order to be used as a deep learning model input. 
This conversion has been done using the [feature embedding technique](https://cloud.google.com/solutions/machine-learning/overview-extracting-and-serving-feature-embeddings-for-machine-learning), to convert each crop code to a vector of fixed dimensions.

All this data comes from claims for payment of CAP subsidies, accesible for us as regional goverment agency, but it cannot be shared in the project, but I thin the data exploration and visualization notebook gives a sufficient idea of the data structure.

# Data Exploration, Visualization and Quality Assessment
Review the data and its structure, check the cultivation codes and see the data distributions they have to see how it can affect the model.
The data set is strongly unbalanced. In all areas there are predominant crops, in our case they are the cereals. This makes the dataset have crops with an extreamly high frequency, to make sure that the less frequent crops have enough representation, a dataset has been created with a minimum frequency per crop.

* [Data exploration notebook](course/eda_sampling.ipynb)

# Model Definition, Training and Evaluation
The initial idea was to use LSTM models, these models take advantage of the contextual information of a series, so they are perfect for modeling data with a temporal component. In our case, the problem is that the time series is short (9 years) and there is no access to a previous series. Different models have been made combining LSTM networks with 1-dimensional convolution networks to extract new characteristics, the result is relatively satisfactory.
As a performance metric, the **f1-score on the test set** has been used, this metric balances between precision and recall and makes it more robust in unbalanced datasets.

Because LSTM networks have a longer training time than other deep learning models, especially if regularization parameters that [disable the use of the cuDNN implementation are used](https://keras.io/api/layers/recurrent_layers/lstm/), a reduced dataset has been extracted for model training to speed up the construction of the different iterations of the model. The frequencies of each crop have been maintained to ensure the representativeness of this dataset.
The sample dataset has been divided into three blocks to have separate data for training and testing with percentages 80% / 20%. Train and test sets are used in the keras callback to measures performance during the training and the test set is used for final evaluation.

**Base model**
The first step is to get a base model that gives us information about the minimun performance expected for the deep learning model. Two basic models have been developt:
* One model base using prior knowledge of crop usage, or using just the last year crop as expected category leads to a **0.25 f1-score**.
* Using TPOT, an AutoML library that automatically trains state of art models, in this case a **0.5 f1-score**  is obtained using a ExtraTreesClassifier model. 
65% of crops have changed the last year of the serie from the previous one, this makes so difficult to predict the next year, and a deep-learning model that could interpret this serie could help. 

* [Model training notebook](course/modeling_base_tpot.ipynb)

**LSTM model** (Three versions)

LSTM models are commonly used as predictors or classifiers for time series data. In this case, we are going to trea
In our case, the time series is relatively short, we only have 9 years (8 historical + 1 prediction), and it has the additional problem of being categorical data.
In order to use them, we will code the categorical variables using embeddings, so that each year it will be coded as a 20-dimensional vector.
The model used is similar to those commonly used for text-based prediction (next word prediction, sentiment prediction, etc.). In these models each word is encoded with a vector and the LSTM model is capable of giving an answer based on the context of the series received as input.
In our case, as an analogy, the LSTM model will be learning the patterns of crop use that occur in our region.
The obtained model has a **f1-score of 0.71, outperfoming the base model**.
* [LSTM model](course/modeling_keras.ipynb)

# Tuning and Deployment
During the entry process, the effectiveness of each characteristic of the model has been measured to see how it affects the f1 metric.
Once the definitive model has been selected, Bayesian optimization has been used to obtain the optimal values for these parameters of the model before the model is trained with de full dataset.
The **tunning process improved the model from 0.71 to 0.72**. 
The test curve is stuck int the 0.71 plateau, the [ReduceLROnPlateau](https://keras.io/api/callbacks/reduce_lr_on_plateau/) callback has been used without success, to try to get the model out of the plain reducing the learning rate. 

* [Tunning the model](course/modeling_keras.ipynb#Parameter-Tunning)

# Project presentations
* [Stakeholders](resources/stake_holders_press.odp)
* [Data science team](resources/datascience_team_press.odp)

# Next steps
This is just a starting point, thank you to this course 
there are many ideas that I have to try:
* Add new categorical variables as new embeddings [as seen here](https://github.com/mmortazavi/EntityEmbedding-Working_Example).
* Use a preclustering to create automatic index for each crop to create a new feature.
* Flatten the timeseries to leghten the context so LSTM can have more steps to learn from context.
* Experiment with 1DConv networks. As an alternative to LSTM models, I have also tried 1-dimensional convolution nets. Convolution nets are widely used models for image recognition. These recesses create filters that store information about the most relevant color structures associated with a category (eg, a cat's ear, a bird's beak). This for two dimensions, when we go to one dimension, the convolution network looks for patterns in the time series. The model kernel moves along the timeline and the network filters will store in their weights the information that encodes the most relevant patterns for each crop category·
* Use [lstm-vis library](http://lstm.seas.harvard.edu/) to visualize the LSTM cells state.



About categorical time series:
* (https://silo.tips/download/categorical-time-series-analysis-modelling-monitoring-christian-h-wei-department)