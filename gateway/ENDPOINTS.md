# Endpoints for the SOSCHAIN Gateway Service

Copyright © 2024 Fernando Latorre López, CTO at [ConnectHealth](https://connecthealth.info).

---

## Introduction

The paths (routes) in SOSCHAIN are case-insensitive and follow **HL7 FHIR API** standards and **Consumer Data Standards (CDS)**, using `territory` (ISO 3166 two-letter country code) and various **sectors** based on the organization's type (see https://github.com/soschain/soschain/README.md).

Organizations can certify identities and documents on the blockchain, generate de-identified metadata for statistics and research, and deliver notifications to the SOSCHAIN Gateway Service as authorized by individuals or their legal guardians (e.g., in the case of children or dependents).

## Endpoint Classification

- **Organizational Data**:

    Certifies organizational identity information across sectors, including practitioners, roles, departments, and locations, along with additional healthcare services and care teams within healthcare and pharmacy sectors.

    Accepted resource types: `Organization` (sub-organization and department), `Location`, `Practitioner`, `PractitionerRole`, `CareTeam`, `HealthcareService`.

    ```
    PUT
    /cds-<country>/v1/<sector>/entity/org.hl7.fhir.<release>/<resourceType>
    ```

    To retrieve a FHIR Bundle of type "searchset" based on search parameters such the identifier of the resource (UUID), the following request will be accepted:

    ```
    POST
    /cds-<country>/v1/<sector>/entity/org.hl7.fhir.<release>/<resourceType>/_search

    id=<resource-UUID-v4>
    ```

    Note the representative of the organization must request joining the SOSCHAIN Network by a signed PDF consent form.
    
    You can download the form for entities [here](../join-entity-representative/).


- **Product Management**: Certifies traceability of products within the insurance, healthcare, and pharmacy sectors, which are related to a patient’s Unified Health Index, including healthcare device (e.g., pacemakers), biologically derived product (e.g., blood bags, human milk), medication (product definition), insurance plan, coverage, contract, invoice, among others.

    Accepted resource types: `DeviceDefinition`, `Device`, `BiologicallyDerivedProduct`, `Medication`, `InsurancePlan`, `Coverage`, `Contract`, `Claim`, `ClaimResponse`, `Invoice`.

    ```
    PUT
    /cds-<country>/v1/<sector>/product/org.hl7.fhir.<release>/<resourceType>
    ```

    To retrieve a FHIR Bundle of type "searchset" based on search parameters such the identifier of the resource (UUID), the following request will be accepted:

    ```
    POST
    /cds-<country>/v1/<sector>/entity/org.hl7.fhir.<release>/<resourceType>/_search

    id=<resource-UUID-v4>
    ```    


- **Individual and Related Person Identity**: Certifies identity information for individuals and related persons, including permissions to enable the Unified Health Index via signed consent, and relationships (e.g., a parent-child relationship).

    Accepted resource types: `Person`, `Patient`, `RelatedPerson`

    ```
    PUT
    /cds-<country>/v1/soschain/individual/org.hl7.fhir.<release>/<resourceType>
    ```

    Note the data is hashed, so no personal identity data is known but their hashes, except the email, phone numbers (securely stored) and relationship code in case of `RelatedPerson` resource.


- **Clinical-Only Certification Data**: Certifies clinical data on the blockchain specifically for clinical purposes, without updating the patient's Unified Health Index. This is applied during active encounters, allowing clinicians to communicate new data to patients or family members responsibly.

    Accepted resource types within the Bundle: `Patient`, `AllergyIntollerance`, ...

    The certification of FHIR resources requires applying a **data normalization** by removing some properties in the JSON data as per the FHIR specification, and sorting the normalized resource before calculating the hash of the data. This job is done by the SOSCHAIN Gateway Service before data certification and verification operations.

    For data certification the resources can be sent as `fhir+json`, or in the `body` property of the payload of the protected request, for example:
    
    ```
    PUT
    /cds-<country>/v1/<sector>/blockchain/org.hl7.fhir.<release>/org.loinc/60591-5/Bundle

    request=<didcomm message>
    ```

    Data certification also extracts and registers metadata for statistics, essential for monitoring epidemic and pandemic trends. This includes the clinical code (e.g. SNOMED or LOINC code) and other metadata such as `year of birth`, `sex at birth`, and `administrative gender`, which are extracted from the `Patient` resource within the Bundle.

    Each resource is certified using its UUID as the asset ID and includes a field to track whether metadata has been extracted (de-identification) for statistical and research purposes, preventing data duplication.


- **Blockchain Data Verification**: Retrieves asset data on the blockchain, returning the original resource hash and additional audit information.

    Accepted resource types: `AllergyIntollerance`, ...

    To verify a specific resource the resource being verified can be sent as `fhir+json`, or in the `body` property of the payload of the protected request, for example:

    ```
    POST
    /cds-<country>/v1/<sector>/blockchain/org.hl7.fhir.<release>/<resourceType>/_verify

    request=<didcomm message>
    ```

    To retrieve the audit data stored on blockchain for a resource, use:

    ```
    POST
    /cds-<country>/v1/<sector>/blockchain/<resourceType>/_search

    id=<resourceID>
    ```    

    To retrieve the resource's entire history, use:

    ```
    POST
    /cds-<country>/v1/<sector>/blockchain/<resourceType>/_history

    id=<resourceID>
    ```


- **Research**: SOSCHAIN supports blockchain-based registration of certified research studies, fostering a trusted, transparent environment where patients can join studies via consent, sharing either their International Patient Summary (IPS) or an anonymized Digital Twin (DT) that evolves with shared data.

    To register or update a research study:

    ```
    PUT
    /cds-<country>/v1/research/index/org.hl7.fhir.<release>/ResearchStudy
    ```

    Each `ResearchSubject` entry links to the study ID, subject, and consent. For anonymized digital twins, only the anonymized `identifier`, `year of birth`, `sex-parameter-for-clinical-use` (extension), and gender are included in the patient resource.

    When registering a patient in a study, a FHIR Bundle containing all necessary resources is created:

    ```
    POST
    /cds-<country>/v1/research/<studyID>/org.hl7.fhir.<release>/Bundle
    ```

    Certified metadata for research and statistical purposes is essential for tracking epidemics and pandemics. The system allows metadata retrieval by birth sex or administrative gender.

    To retrieve metadata stored on blockchain for statistic and research via the FHIR API (formatted for clarity):

    ```
    POST
    /cds-<country>/v1/research/blockchain/<resourceType>/_search

    code=<coding system>|<code>
    &date=geYYYY-MM-DD
    &date=leYYYY-MM-DD
    &location=urn:iso:std:iso:3166|<country-code>
    &birthdate=geYYYY
    &sex-parameter-for-clinical-use=<birth sex>
    &gender=<administrative gender>
    ```

    For research covering a specific time frame, both start and end dates MUST be specified. If only a start date is provided, the current date will be assumed as the end date.


- **Patient self-registration**:
  
    The **Unified Health Index (UHI)** for the patient **is activated in the SOSCHAIN network through a signed consent document** (download the PDF file [here](../join-patient-myself/)). This consent contains data in a PDF form and a digital signature (depending on the country), from which interoperable claims (reverse-DNS based) are extracted and then FHIR resources are constructed in the FHIR version specified.

    The *PDF consent file* can contain groups of related persons with some permissions such us for receiving notifications about appointments and encounters of the patient (e.g. relatives and caregivers), for accessing all the data of the patient (e.g. legal guardians) or specific data (e.g. caregivers).

    It can contain also groups of professionals for accessing all or speciic data in health consultations and emergencies (e.g. medical doctor, nurse, paramedic), but also specific data in case of emergencies by non-healthcare professionals (e.g. life guard, firefighter, flight attendant) such as devices (e.g. associated Bluetooth device for localization, pacemaker implanted, etc.).

    The patient signs the *PDF consent file* to enable the Unified Health Index. Then the file will be sent (pushed) to the SOSCHAIN Gateway Service authorized by the patient in the consent. Thus the data is verified and *verified claims* are extracted following the *OpenID Connect for Identity Assurance (OIDC4IDA)*, where  *reverse DNS FHIR API* is used as format for the claim names, such as "org.hl7.fhir.api.person.citizenship" or "org.hl7.fhir.api.person.identifier:0" where numbers are used in case the property can be present more than one time in the claims (e.g. national identifier, passport number, etc.).

    The signature of the *PDF consent file* is first verified and then the consent is securely stored in the gateway service. Then the PDF consent data is extracted and **converted to a FHIR Consent resource**, which will have some elements in the `contained` property such as *Person*, *Patient*, *Group*, *RelatedPerson*, *CareTeam*.
    
    The FHIR `Consent` resource will be returned by the SOSCHAIN Gateway Service in an standard FHIR or protected respose (see part [D. Protected Response](./REAME.md#d-protected-response)), as in the `body` property of the protected response, containing in this way:
    - **XHTML representation** of the authorization provided through the signed PDF document, as per the FHIR specification;
    - all the **information extracted for the signed PDF**;
    - the **URL** where the PDF consent is available for authorized parties;
    - the OpenID `verified_claims` property will be also included in the protected message, which have an array of verified claims for each entity (Person, Patient, Related Persons, Consent).
 
    Additionally, an **email or SMS** will be sent to the signer of the consent with a code **to confirm the commitment of the consent permissions to the blockchain**, similar to the OpenID PKCE `code` flow but suing Pushed Authroization Request (PAR) and sending the code as a Multifactor Authentication (MFA) or One Time Passworkd (OTP) to confirm the operation.

    The endpoint for sending the PDF consent to activate the index of the patient will be like:

    ```
    POST
    /cds-<country>/v1/soschain/individual/org.hl7.fhir.r5/Consent/par

    request=<didcomm message>
    ```

    The raw PDF data will be accepted in the body of the DIDComm message to construct the FHIR resources from a signed PDF consent form, using the "type" property `application/pdf` in the payload of the DIDComm message.

    The following unsigned JWT/plaintext DIDComm message for the request is shown for educational purposes, which can then be signed and encrypted by the software application sending the data to the gateway service for enhanced data protection:

    ```json
    POST
    request=(base64url( // JWT header, base64url encoded
    {
        "alg": "none", // No signature algorithm (unsigned JWT/plaintext DIDComm message)
        "kid": "", // No signature Key ID (unsigned JWT/plaintext DIDComm message)
        "typ": "didcomm-plain+json" // Type of payload
    }).(base64url( // JWT payload, base64url encoded
    {
        "iss": "<public-client-ID>", // Issuer Claim: the principal that issued (signed) the JWT.
        "sub": "<public-Unified-Health-ID>", // Subject Claim: the principal that is the subject of the JWT.
        "aud": "<gateway-service-URL>", // Audience Claim: recipients that the JWT is intended for.
        "nbf": 123456000, // Not Before Claim: time before which the JWT MUST NOT be accepted.
        "exp": 123456060, // Expiration time after which the JWT MUST NOT be accepted.
        "jti": "<messageID>", // JWT ID Claim: universally unique identifier (UUID) for the request (implied thread ID).
        "response_mode": "jwt", // "jwt" or "form_post.jwt" are accepted for protected messages (JARM)
        "type": "application/pdf", // Type of the embedded body in the payload
        "body": "<raw PDF data>" // PDF binary data (no base64 encoding is required)
    }). // No signature (unsigned JWT/plaintext DIDComm message)
    ```

    Examples for the response received will be provided in the [examples folder](../examples/) or shared in a Google Drive shared folder for the SOSCHAIN partner organizations.

    If the verification result is successful, the response will have a content-url a FHIR Consent with the property "verified_claims" as per the OpenID Connect for Identity Assurance.

    To retrieve consent data based on search parameters:

    ```
    POST
    /cds-<country>/v1/soschain/individual/org.hl7.fhir.r5/Consent/_search

    subject=<public-Unified-Health-ID>
    or
    thid=<jti>
    ```
