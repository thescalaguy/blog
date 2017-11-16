---
title: Unit Testing with Specs
tags:
	- clojure
	- testing
---

[clojure.spec](https://clojure.org/about/spec) is a standard, expressive, powerful, and integrated system for specification and testing. It lets you define the shape of your data, and place contraints on it. Once the shape, and constraints are defined, clojure.spec can then generate sample data which you can use to test your functions. In this post I'll walk you through how you can use clojure.spec in conjunction with other libraries to write unit tests.

## Motivation

As developers, we are accustomed to writing example-based tests - we provide a known input, look at the resulting output, and assert that it matches our expectations. Although there is nothing wrong with this approach, there are a few drawbacks:  

1. It is expensive as it takes longer to complete.
2. It is easier to miss out on the corner cases.
3. It is more prone to pesticide paradox.<sup>[[1]](https://sqa.stackexchange.com/a/6442)</sup>

In contrast, clojure.spec allows you to do generative, property-based testing. Generative testing allows you to specify _what kind_ of data you are looking for. This is done by using generators. A generator is a declarative description of possible inputs to a function.<sup>[[2]](https://nofluffjuststuff.com/conference/raleigh/2013/08/session?id=29335)</sup> Property-based testing allows you to specify _how_ your program is supposed to behave, given an input. A property is a high-level specification of behavior that should hold for a range of inputs.<sup>[[3]](http://www.scalatest.org/user_guide/property_based_testing)</sup>

## Setup

### Creating an App

We'll begin by creating an app using `lein` and defining the dependencies. So go ahead and execute the following to create your project:

{% codeblock lang:clojure%}
lein new app clj-spec
{% endcodeblock %}

### Adding Dependencies

Next we'll add a few dependencies. `cd` into `clj-spec` and open `project.clj`. Add the following to your `:dependencies`  

{% codeblock lang:clojure %}
[org.clojure/clojure "1.8.0"]
[clojure-future-spec "1.9.0-beta4"]  
[org.clojure/test.check "0.9.0"]  
[circleci/bond "0.3.0"] 
[cloverage "1.0.9"]
{% endcodeblock %}  

`clojure.spec` comes as a part of Clojure 1.9 which, as of writing, isn't out yet. If you're on Clojure 1.8, as I am, you can use [`clojure-future-spec`](https://github.com/tonsky/clojure-future-spec) which will give you the same APIs. `circleci/bond` is a stubbing library which we'll use to stub IO, network calls, database calls, etc. `cloverage` is the tool we'll use to see the coverage of our tests.

## Using `clojure.spec`
### Simple Specs

Fire up a REPL by executing `lein repl` and `require` the required namespaces ;)

{% codeblock lang:clojure %}
clj-spec.core=> (require '[clojure.spec.alpha :as s])
nil
clj-spec.core=> (require '[clojure.spec.gen.alpha :as gen])
nil
clj-spec.core=> (require '[clojure.future :refer :all])
nil
{% endcodeblock %}

`spec` will let us define the shape of our data, and constraints on it. `gen` will let us generate the sample data.  

Let's write a simple spec which we can use to generate integers.

{% codeblock lang:clojure %}
clj-spec.core=> (s/def ::n int?)
:clj-spec.core/n
{% endcodeblock %}

We've defined a spec `::n` which will constrain the sample data to only be integers. Notice the use of double colons to create a namespace-qualified symbol; this is a requirement of the spec library. Now let's generate some sample data.

{% codeblock lang:clojure %}
clj-spec.core=> (gen/generate (s/gen ::n))
-29454
{% endcodeblock %}

`s/gen` takes a spec as an input and returns a generator which will produce conforming data. `gen/generate` exercises this generator to return a single sample value. You can produce multiple values by using `gen/sample`:

{% codeblock lang:clojure %}
clj-spec.core=> (gen/sample (s/gen ::n) 5)
(0 0 1 -1 0)
{% endcodeblock %}  

We could have done the same thing more succinctly by using the in-built functions as follows:

{% codeblock lang:clojure %}
clj-spec.core=> (gen/generate (gen/vector (gen/int) 5))
[25 -29 -13 26 -8]
{% endcodeblock %}

### Spec-ing Maps

Let's say we have a map which represents a person and looks like this:  

{% codeblock lang:clojure %}
{:name "John Doe"
 :age 32}
{% endcodeblock %}

Let's spec this. 

{% codeblock lang:clojure %}
(s/def ::name string?)
(s/def ::age int?)
(s/def ::person (s/keys :req-un [::name ::age]))
{% endcodeblock %}

We've defined `::name` to be a string, and `::age` to be an integer (positive or negative). You can make your specs as strict or as lenient as you choose. Finally, we define `::person` to be a map which requires the keys `::name` and `::age`, albiet without namespace-qualification. Let's see this in action:

{% codeblock lang:clojure %}
clj-spec.core=> (gen/generate (s/gen ::person))
{:name "KXYXcbk", :age 107706}
{% endcodeblock %}

By now you must have a fair idea of how you can spec your data and have sample values generated that match those specs. Next we'll look at how we can do property-based testing with specs.

## Using `test.check`

`test.check` allows us to do property-based testing. Property-based tests make statements about the output of your code based on the input, and these statements are verified for many different possible inputs.<sup>[[4]](http://blog.jessitron.com/2013/04/property-based-testing-what-is-it.html)</sup>

### A Simple Function

{% codeblock lang:clojure %}
(defn even-or-odd
  [coll]
  (map #(cond
          (even? %) :even
          :else :odd) coll))
{% endcodeblock %}

We'll begin by testing the simple function `even-or-odd`. We know that for all even numbers we should get `:even` and for all odd numbers we should get `:odd`. Let's express this as a property of the function. Begin by `require`-ing a couple more namespaces.

{% codeblock lang:clojure %}
clj-spec.core=> (require '[clojure.test.check :as tc])
nil
clj-spec.core=> (require '[clojure.test.check.properties :as prop])
nil
{% endcodeblock %}

Now for the actual property.

{% codeblock lang:clojure %}
(def property
  (prop/for-all [v (gen/vector (gen/choose 0 1))]
    (let [result (even-or-odd v)
          zeroes-and-ones (group-by zero? v)
          zeroes (get zeroes-and-ones true)
          ones (get zeroes-and-ones false)
          evens-and-odds (group-by #(= :even %) result)
          evens (get zeroes-and-ones true)
          odds (get zeroes-and-ones false)]
      (and (= (count zeroes) (count evens))
           (= (count ones) (count odds))))))
{% endcodeblock %}

We have a generator which will create a vector of 0s and 1s only. We pass that vector as an input to our function. Additionally, we know that the number of 0s should equal the number of `:even`s returned and that the number of 1s should equal the number of `:odd`s returned.

Next, let's test this property.

{% codeblock lang:clojure %}
clj-spec.core=> (tc/quick-check 100 property)
{:result true, :num-tests 100, :seed 1510754879429}
{% endcodeblock %}

Awesome! We ran the test a 100 times and passed. The added benefit is that the input generated will be different every time you run the test.

### Using `bond`

`bond` is a library which will let you stub side-effecting functions like database calls. We'll require the namespce and modify our code to save even numbers to database.

First, the namespace.

{% codeblock lang:clojure %}
clj-spec.core=> (require '[bond.james :as bond])
nil
{% endcodeblock %}

Next, the code.

{% codeblock lang:clojure %}
(defn save
  [n]
  ;; let's assume there's a database call here
  nil)

(defn even-or-odd
  [coll]
  (map #(cond
          (even? %) (do
                      (save %) ;; save to database
                      :even)
          :else :odd) coll))
{% endcodeblock %} 

Now let's update the property and stub `save`.

{% codeblock lang:clojure %}
(def property
  (prop/for-all [v (gen/vector (gen/choose 0 1))]
    (bond/with-stub [save]
      (let [result (even-or-odd v)
            zeroes-and-ones (group-by zero? v)
            zeroes (get zeroes-and-ones true)
            ones (get zeroes-and-ones false)
            evens-and-odds (group-by #(= :even %) result)
            evens (get zeroes-and-ones true)
            odds (get zeroes-and-ones false)]
        (and (= (count zeroes) (count evens))
             (= (count ones) (count odds))
             (= (count zeroes) (-> save bond/calls count)))))))
{% endcodeblock %}

Notice how we're using `bond/with-stub` and telling it to stub `save` function which calls the database. Later, we assert that the number of times that the databse was called is equal to the number of evens in the vector. Let's verify the property.

{% codeblock lang:clojure %}
clj-spec.core=> (tc/quick-check 10000 property)
{:result true, :num-tests 10000, :seed 1510834022725}
{% endcodeblock %}

Voilà! It works!

The last part of this post is about finding out test coverage using `cloverage`. For that, we'll be moving our code to `core.clj` and writing test under the `test` directory.

## Using `cloverage`  

To see `cloverage` in action, we'll need to add our functions to `core.clj`. Here's what it'll look like:

{% codeblock lang:clojure %}
(ns clj-spec.core
  (:gen-class))

(defn save
  [n]
  ;; let's assume there's a database call here
  nil)

(defn even-or-odd
  [coll]
  (map #(cond
          (even? %) (do
                      (save %) ;; save to database
                      :even)
          :else :odd) coll))

(defn -main
  "I don't do a whole lot ... yet."
  [& args]
  (println "Hello, World!"))
{% endcodeblock %}

Update your `clj-spec.core-test` to the following: 

{% codeblock lang:clojure %}
(ns clj-spec.core-test
  (:require [bond.james :as bond]
            [clojure.test.check.properties :as prop]
            [clojure.spec.gen.alpha :as gen]
            [clojure.test.check.clojure-test :refer :all]
            [clj-spec.core :refer :all]))

(defspec even-odd-test
 100
 (prop/for-all [v (gen/vector (gen/choose 0 1))]
   (bond/with-stub [save]
     (let [result (even-or-odd v)
           zeroes-and-ones (group-by zero? v)
           zeroes (get zeroes-and-ones true)
           ones (get zeroes-and-ones false)
           evens-and-odds (group-by #(= :even %) result)
           evens (get zeroes-and-ones true)
           odds (get zeroes-and-ones false)]
       (and (= (count zeroes) (count evens))
            (= (count ones) (count odds))
            (= (count zeroes) (-> save bond/calls count)))))))
{% endcodeblock %}

Here we are using the `defspec` macro to run the same property-based test a 100 times only this time we'll run the test via command-line using `lein`. Execute the following command to run the test and see the coverage.

{% codeblock %}
lein run -m cloverage.coverage -t 'clj-spec.core-test' -n 'clj-spec.core'
{% endcodeblock %}

This will make use of `cloverage` to run the tests. `-t` denotes our test namespace and `-n` denotes the namespace for whom the tests are written. You'll get an output like this:

{% codeblock lang:clojure %}
...
Produced output in /Users/fasih/Personal/clj-spec/target/coverage .
HTML: file:///Users/fasih/Personal/clj-spec/target/coverage/index.html

|---------------+---------+---------|
|     Namespace | % Forms | % Lines |
|---------------+---------+---------|
| clj-spec.core |   82.61 |   88.89 |
|---------------+---------+---------|
|     ALL FILES |   82.61 |   88.89 |
|---------------+---------+---------|
{% endcodeblock %}

Perfect!! Now we know how much coverage we have. The HTML file has a nice graphical representation of which lines we've covered with our tests.

## Conclusion

This brings us to the end of the post on using `clojure.spec` to write generative, property-based tests in both the REPL and source files. Generative testing automates the task of having to come up with examples for your tests. Where to go from here? Each of these libraries is pretty powerful in itself and will provide you with the necessary tools to write powerful, robust, and expressive tests that require minimal effort. So, head over to the offical docs to learn more.