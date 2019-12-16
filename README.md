

# interactive-dashboard-post

A blog post for PBP

## Running the notebook

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

**troubleshooting**:

If the plotly express plots are not showing then try:

```bash
(env) $ jupyter labextension uninstall @jupyterlab/plotly-extension
```

You might need to install [nodejs](https://nodejs.org/en/) to run them. 



## Launching the dashboard

Follow the instructions above, but instead of launching jupyter lab: 

```bash
(env) $ voila notebooks/Dashboard.ipynb
```

This should launch the dashboard in http://localhost:8866/



