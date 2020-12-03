title: Session Docs | Session | Infrastructure
description: Session follows one of two distinct cases for message routing, depending on the availability of participating clients, Asynchronous Routing and Synchronous Routing.

# Message Routing

Session follows one of two distinct cases for message routing, depending on the availability of participating clients:

## Asynchronous (Offline) Routing 

By default, or when either of the participating clients' statuses is determined as offline (see Synchronous Routing for how client status is determined), Session will use asynchronous routing. In asynchronous routing, the sender determines the recipient's swarm by obtaining the deterministic mapping between the recipient's long-term public key and the currently registered Service Nodes. This information is initially requested from a random Service Node by the sender and updated whenever the client gets an error message in the response that indicates a missing swarm.

Once this mapping is determined, the sender creates the message protobuf and packs the protobuf in an envelope with the information to be processed by Service Nodes: the long-term public key of the recipient, a timestamp, TTL ("time to live") and a nonce which proves the completion of the required proof of work (see Attacks — Spam). The sender then sends the envelope using an onion request to one or more random Service Nodes within the target swarm (in practice, each request is always sent to 3 service nodes to achieve a high degree of redundancy). These Service Nodes then propagate the message to the remaining nodes in the swarm, and each Service Node stores the message for the duration of its specified TTL.

![Routing 1](../../assets/routing1.png)

Alice uses an onion request to communicate with three random Service Nodes in Bob’s swarm. Bob then uses an onion request to retrieve said message, by talking to three random Service Nodes in his swarm.

* Not shown here is the process of Alice’s message being replicated across Bob’s swarm.

## Synchronous (Online) Routing 

Session clients expose their online status in the encrypted protobuf of any asynchronous message they send. Along with their online status, a sending client also lists a Service Node in their swarm which they are listening to via onion request.

When a Session client receives a message which signals the online status of another client, the receiver sends an onion request to the sender's specified listening node. The recipient also exposes their own listening node to the sender. If this process is successful, both sender and receiver will have knowledge of each others' online status and corresponding listening nodes. Messages may now be sent synchronously through onion requests to the conversing clients' respective listening nodes.

![Routing 2](../../assets/routing2.png)

Alice uses an onion request to send a message to Bob’s listening node. Bob receives this message using an onion request, then sends a message to Alice’s listening node.

Messages sent using this synchronous method do not contain proof of work, and listening nodes do not replicate or store messages. To ensure messages are not lost, receiving clients send acknowledgements after receipt of each message. If either device goes offline, this acknowledgement will not be received, and the client which is still online will fall back to using the above asynchronous method of message transmission.

## Encryption and the Signal Protocol

So far, we have discussed both the transport and storage of messages. However, any secure messaging application also requires message encryption in order to preserve user privacy. In order for messages to maintain perfect forward secrecy (PFS) and deniable authentication, we cannot only encrypt messages using the long-term public keys of each Session client. Instead, Session uses the Signal protocol. 

The Signal protocol allows clients to maintain PFS and Deniable Authentication in an asynchronous messaging context after initially establishing a session using long-term keys. The Signal protocol achieves perfect forward secrecy through an Extended Triple Diffie-Hellman (X3DH) key agreement protocol and the Double Ratchet protocol for deriving message keys.

X3DH works in the following way. Consider clients A and B that want to establish a session. A and B each have a long-term identity key: IK_a, IK_b, respectively. Additionally, each client holds a key signed with their identity key (SK_a, SK_b), that they update on a regular basis.
Finally, each client generates a one-time key (OTK) for every session they want to establish.

Client A can start a session with client B if it obtains a set of B's "prekeys", consisting of IK_b(pub), SK_b(pub), OK_b(pub). A then validates the signature on SK_b, generates an ephemeral key EK_a, and performs a series of Diffie-Hellman derivations:
DH1 = DH(IK_a(sec), SK_b(pub))
DH2 = DH(EK_a(sec), IK_b(pub))
DH3 = DH(EK_a(sec), SK_b(pub))
DH4 = DH(EK_a(sec), OK_b(pub))

The DH components are then concatenated and passed through a key derivation function (KDF) to derive a shared secret key K, which is used to initialise the Double Ratchet:

K = KDF(DH1 || DH2 || DH3 || DH4)

Client A is now ready to start deriving message keys using the Double Ratchet, and thus start communicating with B. In the first message that it sends, A includes IK_a(pub), EK_a(pub) necessary for B to derive K.

The Double Ratchet uses a chain of Key derivation functions (KDF), each taking the previous chain key and DH parameters communicated by both clients in each of their messages, and producing the next chain key and the actual message key used for encrypting the next message. Even if some message keys get exposed, only the messages related to those keys would be compromised, and the remaining message history would continue to be hidden (the PFS property) as KDF is a one-way function. Additionally, no future messages would be exposed (the “self healing” property) as the potential attacker would be missing the necessary DH parameters to maintain the ratchet.

The Signal protocol obtains deniability through the same scheme by allowing for all ephemeral keys used in the scheme to be left unsigned by both parties. This allows any user to create ephemeral keys for any other user, combine those ephemeral keys with their own long term and ephemeral keys to produce plausible yet forged transcripts.

The Signal protocol achieves X3DH in an asynchronous environment through the use of prekeys, which contain the required information to asynchronously calculate the ephemeral keys used in the X3DH protocol. In the case of the Signal application, prekeys are stored on a central server, ensuring that these prekeys are available even when a user's device is offline.

### Modifications to the Signal Protocol

Session does not modify the fundamentals of the Signal protocol. However, in order to avoid using centralised servers, we have made some changes to the sharing of prekey bundles. In Session, the sharing of prekey bundles is conducted through the 'friend request' system (see below). We also add additional information to each message, for the purpose of routing the message to its desired recipient and verifying that it was created correctly.

### Friend Requests

Friend requests are sent the first time a client initiates communication with a new contact. Friend requests contain a short message with a written introduction, the sender's prekey bundle, and meta-information like the sender's display name and public key, which the recipient can use to respond. Friend requests are encrypted for the public key of the recipient using ECDH. When a friend request is received, the client can choose whether to accept it. Upon acceptance, the client can use the prekey bundle to begin a session as per the original Signal protocol, and start sending messages asynchronously.
