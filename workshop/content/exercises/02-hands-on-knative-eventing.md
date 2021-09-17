## Your first experience with *Knative Eventing*

Let's first check the current state using the kn CLI.

```terminal:execute
command: kn trigger list && kn source list && kn broker list
clear: true
```

Under the hood, these commands are groping around for `Trigger`, `Source`, and `Broker` records in Kubernetes. You haven’t done anything yet, so there are none.

A `Trigger` combines information about an event filter and an event subscriber together, a `Source` is a description of something that can create events.

In reality, Triggers don’t really do anything in themselves. They’re records that get acted on by a `Broker`, with potentially many `Triggers` per `Broker`.

A `Subscriber` here is anything that *Knative Eventing* knows how to send stuff to.

Let's start with the creation of a `Broker`:

```terminal:execute
command: kn broker create default
clear: true
```

and continue with a `Subscriber`. The simplest thing to put here is a basic web app that can receive `CloudEvents` and perhaps help you to inspect those.

```terminal:execute
command: kn service create cloudevents-player --image ruromero/cloudevents-player:latest --env BROKER_URL=http://default
clear: true
```

After the deployment of the `Service` there should be a URL to access the application in the logs. If you open the URL, you should see a form with fields for all the required attributes we talked about in the previous section to send an event to the `Broker`.
To also consume the events from the broker with our application and view them in the blank area on the right, we have to create a `Trigger`.

```terminal:execute
command: kn trigger create cloudevents-player --sink cloudevents-player
clear: true
```

If you now head back to your browser and try sending an event. You’ll see that the event is both sent and received.
The simple fact here is that I’ve cheated by making the application both the `Source` and `Sink` for events.
The nomenclature of "sinks" and "sources" is already widespread outside of Knative in lots of contexts. `Sources` are where events come from, `Sinks` are where events go.
You didn’t explicitly define a `Source` and only defined a `Sink` in the `Trigger`. This already hints at how flexible *Knative Eventing* actually is.

Let's have a closer look at our created `Trigger`.

```terminal:execute
command: kn trigger describe cloudevents-player
clear: true
```

As you can see it sets `cloudevents-player` as its `Sink`. Many things can act as a `Sink` without knowing it and they don't have to be Knative Services as you will see at the end of the chapter.

Like with the *Knative Serving* Kubernetes records, there are also *`Conditions`*.

Right now the conditions are all `++`, indicating a state of swellness and general good humor. As with other Knative conditions, `Ready` is the logical `AND` of all the others. If any other condition isn’t `OK`, then `Ready` will not be `++`.

Let's look a little closer at the other conditions:

- *`BrokerReady`* This signals that the `Broker` is ready to act on the `Trigger`. When this is false, it means that the `Broker` can’t receive, filter, or send events
- *`DependencyReady`* Because you defined the `CloudEvents` player service as both `Source` and `Sink`, this doesn’t have a deep meaning. This is really meant to tell you how a standalone source, such as PingSource, is doing. It gives you a fighting chance of guessing whether stuff broke because you broke it or whether some external service broke it instead
- *`SubscriberReady`* This is where the naming of things in *Knative Eventing* starts to get squiggly. The practical point here is that `SubscriberReady` is about whether the `Sink` is `OK`. It’s not a reference to the `Subscriber` field of a `Trigger`.
- *`SubscriberResolved`* `BrokerReady`, `DependencyReady`, and `SubscriberReady` are all about the status of software that’s running someplace else. 

That means it can change from time to time.

But `SubscriberResolved` is a once off. It refers to the business of resolving the subscriber (turning the Sink into an actual URL that can actually be reached). That happens right after you submit a `Trigger`. *Knative Eventing* picks up the `Sink` and shakes it to see what’s inside. It might be a directly hard-coded URL that you gave. It might be a Knative service. But it can also be a number of different things, such as a Kubernetes `Deployment`. All of these have some differences in how you get a fully resolved URL, but a fully resolved URL is what the Broker needs to do its job of squirting `CloudEvents` over HTTP.

When it’s `OK`, `SubscriberResolved` can be ignored. But if it’s false, then your `Trigger` is wedged. You won’t fix it directly by tinkering with the `Broker`, `Source` (aka dependency), or `Sink` (aka subscriber). To be sure, if those are misbehaving, fix these first. But you will still need to update the `Trigger` in order to get another go at URL resolving.

The conditions you saw do go beyond a game of true-or-false, so each of these conditions has relatives for things that are broken.

- *`BrokerDoesNotExist`* This appears when there is no `Broker`. It’s possible for platform operators to configure `Brokers` to be injected automatically, but it’s not the default behavior. In any case, if you see this, it means you need to use `kn broker create´
- *`BrokerNotConfigured`*, *`DependencyNotConfigured`*, and *`SubscriberNotConfigured`* These appear when you first create a `Trigger`. These represent that control loops take time to swirl: upon submitting records, it takes time to reconcile the desired world with the actual world. A lot of the time these will be the kind of thing you can miss if you blink. But if these hang around, something is wrong
- *`BrokerUnknown`*, *`DependencyUnknown`*, and *`SubscriberUnknown`* By convention, an `-Unknown` condition literally means that Knative has no idea what’s happening. The sister condition (e.g., `BrokerReady`) isn’t true, it isn’t false, it isn’t in the middle of being configured. It’s just ... unknown.

On the downside, if you see this, something weird is going on. On the upside, Knative at least has the manners to capture some details and log those.

To clean up the environment for the next section run:

```terminal:execute
command: |-
    kn broker delete default
    kn service delete cloudevents-player
    kn trigger delete cloudevents-player
clear: true
```
