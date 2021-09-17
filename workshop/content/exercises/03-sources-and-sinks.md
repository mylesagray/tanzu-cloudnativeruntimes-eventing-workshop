`Sources` and their slightly less visible companion, `Sinks`, are the bread-and-butter concepts you use most often in *Knative Eventing*.

Quickly revisiting: a `Source` describes a thing that can emit `CloudEvents` and a place (the `Sink`) where those events should be sent.

`Sources` are the canonical way of describing how `CloudEvents` will move between points in a *Knative Eventing* design. Knative’s structure is that `Sources` are the top-level concept and that `Sinks` are a component of `Sources`.

## Sources

### Source Types

Knative’s design is that any Kubernetes record having the two fields `Sink` and `CloudEventOverrides` can be treated as a `Source` by Knative.

Let’s first have a look at the Sources that are installed by default.

```terminal:execute
command: kn source list-types
clear: true
```

You should see the `ApiServerSource`, `ContainerSource`, and `PingSource`. Additionally there is a `SinkBinding`, which we won't cover here.

#### Listing Source Types

`kn source list-types` shows what `Source` definitions have been installed, while the `kn source list` shows `Source` records that have been created and submitted:

```terminal:execute
command: kn source list
clear: true
```

The `kubectl` counterparts are:

```terminal:execute
command: kubectl get sources
clear: true
```

With `kubectl get crds -l duck.knative.dev/source=true` you are able to get the `CRD`s for the installed `Source` definitions.

#### Creating an Example Source

For working with `ApiServerSource` and `PingSource`, Knative provides some convenient subcommands. First you have to deploy a `broker` and a `sink`.

```terminal:execute
command: kn broker create default
clear: true
```

Instead of the CloudEvents player application, we will use Sockeye which reveals a slightly lower-level view.

```terminal:execute
command: kn service create sockeye --image docker.io/n3wscott/sockeye:v0.7.0
clear: true
```

Now you are able to create the `PingSource` and specify the sink via the `kn` CLI - this targets the sockeye `Service` that we just deployed:

```terminal:execute
command: kn source ping create ping-player --sink $(kn service describe sockeye -o url)
clear: true
```

If you have a look at the details of the created `Source`, you can see the `Schedule` with the rule `* * * * *` that is satisfied every minute.

```terminal:execute
command: kn source ping describe ping-player
clear: true
```

As you would expect, there is also the HTTP(S) URI to which `PingSource` is meant to send the `CloudEvent` and as always, some `Conditions`.

`Data` is the actual JSON sent onwards to the `Sink` - as you can see from the above output, this is currently empty, because we didn't specify any data at the creation of the `PingSource`, let's do this now:

```terminal:execute
command: |-
  kn source ping update ping-player --data '{"message": "Hello world!"}'
clear: true
```

The kubectl/YAML counter-part for our source looks like this:

```editor:append-lines-to-file
file: ~/source.yaml
text: |
      apiVersion: sources.knative.dev/v1beta2
      kind: PingSource
      metadata:
        name: ping-player
      spec:
        data: '{"message": "Hello world!"}'
        schedule: '* * * * *'
        sink:
          uri: http://sockeye.{{ session_namespace }}.{{ ingress_domain }}
```

If you open the application URL you see the incoming events from our `PingSource`, once every minute - on the minute. You can see both the `attributes` (on the left) and `data` (on the right) which are part of the `CloudEvent` spec, as are the attribute fields for `id`, `source`, `time`, and `type`.

## Sinks

### Using URIs or Refs

In previous examples so far we’ve positioned the `Sink` as being a URI.

It turns out that this is only one way to express "send my `CloudEvents` here." The other is to use a `Ref` — a reference to another Kubernetes record.

```terminal:execute
command: kn source ping update ping-player --sink ksvc:sockeye
clear: true
```

The kubectl/YAML counter-part for our source looks like this:

```sh
kubectl apply -f - << EOF
apiVersion: sources.knative.dev/v1beta2
kind: PingSource
metadata:
  name: ping-player
spec:
  data: '{"foo":"and likewise bar"}'
  schedule: '* * * * *'
  sink:
    ref:
      apiVersion: serving.knative.dev/v1
      kind: Service
      name: sockeye
      namespace: {{ session_namespace }}
EOF
```

There’s no `--sink-ref` or `--ref` argument to signal to Knative that you’re using a `Ref` instead of a `URI`. Your intention is derived from the syntax of what you pass in.

If your argument starts with `http://` or `https://`, the assumption is that you want a `URI`. If instead it starts with `ksvc:`, the assumption is that you want a `Ref`.

You could also have used `service:sockeye` as a more explicit alternative. Whichever you pick, just remember that a `Knative Service` is not the same as a `Kubernetes Service`.

### Which is better?

`URI`s are simpler to start with and allow you to target endpoints that live outside the cluster Knative is running on. But consider using `Ref`s instead. A `URI` is just an address, what lies on the other side is a black box.

But a `Ref` is a Knative construct referring to a `Knative Service`. Knative knows how to open the box to peek at what’s inside.

This is most useful when things go awry. Let’s demonstrate by creating some unexpected havoc as uncovered in this listing.

```terminal:execute
command: |-
  kn service delete sockeye
  kn source ping describe ping-player
clear: true
```

The `Ready` Condition is a top-level rollup of other Conditions, the more diagnostic Condition is that `SinkProvided` is `!!` (not OK), the `NotFound` reason explains why.

This information is surfaced because *Knative Eventing* can go and see if the nominated `Ref` actually exists. That’s not something it can do with `URI`.

## Other Sources

In addition to the first-party `Sources` that enable you to kick the Eventing tires immediately after installation, there are a few places to look for more Sources.

- [*Knative Eventing* documentation “Knative Eventing Sources” page](https://knative.dev/docs/eventing/sources/)
- [Knative’s own eventing-contrib repository](https://github.com/knative/eventing-contrib)
- [TriggerMesh provides a growing collection of `Sources` for AWS](https://github.com/triggermesh/aws-event-sources).
- [Google first-party Sources](https://github.com/google/knative-gcp)
- [VMware has an experimental Source for vSphere and is working on others like for RabbitMQ](https://github.com/vmware-tanzu/sources-for-knative)

To clean up the environment run:

```terminal:execute
command: |-
  kn broker delete default
  kn source ping delete ping-player
clear: true
```
