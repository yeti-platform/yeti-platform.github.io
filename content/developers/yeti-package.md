---
title: Yeti Package
date: 2024-11-08T08:20:00Z
draft: false
cascade:
  type: docs
---

Yeti Package helps creating a bundle of mixed yeti objects. It supports:

* all observable types and will create generic type with a tag if type is not implemented as an observable.
* all available entities and indicators
* link between defined objects

Here's one example:

```json
{
   "timestamp": "2024-09-24T08:31:29.312Z",
   "source": "honeypot",
   "tags": {
      "global": ["honeypot", "exploitation"],
      "88.173.200.156": ["one_tag"]
   }
   "observables": [
      {
         "value": "88.173.200.156",
         "type": "ipv4"
      },
      {
         "value": "Go-http-client/1.1",
         "type": "user_agent"
      },
      {
         "value": "ubuntu:18.04",
         "type": "docker_image"
      },
      {
         "value": "/bin/bash",
         "type": "command_line"
      }
   ],
   "entities": [
      { 
         "name": "docker malicious campaign",
         "type": "campaign",
         "description": "### Docker container creation attempt\n* ```ubuntu:18.04```\n* ```/bin/bash```\n"
      }
   ],
   "indicators": {},
   "relationships": {
         "docker malicious campaign": [
            {
               "target": "88.173.200.156",
               "link_type": "observes"
            },
            {
               "target": "ubuntu:18.04",
               "link_type": "creates"
            },
            {
               "target": "/bin/bash",
               "link_type": "executes"
            },
         ],
         "88.173.200.156": [
            {
               "target": "Go-http-client/1.1",
               "link_type": "uses"
            },
            {
               "target": "ubuntu:18.04",
               "link_type": "creates"
            },
            {
               "target": "/bin/bash",
               "link_type": "executes"
            }
         ]
      }
}
```

This package will create a campaign named "docker malicious campaign" with the following observables:

* ipv4: `88.173.200[.]156`
* user-agent: `Go-http-client/1.1`
* docker_image: `ubuntu:18.04`
* command_line: `/bin/bash`

The following relationships will also be created:

* `88.173.200[.]156` --> uses --> `Go-http-client/1.1`
* `88.173.200[.]156` --> creates --> `ubuntu:18.04`
* `88.173.200.156` --> executes --> `/bin/bash`

The campaign itself will be linked with:

* `88.173.200[.]156` and `observes` link
* `ubuntu:18.04` and `creates` link
* `/bin/bash` and `executes` link

All elements will be tagged with `honeypot` and `exploitation` and `88.173.200[.]156` will be tagged with `one_tag`