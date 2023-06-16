
(For additional context, read up on our [previous deep dive](https://nickswetucsd.github.io/poweroutages/) into power outages.)

<br>

## REINTRODUCTION 🔋

Nobody likes a blackout. They seem to always crop up when it's least convenient, leaving cities high and dry with malfunctioning traffic lights and citizens floundering with a lack of resources for poor laptop battery life.

Last time, we investigated the ins and outs of over **1500 rows** of U.S. power outage data¹ , courtesy of **Purdue University’s LASCI (Laboratory For Advancing Sustainable Critical Infrastructure)**. We found some interesting conclusions:



- Outage data has many factors and causes🌪️ that make it complex to predict, and investigating that data as individual points or as aggregates can reveal contrasting trends.

-  Four states deviate from what is expected of the distribution of the greater United States in terms of OUTAGE.DURATION⏱.

- There is some overlap in the states that more commonly have longer power outages and the states who have a high number of mean CUSTOMERS.AFFECTED🚶 per outage.



**But can we take these conclusions and turn them into something actionable? Perhaps we can!**


¹ Link to data source found [here.](https://engineering.purdue.edu/LASCI/research-data/outages/outagerisks)

<br>

---

## FRAMING THE PROBLEM 🔋


We pose the following prediction problem:
> ### Can we reasonably predict the magnitude of a blackout from other blackout features?

... and here's how we would go about it.



- **RESPONSE VARIABLE: CUSTOMERS.AFFECTED 🚶**
>  For the sake of continuity and expanding our investigation, we thought that CUSTOMERS.AFFECTED 🚶 would be an apt response variable. CUSTOMERS.AFFECTED 🚶 is a column from our dataset that denotes the approximate number of customers affected by a particular blackout. This is great way to define what "magnitude" of a blackout should mean in our question, as we would label blackouts that affect more people as "more severe"!
<br>

- **PREDICTOR CLASS: RandomForestRegressor**
> Since our response variable of CUSTOMERS.AFFECTED 🚶 uses *continuous numerical data*, we'll need to use a regressor instead of a classifier. 
We'll opt for [RandomForestRegressor](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestRegressor.html) from sk.learn, since we are working with a large dataset and we're not necessarily concerned about interpreting how specific splits are made, like how a DescisionTreeRegressor may operate. 
<br>

- **QUALITY METRIC: RMSE**
> To measure how "good" our model is, we'll use the classic regression metric of **Root Mean Squared Error (RMSE).** RMSE will give us the total squared error from each actual and predicted value, which holds up for assessing nonlinear prediction models like ours; this is preferable to other metrics like **R-Squared**, which are *invalid* for assessing nonlinear prediction models. Lower RMSE indicates "better" predictions, while higher RMSE indicates "worse" predictions.
<br>

Now that our sails are set, we can use the cleaned data from our previous project ***to start tinkering with features and build a baseline model.***

<br>

---

## BASELINE MODEL 🔋

Two things we immediately thought of that related to the numbers of customers a blackout affects were **how long** the blackouts were and at **what temperatures** the blackouts were occurring in. After all, everybody *knows* that blackouts are (probably) more severe in summer due to hot weather.

The closest analogue for these factors that we could find in our dataset were the two following variables:

- **OUTAGE.DURATION** ⏱:
>"Duration of outage event (in minutes)." Used in our previous analysis as a measure of power outage severity. **Measured as a *continuous, numerical variable.***
<br>


- **CLIMATE.CATEGORY** 🌤️:
>"Represents the climate episodes corresponding to the years. The categories—'Warm', 'Cold' or 'Normal' episodes of the climate are based on a threshold of ± 0.5 °C for the Oceanic Niño Index (ONI)." **Measured as an *ordinal, categorical variable.***
<br>



Here is what this data might look like before adjustment:

<div style="overflow-x:auto;">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>CUSTOMERS.AFFECTED</th>
      <th>OUTAGE.DURATION</th>
      <th>CLIMATE.CATEGORY</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>70000.0</td>
      <td>3060.0</td>
      <td>normal</td>
    </tr>
    <tr>
      <th>1</th>
      <td>70000.0</td>
      <td>3000.0</td>
      <td>cold</td>
    </tr>
    <tr>
      <th>2</th>
      <td>68200.0</td>
      <td>2550.0</td>
      <td>normal</td>
    </tr>
    <tr>
      <th>3</th>
      <td>250000.0</td>
      <td>1740.0</td>
      <td>warm</td>
    </tr>
    <tr>
      <th>4</th>
      <td>60000.0</td>
      <td>1860.0</td>
      <td>cold</td>
    </tr>
  </tbody>
</table>
</div>

<br>

We proceed by tweaking these two features so that they are suitable to predict CUSTOMERS.AFFECTED 🚶. This is done inside of a [**PipeLine object**](https://scikit-learn.org/stable/modules/generated/sklearn.pipeline.Pipeline.html) from sk.learn.  


- OUTAGE.DURATION ⏱: is fine as is; no need to change it.

- CLIMATE.CATEGORY 🌤 will undergo **One-Hot Encoding (OHE)**, which transforms a *single* categorical feature into *multiple* numerical features through the process of creating seperate data columns for each potential category in the original variable. How it would be applied here is: for each of the three potential categories in CLIMATE.CATEGORY 🌤 ('Warm', 'Normal', and 'Cold'), we will create a seperate a column that will contain only zeroes and ones (1 if the blackout falls into the potential category, 0 if the blackout does not). We will remove the original categorical variable from the feature set. We will additionally remove one of the three new features in order to, more technically, shield ourselves from the effects of multicollinearity.

Our features are now prepared.

<div style="overflow-x:auto;">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>CLIMATE.CATEGORY_normal</th>
      <th>CLIMATE.CATEGORY_warm</th>
      <th>OUTAGE.DURATION</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.0</td>
      <td>1.0</td>
      <td>1406.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1.0</td>
      <td>0.0</td>
      <td>5910.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.0</td>
      <td>0.0</td>
      <td>235.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.0</td>
      <td>1.0</td>
      <td>13140.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1.0</td>
      <td>0.0</td>
      <td>347.0</td>
    </tr>
  </tbody>
</table>
</div>

<br>

### SUMMARY OF FEATURES BEING USED IN BASELINE MODEL:

- 1 CONTINUOUS, NUMERICAL FEATURE.
> OUTAGE.DURATION ⏱


- 2 DISCRETE, NUMERICAL FEATURES AS THE RESULT OF CATEGORICAL FEATURE OHE.
> CLIMATE.CATEGORY (NORMAL) 🌤 <br>
> CLIMATE.CATEGORY (WARM) 🌤


- THERE ARE 3 TOTAL FEATURES.


<br>

### RESULTS:

```
BASELINE MODEL___
Training RMSE: 128375.64574679772
Testing RMSE: 379425.60552213225
```

It's not perfect. The training RMSE is *significantly* lower than our testing RMSE, which implies that our model might be **overfit** to our current data. Regardless, both errors are still quite high. 

This implies that the best practice going forward is to **limit the depth of training** (by considering better hyperparameter configurations in RandomForestRegressor) and, perhaps, **increase the breadth of training** (offer different features to get better predictive resolution).

Let's see if our predictions for the number of CUSTOMERS.AFFECTED 🚶 in power outages improve with these changes.


<br>

---


## FINAL MODEL 🔋


After combing our dataset for a bit longer, we came across a few other variables that had predictive potential. In addition to the features in our basic model, we use feature-engineered versions of:

- **CAUSE.CATEGORY** 🌪:
>"Categories of all the events causing the major power outages." Used in our previous analysis as a measure of power outage severity. Potential vallues are 'Equipment Failure', 'Fuel Supply Emergency', 'Intentional Attack', 'Islanding', 'Public Appeal', 'Severe Weather', and 'System Operability Disruption'.  **Measured as a *nominal, categorical variable.***
<br>
Could be useful because more severe causes (hurricanes) might affect more customers than less severe causes (power grid strain).
<br>


- **OUTAGE.START.TIME** 🏁️:
>"Indicates the time of the day when the outage event started." Uses pd.Timestamp format after data cleaning. **Measured as a *continuous, numerical variable.***
<br>
Could be useful because certain times of day (afternoon) are warmer and might spark more severe blackouts than at other times of day (evening).
<br>


- **POPULATION** 👥️:
>"Population in the U.S. state in a year." Includes customers and non-customers in the count. **Measured as a *discrete, numerical variable.***
<br>
Could be useful because areas with more customers affected somewhat correspond to areas with more citizens in general.
<br>

Here is what this data might look like before adjustment:

<div style="overflow-x:auto;">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>CUSTOMERS.AFFECTED</th>
      <th>OUTAGE.DURATION</th>
      <th>CLIMATE.CATEGORY</th>
      <th>CAUSE.CATEGORY</th>
      <th>OUTAGE.START</th>
      <th>POPULATION</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>70000.0</td>
      <td>3060.0</td>
      <td>normal</td>
      <td>severe weather</td>
      <td>2011-07-01 17:00:00</td>
      <td>5348119.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>70000.0</td>
      <td>3000.0</td>
      <td>cold</td>
      <td>severe weather</td>
      <td>2010-10-26 20:00:00</td>
      <td>5310903.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>68200.0</td>
      <td>2550.0</td>
      <td>normal</td>
      <td>severe weather</td>
      <td>2012-06-19 04:30:00</td>
      <td>5380443.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>250000.0</td>
      <td>1740.0</td>
      <td>warm</td>
      <td>severe weather</td>
      <td>2015-07-18 02:00:00</td>
      <td>5489594.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>60000.0</td>
      <td>1860.0</td>
      <td>cold</td>
      <td>severe weather</td>
      <td>2010-11-13 15:00:00</td>
      <td>5310903.0</td>
    </tr>
  </tbody>
</table>
</div>

<br>

We proceed by tweaking these three new features so that they are also suitable to predict CUSTOMERS.AFFECTED 🚶. We'll have to do quite a bit of adjustment to make these variables more understandable to our regressor.

- OUTAGE.DURATION ⏱ and CLIMATE.CATEGORY 🌤 will undergo the same transformations that were present in our baseline model pipeline.

- CAUSE.CATEGORY 🌪 will undergo **OHE**. How it would be applied here is: for each of the seven potential categories in CAUSE.CATEGORY 🌪 ('Equipment Failure', 'Fuel Supply Emergency', 'Intentional Attack', 'Islanding', 'Public Appeal', 'Severe Weather', and 'System Operability Disruption'), we will create a seperate a column that will contain only zeroes and ones (1 if the blackout falls into the potential category, 0 if the blackout does not). We will remove the original categorical variable from the feature set. We will additionally remove one of the seven new features in order to, more technically, shield ourselves from the effects of multicollinearity.

- OUTAGE.START.TIME 🏁 will be put through a **FunctionTransformer**, which will sort the hour of blackout into buckets of 'Morning', 'Afternoon', or 'Evening' (0-8: Morning, 8-16: Afternoon, 16-24: Evening). These three categories will recieve their own **OHE,** creating *two* (three, and then dropping the first new feature) new features for our model. We will rename these columns to TIME.OF.DAY 🏁.

- POPULATION 👥 will utilize a **StandardScaler,** which takes a distribution and spits out the corresponding z-score for each value, essentially mapping the distribution to a bell-curve. Though we could leave POPULATION 👥 untouched, it is better to standardize the data and dull the effect that extremely large cities and extremely small towns have on our regressor. 



Our features are now prepared.

<div style="overflow-x:auto;">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>TIME.OF.DAY_morning</th>
      <th>TIME.OF.DAY_evening</th>
      <th>CLIMATE.CATEGORY_normal</th>
      <th>CLIMATE.CATEGORY_warm</th>
      <th>POPULATION</th>
      <th>CAUSE.CATEGORY_fuel_supply_emergency</th>
      <th>CAUSE.CATEGORY_intentional_attack</th>
      <th>CAUSE_CATEGORY_islanding</th>
      <th>CAUSE.CATEGORY_public_appeal</th>
      <th>CAUSE.CATEGORY_severe_weather</th>
      <th>CAUSE.CATEGORY_system_operabillity_disruption</th>
      <th>OUTAGE.DURATION</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>-0.709441</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>3060.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-0.712880</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>3000.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>-0.706454</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>2550.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>-0.696369</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>1740.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-0.712880</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>1860.0</td>
    </tr>
  </tbody>
</table>
</div>

<br>

### SUMMARY OF FEATURES BEING USED IN FINAL MODEL:

- 2 CONTINUOUS, NUMERICAL FEATURES.
> OUTAGE.DURATION ⏱ <br>
> POPULATION 👥


- 10 DISCRETE, NUMERICAL FEATURES AS THE RESULT OF CATEGORICAL FEATURE OHE.
> TIME.OF.DAY (MORNING) 🏁  <br>
> TIME.OF.DAY (EVENING) 🏁  <br>
> CLIMATE.CATEGORY (NORMAL) 🌤 <br>
> CLIMATE.CATEGORY (WARM) 🌤 <br>
> CAUSE.CATEGORY (FUEL SUPPLY EMERGENCY) 🌪 <br>
> CAUSE.CATEGORY (INTENTIONAL ATTACK) 🌪 <br>
> CAUSE.CATEGORY (ISLANDING) 🌪 <br>
> CAUSE.CATEGORY (PUBLIC APPEAL) 🌪 <br>
> CAUSE.CATEGORY (SEVERE WEATHER) 🌪 <br>
> CAUSE.CATEGORY (SYSTEM OPERABILITY DISRUPTION) 🌪 <br>


- THERE ARE 12 TOTAL FEATURES.

<br>
<br>

We decided to stick with RandomForestRegressor as our predictor instead of changing it for our final model. To obtain better hyperparamters and train more effectively, we enlisted the help of [GridSearchCV](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.GridSearchCV.html), which tests our model over splits or "folds" of our data to find better hyperparameters. This can prevent **overfitting.**

Multiple runs of GridSearchCV helped identify that perhaps these were the best hyperparameters for our RandomForestRegressor:



```
best_params = {'criterion': 'friedman_mse',
               'max_depth': 3,
               'max_features': 9,
               'n_estimators': 6,
              'random_state': 1}
```


With this in mind, let's see how our model performs.

<br>


### RESULTS:


<br>

```
FINAL MODEL___ 
Final Training RMSE: 214995.14290443418

Final Testing RMSE: 333613.7695914098
Testing Improvement: -45811.835930722475
```



It's an improvement. The training RMSE is still *significantly* lower than our testing RMSE, which implies that our model might be **overfit** to our current data, but the overfit is less extreme than in the case of our baseline model. Training error has increased significantly, but what matters much more is how the testing error has changed, since we'll be testing our model on *data other than our training data.* Testing error has gone down significantly; the improvement is ~40,000 RMSE! This means that our model is much better at predicting *for other given datasets* than our baseline model is.

Clearly, our predictions for the number of CUSTOMERS.AFFECTED 🚶 in power outages has improved with these changes, but there are still limitations to the quality of our model.


<br>

---

## FAIRNESS ANALYSIS 🔋

To test and see if our model is fair, or producing similar RMSEs, in assessing blackouts from different climates, we conducted a permutation test. The two groups we compared were normal climates (blackouts where CLIMATE.CATEGORY 🌤 is 'Normal') and extreme climates (blackouts where CLIMATE.CATEGORY 🌤 is either 'Warm' or 'Cold'). The results of this permutation test would determine the validity of the model being used with regards to climate.

> <b> Null Hypothesis: </b> Our model is fair. The RMSE for locations with extreme weather (hot and cold) is the same as the RMSE of locations with normal weather.

> <b> Alternative Hypothesis: </b> Our model is unfair. The RMSE for locations with extreme weather (hot and cold) is not the same as the RMSE of locations with normal weather.

> *Test Statistic: Absolute difference between normal weather and extreme weather RMSEs.*

> *Significance Level: 0.05.*

<br>

We recieved a p-value of ~0.20 for this comparison, which (at a standard α = 0.05 significance level) **fails to reject the null hypothesis and implies that our specific model is fair, producing similar RMSEs for blackouts where CLIMATE.CATEGORY 🌤 is 'Normal' vs blackouts where CLIMATE.CATEGORY 🌤 is either 'Warm' or 'Cold'.**

## CONCLUSION 🔋

Going back to our original question of:
> ### Can we reasonably predict the magnitude of a blackout from other blackout features?

<br>

... we found that using features like CLIMATE.CATEGORY 🌤, CAUSE.CATEGORY 🌪, TIME.OF.DAY 🏁, POPULATION 👥, and OUTAGE.DURATION ⏱  were mediocre indicators of the magnitude of a blackout (CUSTOMERS.AFFECTED 🚶). Both the training RMSE (~ 220k) and testing RMSE (~ 330k) were fairly high in our final model, yet reasonable improvement was made in testing RMSE from our baseline model (~ 44k). There was not strong enough evidence to suggest that our model was unfair in predicting across climates, passing a permutation test at the 0.05 significance level.

As with all prediction problems, further exploration and assessment is necessary to see where better predictors and undercover patterns may lie in our data.
