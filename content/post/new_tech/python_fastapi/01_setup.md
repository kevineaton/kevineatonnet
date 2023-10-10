---
title: Learning Python and FastAPI - 01 - Set Up
description: Learning Python and FastAPI - 01 - Set Up
date: "2023-06-01 15:00:00"
publishDate: "2023-06-01 15:00:00"
draft: true
---

In this post, we will cover getting set up with Python 3 on an Ubuntu machine and making sure we are good to get started.

## Reminder

This is a documentation of my attempt to learn Python with FastAPI to build a simple API. I prefer to learn new tech by doing. This is akin to journaling and is not intended to be used in a production environment. It is a learning experiment. For more information see:

- [Project Overview and Reasoning]({{< ref "/post/new_tech/project_overview.md" >}} "Project Overview and Reasoning")
- [Previous Post]({{< ref "/post/new_tech/python_fastapi/00_overview.md" >}} "Previous Post")

## Getting Started

So, I am going to leave installing Python 3 up to you. It will depend on your operating system and configuration. I am doing this on my Ubuntu machine running Ubuntu 22.04, the most up to date LTS at the time of this post. I am currently on Python 3.10.6:

```bash
$ python3 --version
Python 3.10.6
```

You will want to follow the [instructions](https://www.python.org/downloads/) for your operating system, but generally most recent versions of Python should work.

I also use Visual Studio Code as my primary editor. I can drop into vim just fine when on a CLI, but when I have a GUI, I do prefer the editor. I know, I'm a filthy casual.

As with most of the time, I already have MySQL up and running on my machine locally, although you could certainly use Docker instead. With that, let's create a new git repo to store the work. I will create a new public repo at GitHub, and save you the details on a basic README.

![Starting Repo](/images/posts/new_tech/python_fastapi/01_01_github.png)

Then clone it locally:

```bash
$ git clone git@github.com:kevineaton/learning_python_fastapi.git
Cloning into 'learning_python_fastapi'...
remote: Enumerating objects: 4, done.
remote: Counting objects: 100% (4/4), done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 4 (delta 0), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (4/4), done.
$ cd learning_python_fastapi/
```

Now is the first unique step, really. We really want to use a virtual environment. There are tons of reasons, including security, ease of use, and avoiding conflicts. There really isn't a reason not to. A good background can be found by [Martin Breuss](https://realpython.com/python-virtual-environments-a-primer/) and [Athreya Anand](https://towardsdatascience.com/why-you-should-use-a-virtual-environment-for-every-python-project-c17dab3b0fd0). Luckily, doing so is easy from within the checked out repository:

```bash
$ python3 -m venv learning_python_fastapi .
```

If you look in the folder now, you will see a lot of different folders that were created. To actually USE the virtual environment, you need to activate it in your shell. You will know this worked because, at least on Ubuntu, the environment will now be listed in the shell:

```bash
$ ls
bin  include  lib  lib64  LICENSE  pyvenv.cfg
$ . ./bin/activate
(learning_python_fastapi) $ 
```

Note: from here out, unless I specify otherwise, it's safe to assume that the `venv` name will be prepended to the `$` in shell examples.

Then open the folder in VS Code, either through the app or through the terminal:

```bash
$ code .
```

## Logging

One of the first things I want to do when building an application is to provide some way of getting information out of the app. However, I also don't want to spend too much time worrying about selecting just the right tool. So, I usually want to just get something quick and simple out that consolidates logging to a central location, so if I want to change it later, it should be easier.

First, from the terminal, let's set up a basic filesystem. Not quite sure we will keep this, but for now, we will create a `src/` folder that will serve as the root of the app. We will also create a `src/utils/` folder to hold our general purpose utils, such as a logger. We will then create two new files. We may move these later, but for now, we will leave them where they are.

```bash
$ mkdir -p src/utils # create both the src and src/utils folders
$ touch ./src/__init__.py ./src/utils/__init__.py # turn both into a module
$ touch src/app.py src/utils/logger.py # create the source
$ ls ./src
app.py  __init__.py  utils
$ ls ./src/utils/
__init__.py  logger.py
```

With those files created, let's build a logger. It will just be a single function that takes in a level string, a message, and optional arguments for more data. We will start with the basic built-in logging. I like to prefer the standard library as the default, and expand out if needed.

So, first we will import the logging module. This module exports a few helpers for the levels, but we should abstract that and then switch on it. A basic first implementation could look something like this:

```python
# src/utils/logger.py
import logging
from datetime import datetime

def log(level, message, **options):
    # normalize the level
    level = level.upper()
    if level not in ["DEBUG", "INFO", "WARNING", "ERROR", "CRITICAL"]:
        level = "INFO"
    
    # for options, we care about module and extra for now
    module = "?"
    extra = {}

    if "module" in options:
        module = options["module"]
    if "extra" in options:
        extra = options["extra"]

    # set the time format
    # we will come back and improve this with our basic config
    time_format = "%Y-%m-%d %H:%M:%SZ"
    now = datetime.now()
    
    line = f"{now.strftime(time_format)}:{level}:Module {module}: {message}"

    # switch on the level
    match level:
        case "DEBUG":
            logging.debug(line)
        case "INFO":
            logging.info(line)
        case "WARNING":
            logging.warning(line)
        case "ERROR":
            logging.error(line)
        case "CRITICAL":
            logging.critical(line)
```

In the future, we may want to turn this into a class and implement the various helpers that would make documentation a bit easier, but for now this will do as a function. You will notice there's... a lot to be desired. The message is mediocre and we are ignoring the extra data.

Let's just set up a simple `app.py` to call into the logger, showing the levels.

```python
# src/app.py
import utils.logger

extra = { "test" : "data" }
utils.logger.log("DEBUG", "this is a sample debug", module="Main", extra=extra)
utils.logger.log("INFO", "this is a sample info", module="Main", extra=extra)
utils.logger.log("WARNING", "this is a sample warning", module="Main", extra=extra)
utils.logger.log("ERROR", "this is a sample error", module="Main", extra=extra)
utils.logger.log("CRITICAL", "this is a sample critical", module="Main", extra=extra)
```

Running it is simple:

```bash
$ clear && python3 ./src/app.py 
WARNING:root:2023-06-01 23:21:00Z:WARNING:Module Main: this is a sample warning
ERROR:root:2023-06-01 23:21:00Z:ERROR:Module Main: this is a sample error
CRITICAL:root:2023-06-01 23:21:00Z:CRITICAL:Module Main: this is a sample critical
```

Eww. So what happened?

### Logging and Configuration

The way this was built has some subtle assumptions, that are not what we really want. I knew this when I wrote it, but felt it was important to demonstrate. Sometimes, Python will make assumptions for you. **That's OK.** You just need to know them. In this case, anything below `WARNING` was ignored. Also, simply calling in with the default message will cause the logging module to prepend the level and module, which causes some duplication. Since I personally prefer to have the timestamp as the very first thing in the log, let's go ahead and modify it using a basic configuration. While we are at it, I really don't like that going to STDOUT, so we will just create an ephemeral log file to dump data into. In the logger file, let's change it to:

```python
# src/utils/logger.py
import logging
from datetime import datetime

# set up a basic config
logging.basicConfig(
    level=logging.DEBUG, # we will show everything while testing, and eventually take this from the environment
    filename="log.log", # write to log.log in the running dir
    filemode="w", # overwrite each time
    format="%(message)s", # format the data according to the template; since we create the line ourselves, this is fine
)

def log(level, message, **options):
    # normalize the level
    level = level.upper()
    if level not in ["DEBUG", "INFO", "WARNING", "ERROR", "CRITICAL"]:
        level = "INFO"
    
    # for options, we care about module and extra for now
    module = "?"
    extra = {}

    if "module" in options:
        module = options["module"]
    if "extra" in options:
        extra = options["extra"]

    # set the time format
    # we will come back and improve this with our basic config
    time_format = "%Y-%m-%d %H:%M:%SZ"
    now = datetime.now()
    
    line = f"{now.strftime(time_format)}:{level}:Module {module}: {message}:: {extra}"

    # switch on the level
    match level:
        case "DEBUG":
            logging.debug(line)
        case "INFO":
            logging.info(line)
        case "WARNING":
            logging.warning(line)
        case "ERROR":
            logging.error(line)
        case "CRITICAL":
            logging.critical(line)
```

Note the addition of the extra in the `line`. We set up a pretty basic configuration on the logger which will get run when the module loads. Subsequent calls won't change this, so we just run it at the module level. We could do more, such as add the ability for stack traces, but for now, this seems good enough. Running the code so far yields:

```bash
$ python3 ./src/app.py 
$
```

Wait, where'd the output go? Since we sent it to the log file, nothing gets written out to STDOUT. So just open up the log file:

```bash
$ cat ./log.log 
2023-06-01 23:41:39Z:DEBUG:Module Main: this is a sample debug:: {'test': 'data'}
2023-06-01 23:41:39Z:INFO:Module Main: this is a sample info:: {'test': 'data'}
2023-06-01 23:41:39Z:WARNING:Module Main: this is a sample warning:: {'test': 'data'}
2023-06-01 23:41:39Z:ERROR:Module Main: this is a sample error:: {'test': 'data'}
2023-06-01 23:41:39Z:CRITICAL:Module Main: this is a sample critical:: {'test': 'data'}
```

Now, I typically prefer to follow the logs when testing. Something like:

```bash
$ tail -f log.log
```

But for now, I am fine just opening it as I need.

## Committing and Wrapping Up

Code that isn't committed was never written.

So, now we will commit. I made an error in that I never created a branch when I pulled, so I will rectify that now. I will create a branch called `initial`. I also don't know what I need in the repo and what I don't, so I will just add everything to `.gitignore` other than the `src/` dir and `pyvenv.cfg` for now. I can add the others back in later. I will add the files, commit them, push them, and then merge them. The final result is an initial commit that you can find [here](j). Note that I will just edit the README in place and won't document that here. It's not super relevant. 

```bash
$ git checkout -b initial
Switched to a new branch 'initial'
$ git add .
$ git commit -m "initial commit with logger"
[initial 712bf8d] initial commit with logger
 4 files changed, 62 insertions(+)
 create mode 100644 pyvenv.cfg
 create mode 100644 src/app.py
 create mode 100644 src/utils/logger.py
$ git push -u origin initial
Enumerating objects: 10, done.
Counting objects: 100% (10/10), done.
Delta compression using up to 8 threads
Compressing objects: 100% (7/7), done.
Writing objects: 100% (8/8), 1.42 KiB | 483.00 KiB/s, done.
Total 8 (delta 1), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
remote: 
remote: Create a pull request for 'initial' on GitHub by visiting:
remote:      https://github.com/kevineaton/learning_python_fastapi/pull/new/initial
remote: 
To github.com:kevineaton/learning_python_fastapi.git
 * [new branch]      initial -> initial
Branch 'initial' set up to track remote branch 'initial' from 'origin'.
```

You should end up with something like this in your editor:

![Editor](/images/posts/new_tech/python_fastapi/01_02_final_editor.png)

And something like [this in your repo](https://github.com/kevineaton/learning_python_fastapi/tree/6e78bedf5d71aefd16885b1ed2e68b1aa77f1d75).

Next, we will get started with our HTTP routing!
