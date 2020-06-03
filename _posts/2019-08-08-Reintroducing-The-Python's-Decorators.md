---
title: Reintroducing The Python's Decorators
author: Caleb Njiiri
date: 2019-08-08 11:33:00 +0800
categories: [Python]
tags: [python classes]
math: true
---

##### Reintroducing The Python's Decorators

## Definition

Python's decorators are a significant part of the language, and one of the features that made me like Python more. A decorator is a function or a class that extends the behavior of the another function or class without the need to explicitly modify it. Decorators act as wrappers to other existing services. This is perfect for many reasons, and keeping our code clean is one of them.

What makes a decorator act this way is that functions are treated as first-class objects in Python, which basically means that functions can be passed around as arguments just like any other value, can be defined inside of other functions, and can be returned from other functions.

## Examples

Let's take a quick example first to understand how decorators work.

> Using Python3 for the examples.

```python
def my_decorator(function_parameter):

    def function_wrapper():
        print("This will be called before the function execution.")

        function_parameter()

    return function_wrapper

@my_decorator
def hello_world():
    print("Hello, World!")

hello_world()
# This will be called before the function execution.
# Hello, World!
```

We start by defining a function that will act as the decorator; naming it `my_decorator`, a very original name by the way. The decorator function accepts a function parameter `function_parameter` that will be the function that the decorator will extend. The in the body of the decorator, we define a function wrapper that will wrap the functionality of the decorator, we name it `function_wrapper`, again, very original. The feature basically is just printing a text before the function execution.

Finally, we define a function, `hello_world`, that the decorator will extend. Now most likely, the function will be already defined in the real case, and the decorator will come to prolong the functionality. Let's take a real world example that I faced before.

I had a function that sends a request to an external API, and that API was unstable for some reason and returning `503 Service Unavailable` sometimes. I had to send the request two or three times repeatedly to get the results back. Now I could obviously modify that function to send the request if I get that HTTP status back, but we already know that is bad coding, especially if we had multiple functions to change. And so the decorator helped in this case where I created a general one that will retry depending on a defined error and another one that `extends` the previous to send the request when getting the `503 Service Unavailable`.

> I am creating the decorator as a class this time to show the power of it.

```python
# The standard retry decorator.
class Retry:

    MAX_TRIES = 5

    def is_valid(self, response):
    # By default, the 200 code status is the only accepted one,
    # thus only retry if a status code is not 200.
        return response.status_code == 200

    def __call__(self, func):
        def retried_func(*args, **kwargs):
            tries = 0
            while True:
                response = func(*args, **kwargs)
                if self.is_valid(response) or tries >= self.MAX_TRIES:
                    break
                tries += 1
            return response
    return retried_func

# Retry class that will recall the API
class RetryUnstableAPICall(Retry):
    def is_valid(self, response):
        return not response['code'] == 503

retry_unstable_api_request = RetryUnstableAPICall()

@retry_unstable_api_request
def send_request():
    #...
    pass
```

The `Retry` class is the one that will act as the retry base decorator, where `MAX_TRIES` is the number of the tries that the request should be sent until it fails. Then the `is_valid()` function is the one that will validate the response according to the HTTP response code. The function call operator, `__call__`, will keep executing the function as long as the `is_valid()` condition is not fulfilled.

Now the `RetryUnstableAPICall` is just extending the `Retry` class and redefining the `is_valid()` function with a new condition that fit my needs for retrying when the HTTP status is `503 Service Unavailable`.

