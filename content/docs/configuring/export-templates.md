---
title: Export Templates
date: 2026-07-01T9:14:05Z
draft: false
weight: 2
---

This section describes how to write and configure the [Jinja](https://jinja.palletsprojects.com/)-style
templates used by export tasks, and where to put them so `yeti` can find
them. See [File Storage Clients & Export configuration](export-configuration)
for how the *output* of an export is stored.

## What Runs a Template?

An export template is rendered by an `ExportTask`, a Celery task that:

1. Fetches the observables to export (filtered by tags, per the task's
   configuration).
2. Loads the named template from disk.
3. Renders it and writes the result to the configured file storage client
   (see [export configuration](export-configuration)).

Templates can also be rendered on demand, without creating an export task,
via `POST /api/v2/templates/render`.

Templates are rendered using [minijinja](https://github.com/mitsuhiko/minijinja),
a Rust implementation of the Jinja2 template language. It supports the
vast majority of standard Jinja2 syntax and filters, but isn't a 1:1 port -
if a filter you rely on from Python's Jinja2 doesn't work, check the
[minijinja filter reference](https://docs.rs/minijinja/latest/minijinja/filters/index.html).

## Where Do Templates Live?

Templates are plain files named `<template_name>.jinja2`, stored in the
directory configured by `system.template_dir` in `yeti.conf`
(`YETI_SYSTEM_TEMPLATE_DIR` as an environment variable), which defaults to
`/opt/yeti/templates`.

```ini
[system]
template_dir = /opt/yeti/templates
```

{{< callout type="warning" >}} **Both the `api` and `tasks` services read
directly from this directory** - the API reads it to list/search templates
and to serve on-demand renders, and the `tasks` worker reads it to run
scheduled export tasks. If you're running Yeti with Docker, mount the same
volume at the same path on both services. There's currently no API
endpoint to upload a template's contents - it must be placed on disk
directly. {{< /callout >}}

## What's Available Inside a Template?

The only variable available inside an export template is `data`: a list of
[`Observable`](https://github.com/yeti-platform/yeti/blob/main/core/schemas/observable.py)
objects matched by the export task's tag filters (or by the observables/search
query passed to `/api/v2/templates/render`).

Every observable exposes at least these fields:

| Field         | Description                                             |
|---------------|----------------------------------------------------------|
| `value`       | The observable's value, e.g. `"1.2.3.4"`                  |
| `type`        | The observable type, e.g. `"ipv4"`, `"hostname"`, `"url"` |
| `tags`        | A list of tag objects, each with a `.name` and `.fresh`   |
| `created`     | Creation timestamp                                        |
| `modified`    | Last modification timestamp                               |
| `context`     | A list of context dicts attached to the observable         |

Some observable types add their own fields on top of these - check the
relevant class under
[`core/schemas/observables/`](https://github.com/yeti-platform/yeti/tree/main/core/schemas/observables)
for the full list (e.g. a `Certificate` observable exposes fields not
present on a plain `Hostname`).

{{< callout type="info" >}} **Note**: rendering a whole observable with the
`tojson` filter does not produce clean JSON - minijinja serializes the
underlying Python object as a list of `(key, value)` pairs rather than a
dict. Build the object you want to export field-by-field instead, as shown
below. {{< /callout >}}

## Example Templates

Annotated example templates are kept in the main repository under
[`extras/templates/`](https://github.com/yeti-platform/yeti/tree/main/extras/templates),
copy them into your `template_dir` as a starting point.

### JSON Lines

```jinja
{% for row in data -%}
{"value": {{ row.value|tojson }}, "type": {{ row.type|tojson }}, "tags": {{ row.tags|map(attribute="name")|list|tojson }}}
{% endfor -%}
```

### CSV

```jinja
value,type,tags
{% for row in data -%}
{{ row.value }},{{ row.type }},"{{ row.tags|map(attribute="name")|join(";") }}"
{% endfor -%}
```
