[VMware Tanzu Cloud Native Runtimes](https://docs.vmware.com/en/Cloud-Native-Runtimes-for-VMware-Tanzu/1.0/tanzu-cloud-native-runtimes-1-0/GUID-cnr-overview.html) is a serverless solution to running applications on top of Kubernetes. Cloud Native Runtimes is based on the opensource project: [Knative](https://knative.dev).

Cloud Native Runtimes simplifies deploying microservices on Kubernetes. With a single command, you can build services based on your containerized applications without having to learn or build various Kubernetes objects. Cloud Native Runtimes automates the backend objects needed to run microservices on top of Kubernetes.

Knative has two major subprojects: **Serving** is responsible for deploying, upgrading, routing and **Eventing** is responsible for connecting disparate systems.

This workshop is intended for folks who want to learn the fundamental components and capabilities of **Knative Eventing** in more detail. A basic understanding of Kubernetes concepts is recommended to be able to understand the full workshop.

#### What you will do in the lab

* Gain an understanding of Knative Eventing
* Understand CloudEvents and why they are used
* Set up Brokers, Sources, Sinks and Triggers
* View events being sent and received through the Knative Eventing system
