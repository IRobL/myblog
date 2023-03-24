---
layout: post
title:  "Containerized CI/ CD Best Practices"
date:   2023-03-03 12:00:00 -0500
categories: software ci cd containerization
---

So first, let's review "what's changed," a brief history on CI/ CD.  

TODO: Fact Check and reference everything, and cleanup into a graphical timeline of CI / CD tools

2001: JSON and the first version of YAML are released.

2002: VMware releases ESX Server 1.5, an awkward, pre-docker, but enterprise-ready isolated execution environment tool suite.  

2005 (Jan): YAML 1.1 releases with Merge Keys which allow YAML to be useful on it's own.

2005: Jenkins/ Hudson CI and CruiseControl are released.

2006: Martin Fowler writes a paper on [Continuous Integration](https://martinfowler.com/articles/continuousIntegration.html)

2006: AWS EC2 is launched.

2009: Heroku is launched as a PaaS with support for Ruby projects

2011: Jenkins open's source fork from Hudson officially takes place.

2012: Travis CI, a yaml based, multi-platform build tool based on becomes 1 year old and becomes slightly popular

2013: Docker is released.

2014: Kubernetes is released.

2015 (Aug): Heroku adds Docker (read "all language") support to its PaaS product.

2015: GitLab CI/ CD initial release similar to GitHub using the Ruby programming language

2016 (~March): GitLab CI/ CD gains job-level configurations for images and variables enabling containerized pipeline definitions.

2016 (April): Jenkins 2.0 releases not using YAML like Travis CI, but using pipeline as code using Groovy-based Jenkinsfiles.  

2016 (Dec): AWS CodeBuild is launched.

2018: GCP Cloud Build is launched with Cloud Builders, containerized build commands that accept configuration in the form of command arguments specified in build yamls.  Build yamls are optional, a default yaml is used if none is present.  

2018 (Sept): Azure DevOps Pipelines is launched.  

2019: Tekton and Drone are released both with a heavy, container-first build task approach.  

2019 (Nov): Github Actions is released.

Ok, by 2015 we finally have YAML, containers/ registries, and Kubernetes.  This marks the easiest time for GitLab (or anyone really) to build a container-first CI/ CD pipeline environment.  It's not immediately clear when the CI/ CD community began using containers to package their pipeline logic, but drone references "Plugins" and describes them as such.

> Plugins are docker containers that encapsulate commands, and can be shared and re-used in your pipeline. Use plugins to build and publish artifacts, send notifications, and more.

Vela uses this plugin terminology too.  Heroku, with its beginnings predating docker, used the term "Build Packs" which strictly transformed software from its source code state and put it into a 'slug' which would be deployed and scaled on AWS, originally as EC2 instances.  Google uses the term "Cloud Builders."


#### Tenants of Well Crafting CI/ CD Pipeline Code

You should develop and maintain your pipeline code with the same long term considerations that you would for "ordinary" application code.  

###### For Your Sake

- *Prefer working Code*: Minimize the need for documentation by codifying as many manual procedures as possible and writing meaningful error messages if a configuration or condition is out of alignment.
- *Document*: Document the development process in the README.md
- *Package*: You should release your pipeline code in packages (containers)
- *Test locally*: You should be able to locally test your pipeline code without much effort
- *Integration test*: You should be able to integration test your pipeline
- *Use CI/ CD*: You should make sure changes to the pipeline code are good before merging them to main (and performing a release).  Feature reviews are not enough to ensure bad releases never happen.  
- *Write skinny yamls and fat containers*: The less logic there is in the individual yaml files, the better.  

###### For Your Consumer's Sake

- *Document*: Document the consumption process in the README.md and link to a working example app.  
- *Log Enough*: You should be able to see helpful log messages so when a build fails it's clear what went wrong
- *Avoid Over-Logging*: You shouldn't show unhelpful log messages.  10mb of log messages is probably too much logging to be useful.  
- *Enable Configuration*: Your pipeline should be configurable for special scenarios
- *Don't Require Configuration*: Your pipeline does not require configuration to work in "many" scenarios
- *Do one thing well, but bundle*: Splitting up concerns is often the best approach to designing well written software, but you may need to deviate from this approach to offer helpful bundles to customers.  

#### Elaboration

###### Write skinny yamls and fat containers
The less code you have in a pipeline yaml, the less copy/ paste a consumer depends on, the less likely project specific updates are needed, and the happier everyone is.  If you can write a default a yaml that is just 2 lines of code (the name of a job/ stage and a docker image to run) you've achieved the ultimate victory.  Likely you'll need more boilerplate that just that, but none of the surrounding boilerplate yaml stuff needs to be specific to your pipeline code.  

###### Do one thing well, but bundle things where helpful
Often time in development we try to reduce complexity by having our code do one thing and do it well.  In the case of CI/ CD it makes sense to have release notes be a separate plugin from say a deployment plugin.  But it's can sometimes be a value add to bundle multiple disparate e steps into a single plugin, such as bundling together the task of performing docker builds along with the task of publishing the container built, and even performing snyk security validations all from the same plugin.  The rule of thumb is that if it seems like it wouldn't get in your way, and it's order within the pipeline is predicatble, then it's probably safe to bundle and expose options to disable the bundled task if customer's complain that the bundling is undesirable.  

###### Prefer working Code
Documentation has its place, but superfluous degrees of documentation is not appropriate.  The amount of documentation required to use your software can be minimized through 1) Automation of manual tasks, 2) following an intuitive design, and 3) developing informative error messages that guide the users when they misconfigure the tool or use the tool incorrectly.  



References:  
- https://getindata.com/blog/different-generations-of-cicd-tools/
- Azure Pipelines: https://azure.microsoft.com/en-us/blog/announcing-azure-pipelines-with-unlimited-ci-cd-minutes-for-open-source/
- AWS CodeBuild: https://aws.amazon.com/about-aws/whats-new/2016/12/announcing-aws-codebuild/
