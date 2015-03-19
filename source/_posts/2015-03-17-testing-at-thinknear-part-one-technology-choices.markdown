---
layout: post
title: "Testing @ Thinknear - Part One: Technology Choices"
date: 2015-03-17 15:35:14 -0700
author: Ciprian Costin
comments: true
categories: 
---

At Thinknear we believe automated tests are essential to any modern reliable system, allowing the team to:

* deploy production code faster than ever before, while maintaining quality
* make bolder changes to the system and experiment
* increase confidence that a certain change won't break important features of the systems
* decrease TCO for the entire product lifecycle
* analyze how the product would scale up/out, through PRS (Performance, Reliability, Stress) tests

Thinknear's systems are based around two main development languages, Ruby and Java. Java is used for the real-time, high frequency bidding engine, while Ruby (on Rails) 
is used for the most part of the systems. 

We test at the unit level, service level and integration level (end-to-end) using a veriety of tools: 

* for the Ruby on Rails systems: RSpec, Capybara, Poltergeist, Karma
* for the Java-based systems: JUnit, Hamcrest, Sonar
* for the build management system: Jenkins

Those tools were chosen in correlation with those development languages, the only exception being that we chose to write the service level tests 
for our Java-based systems in RSpec.

1. Why **RSpec**? The layout of the tests focuses on how the software should behave (BDD) and follows natural language in a very close manner, the vocabulary and 
structure of the tests being more intuitive than in most other testing frameworks. An example is the matchers, supporting both positive and negative expectations.

2. Why **Capybara**? Capybara is used for UI browser testing, it allows easy switching of drivers, provides an efficient DSL for interacting with the DOM. 
In the past the framework was not that robust, developers having to add sleeps and hack their way to good reliability. Now it's greatly improved. Writing 99% reliable 
code still comes with experience, ways to get to that percentage will be discussed in another post.

3. Why **Poltergeist**? Driver for PhantomJS (headless browser built on WebKit), the main reason for using it for our Capybara tests is speed. In our experiments, 
it proved 4-10 times faster than the Selenium (full browser) driver. Poltergeist has very clear error messages and it can re-raise Javascript errors from the page.

4. Why **Karma**? Our UI relies heavily on AngularJS, therefore Karma, built by the AngularJS team, is the default solution for testing Javascript at the unit test level.
It supports many plugins, and lets you test in a cross-browser environment.

5. Why **JUnit**? The standard for testing in Java, very well documented, supported by almost any IDE, it's used for unit level testing, developers writing test cases
while developing the software.

6. Why **Hamcrest**? A library of matcher objects, makes the code more readable by allowing match rules to be defined declaratively (closer to natural language).
We use it in combination with JUnit for our Java-based software.

7. Why **Sonar**? Sonar is pretty much the only choice for static code analysis. It provides a plugin architecture for different analysis tools like FindBugs, Squid etc.
It can also analyze other languages too, although that feature can be immature depending on the language. Besides code coverage, we use it to detect package cycles 
(circular dependencies) and to analyze code complexity.

8. Why **Jenkins**? Popular continuous build tool, free and open source, Jenkins supports a wide array of plugins and features like support for SCM changes, console logs, 
email notifications, build exclusions and many others. Its flexibility allows it to fit in a variety of environments and makes it easy to develop complicated workflows.