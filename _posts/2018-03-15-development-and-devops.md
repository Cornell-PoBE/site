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

Tools such as [Docker](https://www.docker.com/) (containerization), [Jenkins](https://jenkins.io/) (continuous integration), [Puppet](https://puppet.com/) (Infrastructure as Code) and [Vagrant](https://www.vagrantup.com/) (virtualization platform)—among many others—are often used and frequently referenced in DevOps discussions. We will be addressing some of these later in the lecture.

We leverage DevOps to programmatically develop and deploy our application. DevOps solutions vary in upfront work, customizability, management of services, scalability (both horizontal and vertical), programmability, automation, security, and cost. We will be focusing on two solutions: Heroku and AWS.

# Devops with [Heroku](https://www.heroku.com)
Heroku is a cloud platform based on a managed container system, with integrated data services and a powerful ecosystem, for deploying and running modern apps. Its simplicity and abstraction of internal system architecture is what is attractive about this platform for MVP (Minimum Viable Product) / POC (Point of Concept) applications.
Heroku runs applications inside dynos — smart containers on a reliable, fully managed runtime environment. Developers deploy their code written in any language that they prefer to a build system which produces an app that's ready for execution. The system and language stacks are monitored, patched, and upgraded, so it's always ready and up-to-date. The runtime keeps apps running without any manual intervention. Heroku also prioritizes software delivery so developers can focus on creating and continuously delivering applications, without being distracted by servers or infrastructure. Developers deploy directly from popular tools like Git, GitHub or Continuous Integration (CI) systems. The intuitive web-based Heroku Dashboard makes it easy to manage your app and gain greater visibility into performance.

As such, Heroku's edge over other frameworks is the intuitive and easy-to-manage deployment service it provides alongside popular tools like Github and various CI systems.


# Devops with AWS Instructions
Let's say we wish to deploy a generic Django/Flask application leveraging Git source control.

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
Vagrant is a tool for building and managing virtual machine environments in a single workflow. It provides easy to configure, reproducible, and portable work environments built on top of industry-standard technology and controlled by a single consistent workflow to help maximize the productivity and flexibility of you and your team. Similar to virtualenv, Vagrant will isolate dependencies and their configuration within a single disposable, consistent environment, without sacrificing any of the tools you are used to working with (editors, browsers, debuggers, etc.). Once you or someone else creates a single `Vagrantfile`, you just need to `vagrant up` and everything is installed and configured for you to work.

Example: let’s say we want to make it so anyone can develop on our application. Keeping our code on our local machine works fine for a while. We could pass out the instructions we just gave (not too hard, right?). There’s two main challenges that you’ll encounter:

We’ll need to replicate what it’s like to set up a new server. If you’re using your local machine, it’s tough to get it to act like a fresh machine.

It’s possible that your local machine is a different type of box than your server. Perhaps you’re working on a Mac but deploying to a Linux box, for example. Maybe you’re deploying to a different version of the operating system or a different operating system all together.

We’ll use Vagrant for our development environment because it allows us to address these challenges. We’ll set up a virtual machine and run it there but still write code locally.

After running `vagrant init` in your Terminal this should set up a basic Vagrant configuration file that we can change to suit our needs.

Next, in your terminal, you can `vagrant up` to download an Operating System image that we specified as the config.vm.box value and to create your server as a virtual machine.
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

#### Looking into [Docker](https://www.docker.com)
Next step is to take a look at Docker. Docker is the world’s leading software container platform. Developers use Docker to eliminate “works on my machine” problems when collaborating on code with co-workers. Operators use Docker to run and manage apps side-by-side in isolated containers to get better compute density. Enterprises use Docker to build agile software delivery pipelines to ship new features faster, more securely and with confidence for both Linux and Windows Server apps. A [container](https://www.docker.com/what-container) image is a lightweight, stand-alone, executable package of a piece of software that includes everything needed to run it: code, runtime, system tools, system libraries, settings. Available for both Linux and Windows based apps, containerized software will always run the same, regardless of the environment. Containers isolate software from its surroundings, for example differences between development and staging environments and help reduce conflicts between teams running different software on the same infrastructure.

Docker is the most popular file format for Linux-based container development and deployments. It also provides a container-specific toolset that enables you to create and deploy container images to a cloud-based container hosting environment. This can work great for brand-new environments, but it can be a challenge to mix container tooling with the systems and tools you need to manage your traditional IT environments. And, if you’re deploying your containers locally, you still need to manage the underlying infrastructure and environment.

Ansible interacts well with Docker as it functions as a way to automate Docker in your environment. Ansible enables you to operationalize your Docker container build and deployment process in ways that you’re likely doing manually today, or not doing at all.
When you automate your Docker tooling with Ansible, you gain three key things:

* Flexilibity: Ansible Playbooks are portable. If you build a container with a pure Dockerfile, it means that the only way you can reproduce that application is in a Docker container. If you build a container with an Ansible Playbook, you can then reproduce your environment in Docker, in Vagrant, on a cloud instance of your choice, or on bare metal. Plus, you can build your containers up using Ansible Roles, so that even complex container descriptions are easily read, and different container roles can be reused across many environments.

* Auditability: Ansible Playbooks are repeatable and auditable. Just because containers provide some isolation doesn't alleviate the IT department of the burden of knowing what their containers are based on, tracking any potential vulnerabilities within, and knowing with certainty that you can rebuild a container if needed. By using Ansible Playbooks to build containers, you gain the advantages of a simple, repeatable, defined state of your containers that you can easily track. Combine this with Ansible Tower and you can easily track who deployed what container, with what code, where and when.

* Ubiquity: Ansible can manage full environments. With Ansible, you can manage not only the containers, but the environments around the containers. Docker instances still need to run on hosts, and those hosts need to be launched, configured, networked, and coordinated, whether they be local machines or full cloud infrastructures.

Ansible can model containers and non-containers at the same time. This is especially important, as containerized applications are nearly always talking to components — storage, database, networking — that are not containerized, and frequently not even containerizable. And with Ansible Tower, you can deploy your host environments, your containers, and your services with the push of a button.

# Scalability and Load Balancing
If things work out as you’ve envisioned, there will be a time in your application's lifecycle when it’s serving a large number of users. By the time things get to this point, it’s ideal if you’ve architected your application to both scale gracefully to meet this load, and also be resilient to arbitrary failures of underlying compute resources. In this section we will be looking at leveraging Docker containers and Kubernetes to help your example Django/Flask app achieve these architecture goals.

Let’s imagine that you’re working on a Django web application that’s laid out in a fairly standard fashion, that mimics the Flask app you have implemented in A2: All your app’s data resides in a `PostgreSQL` database. The app itself is written in Django-flavoured Python, and is served up using the `Gunicorn` application server. And in front of all this, you have the `NGINX` web server acting both as a reverse proxy and a static content server to serve your static files.

Here is the example layout:
![Standard Django Application](https://harishnarayanan.org/images/writing/kubernetes-django/standard-django-application.svg)

When you’re first starting out with your app and you only have a handful of users, it makes perfect sense to run all these pieces on a single server. In A3 you will be deploying A2 as it was explained above, by using Ansible and Vagrant to host all of the following pieces on a single EC2 server:

![All-In-One-Server](https://harishnarayanan.org/images/writing/kubernetes-django/all-in-one-server.svg)

However if your app begins to get more and more popular you start to begin focusing on scaling. At first, you follow the straightforward approach and simply provision larger and larger single machines to run your app on. This is called vertical scaling and works well until you reach a few thousand users.

And then your app gets even more popular.

At this point in time it is wise to split the components making up your app and put them on separate machines, so that you can scale the components independently. Meaning, for example, that you can run multiple instances of your Django app (called horizontal scaling) to handle your growing user base, while continuing to run your PostgreSQL server on only one (but potentially increasingly powerful) machine.

![Running-may-instances](https://harishnarayanan.org/images/writing/kubernetes-django/on-separate-servers.svg)


This solution is stable and could be automated with Ansible even, but it comes with its own set of inconveniences:

* Ansible will make it difficult to provision, setup and keep up-to-date one server for each component. This is not the level at which you want to be thinking about the system.

* You will have poor resource utilization because each component doesn’t effectively use all the resources that the respective server has to offer. This is primarily because you’re often setup for peak load, not median load.

* In trying to solve the resource utilization problem, you try to run multiple components (e.g. both your app and database) on the same machine, which causes poor resource isolation within a given server because there’s nothing stopping one piece from clobbering the others

The solution, therefore, would be to shift our attention and focus from managing servers and resources to simply running the components of our app on a collection of computing resources. The intention would be that these components would be well isolated from one another and efficiently using their respective resources.

Our deployment architecture would then change to the following:

![Cluster-Management](https://harishnarayanan.org/images/writing/kubernetes-django/scheduled-on-cluster.svg)

where the primary pieces we care about (the application components) are shown in orange. The actual nodes (physical or virtual machines) that the components run on are de-emphasized visually because we don’t care about the details. And we trust our underlying compute infrastructure to offer us some primitives such as persistent storage and load balancers (shown in green) that are common to any non-trivial web application.

##### Note: In industry, managing your database on the same compute cluster is sometimes leveraged to achieve rack(data) locality, where your data and data-science workloads live on the same node, however it is not necessary, and sometimes not encouraged, as the database tend to be on a remote cluster that you can TCP into. These design decisions are made by database administrators who wish to have remote DMZ clusters separate from a data-science computer cluster. As such, for your solutions you can leave your AWS cluster on-prem or, for example, on Amazon RDS, and still leverage a compute cluster for your application.

This philosophical shift — from managing servers to simply running components of our app ideally — is precisely the promise of container technology like Docker and cluster orchestration frameworks like Kubernetes for these tools allow us to easily recreate the ideal deployment scenario shown above.

#### Looking into [Kubernetes](https://kubernetes.io/)
Kubernetes is an open source system for managing clusters and deploying containerized applications, like Docker. Kubernetes abstracts the underlying hardware (of your cloud provider or on-prem cluster), and presents a simple API that allows you to easily control and interface with. You send this API some declarative state, e.g. “I’d like three copies of my Django app container running behind a load balancer, please,” and it ensures that the appropriate containers are scheduled on the nodes of your cluster. Furthermore, it monitors the situation and ensures that this state is maintained, allowing it to be robust to arbitrary changes in the system. This means, for example, that if a container is shut down prematurely because a node runs out of memory, Kubernetes will notice this and ensure another copy is restarted elsewhere.

Kubernetes works by having agents that sit on each node of your cluster. These allow for things like running Docker containers (the Docker daemon), making sure the desired state is maintained (the kubelet), and that the containers can talk to each other (kube-proxy). These agents listen to and synchronize with a centralized API server to ensure that the system is in the desired state.

![Kubernetes-Architecture](https://harishnarayanan.org/images/writing/kubernetes-django/kubernetes-architecture.svg)

The Kubernetes API exposes a collection of cluster configuration resources that we can modify to express the state we want our cluster to be in. The API offers a standard REST interface, allowing us to interact with it in a multitude of ways. `kubectl` is a thin command-line client used to communicate with the API server.

While the API offers numerous primitives to work with, here are a few that are important to conceptually understand if you wish to leverage Kubernetes in your workloads:

* `Pods`: a collection of closely coupled containers that are scheduled together on the same node, allowing them to share volumes and a local network. They are the smallest units that can be deployed within a Kubernetes cluster.

* `Labels`: arbitrary key/value pairs (e.g. name: app or stage: production) associated with Kubernetes resources. They allow for an easy way to select and organize sets of resources.

* `Replication Controllers`: insurances that a specified number of pods (of a specific kind) are running at any given time. They group pods via labels.

* `Services`: offering of a logical grouping of pods that perform the same function. By providing a persistent name, IP address or port for this set of pods, they offer service discovery and load balancing.

# Wrapup:
As you can see there are a ton of tools, systems, and designs that enable you to have Continuous Integration, Continuous Delivery, Microservices, IAAC, Seamless Deployment, and Scalability within your application. It is your choice of how deep you want to get into the actual customization of your deployment strategy and that choice allows for you to explore or narrow done your options of DevOp tools.
