# A College Student's Nightmare: Cooking Times
by Jason Liu (jal134@ucsd.edu)

---
## Introduction

As a college student, I face a constant struggle in trying to decide what to eat every day. Dining at restaurants can be very expensive, so cooking my own meals is what I like to choose. However, on some days, I can be quite limited in what I can actually make, since I might not have a wide variety of ingredients or I might not have enough time. Even then, I think it's important to be eating healthy, so I can have enough energy to get my work done. As such, I'd like to be able to prepare some food that gets me all the calories I need, and it needs to taste good as well (I can be a picky eater!); assuming these conditions are met, I need to know how long it'll take me to prepare my food so I can plan out my time, making sure I can get all of my homework done. All of this leads me to look into the question:

**How do the nutritial values, complexity, and quality of a recipe affect the amount of time it takes to prepare a particular recipe?**

To find out the answer to this question, I will be analyzing two different datasets.

The first dataset is our `raw_recipes` dataset. It contains data about different recipes taken from food.com that were posted after the year 2008.
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
  width="800"
  height="600"
  frameborder="0"
></iframe>

It looks like the recipes tend to have very high ratings in general! This means that most people think the recipes in our dataset are very good, so we'll have to keep this in mind!

Let's also take a look at the distribution of calories in our dataset, so we know what the average recipes' calories are.

<iframe
  src="assets/univariate_calories.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

_Note: Around 1% of the values for calories fall in the range 2500 - 45609, and are excluded from the distribution (outliers)_

The distribution of calories appears to be skeweed right. A majority of the recipes' calories fall under the 25 - 500 range, but there are a couple of skewed values that could potentially introduce bias in later analyses.

### Bivariate Analysis

---
## Hypothesis Testing
---
## Framing a Prediction Problem
---
## Baseline Model
---
## Final Model
---
## Fairness Analysis
