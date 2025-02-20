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
area: "Security"
workgroup: "Messaging Layer Security"
keyword:
 - messaging layer security
 - end-to-end encryption
 - post-compromise security
venue:
  group: "Messaging Layer Security"
  type: "Working Group"
  mail: "mls@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/mls/"
  github: "germ-mark/multi-mls-id"
  latest: "https://germ-mark.github.io/multi-mls-id/draft-xue-multi-mls.html"

author:
 -
    fullname: "Mark Xue"
    organization: Germ Network, Inc.
    email: "mark@germ.network"
 -
    fullname: "Joseph W. Lukefahr"
    organization: US Naval Postgraduate School
    email: "joseph.lukefahr@nps.edu"
 -
    fullname: "Britta Hale"
    organization: US Naval Postgraduate School
    email: "britta.hale@nps.edu"

normative:

informative:
 CAPBR: # Brewer, E., "Towards robust distributed systems (abstract)", ACM, Proceedings of the nineteenth annual ACM symposium on Principles of distributed computing, DOI 10.1145/343477.343502, July 2000, <https://doi.org/10.1145/343477.343502>.


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

# Meeting MLS Delivery Service Requirements

The MLS Architecture Guide {{draft-ietf-mls-architecture-15}} specifies two requirements for an abstract Delivery Service related to message ordering.
First, Proposal messages should all arrive before the Commit that references them.
Second, members of an MLS group must agree on a single MLS Commit message that ends each epoch and begins the next one.

An honest centralized DS, in the form of a message queuing server or content distribution network, can guarantee these requirements to be met.
By controlling the order of messages delivered to MLS participants, for example, it can guarantee that Commit messages always follow their associated Proposal messages.
By filtering Commit messages based on some pre-determined criteria, it can ensure that only a single Commit message per epoch is delivered to participants.

A decentralized DS, on the other hand, can take the form of a message queuing server without specialized logic for handling MLS messages or, prehaps, simply a local area network.
These DS instantiations cannot offer any such guarantees.

The MLS Architecture Guide highlights the risk of two MLS participants generating different Commits in the same epoch and then sending them at the same time.
The impact of this risk is inconsistency of MLS group state among participants.
This perhaps leads to inability of some authorized participants to read other authorized participants' messages, i.e., a loss of availability of the message-passing service provided by MLS.
A decentralized DS offers no mitigation strategy for this risk, so the participants themselves must agree on strategies, or in our terminology, operating constraints.
We could say that the full weight of the CAP theorem is thus levied directly on the MLS participants in this case.
However, use cases exist that benefit from, or even necessitate, MLS and its accompanying security guarantees for group message passing.

The MMLS operating constraints specified above allow honest participants to form a distributed system that satisfies these requirements despite a decentralized DS.

# MMLS functions

~~The primary requirement of MMLS is that for a given MLS group, it adjudicates
the designated sender credential(s) so that the send group policies can be
correctly applied, including updates to the designated sender's leafNode.~~

(An MMLS layer atop these send groups can provide further structure by relating
send groups 
* A recipient _R_ in a send group _A_ can reply-all by creating their own send group
containing the same participants, and initialized with secrets exported from group _A_.
)

Similar to MLS, MMLS provides a participant appliation programming interface (API) with the following functions:

* INIT -- Given a list of MMLS participants, initialize an MMLS context by (1) creating an MLS group, (2) adding all
other participants (generating a set of Welcome messages and a GroupInfo message), and (3) 
* UPDATE
* PROTECT # or encrypt? or create_message?
* UNPROTECT # or decrypt? or process_message?

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