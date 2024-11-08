---
title: Events bus
date: 2024-11-08T08:20:00Z
draft: false
cascade:
  type: docs
---

The system relies on [Kombu](https://github.com/celery/kombu) messaging library which is the underlying system to handle message passing in celery.

It aims to keep it simple, stupid by providing an event producer that will publish new message in `events` or `logs` queues and consumers that will call relevant tasks of type EventTask or LogTask accordingly. 

## Messages

Messages format are validated by relying on pydantic models defined in `core.events.message.py`:

```python
class MessageType(str, Enum):
    log = "log"
    event = "event"

class EventType(str, Enum):
    new = "new"
    update = "update"
    delete = "delete"

class AbstractEvent(BaseModel, abc.ABC):
    def match(self, acts_on: Pattern) -> bool:
        raise NotImplementedError

class ObjectEvent(AbstractEvent):
    type: EventType
    yeti_object: YetiObjectTypes
    [...]

class LinkEvent(AbstractEvent):
    type: EventType
    source_object: YetiObjectTypes
    target_object: YetiObjectTypes
    relationship: "graph.Relationship"
    [...]
    
class TagEvent(AbstractEvent):
    type: EventType
    tagged_object: YetiObjectTypes
    tag_object: "tag.Tag"
    [...]

class AbstractMessage(BaseModel, abc.ABC):
    type: MessageType
    timestamp: datetime.datetime = Field(default_factory=now)

class LogMessage(AbstractMessage):
    type: MessageType = MessageType.log
    log: str | dict

EventTypes = Union[ObjectEvent, LinkEvent, TagEvent]

class EventMessage(AbstractMessage):
    type: MessageType = MessageType.event
    event: EventTypes
```

`YetiObjectTypes` supports all defined schemas in Yeti (even private ones). For example, with `ObjectEvent` message data, `yeti_object` could be a campaign (entity), an ipv4 (observable), a regex (indicator), ...  

## Producer 

To publish an event message, producer must be called with `publish_event` method with the event message argument that can be of type `ObjectEvent`, `LinkEvent` or `TagLinkEvent`:

```python
from core.events.producer import producer

producer.publish_event(event: EventTypes)
```

To publish a log message, producer must be called with `publish_log` method which supports either `str` or `dict` as arguments:

```python
from core.events.producer import producer

producer.publish_log(log: str | dict)
```

The call to publish method is non-blocking and just sends the message to the configured queue and exchange, events and logs respectively.

### Queues management

When there's no events or logs consumers running, Redis queue are growing infinitely and it will lead to OOM kill of the service.

Instead of relying on queue length, producer relies on memory usage of the queue. If the memory usage is greater than a configured threshold, the queue will be trimmed to keep the most recent `keep_ratio` messages.

#### Memory limit

Memory limit is used as a threshold to trim the size of the message queue. It is configured with `memory_limit` key under `[events]` section. If not configured, it fallbacks to 64 MiB. If configured lower than 64MiB, it fallbacks to 64 MiB.

{{< callout type="info" >}}
This memory limit should be below the actual memory of your redis service since redis is also used by celery to run and schedule classical tasks. For example, if your redis service has 128MiB of memory, you could set `memory_limit` to 96.  
{{< /callout >}}

#### Keep ratio

When memory limit is reached, message queue is trimmed to remove oldest message. To define how much messages must be kept in the queue, `keep_ratio` is used. It is defined as a float greater than 0 and lesser than 1. It is configured with `keep_ratio` key under `[events]` section. If not configured, it fallbacks to `0.9` meaning that 10% of the messages will be removed from the oldest messages in the queue. If configured `keep_ratio` is lte 0 or `keep_ratio` is gte 1, it fallbacks to `0.9`.

## Consumers

In order to receive messages and process them, consumers must be created by defining from which queue they must receive the messages.

To handle events messages:

```python
python -m core.events.consumers events
```

To handle logs messages:

```python
python -m core.events.consumers logs
```

By default, the consumer will spawn several `multiprocessing.Process` based on the number of available CPU by using `multiprocessing.cpu_count()`. 

If the number of spawned processes must be changed, `--concurrency` argument can be used, followed by the number of processes to spawn.

### Events Message routing

Plugins of type `EventTask` can define `acts_on` attribute which represents a regex to match an event. For every event message, `EventWorker` calls `match` method implemented by `EventTypes` classes with `acts_on`.

As example, the following `acts_on` can be set to precisely define when a task must be called on events messages:

* Global events -> ObjectEvent, LinkEvent, TagEvent
   * called on all events: `""`
   * called on all `new` events: `"new"`
   * called on all new or update events: `"(new|update)"`

* Yeti objects specialisation -> ObjectEvent
   * called on all events related to observables or entity objects: `"(new|update|delete):(observable|entity)"`
   * called on all events related to observables: `"(new|update|delete):observable"`
   * called on all events related to url: `"(new|update|delete):observable:url"`
   * called on all events related to ipv4 and ipv6: `"(new|update|delete):observable:(ipv4|ipv6)"`
   * called on all new campaign or vulnerability: `"new:entity:(campaign|vulnerability)"`
   * called on tag creation, update or deletion: `"(new|update|delete):tag"`

* Link events specialisation -> LinkEvent
   * called on all events related to links: `"(new|update|delete):link"`
   * called on all events related to links having an observable as source: `"(new|update|delete):link:source:observable"`
   * called on all events related to links having an observable as target: `"(new|update|delete):link:target:observable"`

* Tagged event specialisation -> TagEvent
   * called on all events related to tag links: "(new|update|delete):tagged"
   * called on all events related when an object is tagged with malware or c2: `"(new|update|delete):tagged:(malware|c2)"`