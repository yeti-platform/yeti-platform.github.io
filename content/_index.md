---
title: Welcome to the Yeti documentation site!
date: 2023-11-05T17:39:58Z
draft: false
---

Yeti aims to bridge the gap between CTI and DFIR practitioners by providing a
Forensics Intelligence platform for DFIR teams. It was born out of frustration
of having to answer the question "where have I seen this artifact before?" or
"how do I search for this IOC in my timeline?"

{{< cards >}} {{< card link="/docs" title="Documentation" icon="book-open" >}}
{{< card link="/guides" title="Guides" icon="search-circle" >}}
{{< card link="https://github.com/yeti-platform/yeti" title="Code" icon="github" >}}
{{< /cards >}}

## What is Yeti?

In a nutshell, Yeti allows you to:

- Bulk search observables and get a pretty good guess on the nature of the
  threat, and how to find it on a system.
- Inversely, focus on a threat and quickly list all TTPs, malware, and related
  DFIR artifacts.
- Let CTI analysts focus on adding intelligence rather than worrying about
  machine-readable export formats.

This is done by:

- Storing technical and tactical CTI (observables, TTPs, campagins, etc.) from
  internal or external systems.
- Being a backend for DFIR-related queries: Yara signatures, Sigma rules, DFIQ.
- Providing a web API to automate queries (think incident management platform)
  and enrichment (think malware sandbox).
- Export the data in user-defined formats so that they can be ingested by
  third-party applications (SIEM, DFIR platforms).
