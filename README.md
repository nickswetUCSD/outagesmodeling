
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

## PROBLEM IDENTIFICATION 🔋

<br>

We pose the following prediction problem:
> ### Can we reasonably predict the magnitude of a blackout from other blackout features?

... and here's how we would go about it.



- **Response Variable: CUSTOMERS.AFFECTED**
>  For the sake of continuity and expanding our investigation, we thought that CUSTOMERS.AFFECTED would be an apt response variable. CUSTOMERS.AFFECTED is a column from our dataset that denotes the approximate number of customers affected by a particular blackout. This is great way to define what "magnitude" of a blackout should mean in our question, as we would label blackouts that affect more people as "more severe"!

- **Predictor Type: RandomForestRegressor**
> Since our response variable of CUSTOMERS.AFFECTED uses *continuous numerical data*, we'll need to use a regressor instead of a classifier. We'll use [RandomForestRegressor](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestRegressor.html) from sk.learn to generate predictions; RandomForestRegressor uses multiple decision trees and averages their outputs to make predictions, which is great for datasets with multiple features.

- **Quality Metric: RMSE**
> To measure how "good" our model is, we'll use the classic regression metric of **Root Mean Squared Error (RMSE).** RMSE will give us the total squared error from each actual and predicted value, which holds up for assessing nonlinear prediction models like ours; this is preferable to other metrics like **R-Squared**, which are *invalid* for assessing nonlinear prediction models. Lower RMSE indicates "better" predictions, while higher RMSE indicates "worse" predictions.


Now that our sails are set, we can use the cleaned data from our previous project ***to start tinkering with features and build a baseline model.***

## BASELINE MODEL 🔋


## FINAL MODEL 🔋

## FAIRNESS ANALYSIS 🔋