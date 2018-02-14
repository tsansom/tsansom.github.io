---
layout: post
comments: true
title: "Clustering Neighborhood Change in Austin"
subtitle: "Part 3 - Exploratory Data Analysis"
data: 2018-02-14 04:10:27 -0600
categories: austin clustering census capstone project python
---

The code for this EDA is located in the project repository in the notebook [Exploratory Data Analysis](__insert link__)

## Data Preparation Summary
I'll start by doing a quick summary of the steps taken for data preparation thus far.

* Raw data gathered and cleaned of unnecessary columns
* 2000 census tracts were converted to 2010 census tracts using population relationships
* Binned variables condensed into a single index value
* Monetary variables adjusted for inflation relative to 2016
* Population variables converted to percentages
* Rent index imputed for missing values
* Special neighborhoods that don't fit study were removed

So now my cleaned and prepped dataset contains 343 neighborhoods (a.k.a. census tracts or geoids) spanning 9 years (2000 and 2009-2016) for a total of 3,087 observations with 6 features per neighborhood.

## High-Level Feature Comparisons
For this section, all observations will be investigated together, independent of the year they were collected. This can be done since the data has been adjusted for inflation, indexed, and converted to per capita proportions. Later we will see how each feature and neighborhood changes over time.

### Correlation Matrix
I'll start by seeing how correlated each of the features is to each other.

![correlation_matrix]({{ site.url }}/images/cluster_proj/corr_matrix.png)

From the above correlation matrix, we can see that all features have positive correlations between 0.27 and 0.75. Education attained shows high correlation with income and home value (0.71 and 0.74 respectively) which is not surprising since higher education typically leads to higher household income which in turn leads to the ability to purchase more expensive housing. Rent and income show similar correlation.

I am surprised to see that the correlation between rent and home value is not higher than 0.48. My intuition would be that neighborhoods with higher home values would also be neighborhoods where rent is high. Perhaps this is reflective of the large number of neighborhoods which intermingle residential housing and apartment complexes.

Another interesting point that opposes my initial expectations is that there is very low correlation between the employment percentage of a neighborhood and the home value even though the correlation between employment rate and income is moderate. Thinking a bit deeper, I can reason how neighborhoods with the highest employment rate would probably not coincide with either the lowest or highest home values. On one hand, I don't expect neighborhoods with the lowest home values to have full employment because of higher likelihood of a lack of education and substance abuse. Similarly, neighborhoods with the highest home values probably have a large portion of the population that are independently wealthy or comfortably retired, which don't need to work.

I'll take a closer look at each one of these relationships a bit later.

### Scatter Matrix
While correlation gives a good initial idea of how the features relate to each other, a scatter matrix will give finer relationship details between features. Additionally, the distribution of each feature can be easily visualized with a single line of code.

```python
_ = pd.plotting.scatter_matrix(data[cols], figsize=(12, 12), alpha=0.1,
                               diagonal='kde', marker='.')
```

![scatter_matrix]({{ site.url }}/images/cluster_proj/scatter_matrix.png)

Now we can start to see some of the intricate relationships between the features. Subplots with a large spread in scatter points have a weaker relationship than subplots with tight grouping of points. By using a very low alpha (transparency) value for each point, the darkest color appears where many points overlap. This dark section of each subplot is where the relationship between the features is the most common. For example, the scatter plot of rent versus home value has a very dark region in the lower left corner in which most of the points fall. Beyond that, the relationship gets weaker and point are more spread out, thus the color is lighter. Subplots with a less pronounced dark region represent features with a weak relationship which tend to have a greater spread in scatter points (as seen in the education vs percentage white subplot).

### Select Feature Comparisons
In this section I'll pick a few of the most interesting feature relationships to explain in greater depth. For the comparisons I'll be using a combination of scatterplots, histograms, and kernel density estimation contours. Seaborn offers an information rich combination plot which is perfect for this sort of comparison (jointplot).

#### Income vs Home Value
![income_value]({{ site.url }}/images/cluster_proj/joint_income_value.png)

There is clearly a linear relationship between income and home value, which is not surprising. The lower bound of this linear relationship is very pronounced while the upper bound is more variable. Some outliers in this plot show relationships between income and home value that don't make much sense. For instance, in the lower left-hand corner there are a group of observations that fall outside the density contours that have the lowest income (<\\$20k) but comparatively high home values (\\$200k - \\$300k).

How could this be? It turns out that all of these points belong to two neighborhoods (48453000603 and 48453000604) which are located directly West of the University of Texas campus and have a large student population. The student population drives the income index down while the disirable geographic location drives home values up.

### Education vs Income
To understand this relationship I need to explain how the education index was calculated. Education attained was originally a binned ordinal variable which was converted into an index using a simple scoring method. The scores for each level of attained education are described below:

1 - Less than 9th grade  
2 - Some high school  
3 - Graduated high school  
4 - Some college  
5 - Associate's degree  
6 - Bachelor's degree  
7 - Graduate degree (includes doctorate and professional degrees)

So an education index of 6 means that the average education attained for a neighborhood would be a bachelor's degree. Now, we can move forward with the comparison.

![education_income]({{ site.url }}/images/cluster_proj/joint_education_income.png)

While there is clearly a positive psuedo-linear relationship between education attained and income, there seems to be a breakpoint around an education index of 5 (which corresponds to associate's degree attainment). To the right of this breakpoint, income is much more variable although the potential maximum income is much higher in comparison to lower education attainment. As in the previous comparison of income to home value, the cluster of points at the bottom right of the plot (high education, low income) correspond to neighborhoods with large percentages of students.

#### Home Value vs Percentage White

![value_white]({{ site.url }}/images/cluster_proj/joint_value_white.png)

This plot alone tells a very profound socioeconomic story about Austin. While the city boasts about racial equality and affordability, they fail to tell you as these things are apparently mutually exclusive. For instance, the top 100 neighborhoods in terms of home value are nearly 90% white! I've personally driven through many of these neighborhoods and can attest to their monochromatic demographics.

## Neighborhood Change Over Time
To capture the change in each neighborhood over time, I'll calculate the difference in each feature between the last (2016) and first (2000) years in the dataset. I'll also get some descriptive statistics for the neighborhood change.

```python
changes_2000_2016 = data[data['year'] == '2016'][cols] \
                  - data[data['year'] == '2000'][cols]
changes_2000_2016.describe()
```

![neighborhood_change_describe]({{ site.url }}/images/cluster_proj/neighborhood_change_describe.png)

There are a couple of troubling trends here that highlight the growing price inequality in Austin. First, household income is declining while home values are increasing. On average, household income per neighborhood has decreased by almost \\$5,000 while home values have increased by about \\$62,000.

Second, most neighborhoods are becoming less diverse with the percentage white population changing on average +5.8% with a maximum change of +55.5%! Think about that for a second... over half of a neighborhoods minority population being replaced by only white residents in a span of 16 years. Luckily for the displaced populations, rent has remained relatively stable and affordable, although most would probably prefer to own as opposed to rent.

### Neighborhood Change Map
To get a grasp of the spatial extent of neighborhood change in Austin, I've created an interactive data viewer with Bokeh (which I'll describe in a different post) that captures some different aspects of the issues described throughout this post. There is a separate tab for each feature. Red denotes positive change while blue denotes negative change. Color intensity represents the magnitude of change. The actual value of change can be seen by moving the mouse over each neighborhood.

{% include change_viewer.html %}
