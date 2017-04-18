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

We leverage DevOps to programatically develop and deploy our application. DevOps solutions vary in upfront work, customizability, management of services, scalability (both horizontal and vertical), programmability, automation, security, and cost. We will be focusing on two solutions: Heroku and AWS. 

# Devops with [Heroku](https://www.heroku.com)
Heroku is a cloud platform based on a managed container system, with integrated data services and a powerful ecosystem, for deploying and running modern apps. Its simplicity and abstraction of internal system architecture is what is attractive about this platform for MVP (Minimum Viable Product) / POC (Point of Concept) applications. 

# Devops with AWS Instructions
Let's say we wish to deploy our Flask application leveraging Git source control. 

We’ll perform each step and then automate the action we’ve just done. 

You could skip this process with something like Heroku but it’s helpful to understand as much of your application stack as you can. It’s useful knowledge when something goes wrong.

Your general workflow should be as follows:

* Get the application running on our local machine
* Setup the application on a virtual machine with Vagrant
* Automate setting up the application in Vagrant with Ansible
* Manually set up our Amazon instance and deploy our application to it
* Automate the Amazon setup with Terraform
* Deploy to Amazon with Ansible

#### Application working on your local machine
Your application should already be functional locally. 

#### Setup with [Vagrant](https://www.vagrantup.com)
Vagrant is a tool for building and managing virtual machine environments in a single workflow. It provides easy to configure, reproducible, and portable work environments built on top of industry-standard technology and controlled by a single consistent workflow to help maximize the productivity and flexibility of you and your team. Similair to virtualenv, Vagrant will isolate dependencies and their configuration within a single disposable, consistent environment, without sacrificing any of the tools you are used to working with (editors, browsers, debuggers, etc.). Once you or someone else creates a single `Vagrantfile`, you just need to `vagrant up` and everything is installed and configured for you to work. 

Example: let’s say we want to make it so anyone can develop on our application. Keeping our code on our local machine works fine for a while. We could pass out the instructions we just gave (not too hard, right?). There’s two main challenges that you’ll encounter:

We’ll need to replicate what it’s like to set up a new server. If you’re using your local machine, it’s tough to get it to act like a fresh machine.

It’s possible that your local machine is a different type of box than your server. Perhaps you’re working on a Mac but deploying to a Linux box, for example. Maybe you’re deploying to a different version of the operating system or a different operating system all together.

We’ll use Vagrant for our development environment because it allows us to address these challenges. We’ll set up a virtual machine and run it there but still write code locally.

After running `vagrant init` in your Terminal this should set up a basic Vagrant configuration file that we can change to suit our needs.

Next, in your terminal, you can `vagrant up` to download the Ubuntu image that we specified as the config.vm.box value and to create your server as a virtual machine. 
Once it completes, you can run `vagrant ssh.` to put us into the command line of our virtual machine (VM).

For setting up your VM the steps that you tend to run are the following:

* `sudo apt-get update`
: refresh the system’s package cache.
* `sudo apt-get install git python-pip`
: using our operating system’s package manager, install git and python-pip. We’re going to use git to pull down our project and python-pip to install requirements for our project
* `git pull`
: pull our most recent build from source control
* `sudo pip install -r requirements.txt` 
: install the Python package requirements for the project from the requirements.txt file in the project’s directory.
* `python run.py`
: run your application

#### Setup with [Ansible](https://www.ansible.com/)

Let's say that we to automate the steps we followed above. This automation could be done with a tool called Ansible. 

Ansible is an open-source automation engine that automates software provisioning, configuration management, and application deployment. Ansible has two types of servers: controlling machines and nodes. First, there is a single controlling machine which is where orchestration begins. Nodes are managed by a controlling machine over SSH. The controlling machine describes the location of nodes through its inventory. To orchestrate nodes, Ansible deploys modules to nodes over SSH. Modules are temporarily stored in the nodes and communicate with the controlling machine through a JSON protocol over the standard output. Other types of automation engines include: [Chef](https://www.chef.io/chef/), [Puppet](https://puppet.com/), and [CFEngine](https://cfengine.com/). We will be focusing on Ansible for this class.  

To automate the above steps, we will use Ansible to log in to servers that you specify using ssh and run commands on them. 

To do this we would create a .yml file, lets call it site.yml, in the same folder as your Vagrantfile. This will be our Ansible [playbook](http://docs.ansible.com/ansible/playbooks.html), and it will contain our automation steps. `site` implies that this is the only file needed to get a successful version of our site up and running. The .yml extension tells us that it’s a YAML-formatted file (Ansible’s preference).

What we are doing with Ansible in essence can be broken down with the .yml file: This yml file will have the following lines: 

* Human readable name attributed to the overall step. This shows up in the terminal when we run it.
* Specific list of hosts to apply the playbook to
* Certain commands needed to be run by the superuser.
* Variables defined to be used by the rest of the playbook. (Ansible uses a template engine called Jinja2, so we can use these later to prevent repeating ourselves.)
* Tasks directives
* Application specific commands which in our case will have a package manager (apt) install a set of packages (Jinja will replace the item variable (indicated by the braces in Jinja) with each of the items in the with_items block right below), run git pull to get the most recent build of our application, and run pip install to install the Python requirements from the requirements.txt file.

Now that we have the playbook defined with the .yml file, you will need to tell your VM to use it when setting itself up. As such, you would uncomment, in your Vagrantfile, the bottom-most section on provisioning and change it to look like this:

```python
  config.vm.provision 'ansible' do |ansible|
    ansible.playbook = 'site.yml'
    ansible.verbose = 'v'
  end
```
We’re telling Vagrant to use the site.yml file we created and to use verbose output. Next, you should reprovision your host. This will run all the commands we defined in the playbook on it. `vagrant provision` should take care of it.

You now should be able to `vagrant ssh` into our VM and run the following to get the server up and going with just the following line:

``` bash
python run.py
```

#### Ansible

# Docker

# Development and Layer Seperation

#### App-Layer

#### Cache-Layers

#### DB-Layers

#### Load-Balancing Layers

