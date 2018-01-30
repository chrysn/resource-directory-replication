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

The traffic produced when millions of nodes with the default 24h lifetime amounts to dozens of exchanges per second,
which is doable with equal ease at central network equipment.

However, if the directory has additional interaction with its registered nodes,
for example because it provides proxying to registered endpoints,
resources like file descriptors can be exhausted earlier,
and the traffic load on the registration server grows with the traffic it is proxying for the endpoint.

## Large number of requests

Not all approaches to constrained restful communication use the Resource Directory only in the setup stage;
some are might also utilize a Resource Directory in more day-to-day operation.

@@@ get some numbers on how many requests a single RD can deal with

## Redundancy

With the RD as a central part of CoRE infrastructures, outages can affect a large number of users.

A decentralized RD should be able to deal both with scheduled downtimes of hosts
as well as unexpected outages of hosts or parts of the network,
especially with network splits between the individual parts of the directory.

# Approaches

## Shared authority

With this approach, a single host and port
(or "authority" component in the generic URI syntax)
is used for all interactions with the RD.

This can be implemented using a host name pointing to different IP addresses
simultaneously or depending on the requester's location,
using IP anycast addresses
or both.

From the client's or proxy's point of view,
all interaction happens with same Origin Server.

In this setup, the replication is hidden from the REST interactions,
and takes place inside the RD server implementation or its database backend.

Compared to the other approaches, this is more complex to set up
when it involves managing anycast addresses:
Running an IPv4 anycast network on Internet scale requires running an Autonomous System.
In either variation, all server instances are tightly coupled;
they need shared administration
and probably need to run the same software.

The replication characteristics are laregly inherited from the underlying backend.

As registering endpoints only store the URI constructed from the Location-Path option to their registration request,
registration updates can end up at any instance of the server,
though they are likely to reach the same one as before most of the time.

Spontaneous failure of individual nodes can interrupt endpoints' registrations in scenarious that do not use anycast addresses until the unusable addresses have left DNS caches.

## Plain caching

Caching reverse proxies that are not particularly aware of a Resource Directory can be used to mitigate the effect of large numbers of requests on a single RD server.
In this approach, there exists a single central RD server instance, but proxies are placed in front of it to reduce its load.

Caching is applicable only to the lookup interfaces; the POST request used in registration and renewal are not cacheable.

A prerequisite for successful caching is that fresh copies exist in the cache;
this is likely to happen only if there are many alike requests to the Resource Directory.
The proxy can than serve cached copies, and might find it advantageous to observe frequent queries.

The simplest way to set up such proxying is to have the proxies forward all requests to the central RD
and to advertise only the proxies' addresses.

Due to the discovery process of the RD, operators can also limit the proxies to the lookup interfaces
and advertise the central server for registration purposes.
A sample exchange between a node and its 6LoWPAN border router could be:

    Req: GET coap://[fe80::1]/.well-known/core?rt=core.rd*

    Res: 2.05 Content
    <coap://central-rd.example.com/rd>;rt="core.rd",
    <coap://europe3.proxy.rd.example.com/rd-lookup/ep>;rt="core.rd-lookup-ep",
    <coap://europe3.proxy.rd.example.com/rd-lookup/res>;rt="core.rd-lookup-res"

This approach does not help at all with large numbers of registrations.
It can mitigate issues with large numbers of lookup requests, provided that many identical requests arrive at the proxy.
The effect on the redundancy goal is negligible:
The proxy can provide lookup results only for as long as the cache is fresh during a central server outage,
which is 60 seconds unless the RD server says otherwise.

This approach can be run with off-the-shelf RD servers and proxies.
The only configuration required is for the proxy to have a forwarding address,
and for the RD (or its announcer) tho know which lookup addresses to advertise.

## RD-aware caching

Similar to the above, specialized proxies can be employed that are aware that their target is an RD lookup address.

The "plain caching" approach is limited in that it requires a small set of lookups to be frequently performed.
A proxy that is aware that the address it is forwarding to is of the Resource Type "core.rd-lookup-\*"
can utilize knowledge of how an RD works to serve more specialized requests as well from fresh generic content.

For example, assume that the proxy frequently receives requests of the shape

    Req: GET /rd-lookup/res?rt=core.s&rt=ex.temperature&ex.building=8341&title=X

for arbitrary values of X. Then it can use the following request to keep a fresh cache:

    Req: GET coap://rd.example.com/rd-lookup/res?rt=core.s&rt=ex.temperature&ex.building=8341
    Observe: 1

and from that serve filtered responses to individual requests.

This method shares the advantages of plain caching,
with reduced limitations but requiring specialized proxying software.

### Potential for improvement

Observing a large lookup result is relatively inefficient
as the complete document needs to be transferred when a change happens.
Serializations of web links that are suitable for expressing small deltas are expected to be developed
for PATCH operations on registration resources.
If those formats are compatible with observation, they can be applied directly.
Otherwise, the proxy can try to establish a "push" dynamic link ({{?I-D.ietf-core-dynlink}})
to receive continuous PATCH updates on its resource.

The applicability of the RD-aware approach is further limited to query parameters
of which the proxy knows that they are not subject to lookup filtering on other entities than the queried one.
In the example above, were the variable part the `d` attribute (of endpoints, as opposed to the `title` of resources),
the proxy could not do the filtering on its own becaus it would not have the required information.
Even the above example does not allow for fully accurate replication, as the endpoint *might* register
with a `title` endpoint attribute, even though no such attribute is specified right now.
It might be worth considering to be more explicit about filtering "up" and "down" in the hiearchy in the RD specification.
Also, annotating the links in the endpoint lookup with information about which registration they belong to
would help the proxy keep all the data around to solve more complex queries.
The provenance extension is proposed for that purpose.

In its extreme form, the proxy can observe the complete lookup resources of the Resource Directory.
It can then answer all queries on its own based on the continuously fresh state transferred in the observations.
That form requires the RD to support the provenance extension.

## Distinct entry points

@@@ registration and handover

@@@ replication of contents vs forwarding of requests

@@@ using foreign registration URIs

# Proposed RD extensions

@@@ `/lkp/res?provenance` -> `<coap://abc/foo>;anchor="coap://abc";provenance="/reg/1234"`

--- back
