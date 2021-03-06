NFC cards for blockchain use
============================

NFC chips are widely used for various kinds of human interaction:
payment cards, digital ID, fidelity cards, access badges, travel fares,
and so on.

A large number of people are carrying a few NFC cards in their
wallets. Also there are numerous other form factors on the market, such
as key fobs, stickers, and tags, bearing an NFC chip inside. They can
easily be purchased in bulk at less than 10 cents a piece.

This article proposes some designs and workflows where NFC chip would be
utilized together with an EOSIO blockchain.

*WARNING*: the NFC chips in VISA or Mastercard credit cards are not very
secure. One can read out the card number, cardcolder name, expiration
date, and a list of latest transactions from the card with a basic NFC
reader and some software skills. Therefore it is highly recommended NOT
to use credit cards for any of purposes listed below.


Using UID/NUID as user identifier
---------------------------------

Each ISO/IEC 14443-3 compliant NFC chip has a read-only block 0 of
memory with a card identifier consisting of 4, 7, or 10 bytes. The
4-byte identifiers are typical, and are not guaranteed to be unique.

There are also NFC chips on the market with rewritable block 0.

Also some chips contain a manufacturer's signature, confirming their
authenticity.

So, the UID can be used as a user identifier for tasks that do not
require strong authentication. For example, customer fidelity points at
a cafe or restaurant are of little material value, not worth the effort
of faking or duplicating the card ID.


### Use case 1: fidelity points

A customer pays at a crypto-friendly shop or cafe, and taps one of their
NFC cards on the NFC reader. This can be any NFC-enabled card that the
customer choses to use as fidelity identifier.

The reader program reads block 0 from the chip, and determins the type
of the card and length of the UID. If possible, it also tries to read
other read-only fields, such as manufacturer signature.

Retrieved bytes are hashed with SHA256, and first 64 bits are taken as
user identfier on the blockchain.

Optionally, the hash can be signed with the shop's private key, and the
signature bits would be used as user identifier. This protects other
merchants from knowing that the user has fidelity points from a certain
merchant.

So, we get a 64-bit identifier which is not completely protected from
collisions, and not protected from attacks, but still providing a good
and low-cost means to identify a user.

Once the user ID is calculated, the reader program performs a lookup on
the blockchain if such user has their own eosio account. If it is the
case, fidelity tokens are transferred to the user account.

If the user does not have an account on the blockchain, a smart contract
maintains a table of user balances. Whenever the user tells the
shopkeeper their eosio account, the fidelity tokens would automatically
be transferred at the next purchase.

The main benefit of such a schema is low cost and ease of use for the
customer. Almost everyone has at least one NFC card in their pocket, and
the shop keeper can easily give out NFC items, such as key fobs or tags.

The main drawback is that there's no self-service for the user: the NFC
reader should be phyically controlled by the shopkeeper. If a user wants
to do any operation at home, we do not have any means to avoid spoofing.


NFC Tags
--------

NFC Forum tag type 2, such as NXP NTAG215, can carry about 400 bytes of
information. Typically such tags contain an URL that a smartphone would
open automatically, or a contact card.

The information on these tags can be protected by password, or the whole
tag can be locked and become immutable.

### Use case 2: transaction request tags

A user taps an NFC tag (or scans a QR code), and the smartphone opens an
URL. This URL downloads an EOSIO transaction request in a JSON file,
accompanied by a special MIME type. This MIME type must be recognized by
wallets as a transaction request. It would contain the following fields:

* Chain ID to define for which EOSIO blockchain the request is made;

* Contract name, action name;

* Action arguments, with the user's accout name indicated as `$USER`.

The wallet software interprets the data, substitutes the user account
name, and prompts the user to approve the transaction, showing all the
details.

Only one action is allowed per request, in order to let the user have a
full understanding of what is being requested.

NFC tags also allow adding the chip's UID and an incrementing read
counter to the URL. So, the server may utize this information and also
use it in the transaction request arguments.

At the moment there is no common MIME type that would be recognized by
wallets, and the data format is not defined.

Some work in this direction has been done by the following projects:

* https://github.com/greymass/eosio-uri

* https://github.com/southex/SimpleWallet





Cards with encryption capabilities
----------------------------------

MIFARE Classic chips are most common and widespread. Unfortunately their
built-in cryptography algorithm is insecure, and it takes minutes on a
common computer hardware to crack the protected information. Thus these
cards are not suitable for anything serious, except for using their UID
as described above.


MIFARE DESFire EV1 and EV2 chips provide secure communication with
AES-128 encryption: a shared key is programmed on the card and is known
to the reader. The reader and the card establish a challenge-response
session that verifies that both sides have the same encryption key. As a
result of challenge-response communication, a random session key is
generated and used to read or write the data on the card.

Secure area on the card may contain a number of data or value files. A
data file can be read or written. A value can be incremented or
decremented in a transactional manner.

Such cards are available on the market at approximately $1 a piece.

It wouldn't be wise to store a private key in such securre area, because
the key would be transmitted to the reader device. This may lead to a
leakage of the private key if the device is tampered.


### Use case 3: strong authentication on a trusted terminal

The DESFire card is used to identify the user without supervision
(contrary to the example above where user authentication has to be
happening in the presense of shopkeeper). Also this identification needs
to be possible in an offline, isolated environment.

The AES-128 encryption key is a combination of the 7-byte UID field and
a PIN number that is known to the user only.

Protected area on the card contains a user ID (such as eosio account
name), and a digital certificate of trusted authority confirming the
identity.

Once the user taps a terminal with the NFC card, the terminal asks for
the PIN. Then using the card UID and the PIN, the terminal derives the
AES-128 password and retrieves encrypted data.

Once the authentication has been successful, the terminal can perform an
action on the blockchain, such as timestamping the user presence.

The DESFire chip is configured with maximum number of authentication
attempts, and locks itself up automatically in case of too many
unsuccessful attempts. This protects the card from brute force attacks.

The terminal still needs to be a physically trusted device: the
information stored on the card is not completely secret, and it's
possible to fake the data exchange if the terminal is tampered.


### Use case 4: sequential offline timestamping

In this scenario, a worker needs to visit a number of locations as a
regular routine, in a defined sequence. The terminals in the field are
standalone and not connected to the network.

Like in the above use case, the encryption key is a combination of UID
and user PIN.

Once the user authenticates, the terminal adds a record to the card
containing its digital signature of the event.

The next terminal in the sequence verifies the previous terminal's
signature, thus guaranteeing that the worker performs the full routine
in due sequence.

The last terminal in the sequence is online, and sends the whole chain
of events to the blockchain as a proof of performed work.



JCOP (Java Card)
----------------

A JCOP card is an advanced and secure microcontroller capable of running
applications in the chip. It is also able to store a private key
securely and issue ECC signatures. The card is designed so that the
private key is nonextractable.

JCOP cards are available on the market for about $4 to $20 a piece.

A typical scenario would be a terminal preparing an eosio transaction,
and the user card generating the ECC signature before broadcasting the
transaction to the blockchain. The chip can also be programmed to ask
user a PIN before issuing the signature.

One potential attack vector is that the chip does not have any display,
so the terminal may show a transaction that is different from what the
user is really signing. So, the terminals need to be certified and
secured.



### Use case 5: signature tag

A JCOP applet is initialized with the following parameters:

* domain ID: a 32-bit integer;

* card ID: a 32-bit integer that is supposed to be unique within the
  domain.

During initialization, the applet generates an ECC private key and sets
the event counter to zero.

Initialization is only accepted if signed by administrative private key.

It is possible to re-initialize, and the old private key is destroyed in
that case.

After initialization, the NFC chip is either attached to a specific
equipment, or given to a person.

When an NFC reader gets in contact with the chip, it sends a 32-byte
hash to rhe chip, and the chip returns the information as follows:

* event counter which is incremented by one;

* domain ID and card ID

* ECC signature for SHA256 of concatenated input hash, domain ID, card
  ID, and the event counter.

The NFC reader can also issue a command that returns the chip's public
key, domain ID, card ID, and the current event counter.

Note that the applet does not verify the input and signs whatever the
other side supplies. So, it cannot be used, for example, for secure
payments. 

The purpose if this applet is to have the chip physically attached at a
specific location, or to a specific piece of equipment. Depending on
requirements, it can even be glued onto something.

A worker or an operator needs to visit a place to do some maintenance
work. The worker carries a mobile terminal with NFC reader. Once the
worker has done the job, he or she collects the measurements or other
important data, also a timestamp, and creates a report document. A hash
of this document is signed by the applet attached to the serviced
equipment. Upon returning, the worker submits the signature to the
blockchain.

The event counter prevents the worker from collecting reports in advance
for the whole year. As described above, the maintenance procedure can
include several checkins at different locations, and starting and
finishing checkin would be done against a chip that is observed by
independent parties.

If accompanied with a 2-factor authentication, such as SMS verification,
this applet can also serve as user identity, and provide an electronic
KYC certificate.



### Use case 6: backup for mobile wallet

Mobile smartphones are often used as transaction signing devices. But
they are poorly protected from environment factors, such as water,
mud, or dropping onto a hard surface.

A JCOP smart card can serve as a backup for a mobile wallet. many
different options are available, for example:

1. Storing a copy of private key: the mobile wallet generates a
keypair, and verifies that a trusted JCOP wallet is connected via
NFC. Then a copy of the private key is transported to the card via
secure channel. In case of emergency, the JCOP applet would be able to
sign transactions.

2. Generating a second keypair: EOSIO accounts allow multiple keys for
every permission, a mobile wallet can request the JCOP card to
generate a keypair, and add the card's public key to the account
permissions when a new account is being created. In case of a loss of
the phone, the card would be able to sign transactions (for example, a
transaction that replaces the public keys).

3. Backup with 2-factor authentication: the JCOP card stores a copy of
the private key securely, and it stores an x509 certificate of a
trusted 3rd party authority. Then the recovery procedure would require
a valid x509 certificarte on the reader device. So, if the user has
got a new phone, he or she requests a new cerficate from the trusted
authority by using some kind of authentication or ID check. By
presenting this certificate, the mobile phone would be able to extract
the private key from the JCOP card and continue using the mobile
wallet.











## Copyright and License

Copyright 2019 cc32d9, EOS Amsterdam.

This work is licensed under a Creative Commons Attribution 4.0
International License.

http://creativecommons.org/licenses/by/4.0/
