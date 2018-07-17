---
layout: post
title:  "Setting up a Java Dev Environment with IntelliJ"
date:   2018-07-17 09:05:28 -0500
categories: java env-setup
---

The great thing about 'light weight' languages is that they usually don't require (or even suggest) the use of an IDE that helps complete all of the extra typing required of heavier weight languages such as C# and Java.  The purpose of this post is to document the steps required to get going with a dev environment (spoiler alert: it's more complicated than just installing a couple packages).


###### Download and Install the JDK
[Reference](https://www.lonecpluspluscoder.com/2017/04/27/installing-java-8-jdk-os-x-using-homebrew/)

    brew install java

###### Download and Install IntelliJ CE
[Link](https://www.jetbrains.com/idea/download/)


###### Open your project from the terminal

    cd MyJavaProject
    idea .

###### Add the new SDK, JDK to IntelliJ

[Link](https://stackoverflow.com/questions/13078898/setting-up-jdk-7-for-intellij-on-the-mac)

###### Ensure Java version under 'project structure' is set correctly

[Reference](https://stackoverflow.com/questions/46280859/intellij-idea-error-java-invalid-source-release-1-9)

If Java 1.8, use '8' as the dropdown menu, not 9....


###### Add Gradle?

[Reference](https://stackoverflow.com/questions/26745541/best-way-to-add-gradle-support-to-intellij-project)

    gradle cleanIdea
    gradle idea

###### Setup the play button

In the project explorer, expand

    src -> main -> java -> project ID -> Application -> Run 'Application Main()'

From there on, the 'Play' and 'Debug' buttons in the IDE will play that application.

###### Create Debug Points

Click in a special spot in the middle of the gutter to create a breakpoint.  Execution will pause there when using the 'Debug' button.



