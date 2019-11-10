---
title: The Foo, the Bar, the Ugly
description: Improving code examples
# tags: mentoring, java, foo, teaching
author:       "johannesvollmer"
---

*Note: I originally wrote this post for [dev.to](https://dev.to/ryanous/the-foo-the-bar-the-ugly-5c71).*


When I learned my first programming language, Java, I just threw my questions into google. The web offers tons of helpful guides, and of course we all know StackOverflow, but theres a minor problem with many of them.


## Code Examples

Examples are undeniably one of the most effective way of learning. They are essential for any programming tip, tutorial, or guide. Glady, the majority of guides do have them.

## Specifically: Bad Code Examples

Unfortunately, someone once decided it was a good idea to do code examples using completely fictional identifiers without any meaning; the good, the bad, the ugly: `Foo`, `Bar`, and `Baz`. 

> They have been used to name entities such as variables, functions, and commands whose exact identity is unimportant and serve only to demonstrate a concept. 

That's what *[Wikipedia](https://en.wikipedia.org/w/index.php?title=Foobar&oldid=804411222)* says.

## But why are these words bad?

Let's have a look at an example of (an example using these keywords).
This example demonstrates how __interfaces__ in Java work:
```java
interface Foo {
    void foo();
}

class Bar implements Foo {
    @Override 
    public void foo(){
        System.out.println("Bar");
    }
}

class Baz implements Foo {
    @Override 
    public void foo(){
        System.out.println("Baz");
    }    
}

// ... later
void baz(Foo foo){
    foo.foo();
}
```

Admit it: You didn't _really_ read this example. It's just not fun to read. Relations between classes, interfaces and methods can only be learned by carefully reading the source code, maybe even only when already knowing a little bit about interfaces.


A key property of examples is that they can be related to. Thus, choosing a fictional and meaningless word contradicts the very definition of 'example'. By using `Foo`, you fail to provide context. 

## What should we do instead?

Wouldn't this example of interfaces be much easier to understand:
```java
interface SomethingThatPurrs {
    void purr();
}

class Cat implements SomethingThatPurrs {
    @Override 
    public void purr(){
        System.out.println("*purring cat*");
    }
}

class Kitten implements SomethingThatPurrs {
    @Override 
    public void purr(){
        System.out.println("*purring kitten*");
    }    
}

// ... later
void makeThatThingPurrSomehow(SomethingThatPurrs purring){
    purring.purr();
}
```

Cats and Kittens create a context. It's suddenly very easy to see that both of them can purr, and how SomethinThatPurrs should somehow relate to both. That's because you know that cats and kittens both purr.


I've seen beginners wondering what `Foo` and `Bar` actually mean, until someone tells them that there is no hidden meaning with that. Some even think that Foo and Bar are funny, which in turn motivates teachers to construct examples using these names.


In my opinion, when trying to explain a concept, one should do all that can be done in order to help the learnee. Providing context by using specific names and well-known concepts helps a lot.



## Conclusion
- Examples should provide as much context as possible
- Foo and Bar do not provide relatable context
- Foo and Bar are uncreative and boring
- Kittens are cute and inspiring


_Fun Fact: `Foo` and `Bar` are said to origin in the military acronym "FUBAR", which expands to "Fucked up beyond all recognition". What a coincidence._
