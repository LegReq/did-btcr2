## CRUD Operations

This section defines the Create, Read, Update, and Deactivate (CRUD) operations
for the **did:btcr2** method.

### Create

Creating a **did:btcr2** identifier is entirely offline, requiring no innate
network interactions or onchain anchoring transactions to generate a new identifier. 
Each creation starts either with a public key or a ::Genesis Document::.  
Both creation algorithms first create the ::Genesis Bytes:: that commit 
to an ::Initial DID Document::.

To create a **did:btcr2** identifier from a public key without an Initial DID
Document, use [Algo 1: Create Genesis Bytes from Public Key], then encode those
bytes along with a version, network for the identifier and an identifier type of 
**“key”** as in [Algo 3: Encode Identifier].

To create a **did:btcr2** identifier from a ::Genesis Document::, use
[Algo 2: Create Genesis Bytes from Genesis Document], then encode those
bytes along with a version, network for the identifier and an identifier type of
“external” using [Algo 3: Encode Identifier].

The output of encoding the identifier is the newly created DID.

Note: When creating from a ::Genesis Document::, it is likely that creators
will want to include information, such as ::Beacons:: and other service
endpoints. While services such as ::BTCR2 Beacons:: may require network interactions, 
e.g., to establish a unique ::Beacon Address:: for updates, they do not 
require onchain anchoring. 

#### Algo 1: Create Genesis Bytes from Public Key {.tabbed .unnumbered}

The ::Genesis Bytes:: is a 33-byte compressed representation of a
[secp256k1 public key](https://www.secg.org/sec2-v2.pdf) following the encoding
defined in the [Standards for Efficient Cryptography](https://www.secg.org/sec1-v2.pdf)
(SEC encoding).

##### Hide {.unnumbered .unlisted}

##### Examples {.unnumbered .unlisted}

##### Example code {.unnumbered .unlisted}

#### Algo 2: Create Genesis Bytes from Genesis Document {.tabbed .unnumbered}

The ::Genesis Bytes:: is a 32-byte [SHA256](https://datatracker.ietf.org/doc/html/rfc6234)
hash of an input ::Genesis Document:: canonicalized using the
[JSON Canonicalization Scheme](https://www.rfc-editor.org/rfc/rfc8785). The
::Genesis Document:: is an intermediate representation of an
::Initial DID Document:: with the identifier values replaced with a placeholder
value. The placeholder value MUST be
did:btcr2:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx. This is
‘did:btcr2:’ followed by 60 ‘x’s, one for each character in the method-specific
identifier.

In order for this DID to be updatable, controllers must include at least one
verification method with a [capability invocation](https://www.w3.org/TR/cid-1.0/#capability-invocation) verification relationship and
at least one ::BTCR2 Beacon:: service.

Controllers may also add content to the ::Genesis Document::, including keys and
services.

It is RECOMMENDED that controllers add at least one ::Singleton Beacon:: to
provide a fallback update capability. This ensures the controller can update the
DID without reliance on any ::Beacon Aggregators:: or other parties.

##### Hide {.unnumbered .unlisted}

##### Flowchart {.unnumbered .unlisted}

##### Examples {.unnumbered .unlisted}

##### Imperative Algorithm {.unnumbered .unlisted}

Inputs:

* `genesisDocument` - any intermediate representation of an initial DID document
  with the identifier replaced with the placeholder value throughout all fields
  (e.g., the id field) with
  `did:btcr2:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`.
  REQUIRED; object.

Outputs:

* `genesisBytes` - a newly created **did:btcr2** identifier;
  string

The steps are as follows:

1. Set `canonicalizedDocument` to the result of passing `genesisDocument` into
   the JSON Canonicalization Scheme.
2. Set `genesisBytes` to the SHA256 hash of the `canonicalizedDocument`.
3. Return `genesisBytes`.

##### Example code {.unnumbered .unlisted}

#### Algo 3: Encode Identifier {.tabbed .unnumbered}

The identifier uses a [bech32m](https://en.bitcoin.it/wiki/BIP_0350#Bech32m)
encoding of input bytes composed of the version, network and ::Genesis Bytes::.
The value of the version MUST be 1. The value of the network declares which
Bitcoin network anchors the identifier and MUST be selected from the table below.

The input bytes to the bech32m algorithm are constructed as follows: the first
byte is the version and the network, with the version minus one in the low
nibble and the value from the network table in the high nibble. The Genesis
Bytes are then appended to the first byte to produce the input bytes. Encode the
input bytes using the bech32m algorithm with the human-readable part (hrp) value
set to the ASCII value of either ‘k’ or ‘x’, depending on the type of the
identifier. For btcr2 DIDs generated from an initial secp256k1 public key, use
‘k’ for *did:btcr2* identifiers generated from an Initial DID Document, use ‘x’. The result
of the encoding is the method-specific identifier. Prepend the method-specific
identifier with the ASCII string ‘did:btcr2:’ to create the DID.

NOTE: In future versions of this algorithm, it is expected that the version
could take up more than one nibble with the nibble set to F indicating that the
next nibble should be considered a part of the version.

| Network          | Value |
|:-----------------|:------|
| bitcoin          | 0     |
| signet           | 1     |
| regtest          | 2     |
| testnet3         | 3     |
| testnet4         | 4     |
| mutinynet        | 5     |
| reserved         | 6-B   |
| custom network 1 | C     |
| custom network 2 | D     |
| custom network 3 | E     |
| custom network 4 | F     |

Note: The values 6 through B are reserved by the specification for future use, such as identifying 
new Bitcoin test networks as they become adopted.

##### Hide {.unnumbered .unlisted}

##### Flowchart {.unnumbered .unlisted}

##### Examples {.unnumbered .unlisted}

##### Imperative Algorithm {.unnumbered .unlisted}

Inputs:

* `idType` - required, one of:
    * “key”
    * “external”
* `version` - required, number
* `network` - required, one of:
    * “bitcoin”
    * “signet”
    * “regtest”
    * “testnet3”
    * “testnet4”
    * “mutinynet”
    * “custom network 1”
    * “custom network 2”
    * “custom network 3”
    * “custom network 4”
* `genesisBytes` - required, byte array, one of:
    * a compressed secp256k1 public key if `idType` is “key”
    * a hash of an initiating external DID document if `idType` is “external”

Output:

* identifier - a **did:btcr2** identifier

Encode the **did:btcr2** identifier as follows:

1. If `idType` is not a valid value per above, raise `invalidDid` error.
2. If `version` is greater than `1`, raise `invalidDid` error.
3. If `network` is not a valid value per above, raise `invalidDid` error.
4. If `idType` is “key” and `genesisBytes` is not a valid compressed secp256k1
   public key, raise `invalidDid` error.
5. Map `idType` to `hrp` from the following:
    1. “key” - “k”
    2. “external” - “x”
6. Create an empty `nibbles` numeric array.
7. Set `fCount` equal to `(version - 1) / 15`, rounded down.
8. Append hexadecimal `F` (decimal `15`) to `nibbles` `fCount` times.
9. Append `(version - 1) mod 15` to `nibbles`.
10. If `network` is a string, append the numeric value from the following map to
    `nibbles`:
    1. “bitcoin” - `0`
    2. “signet” - `1`
    3. “regtest” - `2`
    4. “testnet3” - `3`
    5. “testnet4” - `4`
    6. “mutinynet” - `5`
    7. “custom number 1” - C
    8. “custom number 2” - D
    9. “custom number 3” - E
    10. “custom number 4” - F
11. If the number of entries in `nibbles` is odd, append `0`.
12. Create a `dataBytes` byte array from `nibbles`, where `index` is from `0` to
    `nibbles.length / 2 - 1` and
    `encodingBytes[index] = (nibbles[2 * index] << 4) | nibbles[2 * index + 1]`.
13. Append `genesisBytes` to `dataBytes`.
14. Set `identifier` to “did:btcr2:”.
15. Pass `hrp` and `dataBytes` to the
    [bech32m](https://en.bitcoin.it/wiki/BIP_0350#Bech32m) encoding algorithm,
    retrieving `encodedString`.
16. Append `encodedString` to `identifier`.
17. Return `identifier`.

##### Example code {.unnumbered .unlisted}

### Resolve

Resolving a **did:btcr2** identifier takes an input DID and returns the
canonical DID document at the requested target time as well as additional
metadata in a DID Resolution Result. The current time is the default target
time for a resolution request.

Resolution takes part in two steps.

First, establish the ::Initial DID Document::. Start processing the resolution
inputs, parsing the identifier and any resolution options using [Algo 4. Process
Resolution Inputs]. Then establish the ::Initial DID Document:: according to the
type of the identifier. If the identifier encodes a 33-byte compressed secp256k1
public key, the ::Initial DID Document:: can be generated by following [Algo 5.
Deterministically Generate Initial DID Document]. Otherwise, the identifier must
encode a 32-byte hash of a ::Genesis Document::, in this case first retrieve the
::Genesis Document:: using [Algo 6. Retrieve Genesis Document] and then verify
this document according to [Algo 7. Verify Genesis Document].

Second, iterate through ::Beacon Signals:: posted to the Bitcoin blockchain
using [Algo 8. Process Beacon Signals] processing them according to their
::Beacon Type::. Processing Beacon Signals retrieves the announced ::BTCR2
Update:: for the DID being resolved. The algorithms for processing ::Beacon
Signals:: according to their ::Beacon Types:: are as follows: for ::Singleton
Beacons:: use [Algo 9. Process Singleton Beacon Signal];  for ::Map Beacons::
use [Algo 10. Process Map Beacon Signal]; for ::SMT Beacons:: use [Algo 11.
Process SMT Beacon Signal].

::BTCR2 Updates:: announced with ::Beacon Signals:: are then applied to the DID
document. ::BTCR2 Updates:: must be invocations of the capability to update the
DID being resolved. To verify this, the resolver must first derive the root
capability from the DID itself using  [Algo 12. Derive Root Capability], then
verify and apply the invocation using [Algo 13. Apply BTCR2 Update].

Traverse the blockchain and apply updates in temporal order, updating and
maintaining a ::Contemporary DID Document:: until all updates are processed.
At each stage, the Contemporary DID Document defines the ::BTCR2 Beacons::
active at that point in time; those Beacons – and only those Beacons – are checked
for updates.

Continue traversing the chain until all timely, authentic ::Beacon Signals::
have been processed and their announced updates have been applied in order.

The resulting DID document is the canonical DID Document and must be returned to
the caller of the resolver function in a resolution response which can be
constructed following [Algo 14. Construct Resolution Result].

NOTE. If there are no updates, the ::Initial DID Document:: is returned unless a specific versionId has been targetted by the resolution request.

#### Algo 4. Process Resolution Inputs {.tabbed .unnumbered}

This algorithm processes the resolution inputs from a resolution request. First,
parse the identifier string, then process the resolution options.

First take the identifier string, validate it according to the DID syntax and
decode it to retrieve the identifier type, version, network and ::Genesis
Bytes::.

The identifier string must consist of three components separated by a ‘:’
character. The first component must be the string ‘did’, the second component
must be ‘btcr2’ and the third component is the BTCR2-specific identifier.
Otherwise an INVALID_DID error MUST be raised.

The BTCR2-specific identifier is decoded using the [bech32m](https://en.bitcoin.it/wiki/BIP_0350#Bech32m) algorithm which
returns the human readable part (*hrp value*) and the encoded data bytes. The
encoded data bytes must be parsed as follows to retrieve the version, network,
and ::Genesis Bytes::. The low nibble of the first byte indicates the version;
the high nibble of the first byte indicates the specific Bitcoin network used
to anchor the identifier, and all bytes after the first byte are the ::Genesis
Bytes::.

To get the version, add one to the value in the low nibble of the first byte.
At this time, the version must be 1. Any other value results in an INVALID_DID
error.

To get the network, look up the value from the high nibble of the first byte in
this table.

| Value | Network          |
|:------|:-----------------|
| 0     | bitcoin          |
| 1     | signet           |
| 2     | regtest          |
| 3     | testnet3         |
| 4     | testnet4         |
| 5     | mutinynet        |
| C     | custom network 1 |
| D     | custom network 2 |
| E     | custom network 3 |
| F     | custom network 4 |

Any value not in the table results in an INVALID_DID Error.

To validate the ::Genesis Bytes::, check the hrp value. If hrp is ‘k’, the
::Genesis Bytes:: must be a 33-byte representation of a compressed secp256k1
public key. If hrp is ‘x’, the ::Genesis Bytes:: must be a 32-byte SHA256 hash
of a ::Genesis Document::. If these conditions are not met, or hrp has any other
value, it results in an INVALID_DID Error.

Then the resolution options of a resolution request must be processed and the
following fields must be used to execute the request:

* versionId: The versionId of the DID document targeted by this resolution
  request. If there is a DID document with this versionId, it MUST be returned
  in the resolution result.
* versionTime: A UTC timestamp that is targeted by the resolution request. The
  resolve MUST return the contemporary DID document at this time as determined
  by the Bitcoin block time.
* verbose: A boolean value. This client is requesting additional detail as to 
  how the resolution result was constructed.
* sidecar: The set of ::Sidecar Data:: provided to the Resolver by the client as
  part of their resolution request. This data is likely provided to the client
  by the DID controller.
  * documents: An array of JSON documents that must be used to execute the
    resolution request. ::Sidecar:: documents are identified within ::Beacon
    Signals:: by their hash, this should be used to retrieve the relevant
    document from ::Sidecar Data::.
  * smtProofs: An array of SMT proofs that prove inclusion or non-inclusion of a
    ::BTCR2 Update:: announced within an SMT Beacon Signal. Each proof must have
    an id field which is the hex encoded merkle root that is included as the
    Signal Bytes of the Beacon Signal.

##### Hide {.unnumbered .unlisted}

##### Imperative Algorithm {.unnumbered .unlisted}

Given:

* `identifier` - required, a string **did:btcr2** identifier

Decode the **did:btcr2** identifier as follows:

1. Split `identifier` into an array of `components` at the colon `.` character.
2. If the length of the `components` array is not `3`, raise `invalidDid` error.
3. If `components[0]` is not "did", raise `invalidDid` error.
4. If `components[1]` is not "btcr2", raise `methodNotSupported` error.
5. Set `encodedString` to `components[2]`.
6. Decode the `encodedString` using the Bech32m Decoding algorithm defined in [BIP350](https://github.com/bitcoin/bips/blob/master/bip-0350.mediawiki), retrieving `hrp`
   and `dataBytes`.
7. If the decoding algorithm fails, raise `INVALID_DID` error.
8. Map `hrp` to `idType` from the following:
   1. "k" - "key"
   2. "x" - "external"
   3. other - raise `INVALID_DID` error
9. Set `version` to `1`.
10. If at any point in the remaining steps there are not enough nibbles to
    complete the process, raise `INVALID_DID` error.
11. Start with the first nibble (the higher nibble of the first byte) of
    `dataBytes`.
12. Add the value of the current nibble to `version`.
13. If the value of the nibble is hexadecimal `F` (decimal `15`), advance to the
    next nibble (the lower nibble of the current byte or the higher nibble of
    the next byte) and return to the previous step.
14. If `version` is greater than `1`, raise `INVALID_DID` error.
15. Advance to the next nibble and set `networkValue` to its value.
16. Map `networkValue` to `network` from the following:
    1. `0` - "bitcoin"
    2. `1` - "signet"
    3. `2` - "regtest"
    4. `3` - "testnet3"
    5. `4` - "testnet4"
    6. `5` - "mutinynet"
    7. `6`-`B` - raise `INVALID_DID` error
    8. `C`-`F` - `networkValue - 11`
17. If the number of nibbles consumed is odd:
    1. Advance to the next nibble and set `fillerNibble` to its value.
    2. If `fillerNibble` is not `0`, raise `invalidDid` error.
18. Set `genesisBytes` to the remaining `dataBytes`.
19. If `idType` is "key" and `genesisBytes` is not a valid compressed secp256k1
    public key, raise `INVALID_DID` error.
20. Return `idType`, `version`, `network`, and `genesisBytes`.

#### Algo 5. Deterministically Generate Initial DID Document {.tabbed .unnumbered}

This algorithm deterministically generates an ::Initial DID Document:: from a
33-byte compressed representation of a secp256k1 public key encoded into the
identifier as the ::Genesis Bytes::. The DID document generated must be a
conformant DID document according to the DID core v1.1 specification.

The DID document must contain only the following properties:

* A \@context array containing the DID core v1.1 context url ("[https://www.w3.org/ns/did/v1.1](https://www.w3.org/ns/did/v1.1)" )
  and the did:btcr2 context ("[https://btcr2.dev/context/v1](https://btc1.dev/context/v1)").
* A verificationMethod array containing a single verificationMethod. This
  verificationMethod must have an id value of "#initialKey" be of the type
  Multikey with the publicKeyMultibase value a multikey encoding of the
  secp256k1 public key bytes.
* An array containing the verificationMethod id ‘#initialKey’ for each of the
  verification relationships. assertionMethod, authentication,
  capabilityInvocation, capabilityDelegation
* A service array containing three BTCR2 Beacon services. The serviceEndpoint
  for each of these services must be a BIP21 URI encoding of a bitcoin address
  controlled by the public key. The three different address formats are. p2pkh,
  p2wpkh, p2tr. The id value of the service must be "#initialP2PKH",
  #initialP2WPKH” and "#initialP2TR" respectively. Finally, each of these
  services must have a type set to the string "SingletonBeacon"

##### Hide {.unnumbered .unlisted}

##### Imperative Algorithm {.unnumbered .unlisted}

The Deterministically Generate ::Initial DID Document:: algorithm generates an
::Initial DID Document:: from a secp256k1 public key.

It takes the following inputs:

* `identifier` - a valid **did:btcr2** identifier; REQUIRED; string.
* `identifierComponents` - The decoded parts of a **did:btcr2** identifier;
  REQUIRED; object.
  * `idType` - the type of identifier (`KEY` or `EXTERNAL`); REQUIRED; string.
  * `version` - the identifier version; REQUIRED; integer.
  * `network` - the Bitcoin network used for the identifier; REQUIRED; string.
  * `genesisBytes` - the originating public key; REQUIRED; bytes.

It returns the following output:

* `initialDocument` - the valid first version of a DID document for a given
  **did:btcr2** identifier.

The steps are as follows:

1. Set `keyBytes` to `identifierComponents.genesisBytes`.
2. Initialize an `initialDocument` variable as an empty object.
3. Set `initialDocument.id` to the `identifier`.
4. Initialize a `contextArray` to empty array:
   1. Append the DID Core v1.1 context "https://www.w3.org/ns/did/v1.1".
   2. Append the did:btcr2 context "https://btcr2.dev/context/v1".
   3. Set `initialDocument.@context` to `contextArray`.
5. Initialize a `controllerArray` to empty array:
   1. Append the `identifier`.
   2. Set `initialDocument.controller` to `controllerArray`.
6. Create an initial verification method:
   1. Initialize `verificationMethod` to an empty object.
   2. Set `verificationMethod.id` to `{identifier}#initialKey`.
   3. Set `verificationMethod.type` to "Multikey".
   4. Set `verificationMethod.controller` to `identifier`.
   5. Set `verificationMethod.publicKeyMultibase` to the result of the encoding
      algorithm in [BIP340 Multikey](https://dcdpr.github.io/data-integrity-schnorr-secp256k1/#multikey).
7. Set `initialDocument.verificationMethod` to an array containing
   `verificationMethod`.
8. Initialize a `tempArray` variable to an array with the single element
   `verificationMethod.id`.
9. Set the `authentication`, `assertionMethod`, `capabilityInvocation`, and the
   `capabilityDelegation` properties in `initialDocument` to a copy of the
   `tempArray` variable.
10. Set the `initialDocument.services` property in `initialDocument` to the
    result of passing the `identifier`, `keyBytes` and
    `identifierComponents.network` to the [Deterministically Generate Beacon
    Services] algorithm.
11. Return `initialDocument`.

#### Algo 6. Retrieve Genesis Document {.tabbed .unnumbered}

This algorithm uses the ::Genesis Bytes:: encoded into an external **did:btcr2**
identifier to retrieve and verify the ::Genesis Document::. This document is
then transformed into the ::Initial DID Document::, which must be validated as a
conformant DID document according to the DID core v1.1 specification.

The ::Genesis Document:: may be retrieved either from the ::Sidecar Data::
provided in the DID resolution options or from a ::Content Addressable Storage::
(CAS) such as IPFS. If retrieving from ::CAS::, the ::Genesis Bytes:: must be
transformed into a ::content identifier:: according to the CID v1.0
specification.

If the ::Genesis Document:: cannot be retrieved, it results in a FILE_NOT_FOUND.
If the ::Initial DID Document:: is not conformant, then it results in an
INVALID_DID error.

#### Algo 7. Verify Genesis Document {.tabbed .unnumbered}

Verifying the ::Genesis Document:: requires checking its hash against the
::Genesis Bytes:: and then validating that it is a conformant document.

To check its hash, compute the SHA256 of the ::Genesis Document:: canonicalized
using the [JSON Canonicalization Scheme](https://www.rfc-editor.org/rfc/rfc8785)
(JCS). The computed hash must be equal to the ::Genesis Bytes::. If it is not,
this is an INVALID_DID_DOCUMENT_CONSTRUCTION error.

To check that it is conformant, transform the ::Genesis Document:: into the
::Initial DID Document:: by replacing the placeholder DID with the **did:btcr2**
identifier being resolved. The transformed document must be a conformant DID
document according to the DID core v1.1 specification. If it is not, this is an
INVALID_DID_DOCUMENT error.

Algo 8. Find & Process Beacon Signals

1. Algo 9. Scan until Signal, Select Block
2. Algo 10. For block, process Signals
    1. For each signal,
        1. Algo 11,12,13 Beacon Specific Algo
        2. Algo 14 Apply Update
3. Continue Scanning until end state

#### Algo 8. Process Beacon Signals {.tabbed .unnumbered}

Traverse the Bitcoin blockchain from the start of the chain, identify the next
Bitcoin block that contains a ::Beacon Signal::. For all ::Beacon Signals:: in
that block, process these signals to retrieve ::BTCR2 Updates:: announced by
these signals. Apply these updates to the DID document to produce the
::Contemporary DID Document:: with that block. Continue traversing the
blockchain until the latest block.

A ::Beacon Signal:: is a Bitcoin transaction that spends at least one ::UTXO::
controlled by a ::Beacon Address:: defined by a Beacon in the service array of
the Contemporary DID Document. The Beacon Signal contains ::Signal Bytes::, 32
bytes included in the last transaction output of the transaction. The script of
this transaction output has the following form. \[OP_RETURN, OP_PUSH_BYTES,
<32 signal bytes>\].

Process ::Beacon Signals:: according to their type, as identified by the Beacon
Service in the Contemporary DID Document. The table below defines the algorithm
you must use to process a Beacon Signal for  a specific ::Beacon Type::.

| ::Beacon Type::      | Processing Algorithm                      |
|:---------------------|:------------------------------------------|
| ::Singleton Beacon:: | [Algo 9. Process Singleton Beacon Signal] |
| ::Map Beacon::       | [Algo 10. Process Map Beacon Signal]      |
| ::SMT Beacon::       | [Algo 11. Process SMT Beacon Signal]      |

Processing ::Beacon Signals:: may require ::Sidecar Data:: passed as part of the
resolution options in the DID resolution request.

The result of processing the ::Beacon Signals:: is a set of ::BTCR2 Updates::.
These updates must then be applied to the Contemporary DID Document using
[Algo 13. Apply BTCR2 Update].

#### Algo 9. Process Singleton Beacon Signal {.tabbed .unnumbered}

This algorithm processes a ::Beacon Signal:: broadcast from a ::Singleton
Beacon:: to retrieve and validate the ::BTCR2 update:: announced by this signal.

For signals from Singleton Beacons, the ::Signal Bytes:: are the SHA256 of a
canonicalized ::BTCR2 Update::. The BTCR2 update committed to by the Signal
Bytes must be retrieved, canonicalized using JCS, and then hashed. The resulting
hash bytes must equal the Signal Bytes.

A BTCR2 Update may be retrieved from ::Sidecar Data::, or through querying a
::CAS::. If querying a ::CAS::, the ::Signal Bytes:: must be transformed into a
::CID:: according to the IPFS CID v1 specification. Inability to retrieve the
BTCR2 Update must raise a MISSING_UPDATE_DATA error.

##### Hide {.unnumbered .unlisted}

##### Imperative Algorithm {.unnumbered .unlisted}

Given the 32 `signalBytes` from the Singleton Beacon Signal and a
`sidecarDocumentsMap`:

1. Set `id` to the hexadecimal string representation of `signalBytes`.
2. Get `btcr2Update` from `sidecarDocumentsMap` by its `id` if available, or
   from ::CAS:: by its `id` if not and `cas` is defined.
3. If `btcr2Update` is undefined, raise InvalidDidUpdate error.
4. Set `btcr2Update`
5. Return `btcr2Update`.

NOTE. The act of retrieving from `sidecarDocumentsMap` or ::CAS:: validates the
document hash.

#### Algo 10. Process Map Beacon Signal {.tabbed .unnumbered}

This algorithm processes a ::Beacon Signal:: broadcast from a ::Map Beacon:: to
retrieve and validate the ::Beacon Announcement Map:: committed to by the
signal. That map is then used to retrieve and validate a ::BTCR2 Update:: using
the specific identifier being resolved as the key.

The ::Signal Bytes:: must be retrieved from the last transaction output of the
Beacon Signal. These bytes must then be used to retrieve and validate the Beacon
Announcement Map. A ::BTCR2 Update Announcement:: must be retrieved from the map
document using the hex encoded SHA256 hash of the identifier being resolved as
the key. There may be no announcement for the identifier. The announcement is a
32-byte hash of the ::BTCR2 Update::, this must be used to retrieve and validate
the ::BTCR2 Update::.

Retrieval of the Beacon Announcement Map and ::BTCR2 Update:: documents may be
done through querying either ::Sidecar Data:: or a ::CAS::. If querying a
::CAS::, the 32-byte hashes must be transformed into a CID following the IPFS
CID v1 specification.

##### Hide {.unnumbered .unlisted}

##### Imperative Algorithm {.unnumbered .unlisted}

Given the 32 `signalBytes` from the Map Beacon Signal and a `sidecarDocumentsMap`:

1. Set `id` to the hexadecimal string representation of `signalBytes`.
2. Get `map` from `sidecarDocumentsMap` by its `id` if available, or from
3. ::CAS:: by its `id` if not and `cas` is defined.
3. If `map` is undefined, raise InvalidDidUpdate error.
4. Set `updateId` to the value of `map.<did>`.
5. If `updateId` is undefined, return null.
6. Get `btcr2Update` from `sidecarDocumentsMap` by its `updateId` if available,
7. or from ::CAS:: by its `updateId` if not and `cas` is defined.
7. If `btcr2Update` is undefined, raise InvalidDidUpdate error.
8. Return `btcr2Update`.

NOTE. The act of retrieving from `sidecarDocumentsMap` or ::CAS:: validates the
document hash.

#### Algo 11. Process SMT Beacon Signal {.tabbed .unnumbered}

This algorithm processes a ::Beacon Signal:: broadcast from a ::SMT Beacon:: to
retrieve and validate a ::BTCR2 Update:: for a specific DID being resolved.

The ::Signal Bytes:: in the Beacon Signal are the root of a ::Sparse Merkle
Tree::. (SMT). This root must be used to verify the contents of the leaf of the
SMT indexed by the SHA256 hash of the DID being resolved.

The contents of this leaf either contains the double SHA256 hash of a nonce, or
it contains the SHA256 hash of the concatenation of the SHA256 hashes of a nonce
and ::BTCR2 Update::. The ::BTCR2 Update:: and nonce values must be retrieved
from ::Sidecar Data:: along with a proof path that demonstrates the indexed leaf
commits to the provided content.

##### Hide {.unnumbered .unlisted}

##### Imperative Algorithm {.unnumbered .unlisted}

Given the 32 `signalBytes` from the SMT Beacon Signal, a `sidecarDocumentsMap`
and a `smtProofsMap`:

1. Set `id` to the hexadecimal string representation of `signalBytes`.
2. Get `smtProof` from `smtProofsMap` by its `id`.
3. If `smtProof` is undefined, raise InvalidDidUpdate error.
4. Set `index` to `hash(did)`.
5. Set `nonce` to the value of `smtProof.nonce`.
6. Set `updateId` to the value of `smtProof.updateId`.
7. If `updateId` is defined, set `btcr2UpdateAnnouncement` to the binary
   representation of `updateId` and set `verifyHashBytes` to `hash(index +
   hash(nonce ^ btcr2UpdateAnnouncement))`, otherwise set `verifyHashBytes` to
   `hash(index + hash(nonce))`.
8. For each `step` in `smtProof.path`:
   1. Validate that `step` has a single key-value pair.
   2. Extract `key` and `value` from `step`.
   3. If `key` is `"left"`, set `verifyHashBytes` to `hash(value +
       verifyHashBytes)`; otherwise, if `key` is `"right"`, set
       `verifyHashBytes` to `hash(verifyHashBytes + value)`; otherwise, raise
       InvalidDidUpdate error.
9. If `verifyHashBytes` ≠ `signalBytes`, raise InvalidDidUpdate error.
10. If `updateId` is undefined, return null.
11. Get `btcr2Update` from `sidecarDocumentsMap` by its `updateId` if available,
    or from ::CAS:: by its `updateId` if not and `cas` is defined.
12. If `btcr2Update` is undefined, raise InvalidDidUpdate error.
13. Return `btcr2Update`.

NOTE. The act of retrieving from `sidecarDocumentsMap` validates the document
hash.

#### Algo 12. Derive Root Capability {.tabbed .unnumbered}

This algorithm derives a root capability to update a specific **did:btcr2**
identifier from the identifier itself. The root capability must be conformant to
the [Authorization Capabilities v0.3](https://w3c-ccg.github.io/zcap-spec/)
specification. The root capability for a specific **did:btcr2** identifier is a
JSON containing the following fields:

* \@context: The string "[https://w3id.org/zcap/v1](https://w3id.org/zcap/v1)"
* id: A URN of the following format: urn:zcap:root:${encodeURIComponent(DID)}
* invocationTarget: The DID
* btcr2 identifiercontroller: The DID

Note: The controller of this capability is the DID, which resolves to a DID
document. When verifying a BTCR2 update is an invocation of this capability, the
controller resolves to the contemporary DID document. That is the DID document
that was current when the capability was invoked.

##### Hide {.unnumbered .unlisted}

##### Imperative Algorithm {.unnumbered .unlisted}

The algorithm takes in a did:btcr2 `identifier` and returns a `rootCapability`
object.

1. Define `rootCapability` as an empty object.
2. Set `rootCapability.@context` to ‘https://w3id.org/zcap/v1’.
3. Set `encodedIdentifier` to result of calling algorithm
   `encodeURIComponent(identifier)`.
4. Set `rootCapability.id` to `urn:zcap:root:${encodedIdentifier}`.
5. Set `rootCapability.controller` to `identifier`.
6. Set `rootCapability.invocationTarget` to `identifier`.
7. Return `rootCapability`.

#### Algo 13. Apply BTCR2 Update {.tabbed .unnumbered}

This algorithm takes a ::BTCR2 Update::, attempts to verify it and then applies
it to the appropriate DID document as identified by the update.

A ::BTCR2 Update:: must contain the following:

* \@context. A context array containing the follow context URLs
    * `"https://w3id.org/zcap/v1"`
    * `"https://w3id.org/security/data-integrity/v2"`
    * `"https://w3id.org/json-ld-patch/v1"`
    * `"https://btcr2.dev/context/v1"`
* proof. A Data Integrity proof for with the proof purpose set to
  capabilityInvocation. This must be an invocation of the capability to update
  the DID document of the DID being resolved. The root capability to update a
  specific **did:btcr2** identifier's DID document is derived following
  [Algo 12. Derive Root Capability] from **did:btcr2** identifier. While the
  [zCap-LD specification](https://w3c-ccg.github.io/zcap-spec) allows
  delegation, this specification does not define how such delegation might work.
* patch. A [JSON patch](https://datatracker.ietf.org/doc/html/rfc6902) object
  that defines a set of transformations to be applied to a DID document. The
  result of applying the patch must be a conformant DID document according to
  the DID core v1.1 specification.
* targetVersionId. The versionId of the DID document after the patch has been
  applied. The patch must be applied to the DID document with a versionId that
  is one less than the targetVersionId. If the contemporary document is equal to
  or less than the targetVersionId, then the ::Unsecured BTCR2 Update:: must be
  the same as the previously applied ::BTCR2 Update::. If the targetVersionId is
  greater than one plus the contemporary document’s versionId, then a
  LATE_PUBLISHING error must be raised.
* sourceHash. A base64 encoded SHA256 hash of the canonicalized DID document
  that the patch must be applied to. The DID document must be canonicalized
  using the JSON Canonicalization Scheme (JCS).
* targetHash. A base64 encoded SHA256 hash of the canonicalized DID document
  that results from applying the patch to the source document. The DID document
  must be canonicalized using the JSON Canonicalization Scheme (JCS).

#### Algo 14. Construct Resolution Result {.tabbed .unnumbered}

A DID Resolution Result is an object defined according to the [DID Resolution
v1.0](https://w3c.github.io/did-resolution/#did-resolution-result)
specification. The DID document metadata in the resolution response must specify
the following information about the returned DID document:

* The versionId of the returned DID document
* The number of confirmations that the first Beacon Signal that announced the
* ::BTCR2 Update:: that produced the returned DID Document.

DID Resolvers may also support clients that request a verbose response to their
resolution request. This response should contain the history of the DID
document, its ::BTCR2 Updates:: and the transaction identifiers of the ::Beacon
Signals:: that announced these updates on the Bitcoin blockchain.

### Update

The Update algorithm calls a series of subroutines to construct, invoke and
announce ::BTCR2 Updates:: for **did:btcr2** identifiers and their
corresponding DID documents. An update to a **did:btcr2** document is an invoked
capability using the [ZCAP-LD](https://w3c-ccg.github.io/zcap-spec/) data format,
signed by a `verificationMethod` that has the authority to make the update as specified
in the previous DID document. Capability invocations for updates MUST be authorized using Data Integrity
following the [BIP340 Data Integrity Cryptosuite](https://dcdpr.github.io/data-integrity-schnorr-secp256k1/#instantiate-cryptosuite)
with a `proofPurpose` of `capabilityInvocation`.

It takes the following inputs:

* `identifier` - a valid **did:btcr2** identifier; REQUIRED; string.
* `sourceDocument` - the DID document being updated; REQUIRED; object.
* `sourceVersionId` - the version of the DID and DID document, an incrementing integer
   starting from 1; REQUIRED; integer.
* `documentPatch` - JSON Patch object containing a set of transformations to
   be applied to the `sourceDocument`. The result of these transformations MUST
   produce a DID document conformant to the DID Core specification; REQUIRED; object.
* `verificationMethodId` - identifier for a `verificationMethod` within the
   `sourceDocument`. The `verificationMethod` identified MUST be a
   [BIP340 Multikey](https://dcdpr.github.io/data-integrity-schnorr-secp256k1/#multikey);
   REQUIRED; string.
* `beaconIds` - an array containing the IDs that correspond to ::BTCR2 Beacons:: in the DID
   document. These MUST identify service endpoints with one of the three ::Beacon Types::
   `SingletonBeacon`, `MapBeacon`, and `SMTBeacon`; REQUIRED; array.

It returns the following output:

* `signalsMetadata` - A Map from Bitcoin transaction identifiers of ::Beacon Signals:: 
   to a struct containing ::Sidecar Data:: for that signal provided as part of the
   `resolutionOptions`; object containing the following properties:
   * `btcr2Update` - A ::BTCR2 Update:: which SHOULD match the update announced
      by the ::Beacon Signal::. In the case of a ::SMT:: proof of non-inclusion, the
      `btcr2Update` will be null; object.
   * `proofs` - A ::Sparse Merkle Tree:: proof that the provided `btcr2Update` value is
      the value at the leaf indexed by the **did:btcr2** being resolved; array.

The steps are as follows:

1. Set `unsecuredUpdate` to the result of passing `btcr2Identifier`, `sourceDocument`,
   `sourceVersionId`, and `documentPatch` into the [Construct BTCR2 Update]
   algorithm.
1. Set `verificationMethod` to the result of retrieving the verificationMethod from
   `sourceDocument` using the `verificationMethodId`.
1. Validate the `verificationMethod` is a BIP340 Multikey:
    1. `verificationMethod.type` == `Multikey`
    1. `verificationMethod.publicKeyMultibase[4]` == `zQ3s`
1. Set `unsecuredBtcr2Update` to the result of passing `btcr2Identifier`,
   `unsecuredUpdate`, `and `verificationMethod` to the
   [Invoke BTCR2 Update] algorithm.
1. Set `signalsMetadata` to the result of passing `btcr2Identifier`, `sourceDocument`, 
   `beaconIds` and `unsecuredBtcr2Update` to the [Announce DID Update] algorithm.
1. Return `signalsMetadata`. It is up to implementations to ensure that the
   `signalsMetadata` is persisted.

#### Construct BTCR2 Update

The Construct BTCR2 Update algorithm applies a `documentPatch` to a
`sourceDocument` and verifies the resulting `targetDocument` is a conformant
DID document.

It takes the following inputs:

* `identifier` - a valid **did:btcr2** identifier; REQUIRED; string.
* `sourceDocument` - the DID document being transformed by the `documentPatch`; REQUIRED; object.
* `sourceVersionId` - the version of the DID and DID document, an incrementing integer
   starting from 1; REQUIRED; integer.
* `documentPatch` - JSON Patch object containing a set of transformations to
   be applied to the `sourceDocument`. The result of these transformations MUST
   produce a DID document conformant to the DID Core specification; REQUIRED; object.

It returns the following output:

* `unsecuredBtcr2Update` - a newly created ::BTCR2 Update::; object.

The steps are as follows:

1. Check that `sourceDocument.id` equals `btcr2Identifier` else MUST raise
   `invalidDidUpdate` error.
1. Initialize `unsecuredBtcr2Update` to an empty object.
1. Set `unsecuredBtcr2Update.@context` to the following list.
   `["https://w3id.org/zcap/v1", "https://w3id.org/security/data-integrity/v2", 
   "https://w3id.org/json-ld-patch/v1", "https://btcr2.dev/context/v1"]`
1. Set `unsecuredBtcr2Update.patch` to `documentPatch`.
1. Set `targetDocument` to the result of applying the `documentPatch` to the
   `sourceDocument`, following the JSON Patch specification.
1. Validate `targetDocument` is a conformant DID document, else MUST raise
   `invalidDidUpdate` error.
1. Set `sourceHashBytes` to the result of passing `sourceDocument` into
   the [JSON Canonicalization and Hash] algorithm.
1. Set `unsecuredBtcr2Update.sourceHash` to the base64 of `sourceHashBytes`.
1. Set `targetHashBytes` to the result of passing `targetDocument` into
   the [JSON Canonicalization and Hash] algorithm.
1. Set `unsecuredBtcr2Update.targetHash` to the base64 of `targetHashBytes`.
1. Set `unsecuredBtcr2Update.targetVersionId` to `sourceVersionId + 1`
1. Return `unsecuredBtcr2Update`.

#### Invoke BTCR2 Update

The Invoke BTCR2 Update algorithm retrieves the `privateKeyBytes` for the
`verificationMethod` and adds a capability invocation in the form of a Data
Integrity proof following the [ZCAP-LD](https://w3c-ccg.github.io/zcap-spec/)
and Verifiable Credentials (VC) Data Integrity specifications.

It takes the following inputs:

* `identifier` - a valid **did:btcr2** identifier; REQUIRED; string.
* `unsecuredBtcr2Update` - an unsecured ::BTCR2 Update::; REQUIRED; object.
* `verificationMethod` - an object containing reference to keys and/or Beacons to
   use for [ZCAP-LD](https://w3c-ccg.github.io/zcap-spec/); REQUIRED; string.

It returns the following output:

* `btcr2Update` - a ::BTCR2 Update:: object invoking the capability
   to update a specific **did:btcr2** DID document.

The steps are as follows:

1. Set `privateKeyBytes` to the result of retrieving the private key bytes
   associated with the `verificationMethod` value. How this is achieved is left to
   the implementation.
1. Set `rootCapability` to the result of passing `btcr2Identifier` into the
   [Derive Root Capability from **did:btcr2** Identifier] algorithm.
1. Initialize `proofOptions` to an empty object.
1. Set `proofOptions.type` to `DataIntegrityProof`.
1. Set `proofOptions.cryptosuite` to `bip340-jcs-2025`.
1. Set `proofOptions.verificationMethod` to `verificationMethod.id`.
1. Set `proofOptions.proofPurpose` to `capabilityInvocation`.
1. Set `proofOptions.capability` to `rootCapability.id`.
1. Set `proofOptions.capabilityAction` to `Write`.
1. Set `cryptosuite` to the result of executing the Cryptosuite Instantiation
   algorithm from the [BIP340 Data Integrity specification](https://dcdpr.github.io/data-integrity-schnorr-secp256k1) passing in
   `proofOptions`.
1. Set `btcr2Update` to the result of executing the
   [Add Proof](https://www.w3.org/TR/vc-data-integrity/#add-proof)
   algorithm from VC Data Integrity passing `unsecuredBtcr2Update` as the input document,
   `cryptosuite`, and the set of `proofOptions`.
1. Return `btcr2Update`.

#### Announce DID Update

The Announce DID Update algorithm retrieves `beaconServices` from the `sourceDocument`
and calls the [Broadcast DID Update] algorithm corresponding to the type of
the ::BTCR2 Beacon::.

It takes the following inputs:

* `identifier` - a valid **did:btcr2** identifier; REQUIRED; string.
* `sourceDocument` - the DID document being updated; REQUIRED; object.
* `beaconIds` - an array containing the IDs that correspond to ::BTCR2 Beacons:: in the DID
   document. These MUST identify service endpoints with one of the three ::Beacon Types::
   `SingletonBeacon`, `MapBeacon`, and `SMTBeacon`; REQUIRED; array.
* `btcr2Update` - a ::BTCR2 Update:: object invoking the capability
   to update a specific **did:btcr2** DID document; REQUIRED; object.

It returns the following output:

* `signalsMetadata` - a mapping from Bitcoin transaction identifiers of ::Beacon Signals:: 
   to a struct containing ::Sidecar Data:: for that signal, provided as part of the
   `resolutionOptions`; a map with the following properties:
   * `btcr2Update` - a ::BTCR2 Update:: which SHOULD match the update announced
      by the ::Beacon Signal::. In the case of a ::SMT:: proof of non-inclusion, the
      `btcr2Update` will be null; object.
   * `proofs` - A ::Sparse Merkle Tree:: proof that the provided `btcr2Update` value is
      the value at the leaf indexed by the **did:btcr2** being resolved; object.

The steps are as follows:

1. Set `beaconServices` to an empty array.
1. Set `signalMetadata` to an empty array.
1. For `beaconId` in `beaconIds`:
   1. Find `beaconService` in `sourceDocument.service` with an `id` property
    equal to `beaconId`.
   1. If no `beaconService` MUST throw `beaconNotFound` error.
   1. Push `beaconService` to `beaconServices`.
1. For `beaconService` in `beaconServices`:
   1. Set `signalMetadata` to null.
   1. If `beaconService.type` == `SingletonBeacon`:
      1. Set `signalMetadata` to the result of
        passing `beaconService` and `btcr2Update` to the
        [Broadcast Singleton Beacon Signal] algorithm.
   1. Else If `beaconService.type` == `CIDAggregateBeacon`:
      1. Set `signalMetadata` to the result of
        passing `btcr2Identifier`, `beaconService` and `btcr2Update` to the
        [Broadcast CIDAggregate Beacon Signal] algorithm.
   1. Else If `beaconService.type` == `SMTAggregateBeacon`:
      1. Set `signalMetadata` to the result of
        passing `btcr2Identifier`, `beaconService` and `btcr2Update` to the
        [Broadcast SMTAggregate Beacon Signal] algorithm.
   1. Else:
      1. MUST throw `invalidBeacon` error.
  1. Merge `signalMetadata` into `signalsMetadata`.
1. Return `signalsMetadata`

### Deactivate

To deactivate a **did:btcr2**, the DID controller MUST add the property `deactivated`
with the value `true` on the DID document. To do this, the DID controller constructs
a valid ::BTCR2 Update:: with a JSON patch that adds this property and announces
the ::BTCR2 Update:: by broadcasting an ::Authorized Beacon Signal:: following
the algorithm in [Update]. Once a **did:btcr2** has been deactivated this
state is considered permanent and resolution MUST terminate.
