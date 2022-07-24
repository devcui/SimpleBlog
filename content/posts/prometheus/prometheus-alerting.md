---
title: "Prometheus Alerting"
date: 2021-12-05T11:30:03+00:00
tags: ["prometheus"]
series: []
categories: ["时序数据库","数据库"]
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


# 1.Alerting with Prometheus is separated in to two parts

## 1-1Alerting rules

- Alerting rules is a file that formatted by yml,it's define some exp in to the yaml file.
- Alerting rules is in Prometheus servers send alerts to an alert manager and alert manager managed those alerts.

the rule document in there: [the rules doc](https://blog.eiyouhe.com/articles/2021/01/04/1609740000025.html)

## 1-2Alerting Manager

- AlertingManager is a command line tool and the command line tool flags has those functions
  
  - configure immutable system parameters
  - configuration file defines inhibition rules
  - configuration notification routing
  - configuration notification receivers
  - [router editor](https://www.prometheus.io/webtools/alerting/routing-tree-editor/)
- Alerting notifications steps
  
  - Setup and configure the alert manager
  - Configure Prometheus to talk to the alert manager
  - Create alerting rules in Prometheus

### 1-2-1 Configs

- config.file: specify which configuration file to load`./alertmanager --config.file=alertmanager.yml`
  - the file written in the YAML format and defined by the scheme described below
    - `<duration>`: a duration matching the regular expression `[0-9]+(ms|[smhdwy])`
    - `<labelname>`: a string matching the regular expression `[a-zA-Z_][a-zA-Z0-9_]*`
    - `<labelvalue>`: a string of unicode characters
    - `<filepath>`: a valid path in the current working directory
    - `<boolean>`: a boolean that can take the values true or false
    - `<string>`: a regular string
    - `<secret>`: a regular string that is a secret, such as a password
    - `<tmpl_string>`: a string which is template-expanded before usage
    - `<tmpl_secret>`: a string which is template-expanded before usage that is a secret
    - `<int>`: an integer value

there is the [simple file](https://github.com/prometheus/alertmanager/blob/master/doc/examples/simple.yml) and [some configs](https://prometheus.io/docs/alerting/latest/configuration/)

### 1-2-2 Cores

- Grouping
  
  - Grouping categorizes alerts of similar nature into a single notification.
- Inhibition
  
  - Inhibition is a concept of suppressing notifications for certain alerts if certain other alerts are already firing
- Silences
  
  - Silences are a straightforward way to simply mute alerts for a given time.
  - A silence is configured based on matchers, just like the routing tree.
  - Silences are configured in the web interface of the Alert manager.

### 1-2-3 Notification template Reference

#### 1-2-3-1 Data structures

data

| Name              | Type   | Notes                                                                                                             |
| ----------------- | ------ | ----------------------------------------------------------------------------------------------------------------- |
| Receiver          | string | The notification will be send to                                                                                  |
| Status            | string | Defined as firing if at least one alert if firing.otherwise resolved                                              |
| Alerts            | Alert  | List of all alert objects in this group                                                                           |
| GroupLabels       | kv     | The labels these alerts were grouped by                                                                           |
| CommonLabels      | kv     | The labels common to all of the alerts                                                                            |
| CommonAnnotations | kv     | set of common annotations to all of the alerts. Used for longer additional strings of information about the alert |
| ExternalURL       | string | Backlink to the alert manager that send the notification                                                          |

alert

| Name         | Type      | Notes                                                            |
| ------------ | --------- | ---------------------------------------------------------------- |
| Status       | string    | Defines whether or not the alert is resolved or currently firing |
| Labels       | kv        | A set of labels to be attached to the alert                      |
| Annotations  | kv        | A set of annotations for the alert                               |
| StartsAt     | time.Time | The time the alert started firing                                |
| EndsAt       | time.Time | Only set if the end time of an alert is know                     |
| GeneratorURL | string    | A backlink which identifies the causing entity of this alert     |

kv

| Name        | Args      | Returns                               | Notes                                                       |
| ----------- | --------- | ------------------------------------- | ----------------------------------------------------------- |
| SortedPairs | -         | Pairs(list of key/value string pairs) | Return s a sorted list of key/value pairs                   |
| Remove      | [] string | kv                                    | Returns a copy of the key/value map without the given keys. |
| Names       | -         | []string                              | Returns the names of the label names in the labelset        |
| Values      | -         | []string                              | Returns the list of values in the labelset                  |

functions

| Name         | Args                     | Returns                                                |
| ------------ | ------------------------ | ------------------------------------------------------ |
| title        | string                   | strings.Title,capitalises first character of each word |
| toUpper      | string                   | -                                                      |
| toLower      | string                   | -                                                      |
| match        | pattern,string           | Regexp.MatchString.Match a string using Regexp         |
| reReplaceAll | pattern,replacement,text | Regexp.ReplaceAllString Regexp substitution,unanchored |
| join         | sep string,s []string    | strings.join                                           |
| safeHtml     | text string              | html/template.HTML                                     |
| stringSlice  | ...string                | Returns the passed strings as a slice of strings       |