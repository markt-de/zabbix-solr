#### Table of Contents

1. [Description](#description)
1. [Concept](#concept)
1. [Requirements](#requirements)

## Description

A Zabbix template to monitor the Apache Solr platform. It uses JMX and
Javascript preprocessing, so no additional scripts are required.

The template includes the following:

* Discovery rule to monitor 70 items for every Solr Core
* 30+ items to monitor Solr node and JVM status

## Concept

This template was designed around a unique monitoring concept. In order
to understand the design decisions made, a quick summary is outlined below.

Design goals:

* Avoid false-alerts
* Avoid trigger flapping
* Reduce overall monitoring "noise"

To achieve these goals the following principles were applied:

* Trigger severity has a **distinct meaning**
  * `Warning` – Alert is only displayed in Zabbix GUI. No notifications will be send.
  * `High` – All previous, plus the alert is send via e-mail. (Typical 5x8 alert.)
  * `Disaster` – All previous, plus the alert is send via instant messaging or SMS. (Typical 7x24 alert.)
  * Note: Zabbix "Actions" need to be configured accordingly.

* Enforce 3-step escalation and delay alarms
  * After at least two failures a `Warning` trigger may fire.
  * If the problem persists for the time period specified by `{$ESCALATION_1_DELAY}`, then the `High` trigger fires.
  * If the problem persists for the time period specified by `{$ESCALATION_2_DELAY}`, then the `Disaster` trigger fires.
  * These triggers have dependencies defined, i.e. a `Warning` trigger is closed when the `High` trigger fires.

As a result, *every* alarm starts with a `Warning` trigger, which should
not send any notifications. Only if enough time has passed, then a `High`
or `Disaster` trigger will be activated and notifications will be send.

This simple approach ensures that load spikes or small hiccups do not
cause any alarms/notifications. Additionally triggers are not resolved
too soon (which would result in trigger flapping), but instead a certain
amount of time has to pass.

Of course, this delay can be customized by defining `{$ESCALATION_1_DELAY}`
and `{$ESCALATION_2_DELAY}` on a per-host level.

## Requirements

Basic requirements for this template:

* Zabbix Server version 4.4 or later
* Solr version 8.0 or later
* Remote JMX must be enabled
* JMX interface must be configured with host/port information

Besides that the 3-step escalation requires that you define the following
global macros:

* `{$ESCALATION_1_DELAY}`, i.e. with a value of `600`
* `{$ESCALATION_2_DELAY}`, i.e. with a value of `1200`

If you want to completely disable the escalation delay, then you may
set these values to `0` (NOT RECOMMENDED).
