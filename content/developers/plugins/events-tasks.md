---
title: Events tasks
date: 2024-11-08T08:20:00Z
draft: false
cascade:
  type: docs
---

In order to create task that will be triggered based on events, the following example is provided:

```python
from urllib.parse import urlparse

from core import taskmanager
from core.events.message import Message
from core.schemas import observable, task

class HostnameExtract(task.EventTask):
    _defaults = {
        "name": "HostnameExtact",
        "description": "Extract hostname (domain or ip) from new URL observable.",
        "acts_on": "(new|update):observable:url",
    }

    def run(self, message: EventMessage) -> None:
        url = message.event.yeti_object
        self.logger.info(f"Extracting hostname from: {url.value}")
        o = urlparse(url.value)
        if observable.IPv4.is_valid(o.hostname):
            extracted_obs = observable.IPv4(value=o.hostname).save()
        else:
            extracted_obs = observable.Hostname(value=o.hostname).save()
        url.link_to(extracted_obs, "hostname", "Extracted hostname from URL")
        return

taskmanager.TaskManager.register_task(HostnameExtract)
```

The task must inherit `task.EventTask` and define `_defaults` dictionary to define its name, description and the events to acts on.

When a [consumer]({{< ref "developers/events-bus.md/#consumers" >}}) matches a task based on its acts_on, task `run` method will be called with the event as argument. 

In the example, this task will always receive an `EventMessage` with event of type `ObjectEvent` because the consumer will precisely match on `acts_on` which is based on `ObjectEvent`.

When implementing a task with a more generic `acts_on`, the task is responsible for handling the different event types it can receive.

# Testing

## Producer / Consumer

To test the new event task, a new python shell is needed to execute the consumer:

```shell
poetry run python -m core.events.consumer events --debug
```

Then in another shell to trigger producer when saving an observable

```python
>>> from core.schemas import observable
>>> observable.Url(value="http://example.com").save()
```
