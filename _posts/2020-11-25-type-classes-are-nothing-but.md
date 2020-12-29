---
title:          You don't need type classes
description:    Demystifying type classes in pure functional languages such as Haskell and PureScript
image:          "{{ site.baseurl }}/img/regex-nodes/thumbnail.png" 
logo:           "{{ site.baseurl }}/img/logo.svg"
author:         johannesvollmer
published: 	    false
---

Hey! In this post, I try to demystify type classes. In addition, I present a less obfuscated alternative.

# Why should I worry?

Imagine we want to code a `square` function, which returns a number multiplied with itself. Too simple, right? Let's start by implementing a function for all `Float` numbers. Here's your Haskell:

```haskell
square :: Float -> Float -- squaring a float returns a float 
square x = x * x         -- square the single float argument
```

Of course, it won't work with integers! We'll have to duplicate this function for integers, no, rather use a generic implementation:

```haskell
square :: number -> number
square x = x * x
```

When this function is called, the lowercase type `number` will be replaced with either `Float` or `Int`. But, as you probably already realized, this will not compile, because two arbitrary values can not necessarily be multiplied.

Therefore, in our type declaration, we somehow have to specify that we can __only square a number if it can be multiplied__. This can be done with type classes. Look at the following Haskell code:

```haskell
-- square any number which can be multiplied
square :: (Multipliable number) => number -> number
square number = multiply number number


-- declare that being multipliable 
-- means we can use the multiplication operator
class Multipliable number where

   -- define that multiplying requires two numbers 
   -- and returns a single number
   multiply :: number -> number -> number


-- declare that Int can be multiplied 
-- using the `multiplyInt` function defined somewhere else
instance Multipliable Int where
   multiply = multiplyInt

-- declare that Float can be multiplied 
-- using the `multiplyFloat` function defined somewhere else
instance Multipliable Float where
   multiply = multiplyFloat


-- calling the `square` function implicitly looks up that
-- floats are multiplied using the `multiplyFloat` function

nine = square 3.0 -- = `multiplyFloat 3.0 3.0`
```

However, in real Haskell code, the multiplication operator belongs to the `Number` class. This means that squaring a number now unnecessarily requires our number to support all the other possible numerical operations, which is `+`, `-`, `*`, `negate`, `abs`, `signum` and `fromInteger`, just in order to square it.

Plain functional programming is delighting, because it clearly separates behaviour from data. With type classes however, this clear distinction is lost. Suddenly, a piece of data _can_ or _can't_ do a thing. A number should simply be data, and a function should simply operate on numerical data. Why does the number type now _own_ a function?

What if we instead untangle this mess and let functions be functions, and data be data?  

# You Don't Need Type Classes

As you probably already noticed, these type classes require quite a bit of boilerplate code. In essence though, the code boils down to only four fundamental declarations:
- any type of number can be squared, if it can be multiplied
- multiplication always requires two numbers
  and results in a single number
- an float can be multiplied using `multiplyFloat`
- an int can be multiplied using `multiplyInt`

However, we can express these fundamental ideas without using type classes. This even results in less code than using type classes. How do we do that? It's actually quite a simple concept. We just take an additional argument which contains a function that multiplies our number. Using this strategy, we shift the responsibility of multiplication from the number data type to the caller of our `square` function.

> Instead of asking whether a number is multipliable, ask how to multiply numbers.

```haskell
-- given a multiplication function for a number,
-- square that number
square :: Multiply number -> number -> number

-- take the `times` function as an argument
square multiply number = multiply number number 


-- define a function type
-- that requires two numbers and returns a single number
-- which is just the type of `multiplyInt` and `multiplyFloat`
type Multiply number = 
   number -> number -> number


-- squaring now requires us to choose 
-- a function that multiplies numbers
nine = square multiplyInt 3
nine = square multiplyFloat 3.0
```

This method has many advantages. Most importantly, it allows us to choose an implementation. This removes a major limitation of type classes. The implementation is no longer defined by the data type, but can be dynamically chosen. How often did you have a dictionary of strings that should be sorted in a custom way, maybe regardless of case? How often have you created a newtype just to put something into a dictionary? A newtype that wraps a plain type simply to customize how something should be compared? This is only a problem because with type classes, a function now belongs to a data type.

As a bonus, the second method does not use any special syntax except for the plain old function polymorphism. However, to achieve comparable performance, the compiler must propagate constants and then inline the function. If this optimization is not implemented, this method might be slightly slower.

There is one single real disadvantage though. Most of the time, you will want to use the plain old `multiplyFloat` function when multiplying floats. The need to explicitly state that you're using the default behaviour is code noise and hurts readability. 

So, how do type classes choose an implementation? When defining either a data type or a type class, the type class instance is defined like this:

```haskell
instance Multipliable Float where
   multiply = multiplyFloat

-- calling the `square` function implicitly searches
-- a static global table for the Multipliable instance
-- of the type `Float`, which is the `multiplyFloat` function
nine = square 3.0
```
Therefore, given a concrete type like `Float`, we get the `multiplyFloat` function.

One possible improvement would be the following strategy: If a sensible default implementation exists, one could create a simple alias for the concrete implementation.

```haskell

-- the main implementation
square multiply number = multiply number number

-- default behaviours
squareInt number = square multiplyInt number
squareFloat = square multiplyFloat
```

This method scales with complexity as follows:
```haskell

-- the arithmethic mean of two numbers
average :: NumberFunctions number -> 
   number -> number -> number

average functions firstNumber secondNumber =
   let sum = functions.add firstNumber secondNumber in 
   sum `functions.divideBy` (functions.fromInt 2)

-- we optionally provide some types if we desire
type NumberFunctions number =
   { add :: number -> number -> number
   , divideBy :: number -> number -> number
   , fromInt :: Int -> number
   }

-- default float implementation
averageFloat firstNumber secondNumber = average 
   { add = addFloat
   , divideBy = divideByFloat
   , fromInt = intToFloat
   }
   firstNumber
   secondNumber

-- default int implementation
averageInt = average defaultIntFunctions

-- using a separate variable allows us to 
-- reuse this collection of functions 
-- in other functions such as `median` and not only `average`
defaultIntFunctions = 
   { add = addInt
   , divideBy = divideByInt
   , fromInt = identity
   }

-- call the functions
three = averageInt 2 4
half = averageFloat 0.0 1.0
```

Compare it to type classes:

```haskell

-- the arithmethic mean of two numbers
average :: (Number number) => 
   number -> number -> number

average firstNumber secondNumber =
   let sum = addNumber firstNumber secondNumber in 
   sum `divideByNumber` (numberFromInt 2)

-- we are forced to provide some explicit types 
-- when dealing with type classes
class Number number where
   addNumber :: number -> number -> number
   divideByNumber :: number -> number -> number
   numberFromInt :: Int -> number

instance Number Int where
   addNumber = addInt
   divideByNumber = divideByInt
   numberFromInt = identity

instance Number Float where
   addNumber = addFloat
   divideByNumber = divideByFloat
   numberFromInt = intToFloat

-- the concrete implementation cannot be changed on call-site
-- as it is rigidly declared above (or in another module)
three = average 2 4
half = average 0.0 1.0
```

As you can see, the code that avoids type classes exhibits quite a bit of explicit code.

# Real-Life Example

To display further advantages of this pattern, let's have a look at a container. Because this is the `SortedList` container, the elements must be comparable to each other.

First, here's the implementation that uses type classes:

```haskell

-- declare that our sorted list can contain elements
-- which can be compared to each other
type (Comparable element) => 
   SortedList element = 
      { elementList :: List element
      }

class Comparable element where
   compare :: element -> element -> Ordering

data Ordering = Less | Equal | Greater
```

Now, let's reframe our requirement: The comparability is not an inherent property of our elements anymore, but instead we can store any type of element if we also know how to compare one element with another.

```haskell
-- declare that our sorted list can contain any type of elements
-- but only if we also know how to compare them
type SortedList element = 
      { elementList :: List element
      , comparator :: element -> element -> Ordering
      }

data Ordering = Less | Equal | Greater
```

In this example, there's very little benefit when using type classes. The latter approach allows users to customize the sorting behaviour while simultaneously using less specialized syntax, and less code, in total.

# So, what's my point? 
Instead of introducing type classes, a functional language should instead consider reducing the syntactical boilerplate of the proposed alternative, as it is a more general solution. 

-- For example, introducing some kind of inferred parameter   
-- could reduce the boilerplate of this alternative paradigm.

For example, if a language has partial function specialization and default parameters, the following code might be something to consider:
```haskell
squareWith multiply number = multiply number number
square number = squareWith (defaultMultiplier $ typeof number) number

defaultMultiplier :: number => (number -> number -> number)
defaultMultiplier Float = multiplyFloat

-- could possibly also be specialized inside the integer module
defaultMultiplier Int = multiplyInt 

-- any other type than Float or Int would 
-- throw a compile-time error and 
-- require using `squareWith` with an explicit parameter
```


Or instead of having the whole syntax sugar for type classes in a language, one could instead have only the most important syntax sugar:
```haskell
squareWith number multiply = multiply number number
   -- declare multiply choose from some default values 
   -- if no explicit value is provided
   where multiply <| multiplyFloat <|> multiplyInt

```
This, however, requires us to set all defaults within the function declaration.

# Haskell, PureScript, Elm

# Problems with Typeclasses 
- Can not be chosen at call time. Wrappertypes obfuscate the data hierarchy to enable implementing custom behaviour. The behaviour is unconditionally attached to a data type and cannot be adjusted. Implementing a type class for a data type can be compared to inserting an entry to a global mutable table, but with artificially restricted rights.

 All possible interfaces you can think of have to be implemented immediately and can not be added ouside of a module.

- Smells like OOP: Addition and multiplication belong to the number type class. Why?

- A large amount of syntactic sugar is required to specify such a simple idea 