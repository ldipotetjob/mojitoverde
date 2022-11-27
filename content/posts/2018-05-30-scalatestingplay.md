---
title: Api Rest - Playframework - Testing 
date: 2017-04-04 09:00:00
tags:
    - Scala
    - PlayFramework
    - Testing
categories:
    - scala programming 
keywords:
    - Scala
    - PlayFramework
    - Testing
---

As was  argued by [Robert C. Martin in his Clean Code bible](https://www.amazon.es/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882), there are several aspects that we have to pay attention when we design our tests:

- Minimize the number of assert per concept.
- Test just one concept per test function.
- Test should not depend on each other. 
- Test should run in any environment.