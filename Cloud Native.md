# Introduction to Cloud Native Architecture

[Will Schipp](https://enablement-bible.cfapps.io/cloud-native-intro.html)

*   Note that some slides ask you to press the `Down` arrow. **To see a map of the slides, press `Esc`.**
    

## Agenda

*   What is Cloud Native Architecture?
    
*   What’s _Good_ about Cloud Native Architecture?
    
*   What’s _Challenging_ about Cloud Native Architecture?
    
*   What are the Alternatives?
    
*   Things to Consider
    
*   Summary
    

## What is Cloud Native Architecture?

Cloud Native Architecture covers a _huge_ array of topics including, but not limited to;

*   Microservice Architecture
    
*   Container-based Application Design
    
*   Cloud Native Data
    
*   Orchestration
    
*   …​ and more
    

![cloud native architecture](./images/cloud_native_architecture.png)

## What’s _Good_ About Cloud Native Architecture?

## Focus

*   Applications are focused on _real_ customer needs
    
*   Changes are made to only what _matters_ to get the job done
    

## Iterative Without Consequence

*   Applications can be built independent from each other
    
*   With independence allows for inconsequential changes\*
    

\\\* if done right!

## Traceable

*   Containerization aspects of Cloud Native Architectures support traceability and accountability
    

## Choice

*   Cloud Native Architectures support PolyGlot development, allowing you to pick whatever language works, and _still_ enjoy the benefits
    

## What’s _Challenging_ About Cloud Native Architecture?

## _Lots_ of Moving Parts

*   Cloud Native Architecture means there are lots of independent services that are maintained and managed separately to deliver a single use case
    
*   Architectures mean teams need to agree on a _contract_ only, not development
    
*   You can be tempted to reuse DTO’s (DON’T!)
    
*   You need to stand up services to support
    

## What are the Alternatives?

## Monolith

![swiss army knife](./images/swiss_army_knife.jpg)

*   The Swiss Army Knife App
    

## Serverless

*   Microservices split down to the minutiae
    

## Things to Consider

## First Hint; Ignore the 'Native' Part

*   Lots of first adopters built directly for the cloud _vendor_
    
*   This meant that their application architecture was heavily influenced by the _vendor_
    
*   _Vendor_ specific approaches create tightly coupled architectures
    

## Your Cloud Should Run _Anywhere_

*   Make sure your services are capable of running on a disconnected environment
    
*   Your services _can_ have dependencies on other services, but they should at least tell you _why_
    

## \[Opinionated\] Use Spring Cloud

*   Part of the Spring Project, Spring Cloud helps _massively_ when orchestrating and deploying cloud
    

## Build Small

*   Use Spring Boot to build small, self-contained services, then add the Cloud Management components as you grow
    

## Summary

*   Cloud Native Architectures allow you to build small, concise services that are self-contained
    
*   When building, start small and add management as you grow
    
*   Services _can_ be dependent on other services, but _must_ tell you why and what’s wrong if their dependencies are missing
    
*   Cloud Native Architectures allow you to build _what_ you need, and maintain _that_ service, _when you_ need to - Monoliths DO NOT
    

Thanks for reading!