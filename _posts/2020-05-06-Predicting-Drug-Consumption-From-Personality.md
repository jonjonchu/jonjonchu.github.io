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

### The Personality Traits

Seven traits were measured for this dataset: Five from the NEO-FFI-R test, and two from the BIS-Impulsivity test.

They are:

**Neuroticism**: Essentially equivalent to emotional instability; irritability and moody behaviors.

**Extraversion**: Assertive, gregarious, energetic behaviors.

**Openness to Experience**: Indicates inquisitiveness, thoughtfulness, and propensity for intellectually challenging tasks.

**Agreeableness**: Empathic, sympathetic, and kind behaviours.

**Conscientiousness**: Sense of responsibility and duty as well as foresight.

**Impulsiveness**: Tendency to act without thinking.

**Sensation Seeking**: Search for experiences and feelings, that are "varied, novel, complex and intense", and by the readiness to "take physical, social, legal, and financial risks for the sake of such experiences."

## EDA

Turns out that there are a few drugs for which the data shows a strong correlation - the study authors called them pleiades, and they represent drugs which show a strong correlation in usage. The three pleiades are:

1. **The Ecstasy Pleiad**: Comprising amphetamines, ecstasy, cannabis, cocaine, ketamine, LSD, mushrooms, and legal highs, these drugs are often known as 'party drugs.'

2. **The Heroin Pleiad**: Crack, cocaine, methadone, and heroin. These roughly fall under the 'hard drug' category.

3. **The Benzodiazepine Pleiad**: Methadone, amphetamines, cocaine, and benzodiazepines.

There are some drugs which are shared between pleiades, like cocaine and amphetamines and methadone.

### So what does the personality profile of a drug user look like?

Here's a dictionary to interpret the legend:

- **0** : Never used before
- **1** : Used over a decade ago
- **2** : Used in last decade
- **3** : Used in last year
- **4** : Used in last month
- **5** : Used in last week
- **6** : Used in last day

![Personality types for different frequencies of alcohol use.](https://jonjonchu.github.io/images/alochol_pers_by_usage.png)

![Personality types for different frequencies of cannabis use.](https://jonjonchu.github.io/images/cannabis_pers_by_usage.png)

![Personality types for different frequencies of cocaine use.](https://jonjonchu.github.io/images/coke_pers_by_usage.png)

Forgive the state of these charts, they come rough from the EDA and I can't be bothered to clean them up.

Couple things to notice about the charts: There are some clear divisions between cohorts of use for some drugs (see Cannabis), but not for others. All drug users (defined as those who have used the drug before) show higher **Impulsiveness** and **Sensation Seeking** scores than their non-user peers.

It seemed wise to create a new feature, flagging whether or not a respondent had ever used an illicit drug in the past month. This ```monthly_illicit_user``` metric was chosen because many of the drugs showed different personality archetypes at around the monthly-usage mark. However, it is by no means exhaustive, and perhaps a different metric could be used to find more accuracy in the future.

## The Methods

There was a lot of trial-and-error involved, with random forests and naive Bayesian methods, but the choice of classifier models in this project came down to a few criteria:

1. **Minimizing Prediction Time**. Because this project would run on a web app, and there would be one classifier for each of the eighteen drugs, speed was arguably the most important characteristic for the models.

2. **Accuracy**. Getting as accurate of a prediction as possible was the goal. Neither Recall nor Precision were valued more than the other, so the metric chosen to score the models was the [ROC-AUC score](https://towardsdatascience.com/understanding-auc-roc-curve-68b2303cc9c5).

In the end, I chose to use XGBoost classifiers for all drugs (using the XGBClassifier object because I only know how to use sklearn), because XGBoost offered an ensemble method while still maintaining speed. I dropped the idea of using a Naive Bayes approach because a few of the personality traits were correlated. Was this the right decision? I don't know, but the model can be improved once it's running.

![Correlations between personality types](https://jonjonchu.github.io/images/drugC_personality_corr.png)

### Training the models

First, I divided the data into whether or not a person had used a drug in the past month to make this a binary classification problem, then used the the seven personality scores, age, and education as features to try and predict ```monthly_illicit_user``` and a few of the legal drugs, ```alcohol```, ```chocolate```, and ```caffeine```. 

```monthly_illicit_user``` would serve as another feature to feed into the models for the other drugs. I then trained a model for every other drug using RandomSearchCV to try and optimize hyperparameters. From these, only a few drugs showed ROC-AUC scores over 0.5: ```amphetamines```, ```benzodiazepines```, ```cannabis```, ```ecstasy```, and ```nicotine```.

Now, in a bid to increase accuracy, I took the results of the five drugs above and used those as feature inputs for the rest of the drugs yet again. When this was done, the ROC-AUC score for two more drugs increased: ```methadone``` and ```cocaine```.

This process was repeated once more for the last few drugs, where all the personality traits and the results from previous models were used as feature inputs. 

## Results

### [Try the app](https://www.dry-stream-60533.herokuapp.com)

Was it worth it? Yes. Is the approach flawed? I think so. Given the time and resources, I would definitely change a few things:

1. Revisit the cohorts and divide them up differently. Perhaps the monthly user frequency is too high, and better results would be found using the yearly cohort?

2. Reduce the amount of features for each model. Different personality traits had different importances on each drug. For example, crack had its best results from only using Extraversion and Conscientiousness as its features, while ecstasy's most important features were Age and Sensation Seeking.
