Cloud Native Runtimes is a serverless solution to running applications on top of Kubernetes. Cloud Native Runtimes is based on the opensource project: Knative [https://knative.dev]

Cloud Native Runtimes simplifies deploying microservices on Kubernetes. With a single command, you can build services based on your containerized applications without having to learn or build various Kubernetes objects. Cloud Native Runtimes automates the backend objects needed to run microservices on top of Kubernetes.

Knative has two major subprojects: **Serving** is responsible for deploying, upgrading, routing and **Eventing** is responsible for connecting disparate systems.

This workshop is intended for folks who want to learn the fundamental components and capabilities of **Knative Eventing** in more detail. A basic understanding of Kubernetes concepts is recommended to be able to understand the full workshop.

#### What you will do in the lab

* Deploy an app to Kubernetes with Cloud Native Runtimes and * understand what you did
* install app v2 of your application and see how to manage each * version
* Update the service such that incoming traffic is split between * both apps by 50%-50%
* Observe the traffic being split between your applications
* Next, move all the traffic coming to the new app version 2
