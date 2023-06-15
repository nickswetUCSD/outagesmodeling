
## REINTRODUCTION 🔋

Nobody likes a blackout. They seem to always crop up when it's least convenient, leaving cities high and dry with malfunctioning traffic lights and citizens floundering with a lack of resources for poor laptop battery life.

Last time, we investigated the ins and outs of over **1500 rows** of U.S. power outage data¹ , courtesy of **Purdue University’s LASCI (Laboratory For Advancing Sustainable Critical Infrastructure)**. We found some interesting conclusions:



- Outage data has many factors and causes🌪️ that make it complex to predict, and investigating that data as individual points or as aggregates can reveal contrasting trends.

-  Four states deviate from what is expected of the distribution of the greater United States in terms of OUTAGE.DURATION⏱.

- There is some overlap in the states that more commonly have longer power outages and the states who have a high number of mean CUSTOMERS.AFFECTED🚶 per outage.



**But can we take these conclusions and turn them into something actionable? Perhaps we can!**




For additional context, read up on our [previous deep dive](https://nickswetucsd.github.io/poweroutages/) into power outages.




¹ Link to data source found [here.](https://engineering.purdue.edu/LASCI/research-data/outages/outagerisks)

<br>
---

## PROBLEM IDENTIFICATION 🔋


We pose the following prediction problem:
> ### Can we reasonably predict the magnitude of a blackout from other blackout features?

... and here's how we would go about it.



- **RESPONSE VARIABLE: CUSTOMERS.AFFECTED 🚶**
>  For the sake of continuity and expanding our investigation, we thought that CUSTOMERS.AFFECTED 🚶 would be an apt response variable. CUSTOMERS.AFFECTED 🚶 is a column from our dataset that denotes the approximate number of customers affected by a particular blackout. This is great way to define what "magnitude" of a blackout should mean in our question, as we would label blackouts that affect more people as "more severe"!
<br>

- **PREDICTOR CLASS: RandomForestRegressor**
> Since our response variable of CUSTOMERS.AFFECTED 🚶 uses *continuous numerical data*, we'll need to use a regressor instead of a classifier. We'll use [RandomForestRegressor](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestRegressor.html) from sk.learn to generate predictions; RandomForestRegressor uses multiple decision trees and averages their outputs to make predictions, which is great for datasets with multiple features.
<br>

- **QUALITY METRIC: RMSE**
> To measure how "good" our model is, we'll use the classic regression metric of **Root Mean Squared Error (RMSE).** RMSE will give us the total squared error from each actual and predicted value, which holds up for assessing nonlinear prediction models like ours; this is preferable to other metrics like **R-Squared**, which are *invalid* for assessing nonlinear prediction models. Lower RMSE indicates "better" predictions, while higher RMSE indicates "worse" predictions.
<br>

Now that our sails are set, we can use the cleaned data from our previous project ***to start tinkering with features and build a baseline model.***

<br>
---

## BASELINE MODEL 🔋

Two things we immediately thought of that related to the numbers of customers a blackout affects were how long the blackouts were and what at what temperatures the blackouts were occurring in. After all, everybody *knows* that blackouts are (probably) more severe in summer due to hot weather.

The closest analogue for these factors that we could find in our dataset were the two following variables:

- **OUTAGE.DURATION** ⏱:
>"Duration of outage event (in minutes)." Used in our previous analysis as a measure of power outage severity. **Measured as a *continuous, numerical variable.***
<br>


- **CLIMATE.CATEGORY** 🌤️:
>"Represents the climate episodes corresponding to the years. The categories—'Warm', 'Cold' or 'Normal' episodes of the climate are based on a threshold of ± 0.5 °C for the Oceanic Niño Index (ONI)." **Measured as a *ordinal, categorical variable.***
<br>



Here is what this data might look like before adjustment:

{baseline_model_df}



We proceed by tweaking these two features so that they are suitable to predict CUSTOMERS.AFFECTED 🚶. This is done inside of a [**PipeLine object**](https://scikit-learn.org/stable/modules/generated/sklearn.pipeline.Pipeline.html) from sk.learn.  


- OUTAGE.DURATION ⏱: is fine as is; no need to change it.

- CLIMATE.CATEGORY 🌤 will undergo **One-Hot Encoding (OHE)**, which transforms a *single* categorical feature into *multiple* numerical features through the process of creating seperate data columns for each potential category in the original variable. How it would be applied here is: for each of the three potential categories in CLIMATE.CATEGORY 🌤 ('Warm', 'Normal', 'Cold'), create a seperate a column that will contain only zeroes and ones (1 if the blackout falls into the potential category, 0 if the blackout does not). We will remove the original categorical variable from the feature set. We will additionally remove one of the three new features in order to, more technically, shield ourselves from the effects of multicollinearity.

Our features are now prepared.
<br>

### SUMMARY OF FEATURES BEING USED IN BASELINE MODEL:

- 1 CONTINUOUS, NUMERICAL FEATURE.
> OUTAGE.DURATION ⏱:


- 2 DISCRETE, NUMERICAL FEATURES AS THE RESULT OF CATEGORICAL FEATURE OHE.
> CLIMATE.CATEGORY (NORMAL) 🌤 <br>
> CLIMATE.CATEGORY (WARM) 🌤


- THERE ARE 3 TOTAL FEATURES.

<br>
<br>

*( training baseline_model . . . ⚙️⚙️⚙️ )*

. . .
<br>

### RESULTS:

{baseline model data}

It's not perfect. The training RMSE is *significantly* lower than our testing RMSE, which implies that our model might be **overfit** to our current data. Regardless, both errors are still quite high. 

This implies that the best practice going forward is to **limit the depth of training** (by considering better hyperparameter configurations in RandomForestRegressor) and, perhaps, **increase the breadth of training** (offer different features to get better predictive resolution).

Let's see if our predictions for the number of CUSTOMERS.AFFECTED 🚶 in power outages improve with these changes.


<br>

## FINAL MODEL 🔋

...
## FAIRNESS ANALYSIS 🔋

...