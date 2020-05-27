---
layout: post
title: Making a Flask App and deploying it on Heroku
---

A few people have asked me about what it takes to get a Flask app working and deployed.

Over the past few weeks, I've been building a website called [reddit-topics](https://reddit-topics.herokuapp.com) that gives, at a glance, the topics that people are currently talking about on an arbitrary subreddit, plus a little bit of visualization.

This blog post is not about the nuts and bolts of getting an NLP model to work, but rather the process of creating a Flask app and then deploying it on Heroku. Similarly, I won't really touch on making the web page pretty - instead I'll just link you to [Bootstrap](https://getbootstrap.com/docs/4.4/layout/overview/) and give you the tools to make your page as pretty as you like it.

[The source code itself can be found here.](https://www.github.com/jonjonchu/metisproject4)

# the project

The website [reddit](https://www.reddit.com) works as a great news aggregator, with users submitting posts to specific subreddits - but the site is a RAM hog and it can take some time to scroll through enough posts to get an idea of what is currently happening.

This project grew out of a question: **How can we use unsupervised learning to find out what people are talking about in the news?**

The solution involves extracting topics from reddit using natural language processing (NLP).

# modelling methods

We start with scraping reddit: the [Python Reddit API Wrapper](https://praw.readthedocs.io/en/latest/) (PRAW), which allows us to access reddit. [Here](https://towardsdatascience.com/scraping-reddit-data-1c0af3040768) is a wonderful tutorial on TowardsDataScience on how to get PRAW up and running.

After that, it was a relatively straightforward shot - extracting post titles, cleaning them, then passing them into a [TF-IDF vectorizer](http://www.tfidf.com/) before extracting topics with [SVD](https://towardsdatascience.com/svd-8c2f72e264f).

# flask

Below is a sample of code from the flask implementation. We'll go over what each bit means after you've had the chance to read the code.

```python
from flask import Flask, request, render_template
# import other libraries, etc

# Initialize the app
app = Flask(__name__)

# Write other functions
def do_something_with_query(query):
    # Scrape reddit
    # Get titles from scraped data
    # Vectorize with TF-IDF
    # Get topics from vectors using SVD
    return topics_list

@app.route("/", methods=["GET","POST"])
def main_function():
    query = request.args.to_dict()

    topics_list = do_something_with_query(query)

    return (render_template('home.html',
                            subreddit=query['subreddit'],
                            post_type=query['postType'],
                            topics=topics_list,
    ))

if __name__ == '__main__':
    app.run(debug=False)
```

---

Ready?

The first line:

```python
from flask import Flask, request, render_template
```

gives us the functionality we need:

- ```Flask``` is the framework for the app itself
- ```request``` is the library we need to process ```GET``` and ```POST``` requests from the web page itself
- ```render_template``` allows us to load an html template file.

## Let's tackle ```Flask``` first.

```python
app = Flask(__name__)
```

The line above just instantiates an object of the Flask class.

```python
@app.route("/", methods=["GET","POST"])
```

This ```@app``` decorator does two things: first, there's a function defined just after it that I called ```main_function```. The decorator fires when we visit the page associated with the ```"/"``` parameter (in this case, the home page of the website). The ```methods``` parameter means that the same enclosed function will take an HTML ```GET``` or ```POST``` request method.

```python
if __name__ == '__main__':
    app.run(debug=False)
```

Lastly, this line above is necessary for Heroku (amongst other things). It's common to just write ```app.run()```, but this won't fly because Python interpreters will run the script differently if they think it's being imported as a module. If the script is *not* being run as a module (i.e. directly), the interpreter will set the ```__name__``` variable as ```'__main__'```.

## ```request``` and how it's used here

```python
def main_function():
    query = request.args.to_dict()
```

request will automatically pick up if the website has sent data (either using ```POST``` or ```GET```), and the ```.args.to_dict()``` bit simply gets the arguments from the request and slots them into a dictionary.

I used a ```form``` in the html page to provide info:

```html
<form action="/" method="GET">
```

Note the ```action``` and ```method``` attributes in the form need to match those in ```@app.route()```

## ```render_template``` and making life easier

```render_template``` works in one way: you make a folder called ```templates``` in your current working directory, and put some html files in there. Calling ```render_template()``` will then serve the user with said html file. So,

```python
return (render_template('home.html',
                        subreddit=query['subreddit'],
                        post_type=query['postType'],
                        topics=topics_list,
       ))
```

will serve the ```home.html``` file in the ```templates``` folder.

But what are the other parameters? What's up with ```subreddit=query['subreddit']```? Well, the right side of the equation is the query from the ```request``` call we talked about in the last section. But the left side of the equation is a variable that can be placed in the html file, surrounded by two sets of curly braces.

```
{{subreddit}}
```

And that little phrase will be replaced by whatever we set ```subreddit``` equal to. So if ```query['subreddit']``` is ```'Harold had a little lamb'```, then our html page will show ```Harold had a little lamb``` instead of ```{{subreddit}}```.

# Heroku

After the Flask app is ready (i.e. it runs locally), the next logical step is deployment so that everyone else can use it!

Heroku offers a nice free tier of app deployment. [The basics of the setup process can be found here. **FOLLOW THE INSTRUCTIONS**](https://devcenter.heroku.com/articles/git).  Download the heroku Command-Line Interface (CLI) beforehand.

First, though, two things are needed: a ```Procfile``` and a ```requirements.txt``` file.

## The ```Procfile```

[The ```Procfile``` defines what process to run](https://devcenter.heroku.com/articles/procfile
). We are running our app with Gunicorn, so the procfile for this project looks just like this:

```python
web: gunicorn reddit_app:app --log-file=-
```

where ```reddit_app.py``` is the flask app script.

## The ```requirements.txt``` file

```requirements.txt``` is a list of packages needed for the project to run.

```
flask
gunicorn
praw
pandas
numpy
sklearn
nltk
bokeh
```

You can specify versions like ```Flask==1.0.2``` or ```Flask>=1.0.1```. Heroku will automatically install the packages necessary.

# So it's ready?

No. In all honesty, the first Flask app I made didn't deploy and I had to do ten rounds of bug-smashing before it finally got off the ground under its own power. But at that point, problem-solving is very closely tied to Googling ability. Which is to say, *you can do it*.