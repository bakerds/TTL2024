# Introduction

## What is Graylog?

**Graylog is a free and open source log management platform** that provides a means to aggregate, organize, and make sense of log data from devices, applications, and operating systems. Logs may contain information about events such as descriptions, IP addresses, timestamps, and results. The log management system is responsible for ingesting, parsing, storing, correlating and displaying the log data.

## What is NPS?

**Network Policy Server (NPS) is a Windows server role** that allows you to centrally configure and manage network access authentication, authorization, and accounting using RADIUS (1)
{ .annotate }

1.	RADIUS is an acronym for Remote Authentication Dial-In User Service


## What problem are we solving?

- NPS logs can be really useful for troubleshooting and accounting, but...
- NPS logging is badly behaved
    - NPS writes to a flat text file in `C:\Windows\System32\LogFiles`
    - Logs files will grow until they fill your `C:\` drive to 100%
    - Logs are difficult to view and interpret
    - There is no native support for syslog output
    - No support for interpretation, aggregation, retention policies, etc.

We want to be able to easily view, interpret, aggregate, and control retention of these logs




