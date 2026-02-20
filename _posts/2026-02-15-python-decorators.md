---
layout: post
title: "What are Decorators in Python and how do they work?"
thumbnail: /images/decorators.jpg
---

If you are new to python, or have been using it for a while and wanted to dig deeper into learning Python's most elegant and powerful features, <a href="https://peps.python.org/pep-0318/">decorators</a> is a top candidate. In Python, ``decorators`` are a powerful way to modify or enhance the behaviour of a function without changing its original code. Due to this nature, decorators are known as wrappers that ``decorate`` functions to provide extra functionality.

### Why do we need decorators?
Think of a scenario when you were decorating something, say a Christmas tree. You add extra lights, garlands, ribbons etc. But underneath, the original Christmas tree is still there and the decorated result is still a Christmas tree. Similarly, you can give additional functionality to an existing functions and classes via decorators. Its a form of *metaprogramming* - code that manipulates code. Before understanding decorators, we need to better undestand about functions.

### Functions are first class objects.
It is important to understand that in Python, functions are known to be **first-class objects**. What this means is that, similar to other objects, you can pass functions as arguements and return them form other functions and even assign them to variables. 

- To see this lets define a simple function as below.
```Python
def greet(name):
    return f"Hello {name}!"
```

- Now the most common way of using the above function is just invoking it like below
```Python
result = greet("John") # result = Hello John!
```

- But since functions are first-class objects, we can use functions as arguements.
```Python
say_hello = greet # Assigning function to a variable
print(say_hello("John")) # Output: Hello, John!
```

- We can return *greet* function from another function.

```Python
def foo():
    return greet # Returning greet function from another function

greet_from_foo = foo()
print(greet_from_foo("John")) # Output: Hello, John!
```

- Or we can invoke the *greet* function from within another function

```Python
def execute_function(func, value):
    return greet(value)

print(execute_function(greet, "Bob"))  # Hello, Bob!
```

Understanding the above simple concepts make it possible to create decorators in Python.

### Writing a basic Decorator from scratch
Lets define a simple decorator that times how long a function takes to run
```Python
import time

def my_timer_decorator(func):
    """A decorator that times function execution"""
    def wrapper(*args, **kwargs):
        start_time = time.time()
        result = func(*args, **kwargs)  # Call the original function
        end_time = time.time()
        print(f"{func.__name__} took {end_time - start_time:.4f} seconds")
        return result
    return wrapper

# Lets define our function against which we need to do timing
def my_slow_function():
    time.sleep(1)
    return "Done!"
```

Now, there are two ways to use the above decorator. The first way is to treat *my_timer_decorator* as a function that take another function as an input as below
```Python
my_slow_function = my_timer_decorator(my_slow_function) # Notice my_slow_function overrides the original my_slow_function
my_slow_function()  # now everytime my_slow_function is invoked calculating the time ex: my_slow_function took 1.0001 seconds
```

If you have been following, you would see that after overriding `my_slow_function` as ``my_slow_function = my_timer_decorator(my_slow_function)`` it will call the function wrapped by the ``my_timer_decorator`` decorator. This still enables us to call `my_slow_function` from whithin the wrapper with an additional timing feature. Let's disect the `my_timer_decorator` decorator line by line to better understand what each line does.

```Python
# L1 def my_timer_decorator(func):                                                
# L2     """A decorator that times function execution"""                          
# L3     def wrapper(*args, **kwargs):                                            
# L4         start_time = time.time()                                             
# L5         result = func(*args, **kwargs)                                       
# L6         end_time = time.time()                                               
# L7         print(f"{func.__name__} took {end_time - start_time:.4f} seconds")   
# L8         return result                                                        
# L9     return wrapper                                                           
```

- L1 : `my_timer_decorator` is the function that accepts other functions as input to be decorated
- L2 : A doc string about what this decorator does
- L3 : `wrapper` is a child function taking care of the **positional arguements(args)** and **named arguements(kwargs)** of func. If you do not what is meant by positional and named arguements in python, here is a quick refresher.

#### Positional Arguments
These are values passed to a function in a specific order. Python assigns these values to the corresponding parameters based strictly on their sequence. If you change their order, it affects the fuction's logic. Example
```Python
def take_power(*args):
    if len(args) != 2:
        return TypeError("Invalid number of arguements")
    return args[0]**args[1]
    
print(take_power(2, 3)) # gives 2**3 = 8 
print(take_power(3, 2)) # swap the order gives 3**2 = 9
```

#### Named (Keyword) Arguments
These are passed by explicitly specifying the parameter name followed by an equals sign (=) and the value. The advantage is, you can change their order.
```Python
def my_fraction(**kwargs):
    if kwargs["denominator"] == 0:
        return "Error: Cannot divide by zero!"
    return kwargs["numerator"]/kwargs["denominator"]

print(my_fraction(numerator=2, denominator=10))   # prints 0.2
print(my_fraction(denominator=10, numerator=2))   # swap the position, still prints 0.2
```

- L4 : Store the time before calling the decorated func - you can has as many custom logic from L4 to L5
- L5 : Invoke the original function with its original arguements and store its result
- L6 : Store the time after calling the funtion - you can have as many custom logic from L6 to L8
- L7 : Print the total time taken for the decorated function
- L8 : Return the result of original function to the decorator
- L9 : Return the function wrapper to the original caller

### Introducing the @ syntax
As I have mentioned before, there are two ways to use a decorator and you have already seen the first way to use it directly as a function. Python provides a shorthand for applying decorators using ``@`` symbol. As an example, you can apply the same timing decorator as below and it looks pretty neat now!

```python
@my_timer_decorator
def my_slow_function():
    time.sleep(1)
    return "Done!"
```

Having decorated, everytime you call my_slow_function() what you get is not the original function, but the same function decorated with that additional functionality to measure the timing! How cool is that?

### Decorators with arguements
If you could remember why we need decorators in the first place, they can provide additional features to existing functions without having to modify them. In order to do that, the decorators sometimes need additional arguements. In such scenarios, our decorator needs to have another level of nesting. As an example, lets imaging we need to have a decorator to time a function, but this time it needs to repeat the function a configurable number of times.. Here is a sample code.
```Python
def my_repeat_timer(times):
    """This is a decorator that repeat a function a number of times and measure its execution time"""
    def func_consuming_decorated_func(func):
        def func_consuming_decorated_func_arguements(*args, **kwargs):
            start_time = time.time()
            results = []
            for _ in range(times):
                results.extend(func(*args, **kwargs))  # Call the original function
            end_time = time.time()
            print(f"{func.__name__} took {end_time - start_time:.4f} seconds to run {times} times")
            return results
        return func_consuming_decorated_func_arguements
    return func_consuming_decorated_func
```

If you could have a look at each nested function in the above decorator, each has a specific purpose. Each function returns its child function at its level. Lets take a closer look at each function. 
- my_repeat_timer(times) : this is the name of the decorator that taked *times* as a parameter. It returns *func_consuming_decorated_func* function
- func_consuming_decorated_func(func) : this is first nested function that takes the decorated function as a parameter. It returns *func_consuming_decorated_func_arguements* function
- func_consuming_decorated_func_arguements(*args, **kwargs) : this is the second nested function that takes the parameters of the decorated function as positional and named parameters. In addition it executes the input function the number of times as specified by the decorator. This returns the aggrigated results from the original function.

Let's see this new decorator in action. As you would already know, its use the @ syntactic sugar as its much cleaner and easier to use.
```Python
@my_repeat_timer(times=2)
def my_slow_function():
    """This a my_slow_function"""
    time.sleep(1)
    return "Done!"

# Having decorated, this will run my_slow_function 2 times each time its is called.
res = my_slow_function() # my_slow_function took 2.0013 seconds to run 2 times
print(res)               # ["Done!", "Done!"]
```

### Preserving function metadata
When creating decorators of our own, we need to make sure the metadata of the decorated function such as **__name__**, **__doc__**, **__module__** are preserved by the decorator. lets see what we get when checked now, 
```Python
print(my_slow_function.__name__) # func_consuming_decorated_func_arguements
print(my_slow_function.__doc__)  # None
```

This is because, if you could remember the first way of using decorators, the new decorator is equivalent to 
```Python
my_slow_function = my_repeat_timer(times=2)(my_slow_function)
```

what is retuned by ``my_repeat_timer(times=2)(my_slow_function)`` is ``func_consuming_decorated_func_arguements`` function(the second level of nesting) and that is why if you access ``my_slow_function.__name__`` it appears as ``func_consuming_decorated_func_arguements``. <br>

To avoid this, there is a special built-in function we can use ``from functools import wraps``. The modified decorator looks like below.
```Python
from functools import wraps

def my_repeat_timer(times):
    """This is a decorator that repeat a function a number of times and measure its execution time"""
    def func_consuming_decorated_func(func):
        @wraps(func) # This preserves the metadata of the func function
        def func_consuming_decorated_func_arguements(*args, **kwargs):
            start_time = time.time()
            results = []
            for _ in range(times):
                results.extend(func(*args, **kwargs))  # Call the original function
            end_time = time.time()
            print(f"{func.__name__} took {end_time - start_time:.4f} seconds to run {times} times")
            return results
        return func_consuming_decorated_func_arguements
    return func_consuming_decorated_func
```

Now if you apply this updated decorator you can see that the metadata of the interested function are preserved!
```Python
@my_repeat_timer(times=2)
def my_slow_function():
    """This a my_slow_function"""
    time.sleep(1)
    return "Done!"

print(my_slow_function.__name__) # my_slow_function
print(my_slow_function.__doc__)  # This a my_slow_function
```

### Class based decorators
Decorators can also be classes. Lets create an example decorator to see this in action. Please note this decorator does not accept arguements

```Python
from functools import wraps

class CountCalls:
    def __init__(self, func):
        wraps(func)(self) # this preserves the metadata of the decorated function
        self.func = func
        self.count = 0
    
    def __call__(self, *args, **kwargs):
        self.count += 1
        print(f"Call {self.count} to {self.func.__name__}")
        return self.func(*args, **kwargs)

@CountCalls
def say_hello():
    """This is a function to say hello"""
    print("Hello!")

say_hello() # Call 1 to say_hello \n Hello!
say_hello() # Call 2 to say_hello \n Hello!

print(say_hello.__doc__)     # This is a function to say hello
print(say_hello.__name__)    # say_hello
print(say_hello.__module__)  #__main__


# Let's decorate another function with the same decorator
@CountCalls
def say_bye():
    """This is a function to say bye"""
    print("Bye!")

say_bye() # Call 1 to say_bye \n Bye!
say_bye() # Call 2 to say_bye \n Bye!

print(say_bye.__doc__)     # This is a function to say by
print(say_bye.__name__)    # say_bye
print(say_bye.__module__)  #__main__
```

As you can see the name of the class *CountCalls* becomes the name of the decorator and it can individually count the number of times a decorated function is invoked preserving their metadata.

#### What is the ``__call__`` function in above?
``__call__`` is a special **dunder** (stands for double undescore) method that allows an instance of a class to be a function. Basically if you have a class named *Foo*, the general use is make an object of Foo like *f = Foo()* and then call functions on f such as *f.do_something()*. But if the Foo class has ``__call__`` function defined, you can simply use its objects *f* itself as an object. Lets see another example.

```Python
class Greeter:
    def __call__(self, name):
        print(f"Hello, {name}!")

g = Greeter()

g("Alice")   # Hello, Alice
# The object g itself has act like an method!
```

When you call *g("Alice")*, what happens behind the scene is *g.__call__("Alice")* gets invoked. <br>

Similarly when you have a class based decorator, everytime you **use** that decorator, the ``__init__`` of the class is invoked. And everytime the decorated function is called, the ``__call__`` of the class gets invoked. I invite you to test this on your own by placing some print statements to see when each function is called.

### Real worlds applications of decorators
In Python, decorators are everywhere!!! Python has some common built-in decorators such as
- @staticmethod – defines a static method in a class
- @classmethod – defines a class method
- @property – makes a method behave like an attribute
I will create a separate blog post on built-on decorators.

And if you have used frameforks in python, like [Flask](https://flask.palletsprojects.com/en/stable/), [Django](https://docs.djangoproject.com/en/6.0/intro/tutorial01/), decorators are used everywhere
- @app.route('/home')
- @login_required
- @permission_required

In Test frameworks like [pytest](https://docs.pytest.org/en/stable/) or [unittest](https://docs.python.org/3/library/unittest.html), you cannot code without using decorators ex:
- @unittest.mock.patch
- @unittest.skip
- @pytest.mark.parametrize
- @pytest.mark.xfail

### Thank you! 
This marks the end of this blog post, and I hope you have learned something new out from this blog post and be more confident in using and creating your own decorators in your Python projects!! I wish you all the very best and Happy coding! 