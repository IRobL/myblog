---
layout: post
title:  "Containerized CI/ CD Best Practices"
date:   2023-03-03 12:00:00 -0500
categories: software ci cd containerization
---

Let's begin with a review on "what's changed," a brief history on CI/ CD.

TODO: Fact Check and reference everything, and cleanup into a graphical timeline of CI / CD tools

- 2001 (March): CruiseControl, a continuous build system with extremely limited XML-based Pipeline as Code functionality is released.
- 2001: JSON and the first version of YAML are released.
- 2002: VMware releases ESX Server 1.5, an awkward, pre-docker, but enterprise-ready isolated execution environment tool suite.
- 2005 (Jan): YAML 1.1 releases with Merge Keys which allow YAML to be useful on it's own.
- 2005: Jenkins/ Hudson CI and CruiseControl are released.
- 2006: Puppet is released at the very dawn of Infrastructure as Code (IaC) movement.
- 2006: Martin Fowler writes a paper on [Continuous Integration](https://martinfowler.com/articles/continuousIntegration.html)
- 2006: AWS EC2 is launched helping materialize the concept of disposable infrastructure introduced by VMs and IaC.
- 2009: Heroku is launched as a PaaS with support for Ruby projects to be deployed on disposable cloud compute resources.
- 2011: Adam Wiggins (Heroku developer) published The Twelve-Factor App documenting a success path towards writing cloud-ready applications.
- 2011: Jenkins open's source fork from Hudson officially takes place.
- 2012 (Feb): Ansible is released around the time when VM usage was at it's peak.
- 2012: Travis CI, a yaml based, multi-platform build tool based on becomes 1 year old and becomes slightly popular
- 2013: Docker is released.
- 2014: Kubernetes is released.
- 2014 (Sept): Drone is released, the first CI/ CD platform that allows you to execute builds within containers.
- 2015 (Aug): Heroku adds Docker (read "all language") support to its PaaS product.
- 2015: GitLab CI/ CD initial release similar to GitHub using the Ruby programming language
- 2016 (~March): GitLab CI/ CD gains job-level configurations for images and variables enabling containerized pipeline definitions.
- 2016 (April): Jenkins 2.0 releases not using YAML like Travis CI, but using pipeline as code using Groovy-based Jenkinsfiles.
- 2016 (Dec): AWS CodeBuild is launched.
- 2018: GCP Cloud Build is launched with Cloud Builders, containerized build commands that accept configuration in the form of command arguments specified in build yamls.  Build yamls are optional, a default yaml is used if none is present.
- 2018 (Sept): Azure DevOps Pipelines is launched.
- 2019: Tekton is released, a way of building CI/ CD purely from K8S and containers.
- 2019 (Nov): Github Actions is released.


TODO: Add declarative vs imparative origins

###### 2009 CI/ CD Snapshot - Building Pipelines from a GUI and Managing Build Slaves

We have Cruise Control and Jenkins and we're beginning to understand the importance of continuously integrating our code, and some of us at this point are to a maturatity with unit and integration testing that we're even employing continuous deployment.

In 2009, we are still using a GUI to define our pipelines.  We haven't truely realized the power of Configuration as Code.  Further we're struggling to manage our Jenkins service.  Jenkins was designed well before the concept of disposable infrastructure was around.  And so we had a lot of **state** mixed up with our **compute environment** making it difficult for us to re-build our CI/ CD system from scratch routinely.

The CI/ CD execution at this time was often on the same VM that the jenkins http service was running from.  To make build tools available, we often were manually SSHing onto the jenkins instance and installing the tools through apt-get commands.  The alternative was pre-allocating either physical servers to be used as "build-slaves" and so we'd SSH onto those to install the software needed during build time if we weren't current with IaC frameworks available at that time (Ansible would not be released for another 3 years).

>>> TODO: Diagram of a ton of tools installed on a single machine.

The drawbacks of this are that, first, not all of these tools played nicely on the same machine.  Second it proved to be an impossible challenge to guess what tools would be required by other teams.  Therefore individual teams were often left with the burden of requisitioning their own build slaves and then installing the tools they needed to build the software they had authored.

Further, keeping this software up-to-date was another challenge challenge altegether which
1) Often involved (at best) cron jobs that only updated software installed through the system's package manager, and
2) Usually didn't even run correctly but it wasn't anyone's job to log in as the root user and check the email to see if the cronjobs were puking error messages every night.


###### CI/ CD in 2013 - Docker, we needed this, but what is this for?

We're starting to see competition emerge and overtake Jenkins.  Jenkins at this time is doubling down on some of it's GUI concepts.  Jenkins Plugins allow you to build your own GUI interface for defining pipeline behavior in Jenkins.  This turns the already difficult to upgrade Jenkins instance into a update hellscape where plugins will suddenly stop working if you updated Jenkins to a version that doesn't support your existing plugin version.

In the world of Travis CI, you're now able to define your OS environment and it's required packages as a YAML file, following Configuration as Code principales.  A disposable VM would be created to meet your IaC configurations allowing your software to be tested in a manor that was actually repeatable.  Needless to say, the popularity of Travis CI exploded rapidly when people realized they could pivot effortlessly off of Jenkins to a SaaS tool that was free for open source and competitive for enterprises squandering millions in Jenkins-related tech debt.

We have containers at this time, but we're still strugling with how to use them to package our applications.  Very few out there are thinking about them as something that can package our CI/ CD patterns, and there still isn't any CI/ CD software around that can execute builds neatly in a containerized environment.


######  CI/ CD in 2015 - YAML + Docker +

By 2015 we finally have YAML, containers/ registries, and Kubernetes.  This marks the easiest time for GitLab (or anyone really) to build a container-first CI/ CD pipeline environment.  It's not immediately clear when the CI/ CD community began using containers to package their pipeline logic, but drone references "Plugins" and describes them as such.

> Plugins are docker containers that encapsulate commands, and can be shared and re-used in your pipeline. Use plugins to build and publish artifacts, send notifications, and more.

Vela uses this plugin terminology too.  Heroku, with its beginnings predating docker, used the term "Build Packs" which strictly transformed software from its source code state and put it into a 'slug' which would be deployed and scaled on AWS, originally as EC2 instances.  Google uses the term "Cloud Builders."

The desire for packaging pipeline code arose when enterprises began codifying their pipelines and tracking them into repositories throughtout the organization.

- Bugs would eventually be discovered in the pipeline code.
- Breaking changes to infrastructure would be made, requiring updates to the pipeline code.
- Preferred tools changed throughout the course of the enterprise lifecycle which meant we had a new problem.
- Multiple competing CI/ CD platforms would coexist within the same enterprise which would require the duplicative effort of building the pipeline in a second and third platform.

How can we manage pipeline code at an enterprise scale?  If we're serving 200 teams, do we just disrupt each of them everytime a change takes place?  Imagine sending the email, "Hey, Please paste this bug fix into your pipeline.  Thanks! -The Management"

The answer to all of this was clear in 2018, use containers to version, package, and distribute your pipeline in modular steps.

#### Tenants of Well Crafting CI/ CD Pipeline Code

You should develop and maintain your pipeline code with the same long term considerations that you would for "ordinary" application code.  This includes making deliberate accomidations for unit (and integration) testing every PR, iterating towards maintainable configuraable feature-rich code, packaging versioned releases, monitoring production usage, and building a positive end-user experience.

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



#### References:
- GetInData Blog on CI/ CD dated 2021: https://getindata.com/blog/different-generations-of-cicd-tools/
- CircleCI Blog Date's CruiseControl to 2001: https://circleci.com/blog/a-brief-history-of-devops-part-iii-automated-testing-and-continuous-integration/
- Azure Pipelines: https://azure.microsoft.com/en-us/blog/announcing-azure-pipelines-with-unlimited-ci-cd-minutes-for-open-source/
- AWS CodeBuild: https://aws.amazon.com/about-aws/whats-new/2016/12/announcing-aws-codebuild/
- Tekton: https://mkdev.me/posts/what-is-tekton-and-how-it-compares-to-jenkins-gitlab-ci-and-other-ci-cd-systems
