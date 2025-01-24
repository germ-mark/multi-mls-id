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
Secrecy (FS) and Post-Compromise Security (PCS). A group of participants may
find it difficult to agree on a common state - or that reaching eventual consistency
is impractical for the application. This document describes
Multi-MLS (MMLS), a protocol for using MLS sessions to protect messages
among participants without negotiating a common group state.

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

An MLS sender group operates in the following constrained way:
  * The creator of the group, occupying leaf index 0, is the designated sender
  * Participants only accept MLS commits and application messages authored by the designated sender.
  * The designated sender may accept proposal messages from all
  participants.

In this way, the MLS send group allows the sender to encrypt messages to a
a set of recipients the sender chooses (by authoring commits), and update the 
recipient list and recipients' key material transparently to all recipients
through the use of MLS operations.

Assuming an honest designated sender, participants who agree to operate the group
under these constraints will agree on group state. 

(A dishonest sender can send different commits to different participants.
The MMLS layer can help participants gossip about the commits they see in their
own send groups)

# MMLS functions

~~The primary requirement of MMLS is that for a given MLS group, it adjudicates
the designated sender credential(s) so that the send group policies can be
correctly applied, including updates to the designated sender's leafNode.~~

(An MMLS layer atop these send groups can provide further structure by relating
send groups 
* A recipient _R_ in a send group _A_ can reply-all by creating their own send group
containing the same participants, and initialized with secrets exported from group _A_.
)

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