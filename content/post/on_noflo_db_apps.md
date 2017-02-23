+++
tags = ["noflo","fbp","development"]
comments = true
draft = false
description = "A few thoughts on design of NoFlo backends frequently talking to a RDBMS"
image = "/img/post-bg.jpg"
date = "2016-07-27T14:25:04+01:00"
title = "On NoFlo database applications"
categories = ["Flow-based Programming","NoFlo"]
+++

# On NoFlo Database Applications

Thinking over common parts of database applications...

![Scrappy notes](/img/post/on_noflo_db_apps.jpg)

… I came to some conclusions below

## Common application architecture

### Steps common to all route graphs

* The only processing step that is often present and easy to decompose is Validate *(fig. I and II)*.

### String of pearls or Pipeline or Request backbone

* Request flow should be serial, it simplifies synchronization and understanding of the flow.
* Each bone in the spine of the graph or step on the pipeline can process some data in parallel. But as the flow should be unidirectional, scatter-gather topology *(fig. IV)* is preferred over side jobs *(fig. III)*.

## Database operations

* Database queries are all different across the application. Input data needs to be extracted/filtered first, then inserted into specific parts of the query, then the result rowset needs to be filtered too *(fig. V)* to be used down the stream.
* FBP-SQL would need either a lot of components to cover every specific query configuration, or a rather bushy query builder to compose complex queries in the graph.
* Database operations are very granular in the app and appear in any domain of the app, so Flux Store makes no sense.
* Using a single Database component in the graph makes too many wires.
* Using one instance per query uses more memory.
