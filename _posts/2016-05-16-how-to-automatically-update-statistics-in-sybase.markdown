---
published: true
title: How to automatically update statistics in Sybase?
layout: post
tags: [sybase, configuration, statistics]
categories: [database]
---
*Statistics must be accurate as possible, one way to update them is to use `update statistics` but this is a manual operation. Is there a way for Sybase to initiate this automatically?*

<!--excerpt-->

You can configure parameter `sysstatistics flush interval` to a value different to 0.

This value indicates the interval in minutes that ASE will wait before checking  the amount of data changed and initiate an `update statistics`

If set, the housekeepr task will be in charge of checking and flushing those statistics