---
title: Resource Directory Replication
docname: draft-amsuess-core-resource-directory-replication-latest
stand_alone: true
ipr: trust200902
cat: info
pi:
  strict: 'yes'
  symrefs: 'yes'
  sortrefs: 'yes'
  compact: 'yes'
  comments: yes
  subcompact: 'no'
  iprnotified: 'no'
area: Internet
wg: CoRE
kw: CoRE, Web Linking, Resource Directory, Federation, Replication
author:
- ins: C. Amsüss
  name: Christian Amsüss
  street: Hollandstr. 12/4
  code: '1020'
  country: Austria
  phone: "+43-664-9790639"
informative:
  I-D.ietf-core-resource-directory:

--- abstract

Discovery of endpoints and resources in M2M applications over large networks is
enabled by Resource Directories, but no special consideration has been given to
how such directories can scale beyond what can be managed by a single device.

This document explores different ways in which Resource Directories can be
scaled up from single network to enterprise and global scale. It does not
attempt to standardize any of those methods, but only to demonstrate the
feasibility of such extensions and to provide terminology and exploratory
groundwork for later documents.

--- middle

# Introduction {#introduction}

@@@ see abstract for now

# Terminology

@@@

Origin Server from {{?RFC7252}}

# Goals of upscaling

The following sections outline different reasons why a Resource Directory should be scaled beyond a singe device.
Not all of them will necessarily apply to all use cases, and not all solution approaches might be suitable for all goals.

## Large numbers of registrations

Even at 1kB of link data per registration,
modern server hardware can easily keep the data of millions of registrations in RAM simultaneously.
Thus, the mere size of registration data is not expected to be a factor that requires scaling to multiple nodes.

The traffic produced when millions of nodes with the default 24h lifetime amounts to tens of exchanges per second,
which is doable with equal ease at central network equipment.

However, if the directory has additional interaction with its registered nodes,
for example because it provides proxying to registered endpoints,
resources like file descriptors can be exhausted earlier,
and the traffic load on the registration server grows with the traffic it is proxying for the endpoint.

## Large amounts of requests

Not all approaches to constrained restful communication use the Resource Directory only in the setup stage;
some are might also utilize a Resource Directory in more day-to-day operation.

@@@ get some numbers on how many requests a single RD can deal with

## Redundancy

With the RD as a central part of CoRE infrastructures, outages can affect a large number of users.

A decentralized RD should be able to deal both with scheduled downtimes of hosts
as well as unexpected outages of hosts or parts of the network,
especially with network splits between the individual parts of the directory.

# Approaches

## Shared authority / anycast

When used with the CoAP protocol over UDP or DTLS, a Resource Directory can be
served from an anycast addresses, which is reflected in the authority component
of all RD resources. From the clients' point of view, all interaction happens
from the same Origin Server.

In this setup, the replication is hidden from the REST interactions, and takes
place inside the RD server implementation or its database backend.

@@@ delegation to replication under the RD

## Plain caching

## RD-aware caching

## Distinct entry points

@@@ registration and handover

@@@ replication of contents vs forwarding of requests

@@@ using foreign registration URIs

--- back
