---
title: "Vmalert"
date: 2021-12-05T11:30:03+00:00
tags: ["prometheus"]
series: []
categories: ["数据刮取"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Desc Text."
canonicalURL: "https://canonical.url/to/page"
disableHLJS:  false
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
---


- vmalert executes a list of given alerting or recording rules against configured address
- some command line interface tool

# configure vmalert

```bash
./bin/vmalert -rule=alter.rules \
    // PromSQL compatible datasource
    -datasource.url=http://localhost:8428
    // Alert manager url
    -notifier.url=http://localhost:9093
    // Alert manager replica rul
    -notifier.url=http://127.0.0.1:9093
    // remote write compatible storage to persist rules
    -remoteRead.url=http://localhost:8428
    //  PromSQL compatible datasource to restore alerts state from
    -remoteWrite.url=http://localhost:8428
    // External label to be applied for each rule
    -external.label=cluster=east-1 \
    // Multiple external labels may be set
    -external.label=replica=a \
    // Default evaluation interval if not specified in rules group
    -evaluationInterval=3s
```

> if you run multiple `vmalert` services for the same datastore or Alert manager - do not forget to specify different external.label flags in order to define which vmalert generated rules or alerts.

# groups

```yaml
groups: [- <rule_group>]
```

each group has following attributes:

```yaml
# the name of the group. Must be unique within a file
name: <string>
# How often rules in group are evaluated
[interval: <duration> | default = global.evaluation_interval ]
# How many rules execute at once.
# Increasing concurrency may speed
# up round execution speed
[ concurrency: <integer> | default = 1]
# rules
rules :
    [ - <rule> ...]
```

# rules

> there are two types of rules : recording and alerting

alerting rules

```yaml
# the name of the alert,Must be a valid metric name
alert: <string>
# the metricQL expression to evaluate
expr: <string>
# Alerts are considered firing once they have been returned for this long
# Alerts witch have not ye fired for long enough are considered pending
[ for:<duration> | default = 0s ]
# Labels to add or overwrite for each alert
labels:
    [<labelname>:<tmpl_string>]
# Annotations to add to each alert
annotations:
    [<labelname>:<tmpl_string]
```

recording rules

```yaml
# The name of the time series to output to. Must be a valid metric name.
record: <string>

# The MetricsQL expression to evaluate.
expr: <string>

# Labels to add or overwrite before storing the result.
labels:
  [ <labelname>: <labelvalue> ]
```