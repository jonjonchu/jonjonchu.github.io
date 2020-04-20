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

![Anchorman](www.jonjonchu.github.io/images/ensembles_anchorman.jpg)

One way is to measure the box-office impact of an entity's previous films. So for example, to measure Will Ferrell's impact in Anchorman, we would take the median of his previous film's box office grosses. I did this for every category that had something to do with humans (Distributor, Director, Cast1-4, Writer, Producer, Cinematographer).

## Exploring and Modelling

![Top 100 Movies](www.jonjonchu.github.io/images/top100movies.svg)

I used three models to compare against each other: a polynomial linear regression of degree 2, a LASSO model, and a Ridge model. Each was cross-validated on five folds of the training data, but with 20% of the dataset held out for final testing and evaluation.

First, I threw in all the features and recoiled in horror at my output. The results from the LASSO model strongly suggested that I remove most of the variables.
