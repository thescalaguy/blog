---
title: Scalaz Validation
date: 2017-07-11 15:03:41
tags:
  - scalaz
  - scala
  - functional programming
---
 
## Introduction  

In this post we'll look at Scalaz [`Validation`](https://github.com/scalaz/scalaz/blob/c0e4b531847348e1fd533c7d3605fe69320dde91/core/src/main/scala/scalaz/Validation.scala) which you can use to validate the data in your system. Data validation is a part and parcel of software development; you have to check the data that comes into your system or it may lead to unexpected behavior and / or cause your system to fail. Scalaz provides you with `Validation` to validate your data. `Validation` is an applicative functor. An applicative functor has more structure than a functor but less than a monad.<sup>[[1]](https://wiki.haskell.org/Applicative_functor)</sup>  

## Motivating Example

So let's say we have a `Version` class representing the major and minor version of our software like so:<sup>[[2]](http://arosien.github.io/scalaz-base-talk-201208/#slide20)</sup> 

{% codeblock lang:scala %}
case class Version(major: Int, minor: Int)
{% endcodeblock %}   

Then, a negative value in either of major or minor would be invalid. We could ensure that we never get a negative value in either the major or minor by using `require` like so:<sup>[[3]](http://arosien.github.io/scalaz-base-talk-201208/#slide23)</sup>  

{% codeblock lang:scala %}
case class Version(major: Int, minor: Int) {
  require(major >= 0, "major must be >= 0: %s".format(major))
  require(minor >= 0, "minor must be >= 0: %s".format(minor))
}
{% endcodeblock %}  

The problem here is that we'll have to handle exceptions and we don't want side-effects. Let's use `Validation` to do it in a more functional way.  

{% codeblock lang:scala %}
@ import scalaz._
import scalaz._
@ import Scalaz._
import Scalaz._
// paste mode in Ammonite REPL is { .. CODE HERE .. } and not :paste
@ {
  case class Version(major: Int, minor: Int)
  object Version {
        def createNew(major: Int, minor: Int): Validation[IllegalArgumentException, Version] = {
            val isValidMajor = (major >= 0) ? true | false
            val isValidMinor = (minor >= 0) ? true | false

            (isValidMajor, isValidMinor) match {
              case (false, true) => new IllegalArgumentException("major < 0").failure
              case (true, false) => new IllegalArgumentException("minor < 0").failure
              case (false, false) => new IllegalArgumentException("major and minor < 0").failure
              case (true, true) => new Version(major, minor).success
            }
        }
     }
  }
defined class Version
defined object Version
@ Version.createNew(-1, -1)
res5: Validation[IllegalArgumentException, Version] = Failure(
  java.lang.IllegalArgumentException: major and minor < 0
)
@ Version.createNew(1, 1)
res6: Validation[IllegalArgumentException, Version] = Success(Version(1, 1))
{% endcodeblock %}  

What we've done here is add a `createNew` to the companion object of `Version` that returns a `Validation`. Validating the input can result in either a `Success` or `Failure`. We create a `Success` by calling `success` on the value and a `Failure` by calling `failure` on the value. Once we have a `Validation`, there are numerous ways in which we can deal with it.  

### Providing Default Values  

{% codeblock lang:scala %}
@ val invalid = Version.createNew(-1, -1)
invalid: Validation[IllegalArgumentException, Version] = Failure(
  java.lang.IllegalArgumentException: major and minor < 0
)
@ val default = invalid | new Version(1, 0)
default: Version = Version(1, 0)
{% endcodeblock %}  

There's a convenient `|` operator (`getOrElse`) that lets you provide a default value if the result of the validation is a `Failure`. Here we are assigning the value 1 to major and 0 to minor.  

### Folding a Validation  

{% codeblock lang:scala %}
@ val valid = Version.createNew(1, 0)
valid: Validation[IllegalArgumentException, Version] = Success(Version(1, 0))
@ valid fold(
    f => s"Validation failed because ${f getMessage}".println,
    v => s"Version is $v".println
  )
"Version is Version(1,0)"
@ invalid fold(
    f => s"Validation failed because ${f getMessage}".println,
    v => s"Version is $v".println
  )
"Validation failed because major and minor < 0"
{% endcodeblock %}  

Akin to a disjunction, it's possible to `fold` a `Validation`. `Failure` will be the first argument and `Success` will be the second. If you want to fold just on the `Success` part, you can use `foldRight`.  

<hr>
##### NOTE: 
Both `fold` and `foldRight` should return values. Here I've just printed out the values to the console which is a side-effect. I've done this to keep the examples simple.
<hr>

### Composing Validations  

{% codeblock lang:scala %}
@ {
    case class Version(major: Int, minor: Int)
    object Version {
      def isValidMajor(major: Int) = 
        (major >= 0) ?
        major.success[IllegalArgumentException] |
        new IllegalArgumentException("major < 0").failure

      def isValidMinor(minor: Int) = 
        (minor >= 0) ?
        minor.success[IllegalArgumentException] |
        new IllegalArgumentException("minor < 0").failure

      def createNew(major: Int, minor: Int) = {
        (isValidMajor(major).toValidationNel |@| isValidMinor(minor).toValidationNel) {
          Version(_, _)
        }
      }
    }
  }
defined class Version
defined object Version
{% endcodeblock %}  

When we started building the `Version` example, we used pattern matching to check if the major and minor versions were correct. However, there's a more succinct way to collect all the errors. In the example above, we've turned `isValidMajor` and `isValidMinor` into methods that return a `Validation` instead of simply a `Boolean`.   

The magic is in `createNew`. Here we convert the `Validation` into a `NonEmptyList`. `toValidationNel` wraps the `Failure` in a `NonEmptyList` but keeps the `Success` as-is. The `|@|` is an `ApplicativeBuilder` which lets us combine the failures together. If we get all successes, we construct a `Version` out of it. Let's see this in action:  

{% codeblock lang:scala %}
@ val bothInvalid = Version.createNew(-1, -1)
bothInvalid: Validation[NonEmptyList[IllegalArgumentException], Version] = Failure(
  NonEmpty[java.lang.IllegalArgumentException: major < 0,java.lang.IllegalArgumentException: minor < 0]
)

@ val majorInvalid = Version.createNew(-1, 0)
majorInvalid: Validation[NonEmptyList[IllegalArgumentException], Version] = Failure(
  NonEmpty[java.lang.IllegalArgumentException: major < 0]
)

@ val minorInvalid = Version.createNew(1, -1)
minorInvalid: Validation[NonEmptyList[IllegalArgumentException], Version] = Failure(
  NonEmpty[java.lang.IllegalArgumentException: minor < 0]
)

@ val bothValid = Version.createNew(1, 1)
bothValid: Validation[NonEmptyList[IllegalArgumentException], Version] = Success(Version(1, 1))
{% endcodeblock %}  

So, in case of both the major and minor being invalid, we get a non-empty list with both the errors in it. This is very convenient and also very extensible. If you need to add an extra check, you can write a new `isValidXXX` method and make it return a `Validation`. You can then use the `ApplicativeBuilder` to your advantage. Using simply booleans would need checking a large number of possible cases.  

### Mapping `Success` and `Failure`  

{% codeblock lang:scala %}
@ val errMsgs = bothInvalid bimap(
    f => f map { _.getMessage },
    identity,
  )
errMsgs: Validation[NonEmptyList[String], Version] = Failure(NonEmpty[major < 0,minor < 0])
{% endcodeblock %}  

It's possible to map over `Success` and `Failure` and apply any transformations you want. For example, we represent errors with `IllegalArgumentException`s and we may be coding a rest API and we'd like to send back the strings representing the errors. In the example above, I've used `bimap` to map `Success` and `Failure`. For every failure, I am extracting the string using `getMessage` and leaving the success as-is by using `identity`. 

{% codeblock lang:scala %}
@ val theVersion = bothValid bimap(
    f => f map { _.getMessage },
    identity,
  )
theVersion: Validation[NonEmptyList[String], Version] = Success(Version(1, 1))
{% endcodeblock %}  

Similar to `bimap` is `map` which just maps the `Success` part of the `Validation`.  

{% codeblock lang:scala %}
@ val theValue = bothValid map { v => Version(v.major + 1, v.minor * 2) }
theValue: Validation[NonEmptyList[IllegalArgumentException], Version] = Success(Version(2, 2))
{% endcodeblock %}  

Remember that `bimap` and `map` return a `Validation`. They don't extract values out of it. You'll have to use `fold`, etc. to get the values out.  

### Running a Side-Effect on `Success`  

{% codeblock lang:scala %}
@ bothValid foreach { v => s"Begin build process for version $v".println }
"Begin build process for version Version(1,1)"
{% endcodeblock %}  

`foreach` lets you run a side-effecting function if the result of the `Validation` is a `Success`. For example, using the `Version`, you could begin a build process or similar.  

### Checking if it is a `Success` or `Failure`  

{% codeblock lang:scala %}
@ val isSuccess = bothValid isSuccess
isSuccess: Boolean = true

@ val isFailure = bothInvalid isFailure
isFailure: Boolean = true

@ val isSuccessSatisfying = bothValid exists { _.major < 5 }
isSuccessSatisfying: Boolean = true

@ val isSuccessSatisfyingOrFailure = bothValid forall { _.major === 5 }
isSuccessSatisfyingOrFailure: Boolean = false
{% endcodeblock %}  

There are numerous ways in which we can check if the `Validation` resulted in a `Success` or a `Failure`. The simplest way is to use one of `isSuccess` or `isFailure`. Similarly, `exists` will return true if the validation is a success value satisfying the given predicate and `forall` will return true if the validation is a failure value or the success value satisfies the given predicate.  

### Converting to Other Datatypes  

{% codeblock lang:scala %}
@ val list = bothValid toList
list: List[Version] = List(Version(1, 1))

@ val stream = bothValid toStream
stream: Stream[Version] = Stream(Version(1, 1))

@ val either = bothValid toEither
either: Either[NonEmptyList[IllegalArgumentException], Version] = Right(Version(1, 1))
{% endcodeblock %}  

Scalaz also provides convenience methods to convert the `Validation` to different datatypes like `List`, `Stream`, `Either`, etc.  

## Conclusion  

There's a lot more methods than can be covered in a post. Hopefully this post gave you an idea about what's possible with `Validation`s.