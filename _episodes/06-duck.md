---
title: "Duck typing and interfaces"
teaching: 10
exercises: 15
questions:
- "How does Python decide what you can and can't do with an object?"
- "When is inheritance not appropriate?"
- "What alternatives are there to inheritance?"
objectives:
- "Understand how duck typing works, and how interfaces assist with
  understanding this."
- "Understand polymorphism and how Python supports it implicitly."
- "Understand the circumstances where inheritance can be a hindrance
  rather than a help."
- "Be aware of concepts such as composition which can help where
  inheritance fails."
keypoints:
- "Provided a class exposes all required functionality for an
  operation to work, Python allows it."
- "Polymorphism means different types can be used interchangeably if they
  provide the expected behaviour."
- "Only use inheritance to express relationships where the subclass is
  the same kind of thing as the superclass."
- "Implementing interfaces and adding functionality with composition
  can be better alternatives to inheritance in some cases."
---

There is a principle that if something "looks like a duck, and swims
like a duck, and quacks like a duck, then it is probably a duck".

{% include image.html url="../assets/img/duck.jpg"
alt="Photograph of a duck and three ducklings on a body of
water" caption="These ducks are swimming and look like ducks, although
the quacking can't be guaranteed from this image."%}

<br/>

Python’s type system works the same way. If an object behaves in the way we
need, Python usually doesn’t care what type it actually is.

This is an example of duck typing, and it is closely related to polymorphism:
**the ability to use different types interchangeably as long as they provide the
required methods or behaviour.**

Below is a function that assumes only one behaviour: the object must be
repeatable (support * with an integer):

~~~
def repeat_twice(x):
    """Returns two copies of x concatenated together."""
    return x * 2
~~~
{: .language-python}

It works with numbers:

~~~
print(repeat_twice(10))
print(repeat_twice(3.141592653589793))
~~~
{: .language-python}

~~~
20
6.283185307179586
~~~
{: .output}


If you only planned for this to work with numbers, you might
think of adding a check at the start of the function to check the input value.
However, if we think about this in a duck
typed way, we don't really need to care about this&mdash;provided that
the values can be multiplied by an integer, then the algorithm will work.

This means that we can apply this function to cases we may not have
considered. For example, with strings:

~~~
print(repeat_twice("Quack! "))
~~~
{: .language-python}

~~~
Quack! Quack!
~~~
{: .output}

Or with lists:

~~~
print(repeat_twice([1, 2, 3]))
~~~
{: .language-python}

~~~
[1, 2, 3, 1, 2, 3]
~~~
{: .output}

Python doesn’t ask whether something is a number, string or list.
It simply asks: “Does this object support multiplication by an integer?”

If yes, the operation works. This is duck typing and polymorphism in action.

If we tried to enforce types manually, we would lose flexibility. For example,
imagine writing this check:

~~~
if not isinstance(x, int):
    raise TypeError("x must be an integer")
~~~
{: .language-python}

Then strings and lists would unnecessarily stop working, even though the function could handle them.

## Protocols

It is frequently useful to codify exactly what requirements are placed
on an object (or duck) so that we can design classes to match. In
Python, when these requirements are documented, the specification is
called a _protocol_; you may also hear the word (informal) _interface_
used to describe this as well.

An example of a well-known protocol in Python is the iterator
protocol, which should be obeyed by objects returned by the
`__iter__()` method. For a class to support the iterator protocol, it
must have two methods:

* `__iter__()`, which returns the object itself. (This is so that an
  iterator can be given to a `for` loop directly, which is sometimes
  desirable rather than relying on it being returned by the
  `__iter__()` method of a collection-type object.)
* `__next__()`, which returns the next item in the sequence. If there
  are no more items, then this should raise the `StopIteration`
  exception, and successive calls should keep raising this exception.

For instance, an iterator that yields numbers 1 through n may look
something like:

~~~
class CountToN:
    def __init__(self, n):
        self.n = n
        self.current = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self.current >= self.n:
            raise StopIteration
        self.current += 1
        return self.current
~~~
{: .language-python}

This is an example of where we want to use the iterator directly with
the `for` loop, since we want to initialise it with a
`max_value`. Testing this:

~~~
for number in CountToN(5):
    print(number)
~~~
{: .language-python}

~~~
1
2
3
4
5
~~~
{: .output}

> ## Polymorphism again
>
> Any class implementing the iterator protocol can be used in a for loop.
> Python doesn’t care what it is, only what it does.
{: .callout}

> ## Triangular numbers
>
> Write a class that implements the iterator protocol and that returns
> the first \\(n\\) triangular numbers. These are defined such that
> the \\(n\\)th triangular number is the sum of the first \\(n\\)
> positive integers, so the first five are 1, 3, 6, 10, and 15.
>
> What would happen if you removed the upper bound (and so never
> raised `StopIteration`) and used the iterator in a `for` loop? When
> might this behaviour be useful?
>
>> ## Solution
>>
>> ~~~
>> class TriangularIterator:
>>     def __init__(self, n):
>>         self.n = n
>>         self.total = 0
>>         self.index = 0
>>
>>     def __iter__(self):
>>         return self
>>
>>     def __next__(self):
>>         if self.index >= self.n:
>>             raise StopIteration
>>         self.index += 1
>>         self.total += self.index
>>         return self.total
>> ~~~
>> {: .language-python}
>>
>> A loop over an iterator that can't raise `StopIteration` will run
>> forever. This could be useful if you're using `zip()` to iterate
>> over another, bounded, iterable at the same time; then each element
>> will get a corresponding triangular number, no matter how many
>> elements there are.
> {: .solution}
{: .challenge}

> ## Spot the problem
>
> Look back at the solutions for the `QuadraticPlotter`,
> `PolynomialPlotter`, and `FunctionPlotter`. What problems do you see
> with the `plot` method of these classes?
>
>> ## Solution
>>
>> The arguments to `FunctionPlotter.plot()`, `PolynomialPlotter.plot()`,
>> and `QuadraticPlotter.plot()` are all different&mdash;one expects a
>> callable, one expects a list of coefficients as one argument, and
>> one expects three coefficients as separate arguments. In general,
>> specialistations of a class should keep the same interface to its
>> functions, and the parent class should be interchangeable with its
>> specialisations.
> {: .solution}
{: .challenge}


## Abstract base classes

Python also allows us to go a step further than a protocol, and
formalise the requirements we place on our interfaces in code. An
_abstract base class_ is a class that must be inherited from&mdash;you
can't create instances of it directly. Python provides these for many
of its protocols in the `collections.abc` module. For example, the
CountToN iterator above could inherit from `abc.Iterator`. This would
allow other code to check in advance that it supports the protocol,
and also would guard against us forgetting to implement some part of
the protocol. For example, if we forgot the `__next__()` method:

~~~
from collections.abc import Iterator
class CountToN(Iterator):
    def __init__(self, n):
        self.n = n
        self.current = 0

for number in CountToN(5):
    print(number)
~~~
{: .language-python}

In this case Python gives us an error:

~~~
TypeError                                 Traceback (most recent call last)
<ipython-input-3-a96ac2788df3> in <module>
      4         self.n = n
      5         self.current = 0
----> 7 for number in CountToN(5):
      8     print(number)

TypeError: Can't instantiate abstract class CountToN with abstract methods __next__
~~~
{: .output}

This can be useful when working with more complex interfaces. (On the
other hand, removing the `__iter__()` method works fine, because
`abc.Iterator` helpfully defines `__iter__()` for us, so we can
inherit it.)

> ## Implementing multiple interfaces
>
> You may find yourself wanting to implement multiple interfaces in a
> single class. This is possible by making use of _multiple
> inheritance_, where a class inherits from more than one base
> class. This is not supported in all programming languages, and in
> many programming languages it is considered to be problematic. It is
> more common in Python, but we don't have space to go into detail
> about it in this lesson.
{: .callout}

> ## Hashable `Polygon`s
>
> The hashable protocol allows classes to be used as dictionary keys
> and as members of sets. Look up [the hashable protocol][hashable]
> and adjust the `Polygon` class so that it follows this.
>
> Test this by using a `Triangle` instance as a dict key:
>
> ~~~
> triangle_descriptions = {
>     Triangle([3, 4, 5]): "The basic Pythagorean triangle"
> }
> ~~~
> {: .language-python}
>
>> ## Solution
>>
>> The hashable protocol requires implementing one method,
>> `__hash__()`, which should return a hash of the aspects of the
>> instance that make it unique. Lists can't be hashed, so we also
>> need to turn the list of `side_lengths` into a tuple.
>>
>> ~~~
>>     def __hash__(self):
>>         return hash(tuple(self.side_lengths))
>> ~~~
>> {: .language-python}
> {: .solution}
{: .challenge}


## Composition

Composition is a technique where rather than adding more and more
functionality to a single class (either explicitly, or via
inheritance), functionality is added by adding instances of other
classes that group together the related functionality.

An example of a library that makes heavy use of composition is the
Matplotlib object-oriented API. While Matplotlib makes its `pyplot`
API available for basic plotting, it is built on top of a very
intricate hierarchy of classes and objects.

Here is a example using composition instead of inheritance:
~~~
class Multiplier:
    def __init__(self, factor):
        self.factor = factor

    def apply(self, x):
        return x * self.factor

class Calculator:
    def __init__(self, multiplier):
        self.multiplier = multiplier

    def double(self, x):
        return self.multiplier.apply(x)

calc = Calculator(Multiplier(2))
print(calc.double(10))
~~~
{: .language-python}

~~~
20
~~~
{: .output}

You can see that the `Calculator` class doesn't need to know how
multiplication works; it simply relies on the `Multiplier` class to
handle that. This means that if we wanted to change how multiplication
worked, we could do so by changing the `Multiplier` class, or by
passing in a different class that implemented the same interface as
`Multiplier`.

This is why when you have errors in your code, tracebacks from some
libraries can be quite long. Having lots of small methods in classes
that are dedicated to one very specific aspect means that it is easier
to reason about what each one is doing in isolation by itself, but can
make it more complicated to get a view of the big picture.

> ## Composing plotters
>
> How could the `FunctionPlotter`, `PolynomialPlotter`, and
> `QuadraticPlotter` be refactored to make use of composition instead
> of inheritance?
>
>> ## Solution
>>
>> One way of doing this is to define a "plottable function"
>> interface. An object respecting this interface would:
>>
>> * be callable
>> * accept one argument
>> * return \\(f(x)\\)
>>
>> Then, with the `FunctionPlotter` as defined previously, there is no
>> need to subclass to create `QuadraticPlotter`s and
>> `PolynomialPlotter`s; instead, we can define a `QuadraticFunction`
>> class as:
>>
>> ~~~
>> class Quadratic:
>>     def __init__(self, a, b, c):
>>         self.a = a
>>         self.b = b
>>         self.c = c
>>
>>     def __call__(self, x):
>>         return self.a * x ** 2 + self.b * x + self.c
>> ~~~
>> {: .language-python}
>>
>> This can then be passed to a `FunctionPlotter`:
>>
>> ~~~
>> plotter = FunctionPlotter()
>> plotter.plot(Quadratic(1, -1, 1))
>> ~~~
>> {: .language-python}
>>
>> Alternatively, we can _encapsulate_ the function to be plotted as
>> part of the class.
>>
>> ~~~
>> from matplotlib.colors import is_color_like
>>
>> class FunctionPlotter:
>>     def __init__(self, function, color="red", linewidth=1, x_min=-10, x_max=10):
>>         assert is_color_like(color)
>>         self.color = color
>>         self.linewidth = linewidth
>>         self.x_min = x_min
>>         self.x_max = x_max
>>         self.function = function
>>
>>     def plot(self):
>>         """Plot a function of a single argument.
>>         The line is plotted in the colour specified by color, and with width
>>         linewidth."""
>>         fig, ax = subplots()
>>         x = linspace(self.x_min, self.x_max, 1000)
>>         ax.plot(x, self.function(x), color=self.color, linewidth=self.linewidth)
>> ~~~
>> {: .language-python}
>>
>> This could then be used as:
>>
>> ~~~
>> from numpy import sin
>>
>> sin_plotter = FunctionPlotter(sin)
>> quadratic_plotter = FunctionPlotter(Quadratic(1, -1, 1), color="blue")
>> sin_plotter.plot()
>> quadratic_plotter.plot()
>>
>> show()
>> ~~~
>> {: .language-python}
> {: .solution}
{: .challenge}

[hashable]: https://docs.python.org/library/collections.abc.html#collections.abc.Hashable
