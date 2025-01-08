---
layout: default
title: Hangfire
parent: Insight
nav_order: 2
last_modified_date:   2025-01-08 22:55:01
---

# Feature
1. Simple dashboard for displaying server, job and statis.
1. Job type includes Fire-and-Forget, Delays Job, Recurrying Jobs and Continuations.
1. Uses Sql server to store metadata with code first pattern.
1. Support both dynamic job and class job.
1. Support web like asp.net, asp.net core and windowns service and console application.
1. Support Authentication before running the application.
1. simple table schema, and with keyvalue type for some table for more extensible.
1. Worker count configurable.
1. As use keyvalue table structure, can add dynamic properties to job.

# Userful Design
1. Keyvalue table structure and one class to manage tables and per method per table.
1. Configuration for dynamic job.
1. Job executed status notification using email or/and SMS.

# Issue
1. Sometimes worker thread stuck unexpected.
1. Dashboard too simple to manage the job, like lack of add job, batch delete job, and find job.
1. Need to complicated configuration to enable auto-restart after recyle application for asp.net web.
1. DisableConcurrencyExecution only work on class's Attribute, for dynamic job, need use filter with dynamic perperty to prevent concurrency execution.
1. The last execution time always show previous job interval instread of previous success or failed time, if the last job cancel, it will shows cancel in dashboard.