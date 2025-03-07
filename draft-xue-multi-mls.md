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

# Terminology

Send Group: An MLS group where one designated member authors all messages
and other members use the group only to receive from the designated sender.

Universe: A set of MLS participants that comprise a MMLS group.

# Protocol Overview

Within a group $G$ of distributed participants, we can resolve state conflict by
assigning each member local state that only they control. In Multi-MLS, we assign
each member an MLS group to operate as a Send Group. Through the export and
import of epoch secrets across these send groups, the members of G can 
accomplish PCS and FS without reaching consensus on global state.

## Meeting MLS Delivery Service Requirements

The MLS Architecture Guide {{draft-ietf-mls-architecture-15}} specifies two
requirements for an abstract Delivery Service related to message ordering.
First, Proposal messages should all arrive before the Commit that references them.
Second, members of an MLS group must agree on a single MLS Commit message that
ends each epoch and begins the next one.

An honest centralized DS, in the form of a message queuing server or content
distribution network, can guarantee these requirements to be met.
By controlling the order of messages delivered to MLS participants, for example,
it can guarantee that Commit messages always follow their associated Proposal messages.
By filtering Commit messages based on some pre-determined criteria, it can ensure
that only a single Commit message per epoch is delivered to participants.

A decentralized DS, on the other hand, can take the form of a message queuing server without specialized logic for handling MLS messages or, prehaps, simply a local area network.
These DS instantiations cannot offer any such guarantees.

The MLS Architecture Guide highlights the risk of two MLS participants generating different Commits in the same epoch and then sending them at the same time.
The impact of this risk is inconsistency of MLS group state among participants.
This perhaps leads to inability of some authorized participants to read other authorized participants' messages, i.e., a loss of availability of the message-passing service provided by MLS.
A decentralized DS offers no mitigation strategy for this risk, so the participants themselves must agree on strategies, or in our terminology, operating constraints.
We could say that the full weight of the CAP theorem is thus levied directly on the MLS participants in this case.
However, use cases exist that benefit from, or even necessitate, MLS and its accompanying security guarantees for group message passing.

The MMLS operating constraints specified above allow honest participants to form a distributed system that satisfies these requirements despite a decentralized DS.

# Send Group Operation

An MLS Send Group operates in the following constrained way:
  * The creator of the group, occupying leaf index 0, is the designated sender
  * The creator may accept update messages from any participant
  * Other participants only accept messages from the creator
  * Participants only accept messages as defined in Group Operations

To facilitate binding Send Groups together, we define the following exported values:
   * derived groupid: `MLS-Exporter("derivedGroupId", leafNodePublicSigningKey, Length)`

      This is a unique value for each participant derived from the group's current epoch
   * exportPskId: `MLS-Exporter("exportPskId", "Universe identifier", Length)`
   * exportPSK: `MLS-Exporter("exportPSK", "Universe identifier", Length)`

# Group Operations

Similar to MLS, MMLS provides a participant appliation programming interface (API) with the following functions:

* INIT

Given a list of MMLS participants, initialize an MMLS context by (1) creating an MLS group, (2) adding all
other participants (generating a set of Welcome messages and a GroupInfo message), and (3) 
It is the responsibility of an MMLS implementation to define the Universe of
participants and the mechanism of generating the individual send groups. 
"MMLS Requirements" sketches one such approach.

* UPDATE

A member Alice of $U$ can update their leafNode in the universe $U$ by authoring a full 
or empty commit in Alice's send group, which provides PCS with regard to the committer.

This update commit is also an opportunity to update Alice's credential, in which case
Alice should also distribute corresponding update messages in all other send groups.

* COMMIT

When Bob recives Alice's MMLS update (as a full or empty commit in Alice's send group),
Bob can incorporate PCS from Alice's commit by importing a PSK from Alice's send group.
Precisely, Bob:
   * Creates a PSK proposal in Bob's send group using the exportPskId
      and exportPSK from the epoch of Alice's send group after Alice's MMLS update
   * If Alice's commit updated Alice's credential, Bob should have received an
      accompanying update proposal in Bob's send group.
   * Bob generates a commit covering the PSK proposal (for each send group in which
   he has observed a new MMLS update), and any update proposals he received.

The `psk_group_id` for this PSK is more specifically defined as follows:
```
psk_group_id = (opaque<8>) groupEpoch | groupId
```
where `epoch_bytes` is the byte-vector representation of the epoch in which the exporter was generated, in network byte order.
Since epoch is of type `uint64`, this results in a fixed 8-byte vector.
`groupId`, which is of type `opaque<V>`, is then appended to `epoch_bytes`.
When a `exportPskId` is received as part of an incoming PSK proposal, it can then be processed as follows:
```
groupId = exportPskId[8..]
epoch = (uint64) exportPskId[0..7]
```

Per [RFC9420], the `psk_nonce` must be a fresh random value of length `KDF.Nh` when the PSK proposal is generated.
This ensures key separation between a PSK generated by, for example, (1) a PSK generated by Bob from Alice's group and (2) a PSK generated by Charlie from Alice's group.

* PROTECT # or encrypt? or create_message?
A member Bob protects a ciphertext message and encrypting it to $U$ by encrypting it
as an application message in their send group. As in MLS, before encrypting an
application message, Bob should incorporate any MMLS updates he has received.

Each of the 3 MLS configurations of commit are possible:
If Bob has no MMLS update to issue, and has seen no credential updates,
Bob generates a partial commit covering PSK proposals from each updated send group.

If Bob has seen credential updates, Bob generates a full commit, and for each
   sender that has issued a MMLS update since Bob's last commit,
   * Bob injects a PSK for that sender's send group
   * Bob incorporates any updates they've observed.

If Bob has observed no updates but wishes to provide PCS, they can author
an empty commit.

Members are not required to inject a PSK from a send group if they have only
observed partial commits.
(This allows the distibuted state to stabilize,
if incorporating changes with a partial commit doesn't induce other members to commit.
Injecting a PSK from a partial commit covering some PSK proposals
doesn't add any benefit over importing the same PSK's coverered by the partial commit.
)


* UNPROTECT # or decrypt? or process_message?
On receipt of an MLS message, a member can look up the corresponding send group
by the MLS groupId in the message metadata and attempt to decrypt it with that
send group.


(Mark proposal)
* REJOIN

A member of MMLS needs to receive all commits from all other send groups to
continue to receive messages. A member that has been offline or otherwise
fails to receive some commits still has the ability to encrypt messages to (possibly stale)
credentials of the Universe in their own send group. This offline member Charlie can
request to be re-join the other send groups by broadcasting an MMLS message
in their own send group that indicates they are requesting a re-join. Other members
should have access to a keyPackage message for Charlie. This could be attached
to the re-join message if necessary.

The psk proposals in each of Charlie's commit serve as an ack of the commits of
other send groups that Charlie has received, so other members can infer from
Charlie's send group epoch, the newest epoch Charlie has observed for each send
group.

Each member can then determine if Charlie is out of date in their send group.
If necessary, they can author a commit removing Charlie's current leafNode in their
send group and re-add Charlie using a keyPackage for Charlie.

# MMLS requirements

The application layer over MLS has the responsibility to define 
* The Universe $U$ of members of this MMLS group
* A universe identifier (as context for key exports)
* The export key length
* Additional common rules, such as accepted cipher suites

(nothing inherently requires the send groups to agree on a cipher suite - 
each sender could choose their own as long as they agree on export key length
)

The MMLS layer should recommend a policy for issuing MMLS updates.

## Over the wire definition
For example, $U$ can be defined over the wire by inferring it from a newly created
send group.

Assume Alice has keypackages for some other members $M_i$

Alice can construct a MMLS group
   * with a randomly generated groupId
   * constructing a commit adding all other members $M_i$

Alice can distribute the Welcome message with an Application Message that indicates 
   * this is a Send Group for Alice
   * that defines a Universe $U$ as the members of this group
   * with universe identifier equal to the groupId for Alice's send group
   * and defines a common export key length

# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.

# References 
 CAPBR: # Brewer, E., "Towards robust distributed systems (abstract)", ACM, Proceedings of the nineteenth annual ACM symposium on Principles of distributed computing, DOI 10.1145/343477.343502, July 2000, <https://doi.org/10.1145/343477.343502>.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
