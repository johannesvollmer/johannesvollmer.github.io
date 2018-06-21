---
title: Don't be a Bracist
# jekyll-seo-tag
description:  "Why the world is treating braces differently from parantheses"
#image:        "http://placehold.it/400x200" # why nothing visible?
author:       "johannesvollmer"
---

*Note: I originally wrote this post for [dev.to](https://dev.to/ryanous/dont-be-a-bracist-2on).*

*Trigger warning: This post assumes that racism should be avoided.*

Despite being a student, I already know how time-consuming it can be to dig through other people's work. But there's a particular habit that makes reading code less awkward, and we all know it: Proper Formatting.

There are many online guides on how to format your code. Most IDE's even have a nifty tool that automatically formats your source files. But theres a tiny inconsistency which many of those tools and guides choose to ignore. 

## Common Brace Indentation
This post is all about the formatting of parantheses to group arguments and (curly) braces to group statements. JS, Java, C++, Rust, C and hundreds of other languages use these in a similar manner. Let's have a look at how braces are usually indented in Java:

```java
// inside KittenFactory
public List<Kitten> getKittensByAge(int minAge, int maxAge, boolean useGloves){
    final Filter ageFilter = new AgeFilter(minAge, maxAge);
    if (useGloves) {
        ageFilter.useHands();
        if (ageFilter.hasAnyGloves()) {
            ageFilter.removeWatch();
            ageFilter.enableGloves();
        }
    }
    return this.getKittens(ageFilter);
}
```

We start to see a pattern here: Closing braces are indented to the same level as their parent method head or if-keyword. Content inside braces is indented one level more than their parent keyword and closing brace. But now, let me throw that same code with an alternative indentation style at you (sorry, mobile phone users):

```java
public List<Kitten> getKittensByAge(int minAge, int maxAge, boolean useGloves){
    final Filter ageFilter = new AgeFilter(minAge, maxAge);
    if (useGloves) {
        ageFilter.useHands();
        if (ageFilter.hasAnyGloves()) {
            ageFilter.removeWatch();
            ageFilter.enableGloves();}}
    return this.getKittens(ageFilter);}

```

In case you are considering to write a comment about a missing closing brace, look again. It's right there. I hope you enjoy that feeling of confusion, because there's more:

```java
public List<Kitten> getKittensByAge(int minAge, int maxAge, boolean useGloves){
    final Filter ageFilter = new AgeFilter(minAge, maxAge);
    if (useGloves) {ageFilter.useHands();
                    if (ageFilter.hasAnyGloves()) {ageFilter.removeWatch();
                                                   ageFilter.enableGloves();}}
    return this.getKittens(ageFilter);}

```

## Common Paranthesis Indentation
I feel your confusion and disrespect, yet I can sense a weird feeling of familiarity deep inside of you. It's because we all have seen it before, but not with braces, but with parantheses:

```java
factory.getKittensByAge(0.2,
                        0.3,
                        user.glovesAreAvailableFor("Peter",
                                                   2017,
                                                   Location.of("Peter")));

```

Heck, this formatting even was found in [Oracle's formatting guide](http://www.oracle.com/technetwork/java/javase/documentation/codeconventions-136091.html) (officially outdated).

Wow.



## Alternative Indentation

This style of indentation greatly reduces readability. The important source code moves more and more to the right, far away from all the other code. Also, you can see a bunch of closing brackets cuddling at the end. Imagine having to debug a missing parantheses in such a function call. It's a mess. But fear not! There *are* alternatives! And we already use them, but with braces, and not with parantheses. Have a look:


```java
factory.getKittensByAge(
    0.2,
    0.3,
    user.glovesAreAvailableFor(
        "Peter",
        2017,
        Location.of("Peter")
    )
);

```

Yep, it's that simple. This style has some remarkable benefits: Linebreaks actually move code back to the left, and you can easily see, which closing bracket belongs to which function call. That's why we use that style for braces, after all. Also, having one special formatting rule less seems to be easier anyway.

That same indentation style can also be applied to if-statements and function parameter definitions:

```java
if(
    this.factory.filterType.hasGloves()
    && this.user.canUseGloves() // '&&' is important, so place it left
){
    factory.enableGloves();
}

```

```java
public void setAllFieldsAtOnce(
    double fluffiness,
    String name,
    double minAge,
    double maxAge,
    // ...
){
    this.fluffiness = fluffiness * Double.MAX_VALUE;
    // ...
}
```

## Conclusion

So, **please don't be a bracist**. Pay parantheses the same respect as braces. They deserve it! In the end, they are both used to group elements. Plus, your code will be much more readable.

I am aware that you may not be in the position to change the coding style at your company, but you can at least share this idea with friends and coworkers.

I hope you enjoyed my first post ever on dev.to!