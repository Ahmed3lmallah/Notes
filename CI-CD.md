# CI/CD

**Reference:** [CI/CD Walkthrough](https://enablement-bible.cfapps.io/ci-cd-walkthrough.html)

### Table of Content:

* [What & Why?](#what--why)
	* [What/Why Continuous Integration](#why-continuous-integration)
		* [Merging Often](#merging-often)
		* [Protecting Develop](#protecting-develop)
		* [Automate Integration](#automate-integration)
	* [Why Continuous Delivery/Deployment](#why-use-continuous-deliverydeployment)
		* [Delivery vs Deployment](#delivery-vs-deployment)
	* [What is Continuous Delivery/Deployment](#what-is-continuous-deliverydeployment)
	* [FAQ](#faq)
* [Creating Jenkins Pipelines](#creating-jenkins-pipelines)
	* [What is Jenkins?](#what-is-jenkins)
	* [How to use Jenkins](#how-to-use-jenkins)
			
# What & Why?

## Why Continuous Integration?

Tools like Git allow many pairs of developers to work on different aspects of the same codebase at the same time. However, the longer a pair goes without merging a feature branch back into `develop`, the more likely there will be issues with the eventual merge.

This integration problem is compounded whenever multiple branches are being developed. Many identical three-way merges will be needlessly accomplished over and over again.
    
The difficulty or frustration associated with merging into `develop` often can make people avoid it until an entire feature is complete (a _mega commit_)!
    
**How can we solve, or at least mitigate, this problem?** 

By using _Continuous Integration_ (CI)
    

## What is CI?

Said simply, CI is merging into `develop` (integrating changes) _often_, such as every day or every time **all** of our tests are passing.

### Merging Often

*   While we are working on a development task, there will likely be several times where _our task is not yet complete_, but _all of the tests (old and new) are passing_
    
*   By merging feature branches into `develop` often, we can avoid many of the headaches that would be caused by giant, _mega commit merges_ later on
 
### Protecting Develop

We want to merge often into `develop`, however, how do we ensure that we are not just _piling_ a bunch of incoherent code changes into `develop`?
    
*   We protect the integrity of the `develop` branch by using a 'gated commit' pattern for `develop`
    
*   We do this by preventing changes from being made to `develop` directly
    
    *   Instead, `develop` is changed using 'merge requests' from feature branches, into `develop`
    
### Automate Integration

Automated integration removes the element of human-error from the _execution_ of the actual integration process

#### Pipeline as Code

*   An important aspect to CI is that there is a prescribed process to follow
    
    *   Before we merge any feature branch into `develop`, we run all possible tests (unit, integration, etc.) against the feature branch
        
    *   We want to submit new code to a code analysis service (like Sonar) to keep `develop` clean

This need for an _discrete_ and _repeatable_ validation process leads us to find ways to create _discrete_ and _readable_ integration instructions! This brings us to [Jenkins](./Jenkins.md).

#### Jenkins Pipelines

**Jenkins** is an automation tool that is a very useful CI (as well as Continuous Deliver/Deployment, but we’ll get to that later) service.

**Jenkins** allows us to describe our integration workflow as a [Jenkins](./Jenkins.md) pipeline, which uses declarative language. This pipeline description lives as a source file in the same repository that it describes: pipeline as code.

## Why use Continuous Delivery/Deployment

While Continuous _Integration_ ensures that the `develop` branch is always in a deployable state, Continuous _Delivery_ and Continuous _Deployment_ (CD) adds value to that integration process.  

### Delivery vs Deployment

*   In the context of our pipeline, _delivery_ can be used nearly interchangeably with _deployment_
    
    *   Both are tied to the value that we, as developers, want to create for our customers

The key differences are highlighted here.

*   **Deployment**: when we are directly responsible for taking our executable build artifacts (executable JAR files, etc.) into the production environment (PWS, AWS, etc.)
    
*   **Delivery**: When we are building non executable artifacts (library JAR files, Node packages, etc.) and uploading them to a repository (such as Nexus)

## What is Continuous Delivery/Deployment

### Single-piece Flow

Remember that, as mentioned in the Continuous Integration section above, we want to focus on making many small changes to develop, rather than a few massive changes. CD adds to CI by focusing on the Lean methodology’s 'single-piece flow'.

* Single-piece flow requires that each of those small changes gets built, tested, validated, and deployed or delivered before the next small change starts the same process.

### Single-piece Flow vs Batch

**Why is this serial process important?**

Because if we make even one single mistake with one particular change, all future changes that were made after that will suffer. Not only that, but individual changes get to market sooner by following a single-piece flow.

* **Best Case:** We will finish all five in about the same time using either process, but we will get the first increment deployed sooner using single-piece flow

* **Worst Case:** If we have problems with the earlier deployments, the large batch method suffers the most, because time has already been wasted on building the other deployments with those same problems With the single-piece flow, an issue with the first deployments is solved before that increment is fully deployed, and the future deployments are freed from this problem.

The moment we detect a breaking change has made it into the develop branch, we must stop all subsequent builds and fix the change, because it is very likely that they will all have that same breaking change.
   
## FAQ

**Q**: What does it mean that my feature branch has 'diverged from `develop` '?
    
*   **A**: This just means that at some point after creating your feature branch, new commits were added to `develop` that are consequently absent from your feature branch

**Q**: I want to merge my feature branch into `develop`, but it has diverged from `develop`. What do I do?
    
*   **A**: Your feature branch should be rebased onto the latest version of `develop`.

# Creating Jenkins Pipelines    

## What is Jenkins?

*   [Jenkins](./Jenkins.md) is an open source automation tool written in Java with plugins built for Continuous Integration
    
*   [Jenkins](./Jenkins.md) listens for build triggers which kick off a "pipeline"

### Pipelines

*   Pipelines are automated, programmable sequences of named build stages such as "Run unit tests," "Merge pull request," or "Deploy."
    
    *   They’re kicked off by some build trigger

*   They are configured by a `Jenkinsfile`, a groovy file with no extension in top-level source-control
    
## How to use Jenkins

We typically create 3 pipelines per service:
    
*   **Merge**: Runs tests/verification, then merges your merge request for you
	
*   **Develop**: After merge, runs tests/verification, then deploys to develop
	
*   **Master**: Runs tests/verification, then deploys to production