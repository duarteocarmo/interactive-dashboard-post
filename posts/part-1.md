# Creating live dashboards from Jupyter Notebooks

# Part 1: The data



### About the author

Hey everyone! My name is [Duarte O.Carmo](https://duarteocarmo.com) and I'm a Consultant working at [Jabra](https://jabra.com) that loves working with python and data. Make sure to visit my website if you want to find more about me :smile: 



1. The overall goal
2. Getting the data: pushshift
   1. Making the function
3. The Analysis
   1. Comment and Submission activity activity
   2. Upvote tables
   3. Sentiment timeline
4. Using voila
5. Deploying the solution



### 1. The Goal

Jupyter notebooks are one of my favorite tools to work with data, they are simple to use, fast to set up, and flexible. However, they do have their disadvantages: source control, collaboration, and reproducibility are just some of them. I tend to enjoy [seing what I can accomplish with them](https://pbpython.com/papermil-rclone-report-1.html).

An increasing need is the sharing of our notebooks. Sure, you can export your notebooks to html, pdf, or even use something like [nbviewer](https://nbviewer.jupyter.org/) to share them. But what if your data changes constantly? What if every time you run your notebook, you expect to see something different? How can you go about sharing something like that? 

> But what if your data changes constantly? What if every time you run your notebook, you expect to see something different? How can you go about sharing something like that? 



In this article, I'll show you how to create a Jupyter Notebook that fetches live data, and then how to deploy it as a live dashboard. So that all you need to share with someone is a link. 

Let's have some fun with the data first. 

### 2. Getting live Reddit data

Reddit is a tremendous source of information, and there are a million ways to get access to it. 

My favorite? A small API called pushshift, to which the [documentation is right here](https://github.com/pushshift/api). 

Let's say you wanted the most recent comments mentioning the word "python", then, in python, you could:

```python
import requests

url = "https://api.pushshift.io/reddit/search/comment/?q=python"

request = requests.get(url)

json_response = request.json()
```

You can add a [multitude of parameters](https://github.com/pushshift/api#search-parameters-for-comments) to this request (in a certain subreddit, after a certain day, sorted by up votes, etc)

To make my life easier, I built a function that allows me to call this API as a function: 

```python
def get_pushshift_data(data_type, **kwargs):
    """
    Gets data from the pushshift api.
    
    data_type can be 'comment' or 'submission'
    The rest of the args are interpreted as payload.
    
    Read more: https://github.com/pushshift/api
    """
    
    base_url = f"https://api.pushshift.io/reddit/search/{data_type}/"
    payload = kwargs
    
    request = requests.get(base_url, params=payload)
    
    return request.json()
```

Using the `payload` parameter and `kwargs` I can then add any payload I wish as a function. For example,

```python
get_pushshift_data(data_type="comment",  	# give me comments
                   q="python", 				# that mention 'python'
                   after="48h", 			# in the last 48 hours
                   size=1000, 				# maximum 1000 comments
                   sort_type="score", 		# sort them by score
                   sort="desc")				# sort descending
```

returns the json response. Pretty sweet right? 

 

### 3. Analyzing the data

#### 3.1. In what subreddits does the word 'python' appear more?

To answer the above question, we start by getting the data with our function:

```python
data = get_pushshift_data(data_type="comment",
                          q="python", 
                          after="48h", 
                          size=1000, 
                          aggs="subreddit")
```

The `aggs` keyword asks pushshift to return an aggregation into subreddits, which basically means, group the results by subreddit. ([read about it in the documentation](https://github.com/pushshift/api#using-the-subreddit-aggregation))



Since the json response is pretty nested, we'll need to navigate a bit inside of the dictionary. 

```python
data = data.get("aggs").get("subreddit")
```



And we transform the list of dictionaries returned into a pandas DataFrame, and get the top 10. 

```python
df = pandas.DataFrame.from_records(data)[0:10]
```



Here's what our dataframe looks like: 

|      | doc_count | key              |
| ---- | --------- | ---------------- |
| 0    | 352       | learnpython      |
| 1    | 220       | AskReddit        |
| 2    | 177       | Python           |
| .    | 139       | learnprogramming |

These are the names of the subreddits the word `python` appears more in their comments :tada::tada:!



Let's plot our results with the great [ploty express](https://nbviewer.jupyter.org/github/plotly/plotly_express/blob/gh-pages/walkthrough.ipynb) library. Plotly express is great, in my opinion if you want to:

- create figures quickly. :bullettrain_front:
- create figures that are a bit more interactive than [matplotlib](https://matplotlib.org/). :raised_hand_with_fingers_splayed:
- don't mind a bit more installation and (imo) a bit worse documentation. :sweat:



Here's all the code you need:

```python
import plotly.express as px

px.bar(df,				# our dataframe
       x="key", 		# x will be the 'key' column of the dataframe 
       y="doc_count",	# y will be the 'doc_count' column of the dataframe
       title=f"Subreddits with most activity - comments with 'python' in the last 48h",
       labels={"doc_count": "# comments","key": "Subreddits"}, # the axis names
       color_discrete_sequence=["blueviolet"], # the colors used	
       height=500, 
       width=800)
```

Yes, perhaps a bit more verbose than matplotlib, but you get an interactive chart!

![](figure_1.png)

[Visit the notebook](https://github.com/duarteocarmo/interactive-dashboard-post/blob/master/notebooks/Dashboard.ipynb) for this article.

#### 3.2. What are the most up-voted comments with the word 'python'?

To answer this question, our function will again come in handy, let's aggregate thinks a bit. Don't get scared, this is a one liner that will produce similar results to above:

```python
# get the data we need using the function 
data = get_pushshift_data(data_type="comment", q="python", after="7d", size=10, sort_type="score", sort="desc").get("data")

# we only care about certain columns
df = pandas.DataFrame.from_records(data)[["author", "subreddit", "score", "body", "permalink"]]

# we only keep the first X characters of the body of the comment (sometimes they are too big)
df.body = df.body.str[0:400] + "..."

# we append the string to all the permalink entries so that we have a link to the comment
df.permalink = "https://reddit.com" + df.permalink.astype(str)

# style the last column to be clickable and print
df.style.format({'permalink': make_clickable})
```



**sidenote**: to make a DataFrame column clickable you can can apply the following function to it:

 ```python
def make_clickable(val):
    """
    Makes a pandas column clickable by wrapping it in some html. 
    """
    return '<a href="{}">Link</a>'.format(val,val)
 ```

**end of sidenote**



The above code will return the top 10 most upvoted comments of the last 7 days:

|      | author           | subreddit            | score | body                                                  | permalink                                                    |
| ---- | ---------------- | -------------------- | ----- | ----------------------------------------------------- | ------------------------------------------------------------ |
| 0    | Saiboo           | learnpython          | 111   | Suppose you create the following python file calle... | [Link](https://reddit.com/r/learnpython/comments/eb57p0/what_is_the_point_of_name_main_in_python_programs/fb2ofbs/) |
| 1    | Kompakt          | ProgrammerHumor      | 92    | Some languages don't have switch statements...look... | [Link](https://reddit.com/r/ProgrammerHumor/comments/eb2lbc/stacking_if_else_statements_be_like/fb1vzhz/) |
| 2    | clown_world_2020 | MrRobot              | 47    | Just goes to show that Esmail isn't the only brill... | [Link](https://reddit.com/r/MrRobot/comments/eb3u62/esmails_influence_on_mac_quayles_process/fb2kipo/) |
| 3    | Leebertysauce    | AnimalsBeingBros     | 28    | They wont even be mad when the python decide to ta... | [Link](https://reddit.com/r/AnimalsBeingBros/comments/eb0ux7/capybara_is_a_foster_parent_for_a_group_of_husky/fb25ons/) |
| 4    | Kompakt          | ProgrammerHumor      | 23    | Yep it's true, and depending on the design of the ... | [Link](https://reddit.com/r/ProgrammerHumor/comments/eb2lbc/stacking_if_else_statements_be_like/fb253g9/) |
| 5    | niceboy4431      | Cringetopia          | 23    | I have a theory (someone prove me wrong if you kno... | [Link](https://reddit.com/r/Cringetopia/comments/eakrbs/i_hate_these_channels_so_much/fauiic1/) |
| 6    | kingguru         | Denmark              | 22    | Brug af Python: **+1** Brug af Python 3: **+2** ...   | [Link](https://reddit.com/r/Denmark/comments/eb0mzj/debatkulturen_på_rdenmark/fb1b27a/) |
| 7    | MintyAroma       | totalwar             | 20    | We really need Bretonnian Men-At-Arms shouting Mon... | [Link](https://reddit.com/r/totalwar/comments/eb1eyf/wait_thats_illegal/fb1vq6j/) |
| 8    | aspiringtobeme   | gifsthatkeepongiving | 19    | Amazing. Brought [this Monty Python clip](https://... | [Link](https://reddit.com/r/gifsthatkeepongiving/comments/eaqml0/watch_a_new_kid_every_time/faw7ne5/) |
| 9    | CrimsonSpooker   | TwoBestFriendsPlay   | 19    | “Why can’t Three Houses be gritty and “realistic” ... | [Link](https://reddit.com/r/TwoBestFriendsPlay/comments/eb3f6m/i_dont_think_there_are_too_many_of_these_but/fb2nisj/) |



And you can click the link column to be taken right into the comment. Hooray! :tada: 



#### 3.3. What is the sentiment in /r/python across time? 

Alright, the final one is a bit more complicated. We want to see the sentiment in the [/r/python](https://reddit.com/r/python) subreddit in some sort of time line. 

First, we already now how to retrieve the most upvoted comments of the past 2 days:

```python
# get the data with our function
data = get_pushshift_data(data_type="comment",
                          after="2d", 
                          size=1000, 
                          sort_type="score", 
                          sort="desc", 
                          subreddit="python").get("data")

# define a list of columns we want to keep
columns_of_interest = ["author", "body", "created_utc", "score", "permalink"]

# transform the response into a dataframe
df = pandas.DataFrame.from_records(data)[columns_of_interest]
```



This gives us a pandas DataFrame with the columns specified in `columns_of_interest`. But how the hell do we get the sentiment of every comment? 

Enter [TextBlob](https://textblob.readthedocs.io/en/dev/). A simple library that makes it ridiculously easy to get the sentiment of a sentence. Textblob returns two values, the sentiment **polarity** (-1 is negative; 0 is neutral; and 1 is positive) and the sentiment **subjectivity** (0 is objective and 1 is subjective)

Here's an example:

```python
import textblob

sentence1 = "Portugal is a horrible country. People drive like crazy animals."
print(textblob.TextBlob(sentence1).sentiment)
# -> Sentiment(polarity=-0.8, subjectivity=0.95)
# negative and subjective

sentence2 = "Portugal is the most beautiful country in the world because beaches face west."
print(textblob.TextBlob(sentence2).sentiment)
# -> Sentiment(polarity=0.675, subjectivity=0.75)
# positive and less subjective
```

Read more about the library [here](https://textblob.readthedocs.io/en/dev/). 



 






