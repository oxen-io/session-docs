title: Session Docs | Session | Client Side Protection
description: Secure messaging applications have typically focused their development efforts towards providing  protections against network and server level adversaries, which has led to new advances in encryption and metadata protections.

# Client-Side Protections 

Secure messaging applications have typically focused their development efforts towards providing  protections against network and server level adversaries, which has led to new advances in encryption and metadata protections. However, when interviewing high risk individuals researchers, it has been found that client-side privacy and security protections are some of the most-requested features. High-risk individuals may not be focused on protecting themselves against global adversaries, but instead against a small nation state, or corporate entity. For these individuals, endpoint compromise, device seizure, and forced disclosures are described as the biggest risks. To better mitigate these risks, Session implements a number of client-side protections which allow users to better manage the security of the Session app on their device.

## Deletion

Granular message and data deletion controls are important for users who are likely to have their devices physically sized. Session implements standard features like disappearing messages, which are deleted from sending and receiving clients after being viewed, and the ability to fully wipe all client side stored data. However, [Session](https://getsession.org) also features additional ways to manage client side security. 

## Duress Codes 

Users may set a PIN or pattern lock to access the Session app, which adds additional security on top of any device-level passcodes. As an additional layer of security, users may also specify a duress code, which if entered in lieu of the standard Session app PIN, will wipe Session app data on the device. This is useful in cases where users are forced to unlock their devices and wish for it to appear as if there was never data to begin with. 

## Remote Deletion

Remote deletion allows a user to specify a trusted friend and negotiate a shared secret with that friend. Once this secret is generated and stored on the device, the trusted friend can generate a remote deletion message which reveals this prearranged secret. When this message is received by the user’s device, it initiates the immediate destruction of their local database.
Pseudonyms

High-risk users such as whistleblowers often need to create accounts which are not linked to any real-world physical identifiers (e.g. phone numbers and email addresses). Session account creation only requires generation of a public-private key pair, making it trivial for users to establish multiple pseudonyms without needing to link their account to pieces of information which could be used to identify them.  

## Backup and Restore Account States  

Border crossings or checkpoints can be an area of significantly increased risk for high-risk users. In these zones, high-risk users may be forced to disclose passwords and surrender devices so device images can be taken. To protect their data, some high-risk individuals have begun implementing a strategy of backing up device and application data, wiping their device to cross a border or pass through a checkpoint, and then restoring that data once it is safe to do so. To ease this process, Session supports encrypted backups to a number of popular cloud services. Backups are encrypted with a symmetric key derived from the user’s Session long-term private key, meaning the user only needs knowledge of their 12 word mnemonic seed (recovery phrase) to recover their account after completing the border or checkpoint crossing. 
