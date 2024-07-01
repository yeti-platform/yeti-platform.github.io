---
title: Welcome to the Yeti documentation site!
date: 2023-11-05T17:39:58Z
draft: false
---

{{< figure src="logo-black.png" width="200" class="yeti-logo dark:hidden">}}
{{< figure src="logo-white.png" width="200" class="yeti-logo hidden dark:block">}}

Yeti aims to bridge the gap between CTI and DFIR practitioners by providing a
Forensics Intelligence platform and pipeline for DFIR teams. It was born out of
frustration of having to answer the question "where have I seen this artifact
before?" or "how do I search for IOCs related to this threat (or all threats?)
in my timeline?"

{{< rawhtml >}} <br /> <br /> <br />

<div class="yeti-separator"></div>
{{< /rawhtml >}}

{{< cards >}} {{< card link="/docs" title="Documentation" icon="book-open" >}}
{{< card link="/guides" title="Guides" icon="search-circle" >}}
{{< card link="https://github.com/yeti-platform/yeti" title="Code" icon="github" >}}
{{< /cards >}}

## What is Yeti?

In a nutshell, Yeti allows you to:

- Store and manage Forensics Intelligence: [DFIQ objects](https://dfiq.org),
  [forensic artifact](https://github.com/ForensicArtifacts/artifacts)
  definitions, Sigma and Yara rules, reusable queries, etc.
- Bulk search observables and get a pretty good guess on the nature of the
  threat, and how to find it on a system.
- Inversely, focus on a threat and quickly list all TTPs, malware, and related
  forensic artifacts.
- Let CTI analysts focus on adding intelligence rather than worrying about
  machine-readable export formats.
- Easily incorporate your own data sources, analytics, and logic.

This is done by:

- Storing technical and tactical CTI (observables, TTPs, campaigns, etc.) from
  internal or external systems.
- Being a backend for DFIR-related queries: Yara signatures, Sigma rules, DFIQ.
- Providing a web API to automate queries (think incident management platform)
  and enrichment (think malware sandbox).
- Export the data in user-defined formats so that they can be ingested by
  third-party applications (SIEM, DFIR platforms).

## Some screenshots

{{< figure src="scattered.png" title="Example of ScatteredSpider as imported through MITRE ATT&CK" >}}

{{< figure src="attack.png" title="Example: Intrusion sets as imported through the MITRE ATT&CK feed" >}}

{{< figure src="vuln.png" title="Example: CVE-2023-46604 " >}}

{{< figure src="intrusionset.png" title="Example: TeamTNT intrusion set" >}}
