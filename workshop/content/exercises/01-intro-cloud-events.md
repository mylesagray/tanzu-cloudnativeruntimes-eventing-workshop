The underlying model of Eventing is this:

Things happen in the world, some of these happenings are observed, some of these observations are transmitted from place to place. Receivers of the transmission read the observation, deduce some state of the world, then perhaps take actions. These actions perhaps cause still other observations to arise.

*Knative Eventing* is for the bit in the middle. It doesn’t take action, it doesn’t observe the happenings. But it does _take observations_ and _transmit them_ those.

The format used for this purpose is `CloudEvents`, the mechanisms are **Channels** and **Filters**, **Brokers** and **Triggers**, **Sources** and **Sinks**, **Sequences** and **Parallels**. We will only cover a few of the basics in this workshop to get you started.

## `CloudEvents`

Events are everywhere. However, event producers (Sources) tend to describe and format events in different ways.

The lack of a common way of describing/formatting events means developers are constantly re-learning how to consume events. This also limits the potential for libraries, tooling and infrastructure to aid the delivery of event data across environments, like SDKs, event routers or tracing systems.

As a result, the portability and productivity that can be achieved from event data is hindered overall.

`CloudEvents` is a specification for describing event data in common formats to provide interoperability across services, platforms and systems.

`CloudEvents` have a two-part structure with `data` and `attributes`, `data` is what you probably guessed: the part that systems add their payloads to. What is of more interest to us in the context of `knative-eventing` is the `attributes`.

These are roughly analogous to HTTP headers. Like HTTP headers, there is potentially an unlimited number, because anyone can add their own. But only a handful are standardized, which makes the job a bit easier.

### Content Modes

The `CloudEvents` specification defines two content modes for transferring events from a source to a destination: **structured** and **binary**.

With the structured content mode the event is fully encoded using a stand-alone event format and stored in the message body.

```json
{
    "specversion" : "1.0",
    "type" : "com.example.type",
    "source" : "/example/source",
    "id" : "82C32673-0C78",
    "time" : "2020-04-10T01:00:05+00:00",
    "datacontenttype" : "application/json",
    "data" : {
        "foo": "and likewise bar"
    }
}
```

For an event in binary content mode the event data is stored in the message body, and event attributes are stored as part of message meta-data.

```json
POST /example/event HTTP/1.1
Host: example.com
Content-Type: application/cloudevents+json
ce-specversion: 1.0
ce-source: /example/source
ce-id: 82C32673-0C78
ce-time: 2020-04-10T01:00:05+00:00
 
{
    "foo": "and likewise bar"
}
```
