
[![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/duarteocarmo/interactive-dashboard-post/master?urlpath=%2Fvoila%2Frender%2Fnotebooks%2FDashboard.ipynb)  [![Python](https://img.shields.io/badge/python-v3.7-blue)]()

# From notebook to web application :tada:

[![](posts/readme_figure.png)](https://mybinder.org/v2/gh/duarteocarmo/interactive-dashboard-post/master?urlpath=%2Fvoila%2Frender%2Fnotebooks%2FDashboard.ipynb)

[![](posts/readme_figure_1.png)](https://mybinder.org/v2/gh/duarteocarmo/interactive-dashboard-post/master?urlpath=%2Fvoila%2Frender%2Fnotebooks%2FDashboard.ipynb)


## What is this? :thinking:

A blog post/tutorial for the [Practical Business Python](https://pbpython.com/) blog that teaches how to create web applications from jupyter notebooks and then deploy them. 

[Visit the dashboard](https://mybinder.org/v2/gh/duarteocarmo/interactive-dashboard-post/master?urlpath=%2Fvoila%2Frender%2Fnotebooks%2FDashboard.ipynb) (might take a bit to load because of binder)



## How do I run the notebook? :notebook_with_decorative_cover:

Clone the repo:

```bash
$ git clone https://github.com/duarteocarmo/interactive-dashboard-post.git
```

Navigate to it:

```bash
$ cd interactive-dashboard-post
```

Create a [virtual environment](https://virtualenv.pypa.io/en/latest/):

```bash
$ virtualenv env
```

Activate the environment:

```bash
$ . env/bin/activate
```

Install the requirements:

```bash
(env) $ pip install -r requirements.txt 
```

Launch [jupyter lab](https://jupyterlab.readthedocs.io/en/stable/):

```bash
(env) $ jupyter lab
```

It should launch automatically, if not, navigate to http://localhost:8888/lab 

**troubleshooting**

If the plotly express plots are not showing then try:

```bash
(env) $ jupyter labextension install @jupyterlab/plotly-extension
```

If you still have problems, follow [these instructions](https://plot.ly/python/getting-started/#jupyterlab-support-python-35).



## How do I run the dashboard? :bar_chart:

Follow the instructions above until you have the requirements installed, and then: 

```bash
(env) $ voila notebooks/Dashboard.ipynb
```

This should launch the dashboard in http://localhost:8866/
