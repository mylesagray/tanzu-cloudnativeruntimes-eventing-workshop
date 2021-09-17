# Cloud Native Runtimes Eventing Workshop for Tanzu Developer Portal

For more detailed information on how to create and deploy workshops, consult
the documentation for Educates at:

* <https://docs.edukates.io>

If you already have the Educates operator installed and configured, to
deploy and view this sample workshop, run:

```sh
kubectl apply -f https://raw.githubusercontent.com/mylesagray/tanzu-cloudnativeruntimes-eventing-workshop/master/resources/workshop.yaml
kubectl apply -f https://raw.githubusercontent.com/mylesagray/tanzu-cloudnativeruntimes-eventing-workshop/master/resources/training-portal.yaml
```

This will deploy a training portal hosting just this workshop. To get the
URL for accessing the training portal run:

```sh
kubectl get trainingportal/cnr-eventing-getting-started
```
