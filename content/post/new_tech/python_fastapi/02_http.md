---
title: Learning Python and FastAPI - 02 - HTTP and Routing
description: Learning Python and FastAPI - 02 - HTTP and Routing
date: "2023-06-02 00:00:00"
publishDate: "2023-06-02 00:00:00"
draft: true
---

## Reminder

This is a documentation of my attempt to learn Python with FastAPI to build a simple API. I prefer to learn new tech by doing. This is akin to journaling and is not intended to be used in a production environment. It is a learning experiment. For more information see:

- [Project Overview and Reasoning]({{< ref "/post/new_tech/project_overview.md" >}} "Project Overview and Reasoning")
- [Previous Post]({{< ref "/post/new_tech/python_fastapi/01_setup.md" >}} "Previous Post")

## Meanwhile

Since completing the initial set up, I spent a lot of time in the [official FastAPI docs](https://fastapi.tiangolo.com/lo/tutorial/), which are fantastic. It became clear to me that this is a much larger framework than I originally thought. I think I am going to hit some frustration with the database portion since I really don't like ORMs, but we will cross that at that point. Perhaps I will grow to love it, but honestly I typically prefer to handwrite my SQL. Call me old-fashioned...

Oh, I also just added a [README at the repo](https://github.com/kevineaton/learning_python_fastapi). Other than that, not much actual code has been written, so it's time to dive in!

## Getting Started

I haven't read all of the docs yet, so I am sure I will make some mistakes I will need to refactor later. For now, I will just create some basic stubs of the routes in the `src/app.py` file. None of these will do anything other than report back that they were hittable. I will start by removing the logger outputs used last time to verify it connected, but I will leave the logger import. In fact, I don't like the way that feels for now, so before even diving into the API, let's change it into a proper module.

To do this, simply add a `__init__.py` file to the `src/utils/` directory, then change the import:

```bash
$ touch src/utils/__init__.py
```

```python
# src/app.py
from .utils import logger
```

Better. Will still likely change it later, since the options are obfuscated from the caller, but let's not worry about that for now. Back to the API. First, install the fastapi module:

```bash
$ pip install "fastapi[all]"
```

This is going to install a bunch of packages, including `uvicorn`. We will use that to start our local server with hot-reloading. But first, we need to create an app to hook into. In the `src/app.py` file, import `fastapi` and create a new app. We should probably create a root path, just to make sure the server starts up correctly as well. Therefore, add a simple function with the FastAPI decorator that just returns some simple JSON. All said and done, your file should look like:

```python
# src/app.py
from .utils import logger
from fastapi import FastAPI

app = FastAPI() # create a new API

logger.log("DEBUG", "Starting the server")

@app.get("/")
async def root():
    return {"hello": "world"}
```

Keep in mind, the logger we created is set to log to a file. So you can either tail that if you want, or comment out the file entries in the logger config.

Now, start the server:

```bash
$ uvicorn src.app:app --reload
INFO:     Will watch for changes in these directories: ['/home/kevin/dev/python/learning_python_fastapi']
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [3277973] using WatchFiles
INFO:     Started server process [3277975]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
```

The server is now running, and there should be a single GET route exposed at `/`. To test this, you can either visit it in the web browser or simply use cURL:

```bash
$ curl http://localhost:8000
{"hello":"world"}
```

Going to any other path results in an error:

```bash
$ curl http://localhost:8000/foo
{"detail":"Not Found"}
```

It's also important to notice that you have automatic OAS3 docs available at `/docs` in the browser. However, they're pretty lacking right now, especially with only one route, no auth, and no type definitions. We will get there though.

## Building More Routes

I know I am eventually going to go in and use some type checking. Also, I typically build out a single "return good data" and "send an error" functions that allows me to consolidate, but let's first worry about just getting the stubs up. I will change the `/` path to be a health check, and then will add the other routes that just return proof-of-concept data. As FastAPI uses decorators to specify those routes, it's pretty straight forward. After the stubs, the routes should look something like this, with comments not included right now for brevity. Also, it's obvious I need to change the path parameter for the final route. It's also obvious that these will definitely need to be broken up into sub-modules at some point in the future.

```python
@app.get("/")
async def health_route():
    return {"status": "up"}

@app.post("/users")
async def register_route():
    return {"route": "register_route"}

@app.post("/users/verify")
async def user_verify_route():
    return {"route": "user_verify_route"}

@app.post("/login")
async def login_route():
    return {"route": "login_route"}

@app.get("/me")
async def get_profile_route():
    return {"route": "get_profile_route"}

@app.patch("/me")
async def update_profile_route():
    return {"route": "update_profile_route"}

@app.post("/me/refresh")
async def refresh_token_route():
    return {"route": "refresh_token_route"}

@app.get("/users/{id}")
async def get_user_profile_route():
    return {"route": "get_user_profile_route"}
```

Feel free to make the calls all you want to verify, but they are pretty useless. So first step is to get the `id` parameter in the last route there. To use a path parameter, we put the value in `{}` like we did, and then it's super easy to just add that exact name to the function parameters, like so:

```python
@app.get("/users/{id}")
async def get_user_profile_route(id):
    return {
        "route": "get_user_profile_route",
        "user_id": id,
    }
```

Now, checking the return, we see it come back:

```bash
$ curl http://localhost:8000/users/42
{"route":"get_user_profile_route","user_id":"42"}
```

Sweet, but our docs are a bit lacking. Also, I want user_id to be a number. Let's fix that with some typings.

```python
@app.get("/users/{id}")
async def get_user_profile_route(id: int) -> dict:
    return {
        "route": "get_user_profile_route",
        "user_id": id,
    }
```

So, I don't like the `-> dict` but since I already read through the docs, I know I will swap out the actual typing later. So it works for now. Running cURL, we see we get a number now:

```python
$ curl http://localhost:8000/users/42
{"route":"get_user_profile_route","user_id":42}
```

And the docs are a bit more helpful:

![Docs](/images/posts/new_tech/python_fastapi/02_01_docs.png)

## Request Bodies, Models, and Types

Now we're cooking. Let's create two models that we can use for sending and getting users. We're definitely not ready to write things to the DB, and our models may change, but it would be nice to get some basic placeholders in place. We still lack documentation and comments for these routes, but hopefully some data modeling will help us get there. I also know that I will likely want to have `*_request` and `*_response` objects to ensure things like passwords don't make it back. Still, let's start simple.

We will use classes in the same file, knowing we will move them eventually. We will use `Pydantic` since it's included and looks really powerful. Let's start with creating a user and a token class right above the routes. Our User will look something like:

- User
  - id
  - first_name
  - last_name
  - email
  - password -- ENCRYPTED!!!
  - status -- one of `pending`, `active`, `inactive`, `locked`
  - created_on
  - last_logged_in_on

While at it, let's set the tokens as something like:

- Tokens
  - id
  - user_id
  - token_type -- used for verification and authorization, so one of `register`, `refresh`
  - token
  - expires_on

We inherit from BaseModel, so we will want to import it and then build the classes. While at it, a couple of our routes need different fields, such as during the verification process, so let's add an input for that. I came up with an initial stab looking like:

```python
## Models

class User(BaseModel):
    id: int
    first_name: str
    last_name: str
    email: str
    password: str
    status: str
    created_on: str
    last_logged_in_on: str

class Token(BaseModel):
    id: int
    user_id: int
    token_type: str
    token: str
    expires_on: str

class VerifyRegisterInput(BaseModel):
    email: str
    verification_code: str

class RefreshTokenInput(BaseModel):
    access_token: str
    refresh_token: str
```

I immediately see a problem though. If I just use this, then the password will be sent with each response. I also know that, for logging in, I will want to include an `access_token` but I don't want to return that when getting a profile, for example. I am also a bit nervous about the `PATCH` and if all of the fields will be required by the API. For now, let's just leave it to see how the data gets serialized back and forth. I added the type annotations to the functions and added the parameters as appropriate. We also need to actually return the correct typing or you will get an exception raised for not returning the correct data. So, for example, if you have this route:

```python
@app.get("/users/{id}")
async def get_user_profile_route(id: int) -> User:
    return {
        "route": "get_user_profile_route",
        "user_id": id,
    }
```

You will get an error if it is called:

```python
$ curl http://localhost:8000/users/42
Internal Server Error
```

With a stack trace complaining (truncated for brevity):

```bash
pydantic.error_wrappers.ValidationError: 8 validation errors for User
response -> id
  field required (type=value_error.missing)
```

This is nice in that it enforces the typings, but will be a headache without the request/response dynamic. For now, I am just going to create a test user and always return that. Under the models, I added:

```python
# create a quick test user for validation
user: User = User(
    id=1,
    first_name="Kevin",
    last_name="Eaton",
    email="kevinhowardeaton@gmail.com",
    password="",
    status="active",
    created_on="2023-06-02T00:00:00Z",
    last_logged_in_on="2023-06-02T00:00:00Z"
)
```

I then added it as a return where appropriate, for example:

```python
@app.get("/users/{id}")
async def get_user_profile_route(id: int) -> User:
    return user
```

So now my calls work again:

```bash
$$ curl http://localhost:8000/users/42
{"id":1,"first_name":"Kevin","last_name":"Eaton","email":"kevinhowardeaton@gmail.com","password":"","status":"active","created_on":"2023-06-02T00:00:00Z","last_logged_in_on":"2023-06-02T00:00:00Z"}
```

The API docs are now also more helpful:

![Docs](/images/posts/new_tech/python_fastapi/02_02_docs.png)

## Wrapping Up

Now we are in a pretty good spot. We're not doing anything with the data, and I know for a fact I am going to need to change how some of the data is handled or returned for things like passwords and tokens, but things generally work. I am also lacking comments. So I will add some in, commit the changes, and start figuring out the next step, which is the data. I am intentionally waiting to solve the password/token situation until I have a DB in place, just in case that introduces more validation and class changes.

When all is said and done, you should end up with a repo looking something like [this](https://github.com/kevineaton/learning_python_fastapi/tree/51f99901947a540738687eba5486bd24ac2b9b31).
