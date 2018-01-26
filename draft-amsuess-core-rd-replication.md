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

# Use cases

# Approaches

## Anycast 

@@@ delegation to replication under the RD

## Distinct entry points

@@@ registration and handover

@@@ replication of contents vs forwarding of requests

@@@ using foreign registration URIs

--- back
