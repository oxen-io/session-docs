title: Session Docs | Session | Threat Model
description: It is useful to understand the protections Session provides to users, and the threat model which it is effective in defending against.

# Threat Model 

It is useful to understand the protections Session provides to users, and the threat model which it is effective in defending against.

## Protections

Session aims to provide the following protections against attackers within the scope of the threat model:

**Sender Anonymity**: The long-term identity key of the sender is only knowable to the member(s) of the conversation, and the IP address of the sender is unknown to all parties except the first hop in the onion routing path.

**Recipient Anonymity**: The IP address of the recipient is unknown to all parties except the first hop in the onion routing path.

**Data Integrity**: Messages are received intact and unmodified, and if messages are modified they appear as corrupted and are discarded.

**Storage**: Messages are stored and available for the duration of their specified time to live.

**End-to-end encryption**: Messages (with the exception of friend requests) maintain the properties of the Off the Record (OTR) messaging protocol, namely Perfect Forward Secrecy and Deniable Authentication.

## In Scope

### Service Node Operators - Passive/Action attacks

Storage of messages in Session is handled by [Service Node](../../../ServiceNodes/SNOverview/) operators. Since the Service Node network is permissionless (only [sufficient stake](../../../ServiceNodes/StakingRequirement/) is required to join), our threat model considers a highly resourced attacker that has limited financial resources and can only run a fraction of the storage network. A dishonest Service Node operator would be able to perform a range of active or passive attacks. Such passive attacks could include passively reading message headers, logging timestamps of when messages were relayed/received, saving the encrypted contents of a message, and assessing the size of a message. Active attacks could include failing to relay messages, failing to store messages, providing clients with modified messages, and refusing to respond to requests for messages belonging to public keys. 

Service Nodes also operate the onion request system and thus could also attack it. Active attacks on the [onion request](../infrastructure/#onion-requests) system could include dropping arbitrary packets, modifying latency between hops, and modifying packets. Malicious Service Nodes would be able to continue performing these active attacks for as long as they continued to pass inter-Service Node tests. Passive attacks may involve a malicious Service Node collecting and storing all data that passes through it and logging all connections with other Service Nodes. 

### Network adversary - Passive attacks

Session’s threat model also considers a local network adversary such as an ISP or local network provider. This adversary can perform passive attacks such as monitoring all traffic it relays, conducting deep packet inspection, or saving relayed packets for later inspection.

## Out of Scope 

Attackers who are out of the scope of Session’s threat model may be able to break some of the protections Session aims to provide. 

### Network Adversary — Active Attacks 

A network adversary could conduct active attacks including corrupting or rerouting packets, or adding delays. These attacks could compromise the storage and retrieval of messages. This is primarily addressed by encrypting data and using onion requests to store and retrieve messages, making targeted attacks by network adversaries difficult.

### Global Passive Adversary

A global passive adversary (GPA) that can monitor the first and last hops in an onion request path could use traffic analysis to reveal the true IP address of a Session client and the destination that Session client is talking to. This potential attack is a property of the onion request system; onion requests are a low-latency onion routing network, meaning that packets are forwarded to their destinations as fast as possible, with no delays or batching. This behaviour, while beneficial for user experience, makes traffic analysis trivial in the case of a GPA.

### Out of Band Key Discovery 

Session cannot protect users from exposing the pseudonymity provided by the public key-based account system. If a user associates their real world identity with their public key, then other parties will be able to discover if they receive new friend requests.