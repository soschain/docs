![](https://avatars.githubusercontent.com/u/187218241?s=200&v=4)
# SOSCHAIN Gateway Service of the Shared Data Space

Copyright (c) 2024 Fernando Latorre López, CTO at [ConnectHealth](https://connecthealth.info).

---

## SOSCHAIN for Security, Resilience, and Data Communication in Emergencies and Catastrophes

SOSCHAIN is designed with security and resilience at its core. It enhances data exchange security by implementing and extending the Financial API Security Profile, using encrypted messages (based on JOSE and DIDComm standards) to prevent malware implants that could compromise HTTPS security and access confidential information (e.g., malware like Russian Snake that monitors network traffic). These messages can be securely exchanged over various transport protocols (e.g., HTTP, Bluetooth).

SOSCHAIN implements extended security protocols based on *Financial API and DID Communication*, which enable the secure transfer of information even in insecure networks or on infected devices. It utilizes DIDComm messages through the patented *Unified Identification Protocol for Training and Health*, making it resilient to network downtimes (such as malware attacks, wars, natural disasters, etc.). It also implements the patented *Procedure for the Global Unified Registration and Universal Identification of Donors*. In this way, patients or donors and healthcare practitioners can continue exchanging data over Bluetooth or other connections when the Internet is not available, then synchronize new data when the network connection becomes available.

---

### A. Cryptography:
The service provides public cryptographic keys at `./well-known/jwks.json`, which can be verified via the blockchain. These keys utilize post-quantum cryptography algorithms approved by NIST for data signature (ML-DSA) and data encryption (ML-KEM).

Additionally, upon activation, the Gateway Service receives a private certificate from the blockchain network, enabling it to make signed calls to smart contracts within the trusted network.

Clients will have cryptographic keys for communicating with this Gateway Service. For hybrid operation between device-based wallets and cloud-based wallets, the service implements a mathematical formula to split the seed of the cryptographic keys. This way, parts of the keys are stored on the client device while others are stored on the Gateway Service, enhancing security in the following scenarios:

#### Hybrid Wallet for Online Software Applications

SOSCHAIN utilizes a hydrid wallet approach to allow users to use web or app software applications while maintaning security.

The hybrid approach enhances the SOSCHAIN security by enabling users to reconstruct the cryptographic keys after validating their identity using multifactor authentication, while no private keys are stored.

In this hybrid flow, when users are registered through the SOSCHAIN Gateway Service, they receive a "private code" containing essential parts for reconstructing the `seed` required to regenerate the cryptographic keys of the users associated with their DID Document (for data signature and decryption), wich defines their digital identity.

To reconstruct the user identity keys for signing data and decrypting information, the client software application will ask the end user for the "private code" and then 

In case of server-to-server background communications, the private code can be loaded from a secure source (confidential storate).

If an invalid private code is provided the reconstruction of the cryptographic keys will fail.

#### Confidential Wallet for Online and Offline Flow
A confidential software application can securely protect cryptographic keys on devices (e.g., a server or a mobile phone with an unlock screen that activates data encryption). However, devices can be hacked, compromising device storage security. The applications can implement an additional layer of security, such as a PIN or security password, to protect the parts of the seed required to reconstruct the private cryptographic keys when the user starts the software application.

In this scenario, the client's confidential software application generates seed parts and stores them, with the "private code" composed of some parts of the seed communicated to the user for recovery in case of device damage or software uninstallation. The other parts are backed up in the Gateway Service.

This flow supports both online and offline operations, allowing users to access both secure devices and cloud wallets if needed by using the "private code" from the confidential application.

---

### B. Security and Resilience

To avoid exposing data on systems compromised by malware (e.g., the Russian Snake malware, which undermines HTTPS security), the service follows Financial API security profile recommendations, along with the PAR, JAR, and JARM standards to enhance security. It also incorporates post-quantum cryptography for the JWT/JWE messages exchanged between applications on client devices (customers, employees) and servers, ensuring that every request and response is protected even when the security of the channel is compromised.

Malware implants can intercept the keys negotiated in the SSL protocol, compromising the data transmitted by HTTPS. To address this, SOSCHAIN implements an additional data security layer over the network channel, similar to a virtual private network (VPN). This layer operates over distinct protocols at the application level and enhances the resilience of the healthcare ecosystem by enabling the abstraction of the network channel, in order to use any available data communication channel.

---

#### B.1. Protecting Identifiers

Every personal identifiable information (PII) such as identifiers of the patient and related persons are protected before being written to the blockchain is hashed and encoded using multihash. For example, a private UUID:

```
urn:uuid:123e4567-e89b-12d3-a456-426614174000
```

is hashed using the SHA3-256 algorithm to protect the data. It is then prefixed with the hex string `1620`, which is the identifier for this algorithm in the multihash specification, resulting in:

```
urn:multihash:1620455185f4e3fdb5dca61fce3257b666ea4a88512b9c4ec12fe78462d80e30277b
```

This multihash is then encoded in base58 to create a more compact representation. The prefix `zW1` indicates that it is a SHA3-256 multihash encoded in base58 using multibase. The `z` prefix signifies base58 encoding, while `W1` represents the base58 encoding of the SHA3-256 algorithm in multihash, corresponding to the `1620` hex prefix for the hashed data.

```
urn:multibase:zW1e7aTm6KBbmrJPwMQFEqW9Aa5uQtkFtUgnz1pYvuA1V8e
```

For example, when **registering a birth relationship on the blockchain**, the identifiers of both the newborn and the mother are **obfuscated** in this way, **protecting their real identities** on a shared data space (blockchain). If the mother requests a birth certificate, she can provide her identifier as well as that of the newborn. The Gateway Service will then verify that the information was certified on the blockchain by a trusted organization and can issue a signed credential with this information.

In the case of passports and national identity documents, the interoperable URI for the identifier will be similar to:

```
urn:soschain:cds:v1:identity:person:org.hl7.terminology.codesystem.v2-0203.PPN:ES:<mother-passport-number>
```

The protected identifier, created by applying normalization (lowercasing to maintain consistency with the URNs in the FHIR documentation), multihash, and then multibase for a shorter representation in base58, will resemble the following:

```
urn:multibase:zW1oHpc9ELirtmSW2gMniFwaUGxLDcTAyEjUYab6yXP9avJ
```

The first `zW1` characters in the encoded result indicates that it is a base58 encoding of the SHA3-256 hash result (or digest) of the original information.

#### B.2. Financial API Security Profile

FAPI (Financial-Grade API) is a set of standards developed to enhance the security of APIs, particularly for the exchange of sensitive information. FAPI includes advanced specifications such as OAuth PAR (Pushed Authorization Requests), JAR (JWT Secured Authorization Request), and JARM (JWT Secured Authorization Response Mode), which ensure that the exchange of confidential information is protected with high levels of security and integrity.

The Financial API security profile is integrated to enhance security through the use of protected messages, specifically JWE (JSON Web Encryption) with nested JWT (JSON Web Tokens) as per the JOSE specifications. This approach ensures that sensitive information remains secure, even in potentially compromised communication channels (e.g.: HTTPS).

This protection is crucial for mitigating threats like Snake malware and similar attacks, which operate by implanting themselves on servers to monitor network traffic and compromise HTTPS connections.

#### B.3. OAuth PAR, JAR, and JARM Specifications

To facilitate secure, standards-based communication and ensure financial-grade API security, the system will implement OAuth PAR (Pushed Authorization Requests), JAR (JWT Secured Authorization Request), and JARM (JWT Secured Authorization Response Mode) specifications. These protocols will protect data exchange in HTTP requests and responses when accessing confidential information and exchanging sensible information between the parties.

##### PAR (Pushed Authorization Requests)

It is a specification within the OAuth 2.0 framework that allows HTTP requests to be sent securely and directly to the desired server (recipient). This method is implemented in the document to secure API communications with healthcare services.

##### JAR (JWT Secured Authorization Request)

It is a specification that allows HTTP requests to be signed and optionally encrypted using JWT. In this document, JAR ensures the secure exchange of confidential data.

##### JARM (JWT Secured Authorization Response Mode)

It is a specification that secures HTTP responses to be signed and optionally encrypted using JWT.

#### B.4. Blockchain Integration

The Gateway Service is designed to connect with a blockchain network, utilizing private keys provided by a Certificate Authority (CA) to authenticate and facilitate interactions with the blockchain ledger. Each transaction, including consents, data certification and IPS data updates, is recorded on the blockchain, ensuring traceability and auditability, with all actions cryptographically signed.

##### B.4.1. Digital Certificate from Hyperledger Fabric Certifying Authority

When an organization requests to join the shared data space and receives the activation code, the Gateway can download a digital certificate issued by a certifying authority (CA) within the Hyperledger Fabric blockchain. This certificate is essential for making calls to smart contracts on the blockchain network. Since Hyperledger Fabric operates on a permissioned blockchain, this certificate ensures that only authorized entities can interact with the network’s smart contracts, adding a layer of trust and security.

These internal blockchain interactions are conducted within the secure environment of the SOSCHAIN shared data space. The cryptographic keys used in these interactions remain confidential and are never exposed externally, ensuring transaction integrity and protection against threats, such as potential vulnerabilities from quantum computing.

##### B.4.2. Decentralized Generation of Quantum-Resistant Cryptographic Keys

To further enhance security, the Gateway Service must generate cryptographic keys in a decentralized manner that are specifically designed to be quantum-resistant. Quantum computing poses a risk to classical cryptographic algorithms, especially for public keys exposed in the `well-known` configuration of public servers. These quantum-resistant keys provide long-term security for communications with external entities, including customer and employee web and mobile applications.

By implementing quantum-resistant cryptography, the Gateway ensures that external communications remain secure both now and in the future, as quantum computing capabilities evolve. The quantum-resistant keys generated by the Gateway are registered on the blockchain, creating an immutable record. This registration allows for verification and auditing by any entity within the shared data space, enhancing trust and enabling external clients and applications to validate the authenticity and integrity of the cryptographic keys used in their communications.

#### B.5. Authentication

HTTP requests to the SOSCHAIN Gateway Service can include an API key in the header `x-api-key` for client authentication, particularly for partner services within the shared data space. Additionally, requests can be made on behalf of a practitioner using the header `x-on-behalf-of`. *Note that HTTP headers are case-insensitive, so they can be sent in any case. However, for consistency and clarity, lowercase will be used*.

The identity in Hyperledger Fabric for the Gateway Service of an organization (partner in the shared data space) includes an MSP ID, a private certificate, and a public certificate. The Hyperledger Fabric identity is downloaded from the Hyperledger Fabric CA and securely stored on the Gateway Service's local storage during initial activation, using the activation key provided when joining the shared data space. Before signing a request to the smart contract, the Gateway Service retrieves the MSP ID and the private certificate associated with the API key from secured storage.

In scenarios where a practitioner logs in through the EHR software application, a Bearer token can be used for authentication. The credentials of the authenticated practitioner from a valid Identity and Access Management service (IAM) can be forwarded to the Gateway Service instead of using the `x-on-behalf-of` header.

When using PAR and JAR to encrypt information exchanged between the client and the Gateway Service, either the HTTP Bearer token or the headers (`x-api-key` and `x-on-behalf-of`) can be included in the payload of the JWT. This can be achieved using  `payload.meta.http.header["x-api-key"]`, `payload.meta.http.header["x-on-behalf-of"]` or `payload.meta.http.header["bearer"]` for the HTTP headers (all in lower case). This approach helps protect confidential information in the event of malware affecting the security of the HTTPS connection.

#### B.6. Protecting Data with VPN-like Security: JWT/JWE/JWM/DIDComm Messaging

To protect data in HTTP requests and responses, DIDComm messaging extends JWM (JavaScript Web Messaging) and is based on JOSE (JavaScript Object Signing and Encryption) standards, including JWT and JWE (signed and encrypted JSON Web Messages). As a modern standard, DIDComm is preferred for securely sharing data across various communication channels, such as HTTP and Bluetooth.

This approach mitigates the risk of malware compromising the security of HTTPS connections, including threats posed by sophisticated cyber espionage tools like the [Russian Snake malware implant](https://www.cisa.gov/news-events/cybersecurity-advisories/aa23-129a), which can spy on all packets transmitted through the network adapters of clients and servers.

Malware implants can intercept the keys negotiated in the SSL protocol, compromising the data transmitted by HTTPS. This security layer can operate over distinct protocols at the application level and enhances the resilience of the healthcare ecosystem by enabling the abstraction of the network channel, allowing the use of any available data communication channel.

---

### C. Demystifying the 'Content-Type' HTTP Header, JOSE Header 'Content Type' (cty), and 'Type' of the Body Content in the Payload of Protected Messages (type)

The Content-Type HTTP header specifies the expected content type of the request body. For HL7 FHIR data, this is typically `fhir+json`. In the case of PAR and JAR requests that embed FHIR data, the Content-Type will be `x-www-form-urlencoded`, with the protected message sent in the `request` form parameter.

According to the DIDComm specification, the content type describing the embedded body in the JWT (`payload.body`) can also be included in the JWT payload using the "type" property (`payload.type`). This internal "type" property will specify `fhir+json` for HL7 FHIR resources.

Additionally, the protected headers in both the JWT and JWE will adhere to established standards. The JWE header will include the 'cty' claim (content type) specifying `jwt` to indicate that a nested JWT is enveloped, and the `typ` claim specifying `didcomm-encrypted+json` to indicate the format of the JWE. The embedded JWT will have a protected (signed) header containing the `typ` claim as `didcomm-signed+json`, while the payload will include the `type` claim with `fhir+json`, as noted earlier.

In summary, the HTTP Content-Type will be:

- **`fhir+json`**: Indicates that standard HL7 FHIR data is expected in the HTTP body, which may include a specific FHIR resource, a Bundle of type 'batch', or an IPS Bundle of type 'document' containing a Composition resource.

- **`x-www-form-urlencoded`**: Used for sending form parameters (similar to OAuth JAR) via protected JWTs in the `request` parameter or when sending the `jti` of a previously sent JWT using the `request_uri` parameter. This approach ensures data protection even in compromised networks, such as those affected by Snake malware implants sniffing HTTPS traffic.

---

Technical information about the endpoints can be found [here](./ENDPOINTS.md).