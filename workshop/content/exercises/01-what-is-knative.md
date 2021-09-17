***Knative*** brings "serverless" experience to kubernetes. It also tries to codify common patterns and best practices for running applications while hiding away the complexity of doing that on kubernetes.

It does so by providing two sets of components:

- **Serving** - Request-driven compute that can scale to zero
- **Eventing** - Management and delivery of events

At a high level `knative-serving` is an abstraction over the bare k8s **deployment->pod->service->ingress** model. Knative allows you to ask for a `knative-service` with an image at the core. You will be able to provide many configurations that will/can override all the amazing opinionated defaults provided by the abstraction.

By contrast, `knative-eventing` focuses on messaging between the components in `knative-serving`. It enables developers to use an event-driven architecture with serverless applications. An event-driven architecture is based on the concept of decoupled relationships between `event producers` - that create events, and `event consumers` (sinks) - that receive events.

In this Workshop,  we will focus primarily on the `knative-eventing` aspects.
