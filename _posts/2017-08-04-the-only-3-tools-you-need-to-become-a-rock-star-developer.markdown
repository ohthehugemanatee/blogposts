---
layout: post
title: "The 3 skills you need to become a rock star developer"
date: 2017-08-04 12:03:27 +0200
comments: true
published: true
categories: 
  - code
  - php
  - development
---

A lot is made of so-called "rockstar developers" in any given language or framework. They have a seemingly magical knowledge of the language and API, finding obscure methods and writing best-practice code as if by instinct. I'd like to lift the curtain on this: it's not that hard to be this kind of rock star. You can do it, too... even without a fancy computer science degree. If you know how to code in a given language, you just need 3 skills and some patience. It will take about a year of working this way to get there, but you'll find people throwing around "the R word" sooner than you think.

1) Know how to explore the source
---

For most of us this comes down to knowing how to use your IDE, but other tools are available. It doesn't matter what tool you use, as long as it has these functionalities at a minimum:

* **Something that lets you explore the object graph.** In my IDE, a hotkey takes you directly from any usage of a function, method, or property, to the definition in use. It respects overrides and understands class inheritance. A different hotkey shows me the descendants of a method/property/class; how it's extended and overridden by it's children. You need to be able to poke around the entire application with enough ease that you can notice common design patterns. Class structure is where some of the most important design patterns live, and you want to learn from and match the rest of your application or framework.
* **Something that lets you quickly look up documentation.** One of my favorite things about Drupal is that the documentation standard is in the code itself, which makes this one very easy to satisfy. Most good IDEs will integrate with external documentation systems though. Exact types and example usages should be a hotkey away.
* **Something that provides hints as you code.** My IDE pays attention to type hints, and suggests available methods and properties as I type. This is more a speed thing than anything else, but it saves me from having to remember every single corner of the API.
* **Fast, robust searching.** Very often you know what you want to do, but you don't know quite how to do it. The rest of your framework is your friend: look for other code that does something similar. Think about how you might do this searching a codebase: simple text matching probably won't do - you need regex matching, at least. Constraint by location, probably constraints by filename, too. The other day I (really) was looking for an example of custom route access callbacks in Drupal - something that's awkwardly described in the doco, and it turns out, doesn't exactly work as documented anyway. I needed to search files matching `*.services.yml` for services with a tag value "applies_to" that starts with an underscore. The robust search capability of my IDE helped me find my example, and all I had to write was a little regex.

These functionalities are standard-issue in any normal IDE. So perhaps we should just reduce this whole point to "use an IDE, and learn its hotkeys." This will help you learn by example much faster.

2) Know how to use your debugger
---

There are two kinds of developers in the world: those who use a debugger, and those who piss their time into the wind. Most self-taught developers spend years beating their heads against the wall trying to resolve bugs. It's a major effort to remember everything in state, so they can't write more than 2 lines of code without some visible output to validate what's going on. I understand. I did this too. The PHP debugger (XDebug) seemed complicated to use, and I didn't really understand why it was important. So I avoided it for a long time.

**Holy shit, it's important**. A debugger lets you step through your code one line at a time, as it's being executed. You can look at the value of any variable, try out isolated lines of code to see what they return, and modify values as it's running. Honestly half the time, I develop by writing a skeleton of what I want in sloppy, busted code... then fire up the debugger and do the real writing while I'm looking at the actual values involved.

The other really valuable contribution of a debugger to your life, is that you can "step into" any function or method being called. That means you can jump directly inside the function to see what it does, and inside the functions _it_ uses, and so on and so on. Turns out that it's turtles all the way down: this is the best way to learn really how your framework works. Five and six layers deep you find patterns of abstraction that you simply won't get anywhere else. The kind of understanding you get from walking a complete code path to it's full depth, is impossible to get any other way. It's the kind of understanding that rock stars have.

Debuggers typically offer other useful functionalities, like performance profiling. Those are great when you need them, and I don't want to minimize their importance. Compared to the everyday functionalities though, these are edge cases. You need the basic tool of debugging *every time you touch the code.* So invest the time to figure out your debugger. It will take you an afternoon to really figure it out on your local environment. Spend that afternoon now. Thank me later.

3) Know how to write tests
---

I'll let you in on a secret: you don't have to be good at writing tests. Being good (generally) means your tests run faster, and cover more edge cases. Those are both Good Things, but the basic value of even amateur, I'm-just-figuring-it-out tests is 75% of the point. 

The core value of tests is that they run the exact. same. thing. every. damn. time. That's more precise than refreshing your browser, where a hundred factors might come into play before your code fires. Oh, and did I mention that you can trigger your debugger from your tests? Because you can. 

A basic test is just firing off one method or function on it's own, with known values as input. You write "asserts" to check the output. For example:

``` php
public function testAddition() {
  $result = \MathClass::add(2, 4);
  self::assertEquals(6, $result);
}
```

This is not the most complicated concept in the world. But this four lines means I can re-run the `addition` function a thousand times, stepping through it, with the same values every time. For testing a little additon function, this is certainly not necessary. But for testing submitted values entered on a web form, it saves me a ridiculous amount of time copypasting Lipsum text and fighting with form cache. You can also use this for big, integrated tests that require lots of dependencies. For example, Drupal includes base classes for various levels of bootstrap right up to the whole system. If you're writing a complicated migration, it's not crazy to have a test for it. You can determine the exact start state of the system (effectively a db dump), kick off the exact migration scripts in the right order, and validate the result, all in one command, with no clicking or human inaccuracy.

Tests do more than just save you time in coding. They also save time in maintenance, since you can leave your tests in place and know as soon as there's a regression. They also make refactoring possible. Having complete tests that make sure the whole system *works* means that I never have to worry if my refactor broke something. I know it didn't, because it passes the same tests.

OK, we all get it: testing saves you time. But the important part of test writing for this article, is how much you learn about your framework as you're doing it. You see, the hard part of writing tests is paring down the system you need for the test, to get rid of external factors. For example, technically I could bootstrap all of Symfony to run that example test above... but it's much faster if I just instantiate the MathClass class and run the function directly. So in my test, I won't bother starting anything from the system around it.

Most of your tests will be somewhere in between this simple example and a behemoth which needs the full weight of your framework to run. You want to only instantiate the bare minimum. So in Symphony, maybe you bootstrap the service container, but you only load up the services you need for the code you're testing. And of those, probably half of them can be "dummies" that are just there to give a type signature to a constructor. Good testing frameworks (shout out to PHPUnit!) make this easy.

The process of trying to pare down (or build up) dependencies to the minimum you need, *is a fantastic way to learn your framework's internals*. What REALLY are the dependencies for this function? What subsystems from my framework does it need to run, and what parts of those subsystems? This is a lot like the understanding you get from stepping through an entire request in the debugger: it's rock-star level understanding.

I'm not going to write a detailed howto on testing here. There are a thousand of them all over the internet, and it's pretty specific to your individual language. The point is: write tests. It will take a couple of months to get used to it, but after that you'll find you develop faster, refactor faster, experience fewer bugs and regressions... and you'll find you understand your framework better than any of your peers.


Being a rock star developer
---

These three tools are the fast track to understanding your framework and language on an extremely deep level. That understanding is the definition of a "rock star" developer. You will still google for solutions sometimes, but more often you'll look directly at the doco, or for other examples in the code. You'll probably still forget exactly which methods to use for obscure corners of the framework, or what parameters they take, but you'll have the tools to rediscover that information readily at hand. Most importantly, you'll understand how your system _thinks_, which is what being a rock star is all about.
