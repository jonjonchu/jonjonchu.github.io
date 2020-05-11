---
layout: post
title: Predicting Drug Consumption from Personality
---

There's a great dataset on the UCI Machine Learning Repository about [drug consumption](http://archive.ics.uci.edu/ml/datasets/Drug+consumption+%28quantified%29#) and personality types. It asks 1885 respondents questions about their most recent use of eighteen drugs, and records their score for seven personality types along with basic data like age, education, gender, country and ethnicity.

We're going to use this data to create a [Flask app](dry-stream-60533.herokuapp.com) that predicts drug usage using personality traits.

Ready?

## The Data

This set came with every variable already normalized, so data cleaning was a breeze.

I emailed the study authors and downloaded a [pre-print paper](https://www.researchgate.net/publication/338737362_Personality_Traits_and_Drug_Consumption_A_Story_Told_by_Data?) on their findings. This gave me a serious leg up in figuring out where to start.

After doing some EDA, it seemed wise to create a new feature, flagging whether or not a respondent had ever used an illicit drug before. This ```monthly_illicit_user``` feature came recommended from the study.

![Personality types for different frequencies of Cannabis use.](https://jonjonchu.github.io/images/cannabis_pers_by_usage.png)

## The Methods

