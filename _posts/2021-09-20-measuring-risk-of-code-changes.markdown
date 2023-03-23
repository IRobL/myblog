---
layout: post
title:  "How to measure risk of code changes to software"
date:   2021-09-20 12:00:00 -0500
categories: software
---

Regarding the question "How do you measure risk of source code changes?" I've been wondering myself if there's a formal mechanism for measuring and quantifying risk.  Why quantify?  Let's assume we have a big source code repository that's been around for years.  Not all parts of the code-base are going to be of the same quality.  Some sections of it will be too risky to work on until those risks are mitigated (adding tests, improving logging and customer metrics, etc.).

The three aspects of concern of most interested in are:

- What's the code change like?
- What's the existing code like?
- What's the post/ deployment environment like?

Here's a listing of concerns impacting risk (most borrowed from [Predicting Risk of Software Changes](https://mockus.org/papers/bltj13.pdf))

1. Are there a lot of lines of code added and deleted?
2. Are the changes very diffuse? (e.g. number of files, modules, and subsystems touched)?
3. Are we missing useful automated test that build confidence about the change?
4. Does the team lack an understanding of the code region?
5. Is it difficult to observe whether the change has negatively impacted users of the system?
6. Is it difficult to roll back the change in a timely manor?

With these concerns, an engineer could quantify the risk with a number of 0-6.  If all of these conditions are met, the change is very risky.  If none of the conditions are met, then the change is low risk.  Because an assumption can be made that the code base is non-uniform, this scale can be applied across multiple potential code changes, and the less risky changes can be conducted, where as high risk ones can be abandoned, or plans can be made to mitigate the risks before attempting the code change.

As a quick example, let's say we need to change a program to use a specific value in a field instead of something that has been hard-coded.  The code change is very simple; it's only a one line code change.  But let's say that the code is a mess and has some odd coupling with a feature that was released years ago and no one on the current team even knows how the customers use it.  There are unit tests for the program, but deleting large sections of that code region doesn't break the tests.  Given just that much, we could say that the simple code change is slightly risky, it's a 2. It would be a good idea to invest in writing meaningful tests for the area before continueing. In doing so, an engineer will likely develop an understanding of the code region which will bring the risk to a zero. It's a one-line code change so it really shouldn't be done if it's high risk, just on principle, right?

