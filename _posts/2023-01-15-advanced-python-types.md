---
layout: single
title: "Some Advanced Typing Concepts in Python"
date: 2022-11-23
mathjax: true
---

To start off, let me admit that yes, Python is a dynamically ([gradually?](https://peps.python.org/pep-0483/)) typed language. 
However, with modern Python (3.6+, and really with 3.10+) and static tooling (i.e. [mypy](https://mypy.readthedocs.io/en/stable/), [pyright/pylance](https://github.com/microsoft/pyright)) it is a pretty robust *statically* typed language. Most third party libraries either are typed or have types stubs distributed separately. From a developer perspective,
this version of the language making use of full strict static typing is essentially a different language the Python of yesteryear. 

I won't fully make the argument here but treating Python like a statically typed language
greatly increases readability and expressiveness of the code, greatly reduces bugs and even the amount of tests you need to write, and makes it much easier to move on to a "real" statically typed language like Rust (do it, its great!) or Go (do it, its pretty good!) or even oldies like C/C++/Java. 

While there is good material on more advanced typing concepts in Python (the [mypy docs](https://mypy.readthedocs.io/en/stable/) are actually quite good), most tutorials you find online are "hello world" level examples just introducing types to people who have never used Python or are from the dark days of Python pre-3.6. This post assumes that you have a decent grasp of Python and already know the basics of type annotations, or that you are coming from another statically typed language and just want to see if Python is up to snuff.
Ok, rant over, lets get into some semi-advanced typing goodness.

All of the examples in this post are available [here](https://github.com/jellis18/advanced-python-types) if you want to play around with them. I would suggest using vscode and the "strict" Pylance type checking mode.

# Type annotations and the interface segregation principle

Remember the interface segregation principle from [Uncle Bob's SOLID principles](https://en.wikipedia.org/wiki/Interface_segregation_principle). Well if you don't, it basically means that users of your code should not be forced to rely on interfaces/types/classes that they don't need. We will see how Python type annotations can help with this. 

## Some general good practices

First lets look at how we can make our code more robust by defining loose types for input parameters and strict types for outputs. Lets say we are writing a function to filter
out a certain word or phrase from some strings. You could write your function like this
```python
from typing import List

def filter_things(things: List[str], filter_string: str) -> List[str]:
    return [t for t in things if filter_string not in t]
```
This is fine and would work like this
```python
things = ["badger", "snake", "mushroom"]
filtered_things = filter_things(things, "mushroom")
# returns ["badger", "snake"]
```
But do we really need the inputs `things` to be a list, all we are doing is iterating over it. Lots of python types do that. We could guess and do something like
```python
from typing import Any, Dict, List, Set, Tuple

def filter_things(things: List[str] | Tuple[str,...] | Set[str] | Dict[str, Any], filter_string: str) -> List[str]:
    return [t for t in things if filter_string not in t]
```
but that would be terrible and you have not just confused your user even more. Instead
we can use the type that specifies the bare minimum constraint for what we are doing. Since we are just *iterating* over `things` we can do
```python
from typing import Iterable, List

def filter_things(things: Iterable[str], filter_string: str) -> List[str]:
    return [t for t in things if filter_string not in t]
```
Using the [`Iterable` type](https://docs.python.org/3/library/collections.abc.html#collections.abc.Iterable) just states that `things` is a collection of strings that can be iterated over. Now our function can work with lots of different input types like tuples, sets, dicts, even generators since all of those types are iterable. Note that we do return a concrete type of `List[str]` here. It is always a good idea to hav the return type be as narrow as possible so the behavior is deterministic no matter what kind of input you give the function.

To hammer this home, lets look at one more example. 
```python
from typing import List

def get_fifth_element(things: List[str]) -> str:
    if len(things) < 5:
        raise ValueError(f"Length of input must be >= 5, found len={len(things)}")
    return things[4]
```
Again, we don't really need a list here but we aren't iterating. Instead we are checking the length and grabbing the value at a given index. So this means that we must be able to get the size of the input collection and we must be able to get the item in the collection at a given index. If we look at the [docs](https://docs.python.org/3/library/collections.abc.html) we can see that the correct type to use here is `Sequence` since it implements the `__len__` and `__getitem__` methods which allow for us to call `len` on it and to index into it. So we have
```python
from typing import List 

def get_fifth_element(things: List[str]) -> str:
    if len(things) < 5:
        raise ValueError(f"Length of input must be >= 5, found len={len(things)}")
    return things[4]

# good with a tuple
elements = ("earth", "wind", "water", "fire", "multipass")
fifth_element = get_fifth_element(elements)

# or a list
elements = ["earth", "wind", "water", "fire", "multipass"]
fifth_element = get_fifth_element(elements)
```
As we will see later, even this type is a bit too narrow still, but without defining our own type this will usually be fine.

## Protocols

I had a [previous post](../2022-01-11-abc-vs-protocol) about Protocols but here we will
go over them in the context of the interface segregation principle and just give a bit more detail as to why they are useful.

So lets start with the age old example of defining an `Animal`
```python
import abc

class Animal(abc.ABC):
    @abc.abstractmethod
    def make_sound(self) -> None:
        pass

    @abc.abstractmethod
    def act(self) -> None:
        pass

    @property
    @abc.abstractmethod
    def num_legs(self) -> int:
        pass
```
Note, I used an ABC here. I could have used a protocol for this but I'm trying to fake a 3rd party library that may already have a lot of different implementation of a class using standard class hierarchy. Now lets implement some animals
```python
class Dog(Animal):
    def make_sound(self) -> None:
        print("woof")

    def act(self) -> None:
        print("dog is walking")

    @property
    def num_legs(self) -> int:
        return 4


class Fish(Animal):
    def make_sound(self) -> None:
        print("blub")

    def act(self) -> None:
        print("fish is swimming")

    @property
    def num_legs(self) -> int:
        return 0
```
and lets make a function so we can hear some animal sounds
```python
from typing import Iterable

def get_animal_sounds(animals: Iterable[Animal]) -> None:
    for animal in animals:
        animal.make_sound()

animals = [Dog(), Fish()]
get_animal_sounds(animals)
# woof
# blub
```
So this function takes in an `Iterable` (see we learned from the first part!) of `Animal`s. If we run this function with these inputs then we will get `woof` and `blub`. Cool!, now we want to know [what a fox says](https://www.youtube.com/watch?v=jofNR_WkoCE) so we implement a fox
```python
class Fox:
    def make_sound(self) -> None:
        print("redacted")
```
Sweet, lets plug this in
```python
animals = [Dog(), Fish(), Fox()]
get_animal_sounds(animals)
```
Uh oh we get an error from our type checker like this
```python
Argument of type "list[Dog | Fish | Fox]" cannot be assigned to parameter "animals" of type "Iterable[Animal]" in function "get_animal_sounds"
  "list[Dog | Fish | Fox]" is incompatible with "Iterable[Animal]"
    TypeVar "_T_co@Iterable" is covariant
      Type "Dog | Fish | Fox" cannot be assigned to type "Animal"
        "Fox" is incompatible with "Animal" PylancereportGeneralTypeIssues
```
Ah, yes because we just implemented `make_sound` for `Fox` and not the whole animal interface. Well thats dumb, I just want to know what the fox says, why do I need to implement the whole "interface". Protocols to the rescue
```python
from typing import Iterable, Protocol

class CanMakeSound(Protocol):
    def make_sound(self) -> None:
        ...

def get_animal_sounds(animals: Iterable[CanMakeSound]) -> None:
    for animal in animals:
        animal.make_sound()
```
Now, we can find out what the fox says since all three animals now implement the `CanMakeSound` Protocol. 

So of course this is a silly example but there are many real life situation where our functions, classes, etc only need to know about a few methods of the object and don't need to know about all of them. If we use Protocols to only specify what we need this means that a user can implement their own objects to just satisfy this protocol.

If you really are digging this, here is how we could make our `get_fifth_element` function even more flexible by using a custom protocol that only requires the bare minimum.

```python
from typing import Any, Protocol, TypeVar

T = TypeVar("T", covariant=True)

class MySequence(Protocol[T]):
    def __getitem__(self, __idx: Any) -> T:
        ...

    def __len__(self) -> int:
        ...

def get_fifth_element(things: MySequence[str]) -> str:
    if len(things) < 5:
        raise ValueError(f"Length of input must be >= 5, found len={len(things)}")
    return things[4]
```

# Generics with Protocol Constraints
In the examples above we used Protocols to make our functions very general (you could even say generic!); however, we only used them in the arguments and not in the return type. In general, it doesn't really make sense to return a Protocol. Sometimes you need to use generics with custom constraints. We can illustrate this by implementing the simple `max` function that returns the maximum of two inputs. First lets just do it for `int`s
```python
def max(a: int, b: int) -> int:
    if a < b:
        return b
    return a
```
Ok, simple but the python builtin max function works on lots of other types, like `float`, `str`, and even lists and sequences. If we want a generic max function, look at the above function and figure out what the minimum requirement is. If we look, the only requirement is that `a` and `b` can be compared using the "less than" operator. In Python, that means that they both must implement the `__lt__` magic method. So lets make a bounded generic and re-write our function
```python
from typing import Protocol, TypeVar
from typing_extensions import Self

class HasLessThan(Protocol):
    def __lt__(self, __other: Self) -> bool:
        ...

T = TypeVar("T", bound=HasLessThan)

def max(a: T, b: T) -> T:
    if a < b:
        return b
    return a
```
Now if we use this function we will get the right return types and it will allow for many different comparisons
```python
m = max(3, 4) # m is of type: int
m = max("hello", "world") # m is of type: string
m = max([4, 5, 6], [1, 2]) # m is of type: List[int]
```
As always, we could even define our own type. Lets define a line class where we want `max` to return the longer line object
```python
from dataclasses import dataclass
from typing_extensions import Self

@dataclass
class Line:
    x_min: float
    x_max: float

    def __lt__(self, other: Self) -> bool:
        """Compare based on line length"""
        return (self.x_max - self.x_min) < (other.x_max - other.x_min)

m = max(Line(0, 3), Line(0, 5)) # returns Line(0, 5)
```

This is a very powerful method for defining generic functions/classes that only specify the bare minimum type constraints. If you are coming from Rust this is similar to [generics with trait bounds](https://doc.rust-lang.org/beta/rust-by-example/generics/bounds.html).

# Parameter Specification Variables

As Python is a *gradually* typed language, it cannot or could not always have static
typing in all situations.  Until [Parameter Specification Variables](https://peps.python.org/pep-0612/) or just "ParamSpec" we could not get good static typing on functions that are decorated. Once a function was decorated we lost the type information about the input and output parameters of that function. Lets see how we can fix that now with `ParamSpec`
```python
import time
from typing import Callable, ParamSpec, TypeVar

T = TypeVar("T")
P = ParamSpec("P")

def timer(func: Callable[P, T]) -> Callable[P, T]:
    def inner(*args: P.args, **kwargs: P.kwargs) -> T:
        tstart = time.perf_counter()
        out = func(*args, **kwargs)
        print(f"function: {func.__name__} ran in {(time.perf_counter()-tstart)} seconds")
        return out

    return inner

@timer
def add(a: float, b: float) -> float:
    """
    Add two numbers
    """
    return a + b

out = add(3.4, 4.5) 
# function: add ran in 6.902000677655451e-06 seconds
```
Here we have created a `timer` decorator that simply prints the time it took to call the decorated function. When we add the decorator to the simple `add` function it will print out the runtime. This is a simple example but lets look at what is going on. Notice that the outer function oft the decorator takes in the decorated function `func` and we use
a type annotation of `Callable[P, T]`. `T` is just a generic variable stating that the function could return any type. The `P` variable is the `ParamSpec`. This is what allows
to keep that type information in the decorated function. In the `inner` function we use `*args` and `**kwargs` as you always would for unknown inputs but now we can annotate them with `P.args` and `P.kwargs`, respectively. And this is the magic that respect the type
information of the decorated function. Your code editor (I use vscode and it is amazing for this and really all things...) should show you the function definition when you hover over it and the type checker should respect the input and output types. I used this feature quite
heavily in a previous post about [implementing a fully typed lru-cache](../2021-11-25-lru-cache).

# Overloads
Overloads let you specify different combinations of inputs and/or outputs for a single function. This can lead to some really bloated functions that do way too much but, like most things, everything in moderation.

Broadly speaking there are two cases where you would want to consider using overloads
* Optional Second order decorators (i.e. decorators that may or may not take arguments)
* Functions where there are several different combinations of input parameters which may or may not lead to different output types. Note, overloads should only be used here where the job can't be done with generics.

First lets look at the second-order decorator case by modifying our example above. What if we want the ability to format our message that prints the runtime, or what if we want to use a logger instead of just printing, but we still want to have the default behavior without passing in any arguments. We can modify it as follows
```python
import time
from typing import Callable, Optional, ParamSpec, TypeVar, overload

T = TypeVar("T")
P = ParamSpec("P")

def _default_display_fn(function_name: str, call_time: float) -> None:
    print(f"function: {function_name} ran in {call_time} seconds")

@overload
def timer(__func: Callable[P, T]) -> Callable[P, T]:
    ...

@overload
def timer(*, display_fn: Callable[[str, float], None]) -> Callable[[Callable[P, T]], Callable[P, T]]:
    ...

def timer(
    __func: Optional[Callable[P, T]] = None,
    *,
    display_fn: Optional[Callable[[str, float], None]] = None,
) -> Callable[[Callable[P, T]], Callable[P, T]] | Callable[P, T]:

    display_fn = display_fn or _default_display_fn

    def decorator(func: Callable[P, T]) -> Callable[P, T]:
        def inner(*args: P.args, **kwargs: P.kwargs) -> T:
            tstart = time.perf_counter()
            out = func(*args, **kwargs)
            display_fn(func.__name__, time.perf_counter() - tstart)
            return out

        return inner

    if __func is not None:
        return decorator(__func)

    return decorator
```
There is a lot of detail here but the main point is the two overload decorators. These specify the two signatures of the `timer` decorator. The first is a simple decorator just like we had before where you can use it with no parentheses. The second case is our second-order decorator which we can pass in a function to display the run time. We could use it as follows
```python
import logging

def logging_display_fn(function_name: str, call_time: float) -> None:
    logging.info(f"function: {function_name} ran in {call_time} seconds")

@timer(display_fn=logging_display_fn)
def add(a: float, b: float) -> float:
    """
    Add two numbers
    """
    return a + b
```
Now instead of using the default `print` we will use a logger. In both cases the type signature of the original `add` function are preserved. Just by looking at the complexity of the actual `timer` implementation we can start to see why overload can be dangerous since the function can get very complicated and it may be better to split into separate functions. Nonetheless using overload as a way to have both first- and second-order decorators is generally a decent idea.

To demonstrate the second case for overloads lets extend the functionality of our `max` function so that a user can pass in types that don't natively have support for `<`, but we allow them to pass in a key function to determine how to compare values of `a` and `b`:
```python
from typing import Callable, Protocol, TypeVar, overload

from typing_extensions import Self

class HasLessThan(Protocol):
    def __lt__(self, __other: Self) -> bool:
        ...

T = TypeVar("T", bound=HasLessThan)
S = TypeVar("S")

def _default_key(__val):
    return __val

@overload
def max(a: T, b: T) -> T:
    ...

@overload
def max(a: S, b: S, *, key: Callable[[S], T]) -> S:
    ...

def max(a, b, *, key=None):
    key_func = key or _default_key
    if key_func(a) < key_func(b):
        return b
    return a
```
In the example above we have two overloads. The first overload is exactly what we had before where the inputs must support the `<` operator. The second overload now allows a user to pass in any type but they are required to pass in a key function that returns the value that will be used to compare the two inputs.

For example, lets return to our custom line type but this time, we won't implement the `__lt__` method but instead use a key function in `max`
```python
from dataclasses import dataclass

@dataclass
class Line:
    x_min: float
    x_max: float

m = max(Line(0, 3), Line(0, 5)) # this will fail the type check
m = max(Line(0, 3), Line(0, 5), key=lambda line: (line.x_max - line.x_min)) # returns Line(0, 5)
```
So here our overloads let us be more flexible in our types. Note, that in this case I did
not include types in the actual implementation. It is up to you whether or not you do this since the actual implementation signature will not be exposed to the end user, only the overloads. 

The [`max` function](https://github.com/python/typeshed/blob/main/stdlib/builtins.pyi#L1382) in python actually has several more overloads if you want to check it out.


# Wrapping Up
In this post we have covered some more advanced use cases for type annotations. The TLDR version is
* Use general input types only specifying the bare minimum of what is needed 
* Use strict concrete output types
* Use custom Protocols define the minimum interface needed to functions/methods/classes
* Use custom Protocols as bounds on generic types to make code more general
* Use Parameter Specification (i.e. `ParamSpec`) when making decorators
* Use overloads sparingly when generics cannot do the job. Prefer overloads over usage of Union types, especially in return types

As I mentioned in the introduction, none of this is required for valid Python but making use of the type annotation system will make your code more readable and will make it much nicer to work with. 












