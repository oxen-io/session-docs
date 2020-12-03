title: Session Docs | Session | Infrastructure
description: At its core, Session is built on the Loki Service Node network, so it is important to understand what this network is, how it functions, and what properties Session derives from it.

# Infrastructure

## Foundations

At its core, Session is built on the Loki Service Node network, so it is important to understand what this network is, how it functions, and what properties Session derives from it.

### Service Nodes

Many projects have attempted to establish decentralised permissionless networks. These projects have often found themselves struggling with a ‘tragedy of the commons’ of sorts, wherein public servers, required for the operation of the network, are under-resourced and overused. This inadvertently causes the network to provide poor service to users, which discourages further use or expansion of the network. 

Conversely, those projects which are able to create large, public permissionless networks find themselves constantly facing questions about the parties that contribute to running that infrastructure. This can be especially damaging when the operation of that infrastructure can adversely affect the privacy, security, or user experience of an application. For example, the Tor network faces constant questions about evidence of Sybil attacks from unknown parties attempting to run large sections of of the public routing network, which could be used to deanonymise users .  

Session seeks to sidestep these questions by using a different type of public access network: a staked routing and storage network called the Loki Service Node network. This network is based on the Loki blockchain, which itself is based on the Cryptonote protocol. Through the integration of a blockchain network, Session creates a financial precondition for anyone wishing to host a server on the network, and thus participate in Session’s message storage and routing architecture. 

Authorisation for a server to operate on the network is attained through the server operator conducting a special staking transaction, which requires that an operator provably lock an amount of Loki cryptocurrency assigned to their node (approximately 18,550 Loki coins; equivalent USD 7,420 dollars as of 10/02/2020). 

This staking system provides a defense against Sybil attacks by limiting attackers based on the amount of financial resources they have available. The staking system also achieves two other goals which further reduce the likelihood of a Sybil attack.

Firstly, the need for attackers to buy or control Loki to run Service Nodes creates a feedback loop of increasing prices to run large portions of the network. That is, as the attacker buys or acquires more Loki and locks it, removing it from the circulating supply, the supply of Loki is decreased and the demand from the attacker must be sustained. This causes the price of any remaining Loki to increase, furthering the feedback loop of increasing prices. Secondly, the staking system binds an attacker to their stake, meaning if they are found to be performing active attacks, the underlying value of their stake can sharply decline as users lose trust in the network, or could be destroyed or locked by the network, in any case increasing the attackers sunken costs. 

The other main advantage of a staked blockchain network is that Service Nodes earn rewards for the work they do. Service Nodes are paid a portion of the block reward minted upon the creation of each new block. This system makes Session distinct from altruistic networks like Tor and I2P and instead provides an incentive linked directly with the performance of a Service Node. Honest node behaviour and the provision of a minimum standard of operation is ensured through a consensus-based testing suite. Misbehaving nodes face the threat of having their staked capital locked, while the previously-mentioned cryptocurrency rewards function as the positive incentive for nodes to behave honestly and provide at least the minimum standard of service to the network. 

### Onion Requests

The other foundational component of Session is an onion routing protocol, referred to as onion requests, which enables Session clients to obfuscate their IP addresses by creating a 3-hop randomised path through the Service Node network. Onion requests use a simple onion routing protocol which successively encrypts each request for each of the three hops, ensuring: 

the first Service Node only knows the IP address of the client and the IP address of the middle Service Node,
the middle Service Node only knows the IP address of the first and last Service Nodes, and
the last Service Node only knows the IP address of the middle Service Node and the final destination IP address for the request.

Each Session client establishes a path on startup, and once established, all requests for messages, attachments and meta information are sent through this path. Session clients establish a path by selecting three random nodes from their Service Node list (see bootstrapping), which contains each Service Node’s IP address, storage server port and X25519 key. Clients use this information to create an onion, with each layer being encrypted with the X25519 key of its respective service node. This onion is sent to the first Service Node’s storage server; this Service Node then decrypts its layer of the onion. When a Service Node unwraps a layer, the destination key for the next node is revealed. The first Service Node decrypts its layer and initialises a ZMQ connection with the specified downstream node. When the onion reaches the final node in the path, that node sends a path build success message backwards through the path, which indicates a successful path built upon its receipt by the client.

Upon receiving the path build success message, the client will encrypt their messages with the X25519 keys of the final destination, be that a Service Node, file server, open group server, or client. The client also includes an ephemeral X25519 key in their request. When the destination server or client receives the request, they decrypt it and generate a response. This response is then sent back down the previously-established path, encrypted for the initial sender’s (the client’s) ephemeral key, so that the client can decrypt this response upon receiving it.

## Building on Foundations 

Onion requests provide a straightforward anonymous networking layer, and the Service Node network  provides an incentivised, self-regulating network of remote servers which provide bandwidth and storage space. A number of services are built on top of this foundation in order to give Session features commonly expected of modern messaging applications.
Storage 

Message storage is an essential feature for any chat application aiming to provide a good user experience. When a user sends a message, they expect the recipient to receive that message even if they turn off their device after the message has been sent. Users also expect the user on the other end to receive the message when their device wakes up from an offline state. Apps that run on decentralised networks typically cannot provide this experience, because of the lack of incentive structures and, consequently, the ephemeral nature of clients and servers on such a network. Session is able to provide message storage through the incentivised Service Node network and its usage of swarms.

### Swarms 

Although the Loki blockchain incentivises correct Service Node behaviour through rewards and punishments, these incentive models cannot prevent nodes going offline unexpectedly due to operator choice, software bugs, or data center outages. Therefore, for redundancy, a secondary logical data storage layer must be built on top of the Service Node network to ensure reliable message storage and retrieval. 

This secondary logical layer is provided by replicating messages across small groupings of Service Nodes called swarms. The swarm a Service Node initially joins is determined at the time of that Service Node’s registration, with the Service Node having minimal influence over which swarm it joins. This protects against swarms being entirely made up of malicious or non-performant nodes, which is important to maintain the network’s self-regulating properties.

Composition of each swarm inevitably changes as the networks evolves: some nodes leave the network and the newly registered nodes take their place. If a swarm loses a large number of nodes it may additionally "steal" a node from some other, larger swarm. In the unlikely event that the network has no swarms to steal from (i.e., every swarm is at Nmin=5 nodes), the ‘starving’ swarm (a swarm with fewer than Nmin nodes) will be dissolved and its nodes will be redistributed among the remaining swarms. Conversely, when a large number of nodes enter the network that would oversaturate existing swarms (i.e., every swarm is already at max capacity Nmax=10), a new swarm is created from a random selection of Ntarget=7 excess nodes. Note that Nmin < Ntarget < Nmax to ensure that a newly generated swarm doesn’t get dissolved shortly after and that there is still room for growth.

The outcome of this algorithm is the creation and, when necessary, rebalancing of swarms of around Nmin⁠–Nmax Service Nodes which store and serve Session clients’ messages. The goal of the swarm algorithm is to ensure that no swarm is controlled by a single entity and that the network is resilient enough to handle both small and large scale events where Service Nodes are no longer contactable, ensuring data integrity and privacy in both cases.

The following set of simple rules ensure that Service Nodes within swarms remain synchronised as the composition of swarms changes:

When a node joins a new swarm, existing swarm members recognise this and push the swarm’s data records to the new member.
When a node leaves a swarm, its existing records can be safely erased, with the exception of when the node is migrating from a dissolving swarm. In  this case, the migrating node determines the swarms responsible for its records and distributes them accordingly.

### Identity and Long-Term Keys

The majority of popular messaging applications require the user to register with an email or phone number in order to use the service. This provides some advantages, including account verification, for purposes of spam protection, and social network discoverability. However, such requirements also create some major privacy and security issues for users. 

The use of a phone number as the basis for ownership of an identity key/long-term key pair weakens security against user accounts being compromised, such as in the cases of popular applications like Signal and WhatsApp. This weakness primarily stems from the fact that phone numbers are managed by centralised service providers (i.e. telecommunications service providers) who can circumvent user control, allowing these providers to assume direct control of specific users’ numbers. Widespread legislation already exists to compel service providers to take this kind of action. Additionally, methods such as SIM swapping attacks, service provider hacking, and phone number recycling can all be exploited by lower-level actors .

Signal and Whatsapp put forward varying degrees of protection against these types of attacks. Signal and WhatsApp both send a 'Safety numbers have changed' warning to a user's contacts if identity keys are changed. In practice, however, users rarely verify these details out-of-band . 

Both Signal and WhatsApp also allow users to set a "registration PIN lock"  . This protection means that an attacker (including a service provider or state-level actor) needs access to both the phone number and the registration PIN code to modify identity keys. However, this feature is off by default, difficult to find in the settings menu, and automatically disabled after periods of user inactivity. These factors all significantly reduce the efficacy of registration PIN locks as a protective measure against the security risks of phone number-linked accounts.

Using phone numbers as the basis for account registration also greatly weakens the privacy achievable by an average user. In most countries, users must provide personally identifiable information such as a passport, drivers' license or identity card to obtain a phone number — permanently mapping users’ identities to their phone numbers.These identity mappings are kept in private databases that can be queried by governments or the service providers that own them. There are also a number of web scrapers and indexers that automatically scrape phone numbers associated with individuals. These scrapers may target sources such as leaked databases, public social media profiles, and business phone numbers to link people to their phone numbers. Since the only method of initiating contact with a user in Signal, WhatsApp, or similar application is to know the user’s phone number, this immediately strips away user anonymity — a significant concern for whistleblowers, activists, protestors and other such users.

Account systems based on phone numbers also limit the potential for the establishment of multiple identities by a single user. These systems also prevent high-risk users without access to a phone number from accessing these services.

Session does not use email addresses or phone numbers as the basis of its account system. Instead, user identity is established through the generation of X25519 public-private key pairs. These key pairs are not required to be linked with any other identifier, and new key pairs can be generated on-device in seconds. This means that each key pair (and thus, each account) is pseudonymous, unless intentionally linked with an individual identity by the user through out-of-band activity.

### Restoration

Because Session does not have a central server to keep records of user identities, the commonly expected user experience of being able to recover an account using a username and password is not possible. Instead, users are prompted to write down their long-term private key (represented as a mnemonic seed, referred to within Session as a recovery phrase) upon account generation. A user can use this backup key to recover their account if their device is lost or destroyed, and the user's contacts will be able to continue contacting that same user account, rather than having to re-initiate contact with a new key.