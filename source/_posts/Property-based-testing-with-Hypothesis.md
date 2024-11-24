---
title: Property-based testing with Hypothesis
tags:
  - python
  - testing
date: 2024-11-24 19:37:21
---


I'd [previously written about property-based testing in Clojure](/2017/11/16/unit-testing-with-specs/). In this blog post I'd like to talk about how we can do the same in Python using the [Hypothesis](https://hypothesis.readthedocs.io/en/latest/quickstart.html) library. We'll begin with a quick recap of what property-based testing is, and then dive head-first into writing some tests.  

## What is property-based testing?  

Before we get into property-based testing, let's talk about how we usually write tests. We provide a known input to the code under test, capture the resulting output, and write an assertion to check that it matches our expectations. This technique of writing tests is called example-based testing since we provide examples of inputs that the code has to work with.  

While this technique works, there are some drawbacks to it. Example-based tests take longer to write since we need to come up with examples ourselves. Also, it's possible to miss out on corner cases.

In contrast, property-based testing allows us to specify the properties of the code and test that they hold true under a wide range of inputs. For example, if we have a function `f` that takes an integer and performs some computation on it, a property-based test would test it for positive integers, negative integers, very large integers, and so on. These inputs are generated for us by the testing framework and we simply need to specify what kind of inputs we're looking for.  

Having briefly discussed what property-based testing is, let's write some code.  

## Writing property-based tests  

### Testing a pure function

Let's start with a function which computes the n'th Fibonacci number.   

{% code lang:python %}
@functools.cache
def fibonacci(n: int) -> int:
    """
    Computes the nth number in the Fibonacci sequence.
    """
    if n <= 1:
        return n

    return fibonacci(n - 1) + fibonacci(n - 2)
{% endcode %}  

The sequence of numbers goes 0, 1, 1, 2, 3, etc. We can see that all of these numbers are greater than or equal to zero. Let's write a property-based test to formalize this.  

{% code lang:python %}
from hypothesis import given, strategies as st
from functions import fibonacci


@given(st.integers())
def test_fibonacci(n):
    assert fibonacci(n) >= 0
{% endcode %}  

In the code snippet above, we've wrapped our test using the `@given` decorator. This makes it a property-based test. The argument to the decorator is a search strategy. A search strategy generates random data of a given type for us. Here we've specified that we need integers. We can now run the test using pytest as follows.  

{% code %}
PYTHONPATH=. pytest .
{% endcode %}  

The test fails with the following summary.  

{% code %}
FAILED test/functions/test_fibonacci.py::test_fibonacci - ExceptionGroup: Hypothesis found 2 distinct failures. (2 sub-exceptions)
{% endcode %}  

When looking at the logs, we find that the first failure is because the maximum recursion depth is reached when the value of `n` is large. 

{% code %}
n = 453
... lines omitted ...
RecursionError: maximum recursion depth exceeded
{% endcode %}  

The second failure is because the function returned a negative integer when the value of `n` is negative; in this case it is `n=-1`. This violates our assertion that the numbers in the Fibonacci sequence are non-negative.

{% code %}
+---------------- 2 ----------------
| Traceback (most recent call last):
|   File "/Users/fasih/Personal/pytesting/test/functions/test_fibonacci.py", line 8, in test_fibonacci
|     assert fibonacci(n) >= 0
| AssertionError: assert -1 >= 0
|  +  where -1 = fibonacci(-1)
| Falsifying example: test_fibonacci(
|     n=-1,
| )
+------------------------------------
{% endcode %}  

To remedy the two failures above, we'll add an assertion at the top of the function which will ensure that the input `n` is in some specified range. The updated function is given below.  

{% code lang:python %}
@functools.cache
def fibonacci(n: int) -> int:
    """
    Computes the nth number in the Fibonacci sequence.
    """
    assert 0 <= n <= 300, f"n must be between 0 and 300; {n} was passed."

    if n <= 1:
        return n

    return fibonacci(n - 1) + fibonacci(n - 2)
{% endcode %}  

We'll update our test cases to reflect this change in code. The first test case checks the function when `n` is between 0 and 300.  

{% code lang:python %}
@given(st.integers(min_value=0, max_value=300))
def test_fibonacci(n):
    assert fibonacci(n) >= 0
{% endcode %}  

The second case checks when `n` is large. In this case we check that the function raises an `AssertionError`. 

{% code lang:python %}
@given(st.integers(min_value=5000))
def test_fibonacci_large_n(n):
    with pytest.raises(AssertionError):
        fibonacci(n)
{% endcode %}  

Finally, we'll check the function with negative values of `n`. Similar to the previous test case, we'll check that the function raises an `AssertionError`.  

{% code lang:python %}
@given(st.integers(min_value=-2, max_value=-1))
def test_fibonacci_negative(n):
    with pytest.raises(AssertionError):
        fibonacci(n)
{% endcode %}  

### Testing persistent data

We'll now use Hypothesis to generate data that we'd like to persist in the database. The snippet below shows a `Person` model with fields to store name and date of birth. The `age` property returns the current age of the person in years, and the `MAX_AGE` variable indicates that the maximum age we'd like to allow in the system is 120 years. 

{% code lang:python %}
class Person(peewee.Model):

    MAX_AGE = 120

    class Meta:
        database = db

    id = peewee.BigAutoField(primary_key=True, null=False)
    name = peewee.CharField(null=False, max_length=120)
    dob = peewee.DateField(null=False)

    @property
    def age(self) -> int:
        return (datetime.date.today()).year - self.dob.year
{% endcode %}  

We'll add a helper function to create `Person` instances as follows.  

{% code lang:python %}
def create(name: str, dob: datetime.date) -> Person:
    """
    Create a new person instance with the given name and date of birth.
    :param name: Name of the person.
    :param dob: Date of birth of the person.
    :return: A Person instance.
    """
    assert name, f"name cannot by empty"
    return Person.create(name=name, dob=dob)
{% endcode %}  

Like we did for the function which computes Fibonacci numbers, we'll add a test case to formalize this expectation. This time we're generating random names and dates of birth and passing them to the helper function.

{% code lang:python %}
@given(
    text=st.text(min_size=1),
    dob=st.dates(),
)
def test_create_person(text, dob, create_tables):
    person = pr.create(name=text, dob=dob)
    assert 0 <= person.age <= Person.MAX_AGE
{% endcode %}  

I'm persisting this data in a Postgres table and the `create_tables` fixture ensures that the tables are created before the test runs.   

Upon running the test we find that it fails for two cases. The first case is when the input string contains a NULL character `\x00`. Postgres tables do not allow strings will NULL characters in them.

{% code %}
ValueError: A string literal cannot contain NUL (0x00) characters.
Falsifying example: test_create_person(
     create_tables=None,
     text='\x00',
     dob=datetime.date(2000, 1, 1),  # or any other generated value
)
{% endcode %}  

The second case is when the date of birth is in the future. 

{% code %}
AssertionError: assert 0 <= -1
  +  where -1 = <Person: 5375>.age
 Falsifying example: test_create_person(
     create_tables=None,
     text='0',  # or any other generated value
     dob=datetime.date(2025, 1, 1),
)
{% endcode %}  

To remedy the first failure, we'll have to sanitize the `name` input string that gets stored in the table. We'll create a helper function which removes any NULL characters from the string. This will be called before `name` gets saved in the table.

{% code lang:python %}
def sanitize(s: str) -> str:
    return s.replace("\x00", "").strip()
{% endcode %}  

To remedy the second failure, we'll add an assertion ensuring that the age is less than or equal to 120. The updated `create` function is shown below.

{% code lang:python %}
def create(name: str, dob: datetime.date) -> Person:
    """
    Create a new person instance with the given name and date of birth.
    :param name: Name of the person.
    :param dob: Date of birth of the person.
    :return: A Person instance.
    """
    name = sanitize(name)

    assert name, f"name cannot by empty"
    assert 0 <= (datetime.date.today().year - dob.year) <= Person.MAX_AGE

    return Person.create(name=name, dob=dob)
{% endcode %}  

We'll update the test cases to reflect these changes. Let's start by creating two variables that will hold the minimum and maximum dates allowed.  

{% code lang:python %}
MIN_DATE = datetime.date.today() - datetime.timedelta(days=Person.MAX_AGE * 365)
MAX_DATE = datetime.date.today()
{% endcode %}  

Next, we'll add a test to ensure that we raise an `AssertionError` when the string contains only NULL characters.  

{% code lang:python %}
@given(text=st.text(alphabet=["\x00"]))
def test_create_person_null_text(text, create_tables):
    with pytest.raises(AssertionError):
        pr.create(name=text, dob=MIN_DATE)
{% endcode %}

Next, we'll add a test to ensure that dates cannot be in the future.  

{% code lang:python %}
@given(
    text=st.text(min_size=1),
    dob=st.dates(min_value=MAX_DATE + datetime.timedelta(days=365)),
)
def test_create_person_future_dob(text, dob, create_tables):
    with pytest.raises(AssertionError):
        pr.create(name=text, dob=dob)
{% endcode %}

Similarly, we'll add a test to ensure that dates cannot be more than 120 years in the past.  

{% code lang:python %}
@given(
    text=st.text(min_size=1),
    dob=st.dates(max_value=MIN_DATE - datetime.timedelta(days=365)),
)
def test_create_person_past_dob(text, dob, create_tables):
    with pytest.raises(AssertionError):
        pr.create(name=text, dob=dob)
{% endcode %}

Finally, we'll add a test to ensure that in all other cases, the function creates a `Person` instance as expected.  

{% code lang:python %}
@given(
    text=st.text(min_size=5),
    dob=st.dates(
        min_value=MIN_DATE,
        max_value=MAX_DATE,
    ),
)
def test_create_person(text, dob, create_tables):
    person = pr.create(name=text, dob=dob)
    assert 0 <= person.age <= Person.MAX_AGE
{% endcode %}  

The tests pass when we rerun them so we can be sure that the function behaves as expected.  

### Testing a REST API.  

Finally, we'll look at testing a REST API. We'll create a small Flask app with an endpoint which allows us to create `Person` instances. The API endpoint is a simple wrapper around the `create` helper function and returns the created `Person` instance as a dictionary.  

{% code lang:python %}
@api.route("/person", methods=["POST"])
def create_person():
    name = request.json["name"]

    dob = request.json["dob"]
    dob = parse(dob).date()

    person = pr.create(name, dob)

    return model_to_dict(person)
{% endcode %}  

We'll add a test to generate random JSON dictionaries which we'll pass as the body of the `POST` request. The test is given below.  

{% code lang:python %}
@given(
    json=st.fixed_dictionaries(
        {
            "name": st.text(min_size=5),
            "dob": st.dates(min_value=MIN_DATE, max_value=MAX_DATE),
        }
    )
)
def test_create_person(json, test_client):
    response = test_client.post(
        "/api/person",
        json=json,
        headers={"Content-Type": "application/json"},
    )

    assert response.status_code == 200
{% endcode %}  

Similar to the tests for `create` function, we test that the API returns a response successfully when the inputs are proper.  

That's it. That's how we can leverage Hypothesis to test Python code.