# A College Student's Nightmare: Cooking Times
by Jason Liu (jal134@ucsd.edu)

---
## Introduction

As a college student, I face a constant struggle in trying to decide what to eat every day. Dining at restaurants can be very expensive, so cooking my own meals is what I like to choose. However, on some days, I can be quite limited in what I can actually make, since I might not have a wide variety of ingredients or I might not have enough time. Even then, I think it's important to be eating healthy, so I can have enough energy to get my work done. As such, I'd like to be able to prepare some food that gets me all the calories I need, and it needs to taste good as well (I can be a picky eater!); assuming these conditions are met, I need to know how long it'll take me to prepare my food so I can plan out my time, making sure I can get all of my homework done. All of this leads me to look into the question:

**How do the nutritional values and quality of a recipe affect the amount of time it takes to prepare a particular recipe?**

To find out the answer to this question, I will be analyzing two different datasets.

The first dataset is our `raw_recipes` dataset. It contains data about different recipes taken from [food.com](https://www.food.com/) that were posted after the year 2008.
This dataset has 83782 rows and 12 columns. Each row corresponds to a unique recipe. The data stored in the columns is as follows:

| Column   | Description
| :-----  | :-----
| `'name'` | Recipe name
| `'id'` | Recipe ID
| `'minutes'` | Minutes to prepare recipe
| `'contributor_id'` | User ID who submitted this recipe
| `'submitted'` | Date recipe was submitted
| `'tags'` | Food.com tags for recipe
| `'nutrition'` | Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for “percentage of daily value”
| `'n_steps'` | Number of steps in recipe
| `'steps'` | Text for recipe steps, in order
| `'description'` | User-provided description
| `'ingredients'` | Text for ingredients used in recipe
| `'n_ingredients'` | Number of ingredients used in recipe

The other dataset is our `raw_interactions` dataset. This dataset contains the ratings and reviews that other users for the posted recipes.
This dataset has 731927 rows and 5 columns. Each row corresponds to a review that a user has left. The data stored in the columns is as follows:

| Column   | Description
| :-----  | :-----
| `'user_id'` |	User ID
| `'recipe_id'` | Recipe ID
| `'date'` | Date of interaction
| `'rating'` | Rating given
| `'review'` | Review text

These datasets provide some good information that helps me answer my question. The first dataset provides all of the relevant nutritional information that I will want to analyze. It also provides data for each recipe's complexity, which can be generalized as the amount of ingredients a recipe has, as well as the number of steps in the preparataion process. The second dataset gives us information on the ratings and reviews of specific recipes, which will tell us the quality of a recipe in general (a good recipe tends to have higher ratings).

We will use these datasets in conjuction with each other in order to help us look into the possible relationships between nutrition, recipe complexity, recipe quality, and the amount of time needed to prepare recipes. The next section will go into more detail on how we can do this.

---
## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

To start, we should clean up our datasets so we can do our analysis easily. First, let's transform some of the columns in the `raw_recipes` dataframe so our data is easy to work with.

1. The `'tags'`, `'steps'`, and `'ingredients'` columns in the dataframe are supposed to be lists, but they're currently strings. We will simply transform all of the values into lists.
2. The `'nutrition'` column is also given as a string that represents a list of nutrition values. We will split the nutrition values across multiple columns, so we can easily extract specific nutrition values we need easily. The steps taken to do this are as follows:
- First, split the strings into their proper list format. Again, the format will be: [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)].
- Take all of the specific nutrition values for the recipes and turn each one into their individual columns. Our resulting dataframe will have 7 new columns, one for each nutrition label.
3. For our 7 new nutriton columns, we will convert each of the string values into floats since they are quantitative. These values represent the percent daily values for each recipe.
4. Now, we can combine the two dataframes, `raw_recipes` and `raw_interactions`, so that each review can be assigned to the recipe it was submitted for. To do this, we will perform a left merge for the two dataframes on the recipe IDs.
5. In the merged dataset, we will fill all ratings of 0 with np.nan. This is because ratings of 0 indicate that the user who submitted the review did not submit an associated rating. Since we have no way of telling what the rating would've been, and having a value of 0 will lower the average rating of the recipe, we will make the value null so it will not affect our data later on.
6. Find the average rating per recipe. To do this, we will groupby each specific recipe, and take the average of the ratings provided for that recipe. Our result is a pandas Series.
7. Add the average rating series back into our merged dataframe using another left merge. The merged dataframe will now include a new column that includes the average rating for each recipe.
- The merge itself will be on the recipe ID in the merged dataframe, and the index of the average rating series (since we created it from groupby, the index is the recipe ID). This results in the recipe ID appearing twice in the resulting merge, so we will drop one of them.

We now have a cleaned dataset, `merged_recipes`, that we can use! This is what the head of the dataframe looks like (Note: I included only relevant columns that I'll be using in my analyses):

| name                               | minutes | n_steps | n_ingredients | Calories (#) | Total Fat (PDV) | Sugar (PDV) | Sodium (PDV) | Protein (PDV) | Carbohydrates (PDV) | rating | review                                            | average_rating |
| :-----                             | :-----  | :-----  | :-----        | :-----       | :-----          | :-----      | :-----       | :-----        | :-----              | :----- | :-----                                            | :-----         |
| 1 brownies in the world best ever	 | 40      | 10      | 9             | 138.4        | 10.0            | 50.0        | 3.0          | 3.0           | 6.0                 | 4.0    | These were pretty good, but took forever to ba... | 4.0            |
| 1 in canada chocolate chip cookies | 45      | 12      | 11            | 595.1        | 46.0            | 211.0       | 22.0         | 13.0          | 26.0                | 5.0    | Originally I was gonna cut the recipe in half ... | 5.0            |
| 412 broccoli casserole             | 40      | 6       | 9             | 194.8        | 20.0            | 6.0         | 32.0         | 22.0          | 3.0                 | 5.0    | This was one of the best broccoli casseroles t... | 5.0            |
| 412 broccoli casserole             | 40      | 6       | 9             | 194.8        | 20.0            | 6.0         | 32.0         | 22.0          | 3.0                 | 5.0    | I made this for my son's first birthday party ... | 5.0            |
| 412 broccoli casserole             | 40      | 6       | 9             | 194.8        | 20.0            | 6.0         | 32.0         | 22.0          | 3.0                 | 5.0    | Loved this. Be sure to completely thaw the br...  | 5.0            |

The `merged_recipes` dataframe (that will be used for the rest of the project) has 234429 rows and 25 columns.

These are what data types the columns contain:

| Column   | Type
| :-----  | :-----
| `'name'` | object
| `'id'` | int
| `'minutes'` | int
| `'contributor_id'` | int
| `'submitted'` | object
| `'tags'` | object
| `'nutrition'` | object
| `'n_steps'` | int
| `'steps'` | object
| `'description'` | object
| `'ingredients'` | object
| `'n_ingredients'` | int
| `'Calories (#)'` | float
| `'Total Fat (PDV)'` | float
| `'Sugar (PDV)'` | float
| `'Sodium (PDV)'` | float
| `'Protein (PDV)'` | float
| `'Saturated fat (PDV)'` | float
| `'Carbohydrates (PDV)'` | float
| `'user_id'` | float
| `'date'` | object
| `'rating'` | float
| `'review'` | object
| `'average_rating'` | float

### Univariate Analysis

The first few recipes we have are all rated very highly. Let's take a look at all of the non-null recipe ratings to see how people feel about the recipes on food.com in general!

<iframe
  src="assets/univariate_ratings.html"
  width="600"
  height="400"
  frameborder="0"
></iframe>
It looks like the recipes tend to have very high ratings in general, since most of them were 5! This means that most people think the recipes in our dataset are very good, so we'll have to keep this in mind.

Let's also take a look at the distribution of calories in our dataset, so we know what the average recipes' calories are.

<iframe
  src="assets/univariate_calories.html"
  width="600"
  height="400"
  frameborder="0"
></iframe>
_Note: Around 1% of the values for calories fall in the range 2500 - 45609, and are excluded from the distribution (outliers)_

The distribution of calories appears to be skewed right. A majority of the recipes' calories fall under the 25 - 500 range, but there are a couple of skewed, larger values that could potentially introduce bias in later analyses.

### Bivariate Analysis

Each recipe has several different nutritional values associated with it. Is there a relationship between these nutritional values themselves? Let's take a look.

First, we'll see if there could be a correlation between the amount of sugar in a recipe and the number of calories by looking at a scatterplot.

<iframe
  src="assets/bivariate_sugar.html"
  width="600"
  height="400"
  frameborder="0"
></iframe>
_Note: The 5 recipes with the largest sugar content and the 5 recipes with the most calories were excluded from the graph since they were extreme outliers, making most of the data points hard to see._

By observing the plot, it seems like in general, there is a slight positive correlation between the amount of sugar and the calories in a recipe. However, there are some recipes without much sugar that still contain very high calories, so the association may be weak.

Another relationship we can look at is between the amount of carbohydrates and the number of calories in a recipe.

<iframe
  src="assets/bivariate_carbs.html"
  width="600"
  height="400"
  frameborder="0"
></iframe>
_Note: The 5 recipes with the highest calorie content and the 5 recipes with the most carbohydrates were excluded for the same reason as above._

We can also notice a slight positive correlation between the amount of carbohydrates and the calories in a recipe, since recipes with more carbohydrates seem to have higher calories, too. The correlation appears to be is slightly stronger than what we see in the sugar scatterplot, but we can't be 100% sure.

Both of these plots tell us that the `'Sugar (PDV)'` and `'Carbohydrates (PDV)'` columns may have a positive correlation with a recipe's calories. 

### Interesting Aggregates

Another relationship that we can look into is trying to see if a recipe's calories are affected by its complexity and quality. We defined a recipe's complexity as the number of steps and ingredients, but here we'll look at the number of ingredients to determine complexity. We can also measure the quality of a recipe by the reviews it's received. The following pivot table lets us see the average calories for each combination of rating and number of steps, and we'll try and find some interesting patterns in the aggregation!

| rating | 1.0 | 2.0 | 3.0 | 4.0 | 5.0 | All |
| n_ingredients | | | | | | |
| :----- | :----- | :----- | :----- | :----- | :----- | :----- |
| 1 | 0.00 | 199.50 | 136.05 | 758.68 | 1263.21 | 1194.49 |
| 2 | 732.77 | 349.98 | 292.77 | 347.40 | 390.09 | 388.47 |
| 3 | 419.62 | 329.93 | 335.45 | 268.41 | 276.89 | 279.54 |
| 4 | 394.97 | 345.58 | 325.21 | 295.25 | 301.29 | 302.57 |
| 5 | 487.56 | 362.73 | 332.41 | 332.65 | 322.76 | 327.01 | 
| ... | ... | ... | ... | ... | ... | ... |
| 33 | 0.00 | 0.00 | 0.00 | 0.00 | 338.20 | 338.20 |
| 37 | 0.00 | 0.00 | 0.00 | 0.00 | 10687.70 | 10687.70 |
| All | 486.60 | 446.60 | 425.79 | 405.05 | 415.21 | 415.10 |

This pivot table has several interesting features to discuss. First, by looking at the bottom row, it would seem like on average, the recipes with lower ratings typically had more calories. We can note that since a majority of the ratings were 5 (from earlier), it's likely that many large outliers with high calories can also have a rating of 5. As such, the average calories for recipes with a rating of 5 could actually be higher than what we should expect compared to the other ratings. This is seen in the bottom few rows, where the few recipes that used over 30 ingredients only received ratings of 5. 

The pivot table also gives us some important information on how the number of ingredients can affect the calories as well. On average, looking at the first few rows, it seems like recipes that only use one ingredient typically contain many more calories than other recipes. However, aside from that, there isn't a clear pattern for how the number of ingredients could affect the calories, even though we would expect a recipe with more ingridents to produce food that has more calories. This is likely because we don't acutally have information on what the ingredients are, and having certain ingredients like oil or butter in a recipe would contribute more calories to the recipe than one that uses a lot of vegetables; therefore, the information found in `'n_ingredients'` alone might not be quantitative!

---
## Assessment of Missingness

### NMAR Analysis

There are three columns in our dataset that have a non-trivial amount of missingness: `'description'`, `'rating'`, and `'review'`. A column that could potentially have data that is Not Missing at Random (NMAR) is the `'description'` column.

The column that includes the description of recipes have a few values that are missing. The descriptions themselves are probably missing because the user who submitted the recipe did not include a description when posting their recipe on the website. The missingness of the description itself is a good indicator for why the description could be missing, as the user who submitted the recipe could have felt like their recipe is quite simple or popular and doesn't require a description, which results in there being none. As such, I suspect that the `'description'` column is NMAR.

### Missingness Dependency

The `'review'` column could contain missing data that is potentially MAR on another column. Let's try and find out what columns the missingness could depend on!

First, let's see if the missingness of the reviews could depend on the amount of time needed to prepare the recipe. 

<iframe
  src="assets/permutation_minutes_dist.html"
  width="600"
  height="400"
  frameborder="0"
></iframe>

To perform this permutation test, we will use the following:

**Null Hypothesis**: The distribution of minutes needed to prepare a recipe is the same for both missing and non-missing reviews.

**Alternative Hypothesis**: The distribution of minutes needed to prepare a recipe is different for missing and non-missing reviews.

**Test Statistic**: Absolute difference in means of minutes for missing reviews and non-missing reviews

**Significance Level**: 0.05

We shuffled the `'minutes'` column to simulate data under the null hypothesis 10000 times. Each time, we calculated the absolute difference in means for minutes with missing and non-missing reviews.

<iframe
  src="assets/permutation_minutes_test.html"
  width="600"
  height="400"
  frameborder="0"
></iframe>

**Observed Statistic**: 33.5635

**P-Value**: 0.6721

Since our P-value was larger than our significance level (0.05), we **fail to reject** the null hypothesis. We do not have convincing evidence that the missingness of the `'review'` column depends on the number of minutes taken to prepare a recipe!

Let's test a different column. Could the missingness of the reviews depend on the number of calories in a recipe?

<iframe
  src="assets/permutation_cals_dist.html"
  width="600"
  height="400"
  frameborder="0"
></iframe>

Our setup is much the same. To perform this permutation test, we will use the following:

**Null Hypothesis**: The distribution of calories in a recipe is the same for both missing and non-missing reviews.

**Alternative Hypothesis**: The distribution of calories in a recipe is different for missing and non-missing reviews.

**Test Statistic**: Absolute difference in means of calories for missing reviews and non-missing reviews

**Significance Level**: 0.05

This time, we will shuffle the `'Calories (#)'` column 10000 times. For each simulation, we will calculate the absolute difference in means for calories with missing and non-missing reviews.

<iframe
  src="assets/permutation_cals_test.html"
  width="600"
  height="400"
  frameborder="0"
></iframe>

**Observed Statistic**: 319.2691

**P-Value**: 0.0048

Since our P-value was lower than our significance level (0.05), we can **reject** the null hypothesis. It's possible that the missingness of the `'review'` column can depend on the number of calories present in a recipe!

---
## Hypothesis Testing

Part of the question we discussed in the introduction asks about a relationship between preparation time and nutritional values for recipes. The calorie count is one specific nutritional value, and I personally believe that it should be prioritized since it's what I struggle with managing the most (I mostly eat food low in calories). As such, I want to see if there is a relationship between amount of calories and preparation time, and if it will take longer to prepare meals that will give me more calories. Specificially, the question we are analyzing here is:

**Does it take longer to prepare recipes that have more calories?**

To answer this question, we will run a permutation test with the following:

**Null Hypothesis**: The average time taken to prepare recipes with a high calorie content is equal to the average preparation time of recipes with a low calorie content.

**Alternative Hypothesis**: It takes longer to prepare recipes with a high calorie content when compared to recipes with a low calorie content.

**Test Statistic**: Difference in means for preparation time of recipes with a high and low calorie count

**Significance Level**: 0.05

First, we have to define what is considered a "high" calorie count. According to [Livestrong.com](https://www.livestrong.com/article/440135-recommended-calorie-intake-for-one-meal/), the average calorie intake per meal for women is between 533 to 800, and between 667 to 1000 for men, though it can vary. For this test, we will classify recipes with a high calorie count as recipes that include **over 1000 calories**.

To conduct the permutation test, we will shuffle the `'Calories (#)'` column 10000 times, and we will calculate the difference in means for the preparation time in minutes between recipes with at least 1000 calories and recipes with less than 1000 calories. I chose to use signed difference in means for the test statistic since my alternative hypothesis is one-tailed, so we need to know which mean is greater in the simulations.

<iframe
  src="assets/hypothesis_test.html"
  width="600"
  height="400"
  frameborder="0"
></iframe>

**Observed Statistic**: 66.0130

**P-Value**: 0.0666

The P-value here is higher than our significance level (0.05), meaning we **fail to reject** the null hypothesis. We do not have convincing evidence that the average time taken to prepare recipes high in calories is longer than the average preparation time for recipes low in calories!

---
## Framing a Prediction Problem

Now it's time to really address our question from the introduction as a whole: **How do the nutritional values and quality of a recipe affect the amount of time it takes to prepare a particular recipe?** We'll do this by attempting to fit a model to our data to try and predict the time taken to prepare a recipe using its nutritional values and overall quality (average ratings). The response variable is the **number of minutes to prepare a recipe**. This will be a **Regression Problem**, because the response variable we're predicting is quantitative, and the implementation will use a **Decision Tree Regressor** to make predictions, since it is better than a linear model at identifying complicated non-linear relationships.

I chose the specific response variable since each individual recipe in the merged dataset has a preparation time associated with it, as well as nutritional values. Since we have access to all of the data we need, we can train our model very effectively. At the time of prediction, the data that we will have available is the average rating for each recipe, the nutritional information, and the steps and ingredients used in the recipe. This is because when using the model for predictions, I'll generally have a good idea for what ingredients I will use and how I might go about using them, and I simply want to find out how long it'll take me to prepare my food!

To evaluate the model, we will use the **Root Mean Squared Error (RMSE)** as the evaluation metric. I chose this metric because the RMSE is a good indicator of how well a model's predictions are, and the value itself can be easily interpreted. A lower RMSE value would indicate that our model is making predictions that are closer to the actual values, and has better performance. 

---
## Baseline Model

Let's create our baseline DecisionTreeRegressor model. The model will make predictions using the following features:

| Feature   | Data Type | Inclusion | Encoding
| :-----  | :----- | :----- | :-----
| `'Calories (#)'` | Quantitative | The number of calories is important, since it's the nutrition value that I prioritize the most, which is why I included it as a feature | The calories distribution is skewed right, and include some very large outliers, so we will use a **Log Encoding** to help scale the values
| `'Protein (PDV)'` | Quantitative | The amount of protein is also very important, since meat can sometimes take a long time to cook to ensure it's safe to eat, and must be factored in when trying to predict cooking times | Like calories, the distribution is skewed right and we will use a **Log Encoding** to transform this column as well
| `'average_rating'` | Quantitative | People might rate recipes differently depending on they're quick and easy or time-consuming to prepare, so the average rating could be an indicator for the prep time | This column represents the quality of a recipe, includes values from 1 to 5 only, and we will not need to encode it since its values are quantitative

The model will predict the preparation time of a recipe in minutes. Something important to note is that the `'minutes'` column contains a lot of skew, as the average minutes is 115.03 while there are values as high as 1051200! To handle this, we will identify the outliers using the **Interquartile Range**, and values greater than the third quartile by 1.5 times the IQR will be considered outliers. Recipes that contain these outliers for minutes will be dropped, since they are not a good representation of what our dataset will want to predict on average (it doesn't make sense to spend 1 million minutes making dinner).

To keep things simple, I will be using the default hyperparameters for the DecisionTreeRegressor.

After fitting the model, we have:

**Training Set RMSE**: 5.7549

**Test Set RMSE**: 36.0187

The RMSE indicates that our model is **not very good** at predicting the preparation time in minutes for a recipe using the provided features. Our RMSE is several times higher for the test data than the training data, meaning this baseline model is good at predicting values from the training data but bad at generalizing to unseen data. This could potentially be because decision trees are prone to overfitting on the training set, resulting in our model capturing a lot of noise from our training data.

---
## Final Model

Since decision trees are prone to overfitting on training data, a better idea would be to use a **Random Forest Regressor** instead as our model. By fitting multiple different decision trees using random subsets of our training data and having them "vote" on predictions, there wouldn't be as much overfitting since the decision trees aren't very correlated with each other, and will not fit to training noise as much in general.

We will also add some more features to help improve our model.

| Feature   | Data Type | Inclusion | Encoding
| :-----  | :----- | :----- | :-----
| `'Sugar (PDV)'` | Quantitative | The amount of sugar can impact the amount of time taken to prepare a recipe, since a lot of sugar might indicate a recipe is a dessert, and could require baking which might take a different amount of time | The distribution is skewed right, and include some very large outliers, so we will use a **Log Encoding** to help scale the values
| `'n_ingredients'` | Categorical | Having more ingredients could mean that a recipe is more complicated, and will need more time to prepare all of the ingredients, so we should include this column | This data will be encoded using **One-Hot Encoding** since the number of ingredients is categorical; we don't know what the ingredients are, and different types of ingredients could make the values have a different interpretation - as such, they aren't the standard numbers we're used to working with
| `'n_steps'` | Categorical | If a recipe has more steps, then it's possible that more time will need to be spent overall since we'll need to complete more tasks to prepare a recipe, making this a good feature to include | This data will also be encoded using **One-Hot Encoding** since the number of steps is categorical. All the steps might be different and take different times, meaning like the column containing the number of ingredients, the data here is not exactly quantitative

The hyperparameters that were tuned in this model are the `max_depth` and the `n_estimators` of the Random Forest Regressor. 

I chose to tune the `max_depth` because if the maximum depth for the trees in the model are too high, it could result in each tree being more biased to the training data, while having a low maximum depth could make the model too simple and fail to capture a relationship. It's important to find a value that works well!

I chose to tune the `n_estimators` because I want to figure out what the optimal number of decision trees to use in the random forest is. Generally, more decision trees will make our predictions more accurate, but it's good to make sure.

To tune the hyperparameters, I used `GridSearchCV` to find the best hyperparameters for a `RandomForestRegressor` model using 5 folds. This gave us the following best combination:

`n_estimators`: 64

`max_depth`: 8

Now, let's fit our final model using the new features and `RandomForestRegressor` with the best hyperparameters. To evaluate our new model, I found the RMSE for the training and test data using the same train-test splits that the baseline model used.

**Training Set RMSE**: 22.4137

**Test Set RMSE**: 23.2115

The final model has higher training error than the baseline model, and lower test set error than the baseline model. This means that our final model is less accurate when it comes to making predictions on the training data, but is more accurate when making predictions on unseen data. This is likely because our final model is not overfitting to the training data as much compared to the baseline model, allowing its predictions to be more accurate for our test data. As such, our final model is an improvement over our baseline!

---
## Fairness Analysis

Let's perform a fairness analysis on our final model to see if it's able to draw fair conclusions for two different groups. 

Specifically, we want to know: **Does the model perform worse for recipes with a large number of total fats than for recipes with a low number of total fats?**

First, we should define our two groups more clearly. People should be eating three meals a day, and also not consume over 100% the PDV of fats to stay healthy. Therefore, on average, a meal shouldn't contain over 33% the PDV of total fats. This allows us to properly define our two groups:

Group 1: Recipes high in fat content (total fat content of over 33% PDV)

Group 2: Recipes with a low fat content (total fat content lower than or equal to 33% PDV)

Our evalutaion metric will again be the RMSE of our model. To test for fairness between the two groups, we will conduct another permutation test with the following:

**Null Hypothesis**: Our model is fair. The RMSE of predicted cooking times for recipes high in fat content and low in fat content are roughly the same, and any differences are due to random chance.

**Alternative Hypothesis**: Our model is unfair. The RMSE of predicted cooking times for recipes high in fat content is greater than the RMSE for recipes low in fat content.

**Test Statistic**: Difference in RMSE (High fat recipes - Low fat recipes)

**Significance Level**: 0.05

In this permutation test, we will generate 500 simulated test statistics by randomly shuffling the `'Total Fat (PDV)'` column and splitting the shuffled data into their respective groups for high or low fat content. In each simulation, we will then use the final model to make predictions for each of the groups and then calculate the RMSE for each group's predictions, then calculate our test statistic, which is the difference of the RMSE values.

<iframe
  src="assets/fairness_analysis.html"
  width="600"
  height="400"
  frameborder="0"
></iframe>

**Observed Statistic**: 66.0130

**P-Value**: 0.0

Since our p-value is lower than our significance level, we reject the null hypothesis. There is evidence that the RMSE for recipes with a high total fat content is greater than the RMSE for recipes with a lower fat content. This means that our model is not fair, as it performs worse for recipes that contain a lot of fats!
