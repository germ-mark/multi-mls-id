---
title: "MLS without a centralized DS"
abbrev: "Multi MLS"
category: info

docname: draft-xue-multi-mls-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: ""
workgroup: "Multiformats"
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
  group: "Multiformats"
  type: ""
  mail: "multiformats@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/multiformats/"
  github: "germ-mark/multi-mls-id"
  latest: "https://germ-mark.github.io/multi-mls-id/draft-xue-multi-mls.html"

author:
 -
    fullname: "Mark @ Germ"
    organization: Germ Network, Inc.
    email: "mark@germ.network"

normative:

informative:


--- abstract

The Messaging Layer Security (MLS) protocol enables a group of participants to
negotiate a common cryptographic state for messaging, providing Forward 
Secrecy (FS) and Post-Compromise Security (PCS). This document describes
Multi-MLS (MMLS), a protocol for using MLS sessions to protect messages
among participants without negotiating a common group state

--- middle

# Introduction

Participants operating in peer to peer or partitioned network topologies
may find it impractical to access a centralized Delivery Service, or reach
consensus on message sequencing to arrive at a consistent commit for each
MLS epoch.

Multi-MLS is a protocol for users to encrypt messages to multiple recipients
and update participant's keys to provide FS and PCS. It facilitates group
messaging by using an MLS group per participant as that participant's send
group. This allows each participant to locally and independently determine
their view of the group membership, and encrypt messages using MLS to that
set of recipients.

# Send Group Operation

MMLS designates each constituent MLS group as the send group for a
particular AS credential. Within this send group, we impose these additional
rules:
  * Participants only accept MLS commits and application messages authored by the designated sender.
    * If a commit updates the designated sender's credential, MMLS must authorize the replacement credential as a sender for this MLS group.
    * 
  * The designated sender may additionlly accept proposal messages from all
  participants.

Participants can use the MLS proposal to update their leaf keys and 
credentials. 

In this way, the MLS send group allows the sender to encrypt messages to a
a set of recipients the sender chooses (by authoring commits), and update the 
recipient list and recipients' key material transparently to all recipients
through the use of MLS operations.

# MMLS functions

MMLS, as an application layer atop MLS, has to bind the constituent MLS groups
into a shared context and designate a sender for each constituent group. For
example, given an MMLS context identifier, an MLS groupId, and an AS 
credential, MMLS should validate if that groupId is the designated send 
group for that AS credential in the bundle of groups represented by
the MMLS context identifier.

# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
