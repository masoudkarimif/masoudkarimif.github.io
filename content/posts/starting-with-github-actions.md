---
date: "2023-03-26"
title: "Starting with GitHub Actions"
tags: ["github", "github-actions", "cicd"]
author: "masoudkf"
description: "GitHub Actions is a Continuous Integration (CI) and Continuous Delivery (CD) platform that helps you automate your software development and delivery process right from the place you store your code."
---

## Overview
GitHub Actions is a Continuous Integration (CI) and Continuous Delivery (CD) platform that helps you automate your software development and delivery process right from the place you store your code. It lets you build, test, and deploy your applications. It has a generous free-tier (2,000 minutes/month for private repos) that you most likely won't cross.

With GitHub Actions, you create Workflows to automate the different steps of your software delivery system. GitHub provides Linux, Mac, and Windows machines for running Workflows. GitHub Actions calls these platforms Runners; you can find the list of available runners, plus the tools that come with them [here](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners#supported-runners-and-hardware-resources). Workflows are written in [YAML](https://yaml.org/) and stored inside `.github/workflows` folder in your GitHub repository.

Each Workflow has 4 main sections:
- Event (events happening in your GitHub repo that trigger your workflows, such as `push` and `pull_request`)
- Job (a series of steps with a shared purpose)
- Steps (phases of a job that need to happen one after another)
- Actions/Command (code or commands that need to be executed in a particular step)

## Workflows
As mentioned above, Workflows can automate different steps in your software delivery system, from building and testing, to deploying to production. Here's an example of a GitHub Actions Workflow.

`.github/workflows/sample.yaml`
```yaml
# name of the workflow.
# this is optional.
name: Continuous Integration

# events that will trigger this workflow.
# here, we only have "pull_request", so the workflow will run
# whenever we create a pull request.
# other examples: [push] and [pull_request, push]
on: [pull_request]

# each workflow must have at least one job.
# jobs run in parallel by default (we can change that).
# each job groups together a series of steps to accomplish a purpose.
jobs:
  # name of the job
  first:
    # the platform or OS that the workflow will run on.
    runs-on: ubuntu-latest

    # series of steps to finish the job.
    steps:
      # name of the step.
      # steps run sequentially.
      # this is optionale
      - name: checkout
        # each step can either have "uses" or "run".
        # "uses" run an action written somewhere other than this workflow .
        # usually from the community.
        # this action checks out the repo code to the runner (instance)
        # running the action
        uses: actions/checkout@v3

      # another step.
      # this step runs a bash (Ubuntu's default shell) command
      - name: list files
        run: ls
```

### Test Python Code with GitHub Actions
We'll be creating a GitHub Actions Workflow to test our Python code whenever we create a Pull Request. We'll use [Pytest](https://docs.pytest.org/en/7.2.x/) to test our code.

We start by writing some rudimentary Python to do simple math operations. Create a `main.py` file and add the following code:

`main.py`
```python
def add(a, b):
    return a+b

def subtract(a, b):
    return a-b

def divide(a, b):
    return a//b

def multiply(a, b):
    return a*b
```

Now, we create a test file named `test_main.py` in the same directory as `main.py`:

`test_main.py`
```python
from main import *

def test_add():
    res = add(5, 5)
    assert res == 10

def test_substract():
    res = subtract(10, 5)
    assert res == 5

def test_multiply():
    res = multiply(5, 5)
    assert res == 25

def test_divide():
    res = divide(25, 3)
    assert res == 8
```

We can now run our tests locally using the `python -m pytest -v` command. Create a GitHub repository and link it to the project (current directory):

```bash
# make the current directory a git project
git init

# add the GitHub repo as a remote
git remote add origin <github-repo-address>

# stage everything
git add -A

# commit 
git commit -m"initial"

# make upstream and push
git push -u origin main
```


Now, let's create a GitHub Actions Workflow to run our tests automatically whenever we create a Pull Request.

We create a Workflow file named `ci.yaml` in `.github/workflows`:

`.github/workflows/ci.yaml`

```yaml
name: Continuous Integration

on: 
- pull_request

jobs:
  testing:
    runs-on: ubuntu-latest

    steps:
      - name: checkout
        uses: actions/checkout@v3

      - name: install dependencies
        run: pip install pytest

      - name: test
        run: python -m pytest -v
```

Create a new branch, commit, and push:

```bash
# create a new branch and checkout to it
git checkout -b gh-action

# stage everything
git add -A

# commit and push
git commit -m"adds workflow"
git push origin gh-action
```

Now, we need to create a Pull Request on GitHub, and, if everything goes according to plan, our Workflow should start automatically once the Pull Request is created.

![checks-passed](https://res.cloudinary.com/mkf/image/upload/v1679895578/checks-passed_de9vy2.png)
