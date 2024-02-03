---
title: Creating facades in Python
tags:
  - python
date: 2024-02-03 16:50:52
---

One of the GoF design patterns is the [facade](https://en.wikipedia.org/wiki/Facade_pattern). It lets us create a simple interface that hides the underlying complexity. For example, we can have a facade which lets the client book meetings on various calendars like Google, Outlook, Calendly, etc. The client specifies details about the meeting such as the title, description, etc. along with which calendar to use. The facade then executes appropriate logic to book the meeting, without the client having to deal with the low-level details. 

This post talks about how we can create a facade in Python. We'll first take a look at [singledispatch](https://docs.python.org/3/library/functools.html#functools.singledispatch) to see how we can call different functions depending on the type of the argument. We'll then build upon this to create a function which dispatches based on the value instead of the type. We'll use the example given above to create a function which dispatches to the right function based on what calendar the client would like to use.  

## Single Dispatch

The official documentation defines single dispatch to be a function where the implementation is chosen based on the type of a single argument. This means we can have one function which handles integers, another which handles strings, and so on. Such functions are created using the `singledispatch` decorator from the `functools` package. Here's a trivial example which prints the type of the argument handled by the function.  

{% code lang:python %}
import functools


@functools.singledispatch
def echo(x):
    ...


@echo.register
def echo_int(x: int):
    print(f"{x} is an int")


@echo.register
def echo_str(x: str):
    print(f"{x} is a str")


if __name__ == "__main__":
    echo(5)
    echo("5")
{% endcode %}  

We start by decorating the `echo` function with `singledispatch`. This is the function we will pass our arguments to. We then create `echo_int`, and `echo_str` which are different implementation that will handle the various types of arguments. These are registered using the `echo.register` decorator.  

When we run the example, we get the following output. As expected, the function to execute is chosen based on the type of the argument. Calling the function with a type which is not handled results in a noop as we've set the body of `echo` to ellipses.

{% code %}
5 is an int
5 is a str
{% endcode %}  

When looking at the source code of `singledispatch`, we find that it maintains a dictionary which maps the type of the argument to its corresponding function. In the following sections, we'll look at how we can dispatch based on the value of the argument.  

## Example  

Let's say we're writing a library that lets the users book meetings on a calendar of their choosing. We expose a `book_meeting` function. The argument to this function is an instance of the `Meeting` data class which contains information about the meeting, and the calendar on which it should be booked.   

## Code  

### Model

We'll start by adding an enum which represents the calendars that we support.  

{% code lang:python %}
import enum


class Calendar(str, enum.Enum):
    GOOGLE = "google"
    OUTLOOK = "outlook"
{% endcode %}  

Next we'll add the data class which represents the meeting as a `dataclass`.

{% code lang:python %}
import dataclasses as dc 
import datetime


@dc.dataclass(frozen=True)
class Meeting:
    title: str
    description: str
    time: datetime.datetime
    calendar: Calendar
{% endcode %}  

Finally, we'll start creating the facade by adding functions which will dispatch based on the value of `calendar` contained within the instance of `Meeting`.  

### Dispatch  

We'll create a registry which maps the enum to its corresponding function. The function takes as input a `Meeting` object and returns a boolean indicating whether the meeting was successfully booked or not.

{% code lang:python %}
from typing import Callable, TypeVar 
MeetingT = TypeVar("MeetingT", bound=Callable[[Meeting], bool])
registry: dict[Calendar, MeetingT] = {}
{% endcode %}  

Next we'll add the `book_meeting` function. This is where we dispatch to the appropriate function depending on the meeting object that is received as the argument.  

{% code lang:python %}
def book_meeting(meeting: Meeting) -> bool:
    func = registry.get(meeting.calendar)
    
    if not func:
        raise Exception(f"No function registered for calendar {meeting.calendar}")
    
    return func(meeting)
{% endcode %}  

To be able to register functions which contains the logic for a particular calendar, we'll create a decorator called `register`.  

{% code lang:python %}
def register(calendar: Calendar):
    def _(func: MeetingT):
        if registry.get(calendar):
            raise Exception(f"A function has already been registered for {calendar}")
        registry[calendar] = func
        return func
    return _
{% endcode %}  

`register` accepts as argument the calendar for which we're registering a function. It returns another higher-order function which puts the actual function in the registry. Since the actual logic of the execution is in the decorated function, we simply return the original function `func`.  

Finally, we register functions for different calendars.  

{% code lang:python %}
@register(Calendar.GOOGLE)
def book_google(meeting: Meeting) -> bool:
    print(f"Booked Google meeting")
    return True


@register(Calendar.OUTLOOK)
def book_outlook(meeting: Meeting) -> bool:
    print(f"Booked Outlook meeting")
    return True
{% endcode %}


We'll put all of this code in action by trying to book a meeting on Google calendar.  

{% code lang:python %}
if __name__ == "__main__":
    meeting = Meeting(
        title="Hello",
        description="World",
        time=datetime.datetime.now(),
        calendar=Calendar.GOOGLE
    )

    book_meeting(meeting=meeting)
{% endcode %}  

This prints "Booked Google meeting", like we'd expect. We can now continue to add more functions which contain logic for specific calendars. Our library can evolve without any change to the exposed interface. It's also possible to organise functions into their own modules, import the `register` decorator, and decorate them to add them to the registry. This has two main benefits. One, we keep the code well structured. Two, the code for different versions of the same calendar can stay separated; we avoid having to write `if` checks to see the calendar version since that can be made part of the enum itself, like `GOOGLE_V1`.  

That's it. That's how you can create a facade in Python.