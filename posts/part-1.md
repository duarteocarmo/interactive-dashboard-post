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
                   after="24h", 			# in the last 24 hours
                   size=1000, 				# maximum 1000 comments
                   sort_type="score", 		# sort them by score
                   sort="desc")				# sort descending
```

returns the json response. Pretty sweet right? 

 















 



Here is [the notebook](https://github.com/duarteocarmo/interactive-dashboard-post/blob/master/notebooks/Dashboard.ipynb) for this article.



### 

