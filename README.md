# A College Student's Nightmare: Cooking Times
by Jason Liu (jal134@ucsd.edu)

---
## Introduction

As a college student, I face a constant struggle in trying to decide what to eat every day. Dining at restaurants can be very expensive, so cooking my own meals is what I like to choose. However, on some days, I can be quite limited in what I can actually make, since I might not have a wide variety of ingredients or I might not have enough time. Even then, I think it's important to be eating healthy, so I can have enough energy to get my work done. As such, I'd like to be able to prepare some food that gets me all the calories I need, and it needs to taste good as well (I can be a picky eater!); assuming these conditions are met, I need to know how long it'll take me to prepare my food so I can plan out my time, making sure I can get all of my homework done. All of this leads me to look into the question:

**How do the nutritial values, complexity, and quality of a recipe affect the amount of time it takes to prepare a particular recipe?**

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

These datasets provide some good information that helps me answer my question. The first dataset provides all of the relevant nutritional information that I will want to analyze. It also provides data for each recipe's complexity, which can be generalized as the amount of ingredients a recipe has, as well as the number of steps in the preparataion process. The second dataset gives us information on the ratings and reviews of specific recipes, which will tell us the quality of a recipe in general (a good recipe will have higher ratings).

We will use these datasets in conjuction with each other in order to help us look into the possible relationships between nutrition, recipe complexity, recipe quality, and the amount of time needed to prepare recipes. The next section will go into more detail on how we can do this.

---
## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

To start, we should clean up our datasets so we can do our analysis easily. First, let's transform some of the columns in the `raw_recipes` dataframe so our data is easy to work with.

1. The `'tags'`, `'steps'`, and `'intredients'` columns in the dataframe are supposed to be lists, but they're currently strings. We will simply transform all of the values into lists.
2. The `'nutrition'` column is also given as a string that represents a list of nutrition values. We will split the nutrition values across multiple columns, so we can easily extract specific nutrition values we need easily. The steps taken to do this are as follows:
- First, split the strings into their proper list format. Again, the format will be: [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)].
- For each specific nutrition value, take all the values for each recipe and turn them into their individual columns. Our resulting dataframe will have 7 new columns, one for each nutrition label, which will replace the old `'nutrition'` column.
3. For our 7 new nutriton columns, we will convert each of the string values into floats. These values represent the percent daily values for each recipe.
4. Now, we can combine the two dataframes, `raw_recipes` and `raw_interactions`, so that each review can be assigned to the recipe it was submitted for. To do this, we will perform a left merge for the two dataframes on the recipe IDs.
5. In the merged dataset, we will fill all ratings of 0 with np.nan. This is because ratings of 0 indicate that the user who submitted the review did not submit an associated rating. Since we have no way of telling what the rating would've been, and having a value of 0 will lower the average rating of the recipe, we will make the value null so it will not affect our data.
6. Find the average rating per recipe. To do this, we will groupby each specific recipe, and take the average of the ratings provided for that recipe. Our result is a pandas Series.
7. Add the average rating series back into our merged dataframe using another left merge. The merged dataframe will now include a new column that includes the average rating for each recipe.
- The merge itself will be on the recipe ID in the merged dataframe, and the index of the average rating series (since we created it from groupby, the index is the recipe ID). This results in the recipe ID appearing twice in the resulting merge, so we will drop one of them.

We now have a cleaned dataset, `merged_recipes`, that we can use! This is what the head of the dataframe looks like (Note: I included only relevant columns that I'll be using in my analysis):

| name                               | minutes | n_steps | n_ingredients | Calories (#) | Protein (PDV) | Carbohydrates (PDV) | rating | review                                            | average_rating |
| :-----                             | :-----  | :-----  | :-----        | :-----       | :-----        | :-----              | :----- | :-----                                            | :-----         |
| 1 brownies in the world best ever	 | 40      | 10      | 9             | 138.4        | 3.0           | 6.0                 | 4.0    | These were pretty good, but took forever to ba... | 4.0
| 1 in canada chocolate chip cookies | 45      | 12      | 11            | 595.1        | 13.0          | 26.0                | 5.0    | Originally I was gonna cut the recipe in half ... | 5.0
| 412 broccoli casserole             | 40      | 6       | 9             | 194.8        | 22.0          | 3.0                 | 5.0    | This was one of the best broccoli casseroles t... | 5.0
| 412 broccoli casserole             | 40      | 6       | 9             | 194.8        | 22.0          | 3.0                 | 5.0    | I made this for my son's first birthday party ... | 5.0
| 412 broccoli casserole             | 40      | 6       | 9             | 194.8        | 22.0          | 3.0                 | 5.0    | Loved this. Be sure to completely thaw the br...  | 5.0

The `merged_recipes` dataframe has 234429 rows and 25 columns.

### Univariate Analysis

The first few recipes we have are all rated very highly. Let's take a look at all of the non-null recipe ratings to see how people feel about the recipes on food.com in general!

<iframe
  src="assets/univariate_ratings.html"
  width="600"
  height="400"
  frameborder="0"
></iframe>
It looks like the recipes tend to have very high ratings in general! This means that most people think the recipes in our dataset are very good, so we'll have to keep this in mind!

Let's also take a look at the distribution of calories in our dataset, so we know what the average recipes' calories are.

<iframe
  src="assets/univariate_calories.html"
  width="600"
  height="400"
  frameborder="0"
></iframe>
_Note: Around 1% of the values for calories fall in the range 2500 - 45609, and are excluded from the distribution (outliers)_

The distribution of calories appears to be skeweed right. A majority of the recipes' calories fall under the 25 - 500 range, but there are a couple of skewed values that could potentially introduce bias in later analyses.

### Bivariate Analysis

---
## Assessment of Missingness

### NMAR Analysis

There are three columns in our dataset that have a non-trivial amount of missingness: `'description'`, `'rating'`, and `'review'`. Two of these columns could potentially have NMAR data. Specifically, the `'description'` and `'rating'` column could both contain data that is Not Missing at Random (NMAR).

The column that includes the description of recipes have a few values that are missing. The descriptions themselves are probably missing because the user who submitted the recipe did not include a description. However, there isn't really a clear relationship between a recipe's description and the rest of the data. The user who provided the recipe can include whatever they want in the description, and it could be completely unrelated to the recipe itself; there is no way to predict what the missing values could be. As such, I suspect that the `'description'` column is NMAR.

Another column that I suspect could be NMAR is the `'rating'` column. The values in this column are missing because we replaced all ratings of 0 with np.nan, but the rating was 0 in the first place because the user who posted the review did not include a rating. We cannot predict what the user's rating would have been, since it's difficult to predict what the rating the user would have given simply based off of their review text. Furthermore, some reviews might not even be reviews - users might have left comments or questions about the recipe instead, which makes it impossible to tell if a user would have left a high or low rating. Therefore, I suspect that the missingness of the `'rating'` column is NMAR.

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

**P-Value**: 0.6799

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

**P-Value**: 0.0063

Since our P-value was lower than our significance level (0.05), we can **reject** the null hypothesis. It's possible that the missingness of the `'review'` column can depend on the number of calories present in a recipe!

---
## Hypothesis Testing

Now, let's answer part of the question we stated in the introduction. Specificially, the question we are analyzing here is:

**Does it take longer to prepare recipes that have more calories?**

To answer this question, we will run a permutation test with the following:

**Null Hypothesis**: The average time taken to prepare recipes with a high calorie content is equal to the average preparation time of recipes with a low calorie content.

**Alternative Hypothesis**: It takes longer to prepare recipes with a high calorie content when compared to recipes with a low calorie content.

**Test Statistic**: Difference in means for preparation time of recipes with a high and low calorie count

**Significance Level**: 0.05

First, we have to define what is considered a "high" calorie count. According to [Livestrong.com](https://www.livestrong.com/article/440135-recommended-calorie-intake-for-one-meal/), the average calorie intake per meal for women is between 533 to 800, and between 667 to 1000 for men, though it can vary. For this test, we will classify recipes with a high calorie count as recipes that include **over 1000 calories**.

To conduct the permutation test, we will shuffle the `'Calories (#)'` column 10000 times, and we will calculate the difference in means for the preparation time in minutes between recipes with at least 1000 calories and recipes with less than 1000 calories.

<iframe
  src="assets/hypothesis_test.html"
  width="600"
  height="400"
  frameborder="0"
></iframe>

**Observed Statistic**: 66.0130

**P-Value**: 0.0656

The P-value here is higher than our significance level (0.05), meaning we **fail to reject** the null hypothesis. We do not have convincing evidence that the average time taken to prepare recipes high in calories is longer than the average preparation time for recipes low in calories!

---
## Framing a Prediction Problem
---
## Baseline Model
---
## Final Model
---
## Fairness Analysis
