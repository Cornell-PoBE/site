---
layout: post
---

# (1 Week)

# Introduction to DevOps

DevOps ("software DEVelopment" and "information technology OPerationS") is a term used to refer to a set of practices that allow you as a software engineer to automate the process of software delivery and infrastructure changes. It aims at establishing a culture and environment where building, testing, and releasing software can happen rapidly, frequently, and more reliably. DevOps is the combination of cultural philosophies, practices, and tools that increases an organization’s ability to deliver applications and services at high velocity: evolving and improving products at a faster pace than organizations using traditional software development and infrastructure management processes. This speed enables organizations to better serve their customers and compete more effectively in the market. 

The process can be broken down into 7 different steps:

* Code — Code development and review, version control tools, code merging
* Build — Continuous integration tools, build status
* Test — Test and results determine performance
* Package — Artifact repository, application pre-deployment staging
* Release — Change management, release approvals, release automation
* Configure — Infrastructure configuration and management, Infrastructure as Code tools
* Monitor — Applications performance monitoring, end–user experience


As a whole, here is an overview of important DevOps practices: 

#### Continuous Integration

A software development practice where developers regularly merge their code changes into a central repository, after which automated builds and tests are run. The key goals of continuous integration are to find and address bugs quicker, improve software quality, and reduce the time it takes to validate and release new software updates.

#### Continuous Delivery

A software development practice where code changes are automatically built, tested, and prepared for a release to production. It expands upon continuous integration by deploying all code changes to a testing environment and/or a production environment after the build stage. When continuous delivery is implemented properly, developers will always have a deployment-ready build artifact that has passed through a standardized test process.

#### Microservices

A design approach to build a single application as a set of small services. Each service runs in its own process and communicates with other services through a well-defined interface using a lightweight mechanism, typically an HTTP-based application programming interface (API). Microservices are built around business capabilities; each service is scoped to a single purpose. You can use different frameworks or programming languages to write microservices and deploy them independently, as a single service, or as a group of services.

#### Infrastructure as Code: 

A practice in which infrastructure is provisioned and managed using code and software development techniques, such as version control and continuous integration. The cloud’s API-driven model enables developers and system administrators to interact with infrastructure programmatically, and at scale, instead of needing to manually set up and configure resources. Thus, engineers can interface with infrastructure using code-based tools and treat infrastructure in a manner similar to how they treat application code. Because they are defined by code, infrastructure and servers can quickly be deployed using standardized patterns, updated with the latest patches and versions, or duplicated in repeatable ways.

Tools such as [Docker](https://www.docker.com/) (containerization), [Jenkins](https://jenkins.io/) (continuous integration), [Puppet](https://puppet.com/) (Infrastructure as Code) and [Vagrant](https://www.vagrantup.com/) (virtualization platform)—among many others—are often used and frequently referenced in DevOps discussions. We will be addresing some of these later in the lecture. 

We leverage DevOps to programatically develop and deploy our application. DevOps solutions vary in upfront work, customizability, management of serices, scalability (both horizontal and vertical), programmability, automation, security, and cost. We will be focusing on two solutions: Heroku and AWS. 

# Devops with [Heroku](https://www.heroku.com)
Heroku is a cloud platform based on a managed container system, with integrated data services and a powerful ecosystem, for deploying and running modern apps. Its simplicity and abstraction of internal system architecture is what is attractive about this platform for MVP (Minimum Viable Product) / POC (Point of Concept) applications. 

# Devops with AWS

# Docker

# Development and Layer Seperation

#### App-Layer

#### Cache-Layers

#### DB-Layers

#### Load-Balancing Layers

