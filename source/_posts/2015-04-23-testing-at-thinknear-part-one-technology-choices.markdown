---
layout: post
title: "Testing @ Thinknear - Part One: Technology Choices"
date: 2015-04-23 12:50:04 -0700
author: Ciprian Costin
comments: true
is_newest: true
categories: 
---

At Thinknear we believe automated tests are essential. Having strong automated test coverage allows the team to:

* deploy production code faster, while maintaining quality
* make bolder changes to the system with confidence
* increase confidence that a certain change won't break important features of the systems
* decrease TCO for the entire product lifecycle
* analyze how the product would scale up/out, through PRS (Performance, Reliability, Stress) tests

Thinknear's systems are based around two main development languages, Ruby and Java. Java is used for the real-time, high frequency bidding engine, while Ruby (on Rails) 
is used elsewhere. 

We test at the unit level, service level and integration level (end-to-end) using a veriety of tools: 

* for the Ruby on Rails systems: RSpec, Capybara, Poltergeist, Karma
* for the Java-based systems: JUnit, Hamcrest, Sonar
* for the build management system: Jenkins

Those tools were chosen in correlation with those development languages, the only exception being that we chose to write the service level tests 
for our Java-based systems in RSpec.

1. Why **RSpec**? The layout of the tests focuses on how the software should behave (BDD) and follows natural language in a very close manner, the vocabulary and 
structure of the tests being more intuitive than we found in other testing frameworks. An example is the matchers, supporting both positive and negative expectations.

2. Why **Capybara**? Capybara is used for UI browser testing, it allows easy switching of drivers, provides an efficient DSL for interacting with the DOM. We also 
evaluated Selenium, but chose Capybara because we found it to allow faster test development.
Writing 99% reliable code still comes with experience, ways to get to that percentage will be discussed in another post.

3. Why **Poltergeist**? Driver for PhantomJS (headless browser built on WebKit), the main reason for using it for our Capybara tests is speed. In our experiments,
it proved 4-10 times faster than Selenium's full browser driver. Poltergeist has very clear error messages and it can re-raise Javascript errors from the page.

4. Why **Karma**? Our UI relies heavily on AngularJS, therefore Karma, built by the AngularJS team, is the default solution for testing Javascript at the unit test level.
It supports many plugins, and lets you test in a cross-browser environment.

5. Why **JUnit**? The standard for testing in Java, JUnit is very well documented, supported by almost any IDE, is used for unit level testing, 
developers writing test cases while developing the software. TestNG is also a popular framework, we went with JUnit since this is what the team was most experienced with.

6. Why **Hamcrest**? A library of matcher objects, makes the code more readable by allowing match rules to be defined declaratively (closer to natural language).
We use it in combination with JUnit for our Java-based software.

7. Why **Sonar**? Sonar provides a plugin architecture for different analysis tools like FindBugs, Squid etc. It can aggregate information from those analyzers,
including those for understanding unit test code coverage. It has dashboards and historical views across all projects, and works well with the Jenkins build pipeline.
Sonar can also analyze other languages, although that feature can be immature depending on the language. Besides code coverage, we use it to detect package cycles 
(circular dependencies) and to analyze code complexity.

8. Why **Jenkins**? Popular continuous build tool, free and open source, Jenkins supports a wide array of plugins and features like support for SCM changes, console logs, 
email notifications, build exclusions and many others. Its flexibility allows it to fit in a variety of environments and makes it easy to develop complicated workflows.