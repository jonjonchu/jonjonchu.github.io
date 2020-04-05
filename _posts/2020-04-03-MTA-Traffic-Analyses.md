---
layout: post
title: MTA Traffic Analyses- Finding outliers
---

Today we'll look at analyzing NYC's MTA subway transit data - specifically, removing outliers and doing general cleanup. 

## Downloading and cleaning the data

Head on over to [the MTA developers site](http://web.mta.info/developers/turnstile.html) to find the data. It's updated weekly and goes back to 2010 (!). For the most part, the dataset seems pretty clean; there's only a few cleaning operations that need to be done:

* Entries and Exits are calculated as cumulative running totals, but we'd like to see the number of Entries/Exits per measuring interval.
* Sometimes turnstiles count down Entries/Exits or inexplicably change counts in the middle of the day, which results in enormous counts or incredibly negative numbers.

Mosey on over to [this iPython Notebook](https://github.com/jonjonchu/jonjonchu.github.io/resources/CHU_turnstilesFindOutliers.ipynb) to see the actual implementation.

## Getting Entries/Exits per measuring interval

We're going to do two distinct operations in order to get our result:

1) Group the data by turnstile and day - for each turnstile there will be X groups, where X is the number of days

2) Find a method (hint: it's **diff()**) to compare the entries in each day group

```
df['ENTRIES_DIFF'] = ( df.groupby(['STATION_ID','UNIT','SCP'],as_index=False)['ENTRIES']
                           .transform(pd.Series.diff)['ENTRIES']
)
```
What's happening here?

First we have a groupby with three criteria: 'STATION_ID', 'UNIT', and 'SCP'. Those three values point to unique turnstiles. We consider only the ['ENTRIES'] column of that group.

Secondly we have a transform() function that applies the diff() function to the ['ENTRIES'] column yet again.

It's important to specify ['ENTRIES'] in both parts of the command. Pandas is in part so powerful because we could have specified other columns and come up with completely different analyses.

## Removing outliers

Some turnstiles will output numbers in the billions, which can be a real pain when trying to add up total daily entries (or adding up entries in any way, really). 

There are a few ways (conceptually) to remove outliers, but for this implementation we'll be using the standard deviation of the entries *for each turnstile* to find out which values to remove. 

If we assume the data to be normally distributed (which is its own can of worms, but let's roll with it), that means that **99.7% of the data lies within +/- three standard deviations of the mean.** Look it up if you don't believe me.

So we need to do two things:

1) Get the standard deviation for each turnstile (for all dates/times, mind you)

2) Remove all values greater than +/- three standard deviations from the mean.

```
df = df[df.groupby(['STATION_ID','SCP'])['ENTRIES_DIFF']

.apply(lambda x: np.abs(x - x.mean()) / x.std() < 3)]
```

So, what just happened?

The first part of the codeline is *almost* the same as in our previous example up above. We're grouping the data by 'STATION_ID', 'SCP', and 'ENTRIES_DIFF'. Note that we're grouping by 'ENTRIES_DIFF' and not 'ENTRIES' this time because we want our entries per measuring interval. 

The second part is an apply(), which (oh surprise) applies the function specified within its parentheses to a DataFrame/Series object.

In our case, the function being applied is a lambda function - for more on these, talk to my friend Google. This particular lambda function takes in a variable **x** and only returns it if the absolute value of x - the mean of all x's, divided by the standard deviation of all x's, is less than three.

(abs( x - x.mean ) / x.std ) < 3

If we did it right, that means the 0.3% of 'ENTRIES_DIFF' outliers in each turnstile will be removed.