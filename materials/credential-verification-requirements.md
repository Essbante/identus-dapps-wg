# Credential Verification Requirements for Identus
## Overview
This document provides a explanation of the requirements for verifying Credentials created by the Identus Cloud agent.
While Identus can also create AnonCreds and, more recently, a flavor of Selective Disclosure Credentials, this document focuses solely on the W3C Verifiable Credentials Data Model 1.1/2 variant, as it is the most common and easiest to understand. Furthermore, we'll look only at the Credential (VC) itself and not the Presentation (VP) for the sake of simplicity.

## Structure of a Verifiable Credential (VC)
The data structure of a VC is defined by the W3C Verifiable Credentials [Data Model 1.1 specification](https://www.w3.org/TR/vc-data-model/#what-is-a-verifiable-credential) and [Data Model 2.0 specification](https://www.w3.org/TR/vc-data-model-2.0/). As of September 2024, Identus supports the 1.1 variant, with the new 2.0 variant being on the roadmap. In terms of challenges regarding integration with dApps, this does not matter.
At its most basic interpretation, it can be seen as a strictly defined JSON data structure.

```json
{
  "@context": [
    "https://www.w3.org/2018/credentials/v1"
  ],
  "id": "http://example.edu/credentials/3732",
  "type": ["VerifiableCredential"],
  "issuer": "did:example:123",
  "issuanceDate": "2020-03-10T04:24:12.164Z",
  "expirationDate": "2025-03-10T04:24:12.164Z",
  "credentialSubject": {
    "id": "did:example:123",
    "degree": {
      "type": "BachelorDegree",
      "name": "Bachelor of Science and Arts"
    }
  }
}
``` 
**Example of a (unsigned) Verifiable Credential**

The data structure has fields for (simplified):
- The issuer,
- The subject (called credentialSubject),
- The type of the credential,
- The issuance date and optionally the expiration date,
- The claims (i.e., the actual data that the credential is about).
All is protected by a digital signature, which is created by the issuer.

## Credential Verification
When someone obtains such a credential, all this information can be extracted and taken at face value. Depending on the source of the credential and the communication channel, someone may, in theory, trust the information in the credential without any verification.
However, since clients like Identus are software systems designed to handle the verification of credentials, this is basically always done.
Nonetheless, challenges lie in two areas: the openness of the system, and the surrounding governance framework and trust infrastructure.
Let's start with the first one:
The VC data model allows for different mechanisms to secure a credential with a signature. These are [JWT (JSON Web Tokens)](https://www.rfc-editor.org/rfc/rfc7519) as well as [Data Integrity Proofs](https://www.w3.org/TR/vc-data-integrity/). For an explantion see [here](https://www.w3.org/TR/vc-data-model-2.0/#securing-mechanisms). Identus supports only the former.
Furthermore, the JWT specification allows for a wide range of different algorithms to sign the token (in this case, the credential), but Identus supports only the ES256K1 algorithm.
This means that Identus, in its current state, cannot verify the validity of every Credential allowed by the spec, but only a very specific subset. This isn't unusual in the world of SSI, but it is important to note, since potential limitations in the verification within a dApp might be circumvented by using either a different proof mechanism (Linked Data Proofs) or a different signature algorithm. Both can theoretically be easily implemented into Identus if the need arises.
The second challenge is the governance framework and trust infrastructure.
While a valid signature is an obvious technical requirement, the question of trust is more complex. It raises questions like: Does the Issuer-DID really belong to the issuer? Is the issuer allowed to issue credentials for the subject? How was the resolution of the Issuer-DID done? These are questions that can be easily overlooked initially, but are crucial nonetheless.

### Verifiying the Signature (JWT)
Verifying the signature means proving that the credential was not tampered with as a whole.
Doing so requires four things:
- Knowledge and technical support of the algorithm used for signing (given in our case),
- The full credential (the first two parts of the JWT),
- The signature (the third part of the JWT),
- The public key of the issuer.

The last three parts are then run through a verification function provided by a cryptographic library, which returns true or false.
While the last part might seem trivial, it might not be, given that the specific algorithm used by Identus might not always be supported on every platform/library. Keep this limitation in mind when we are talking about how to do this with a smart contract.
``` 
eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCJ9.
eyJ2YyI6eyJAY29udGV4dCI6WyJodHRwczovL3d3dy53My5vcmcvMjAxOC9jcmVkZW50aWFscy92MSIsImh0dHBzOi8vd3d3LnczLm9yZy8yMDE4L2NyZWRlbnRpYWxzL2V4YW1wbGVzL3YxIl0sImlkIjoiaHR0cDovL2V4YW1wbGUuZWR1L2NyZWRlbnRpYWxzLzM3MzIiLCJ0eXBlIjpbIlZlcmlmaWFibGVDcmVkZW50aWFsIiwiVW5pdmVyc2l0eURlZ3JlZUNyZWRlbnRpYWwiXSwiaXNzdWVyIjoiaHR0cHM6Ly9leGFtcGxlLmVkdS9pc3N1ZXJzLzE0IiwiaXNzdWFuY2VEYXRlIjoiMjAxMC0wMS0wMVQxOToyMzoyNFoiLCJjcmVkZW50aWFsU3ViamVjdCI6eyJpZCI6ImRpZDpleGFtcGxlOmViZmViMWY3MTJlYmM2ZjFjMjc2ZTEyZWMyMSIsImRlZ3JlZSI6eyJ0eXBlIjoiQmFjaGVsb3JEZWdyZWUiLCJuYW1lIjoiQmFjaGVsb3Igb2YgU2NpZW5jZSBhbmQgQXJ0cyJ9fSwidGVybXNPZlVzZSI6W3sidHlwZSI6Iklzc3VlclBvbGljeSIsImlkIjoiaHR0cDovL2V4YW1wbGUuY29tL3BvbGljaWVzL2NyZWRlbnRpYWwvNCIsInByb2ZpbGUiOiJodHRwOi8vZXhhbXBsZS5jb20vcHJvZmlsZXMvY3JlZGVudGlhbCIsInByb2hpYml0aW9uIjpbeyJhc3NpZ25lciI6Imh0dHBzOi8vZXhhbXBsZS5lZHUvaXNzdWVycy8xNCIsImFzc2lnbmVlIjoiQWxsVmVyaWZpZXJzIiwidGFyZ2V0IjoiaHR0cDovL2V4YW1wbGUuZWR1L2NyZWRlbnRpYWxzLzM3MzIiLCJhY3Rpb24iOlsiQXJjaGl2YWwiXX1dfV19LCJpc3MiOiJodHRwczovL2V4YW1wbGUuZWR1L2lzc3VlcnMvMTQiLCJuYmYiOjEyNjIzNzM4MDQsImp0aSI6Imh0dHA6Ly9leGFtcGxlLmVkdS9jcmVkZW50aWFscy8zNzMyIiwic3ViIjoiZGlkOmV4YW1wbGU6ZWJmZWIxZjcxMmViYzZmMWMyNzZlMTJlYzIxIn0.
YMeGs_5KDNzta-h_ehAY9RS6jyyJ8kB36QOXQL6GR1WCkWUz7S2ZUUsBblvipkcgnA4fCfdjI5c_aZI3Ce8byw
``` 
**Example of a JWT**

We haven't yet touched on the public key of the issuer. One has to know or find out that public key. In the case of Identus, this is done by using a DID.
Instead of just including the public key inside the credential itself, the credential references a DID instead.
e.g., issuer: "did:prism:123456789abcdefghi"

To get the public key used for signing, one has to [resolve the DID](https://www.w3.org/TR/did-core/#did-resolution), i.e., find the correct DID Document for that identifier. [See the DID spec here](https://www.w3.org/TR/did-core/).
Depending on the DID method used, this might be a very simple or quite complex process. We can basically differentiate among three different kinds of DID methods:
- The simple ones, where one can just take the identifier (which is often quite long) and decode it to get the public key. Examples of this are [did:key](https://w3c-ccg.github.io/did-method-key/) or [did:peer](https://github.com/decentralized-identity/peer-did-method-spec).
- The slightly more complex ones, where one has to query a website (e.g., [did:web](https://w3c-ccg.github.io/did-method-web/)) or a DNS server (e.g., [did:dns](https://danubetech.github.io/did-method-dns/)).
- The complex ones, where one has to query a web service, which then queries a ledger in the background—like [did:prism](https://github.com/input-output-hk/prism-did-method-spec/blob/main/w3c-spec/PRISM-method.md)
For the task of Credential Verification, Identus uses only did:prism. That means verifying a signature usually also involves making a network request to a service which queries the ledger for the correct public key used for signing that credential.
Depending on the application/dApp/smart contract, this might be a constraint on what is possible. We'll come back to this later.
After we have ensured that the verification method is provided with the right input and it returns true, we can proceed to the next steps (in any order):

```json
{
  "@context": [
    "https://www.w3.org/ns/did/v1",
    "https://w3id.org/security/suites/jws-2020/v1",
    "https://identity.foundation/.well-known/did-configuration/v1"
  ],
  "id": "did:prism:b4a64c37bb3d22c50ecd9234ede2bfbb74f7b7dcda73ea9b69daeeb9d99e2d10",
  "verificationMethod": [
    {
      "id": "did:prism:preprod:b4a64c37bb3d22c50ecd9234ede2bfbb74f7b7dcda73ea9b69daeeb9d99e2d10#key-1",
      "controller": "did:prism:preprod:b4a64c37bb3d22c50ecd9234ede2bfbb74f7b7dcda73ea9b69daeeb9d99e2d10",
      "type": "JsonWebKey2020",
      "publicKeyJwk": {
        "crv": "secp256k1",
        "kty": "EC",
        "x": "Rumtr0Z3AmljqH3MTSx1ae9eZn-N08ZMiChu44miYPk",
        "y": "FFo9QGpERYe9lMHHOUOHzBQSeBGWDRvgw_aFuVG-gy8"
      }
    }
  ],
  "authentication": [
    "did:prism:preprod:b4a64c37bb3d22c50ecd9234ede2bfbb74f7b7dcda73ea9b69daeeb9d99e2d10#key-1",
  ],
  "service": []
}
``` 
### Issuance Date and Expiration Date ([See spec here](https://www.w3.org/TR/vc-data-model-2.0/#validity-period))

The next step is to check the issuance date and the expiration date. Every credential has a field for the issuance date. In most cases, this issuance date is indeed the date of issuance, but it can also be in the future. This is a bit confusing; this was renamed to "ValidFrom" in the 2.0 spec.
Of course, a credential is only valid if the current date is after the issuance date. This requires knowledge of the exact time—also a potential constraint for smart contracts.
The expiration date is optional, but if it is given, the credential is only valid if the current date is before the expiration date.

### Credential Subject and Claims
Depending on the type of credential, it is usually also necessary to check either the subject or the claims (content) of the credential.
This might just involve a very simple yes/no check on a string/property, but might also be more complicated and could result in a network request, depending on the implementation.
For the sake of simplicity, we'll assume that this kind of check is possible and rather trivial in most cases.

### Revocation ([See spec here](https://www.w3.org/TR/vc-data-model-2.0/#status))

W3C Credentials allow for revocation, and Identus optionally supports the use of Revocation Status Lists. These Status Lists (basically just a fancy way of looking up the validity of a credential on a web server) allow the issuer to revoke a credential before it expires.
Identus implements the StatusList2021 specification.
Adding revocation to the mix of verification steps makes it far more complex, since it requires a network request, a signature verification, RDF canonicalization, and a hash calculation. All in all, not trivial if the correct libraries are not available.
But while revocation sounds like a nice feature, in practice it is rarely used, since often the use of a short-lived validity period makes revocation unnecessary.
An alternative way of revocation might include the deactivation of the public key used for signing the credential (or the complete deactivation of that DID), or the invalidation of the executing dApp/smart contract—all more drastic methods, but which still might fit the use case.

### Trust-Registry
Depending on the use case, there might be multiple "valid" issuers of the credentials that have to be verified, not just a single one. In this case, it is not only enough to just look up the public key of the issuer, but also to check if the issuer is actually allowed to issue credentials for the subject.
This is done by a trust registry. Without going into the technical details here (since there is no overarching standard at the moment), this involves a network request to a source of valid issuer DIDs. This might be a simple list, a complex data structure with rules on who is allowed to issue what, or even a service with a database behind it.
Identus currently does not accommodate the listing of trust registries inside the VC itself, but this can be seen as just a minor limitation or an easy fix if so required.

### Other Trust-Issues / Things to consider
//TODO


## Building an application for verifying Identus Credentials
If someone wants to build an application to verify a credential, the following options are available:
- Use the API of the Identus cloud agent directly.
- Use the components of Identus independently, such as specific modules or libraries provided by Identus.
- Use an alternative existing library or components like the [Veramo SDK](https://veramo.io/) or integrate with Hyperledger Aries. There are many SSI libraries available in various languages that can handle most of the heavy lifting.
- Build a custom solution from scratch (but of course not the cryptographic parts).

But in any case, DID resolution has to be done if one uses PRISM DIDs (which are not long-form). Here we have the following options:
- Use Identus, including the PRISM Node and a Cardano Node, or a hosted solution.
- Use [OPN (OpenPrismNode)](https://github.com/bsandmann/OpenPrismNode) or [NeoPRISM](https://github.com/patextreme/neoprism), both either self-hosted or hosted solutions.
- Use the [Universal Resolver](https://github.com/decentralized-identity/universal-resolver).













