---
layout: post
title: Box Office Gross - Regression
---

Can one predict Domestic Box Office Gross by using data from BoxOfficeMojo.com?

Check out the code on github [here](https://www.github.com/jonjonchu/metisproject2)

I pulled movie data from the years 1980-2020 and used linear regression models to predict domestic gross.

## Web Scraping

Using BeautifulSoup4, I wrote a quick [Jupyter notebook](https://github.com/jonjonchu/metisproject2/blob/master/boxOfficeMojoPipeline.ipynb) to pull the required films. In essence it scrapes every film by year on BoxOfficeMojo.com, then pulls the following information:

- Movie Title
- Domestic Distributor
- Domestic Total Gross
- Runtime
- Rating
- Release Date
- Budget
- Cast (First four billings)
- Director
- Writer
- Producer
- Cinematographer

The data is inserted into a Pandas DataFrame and then pickled for later use.

## Data Cleaning

Once all the movie data for every year was collected, I used a [separate notebook](https://github.com/jonjonchu/metisproject2/blob/master/boxOfficeMojoCleaner.ipynb) to combine records for every year into one dataframe, did a little bit of normal data cleaning (timestamps, etc), then moved on to some feature engineering.

The biggest blocker here is the categorical nature of most of the variables in the data - linear regression doesn't really work on non-numerical values. So the real question becomes, how can we quantify the impact of a distributor, director, writer, or cast member?

![Anchorman](https://jonjonchu.github.io/images/ensembles_anchorman.jpg)

One way is to measure the box-office impact of an entity's previous films. So for example, to measure Will Ferrell's impact in Anchorman, we would take the median of his previous film's box office grosses. I did this for every category that had something to do with humans (Distributor, Director, Cast1-4, Writer, Producer, Cinematographer).

## Exploring

First thing to do was some exploratory data analysis. Below you can see the top 10 movies by US domestic gross. Clearly there's some sort of exponential function going on, so it would behoove us to do a log-transform of the gross and do our regression on that.

![Top 100 Movies](https://jonjonchu.github.io/images/top100movies.svg)

Secondly, this chart below shows gross by runtime. Perhaps there's a relationship? We'll add it to the list of features to investigate.

![Movies by Runtime](https://jonjonchu.github.io/images/GrossbyRuntime.png)

Lastly, the chart below shows a clear bimodal split in gross. Quite a few films only make (converting from log $ back to $) $60K to $250K, and there's another peak from $20M to $50M. 

![Log Gross](https://jonjonchu.github.io/images/bimodalLogDTG.svg)

For this project, let's look at the right-side hump of $20M to $50M-grossing films. After inspecting the budgets, it's clear that only around 20% of films in the database had budgets reported. And, in fact, the budgets line up more or less with those same right-hump films!

So we will do a few things before modelling:

1. Consider only films with gross between $1M and $250M
2. Consider only films with budgets > $1M
3. Take the log of both budget and domestic total gross, to help linearize those parameters.

## Modelling

I used three models to compare against each other: a polynomial linear regression of degree 2, a LASSO model, and a Ridge model. Each was cross-validated on five folds of the training data, but with 20% of the dataset held out for final testing and evaluation.

First, I threw in all the features and recoiled in horror at my output. The results from the LASSO model strongly suggested that I remove most of the variables.

Surprisingly, runtime seemed to make little difference to gross, or at least any difference that the model could discern. Rating, too, held little significance, except for one: R-rated films tend to make slightly less money.

In addition, none of the impact scores (save Domestic Distributor) affected gross significantly. The model actually performed betterr when they were removed (in terms of adjusted R2 values).

In the end, I found only a few features that significantly (p < 0.05) affected Domestic Total Gross:

- Budget
- Domestic Distributor Score
- R rating

Below is a chart of predicted vs obesrved values from all the models I used. The OLS polynomial model did better than the other two, but not by much. There is still a predilection toward underestimating gross in the model, which means that there is likely a factor that is not captured by this model.

![Pred. vs. Obs.](https://jonjonchu.github.io/images/predvsobsDTG.png)

The residuals show the same bias: as the gross gets larger, the prediction underestimates the true value by larger and larger margins. Note that both Prediction and Residual here are on a log scale ( log(gross) ).

![Residuals](https://jonjonchu.github.io/images/Residuals.svg)

This QQ plot, however, shows that the model does well when fitting films close to the mean, but struggles near the outliers.

![QQ plot, OLS poly=2](https://jonjonchu.github.io/images/OLSQQ.svg)

## Review and Future Work

It's clear that some feature is missing that could help explain the systemic error in the model. I posit that adding marketing budget (or some other marketing metric) would help to increase the model's accuracy, since widely-known films tend to draw attention (and thus money).

I would also like to attempt to reverseo-engineer budget from gross, and even construct a model for the films that gross less than $1M.
