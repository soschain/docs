# Endpoints of the SOSCHAIN Gateway Service

Copyright © 2024 Fernando Latorre López, CTO at [ConnectHealth](https://connecthealth.info).

---
## Introduction

The paths (routes) in SOSCHAIN are case-insensitive and follow **HL7 FHIR API** standards and **Consumer Data Standards (CDS)**, using `territory` (ISO 3166 two-letter country code) and various **sectors** based on the organization's type (see https://github.com/soschain/soschain/README.md).

Organizations can certify identities and documents on the blockchain, generate de-identified metadata for statistics and research, and deliver notifications to the SOSCHAIN Gateway Service as authorized by individuals or their legal guardians (e.g., in the case of children or dependents).

## Endpoint Classification

- **Organizational Data**: Certifies organizational identity information across sectors, including practitioners, roles, departments, and locations, along with additional healthcare services and care teams within healthcare and pharmacy sectors.

    Accepted resource types: `Organization` (sub-organization and department), `Location`, `Practitioner`, `PractitionerRole`, `CareTeam`, `HealthcareService`.

    ```
    PUT
    /cds-<country>/v1/<sector>/entity/org.hl7.fhir.<release>/<resourceType>
    ```

    To retrieve a FHIR Bundle of type "searchset" based on search parameters such the identifier of the resource (UUID), the following request will be accepted:

    ````
    POST
    /cds-<country>/v1/<sector>/entity/org.hl7.fhir.<release>/<resourceType>/_search

    id=<resource-UUID-v4>
    ```   

- **Product Management**: Certifies traceability of products within the insurance, healthcare, and pharmacy sectors, which are related to a patient’s Unified Health Index, including healthcare device (e.g., pacemakers), biologically derived product (e.g., blood bags, human milk), medication (product definition), insurance plan, coverage, contract, invoice, among others.

    Accepted resource types: `DeviceDefinition`, `Device`, `BiologicallyDerivedProduct`, `Medication`, `InsurancePlan`, `Coverage`, `Contract`, `Claim`, `ClaimResponse`, `Invoice`.

    ```
    PUT
    /cds-<country>/v1/<sector>/product/org.hl7.fhir.<release>/<resourceType>
    ```

    To retrieve a FHIR Bundle of type "searchset" based on search parameters such the identifier of the resource (UUID), the following request will be accepted:

    ````
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

    FHIR certification requires normalization by removing properties such as `meta`, internal `id`, and `contained`.

    ```
    PUT
    /cds-<country>/v1/<sector>/blockchain/org.hl7.fhir.<release>/org.loinc/60591-5/Bundle
    ```

    Certified data includes metadata for statistics, essential for monitoring epidemic and pandemic trends. This includes clinical codes and metadata such as `year of birth`, `sex at birth`, and `administrative gender`, extracted from the `Patient` resource within the Bundle.

    Each resource is certified using its UUID as the asset ID and includes a field to track whether metadata has been extracted (de-identification) for statistical and research purposes, preventing data duplication.

- **Blockchain Data Verification**: Retrieves asset data on the blockchain, returning the original resource hash and additional audit information.

    Accepted resource types: `AllergyIntollerance`, ...

    To verify a specific resource, use:

    ```
    POST
    /cds-<country>/v1/<sector>/blockchain/org.hl7.fhir.<release>/<resourceType>/_search

    id=<resourceID>
    ```

    To retrieve the resource's entire history, use:

    ```
    POST
    /cds-<country>/v1/<sector>/blockchain/org.hl7.fhir.<release>/<resourceType>/_history

    id=<resourceID>
    ```

- **Unified Health Index Update**: Updates the indexed compartments of a patient's clinical data using the standardized IPS Bundle structure (LOINC code `60591-5`). Authorized persons will receive notifications of updates via signed consents. All resources in the Bundle are included in the Composition resource.

    ```
    PUT
    /cds-<country>/v1/healthcare/index/org.hl7.fhir.<release>/org.loinc/60591-5/Bundle
    ```

    Each resource is certified with its UUID as the asset ID, with a flag to indicate if metadata extraction (de-identification) for research has already been performed, avoiding duplication. All resources in the Bundle are verified against the blockchain for certification status and de-identification.

- **Retrieving the Unified Health Index of a Patient**: Retrieves the Unified Health Index using the `Composition` resource, following FHIR standards. All resources in the IPS document are included within the Composition.

    ```
    POST
    /cds-<country>/v1/healthcare/index/org.hl7.fhir.<release>/Composition/_search

    subject=<public Unified Health ID>
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
    PUT
    /cds-<country>/v1/research/<studyID>/org.hl7.fhir.<release>/Bundle
    ```

    Certified metadata for research and statistical purposes is essential for tracking epidemics and pandemics. The system allows metadata retrieval by birth sex or administrative gender.

    To retrieve metadata via the FHIR API (formatted for clarity):

    ```
    POST
    /cds-<country>/v1/research/index/org.hl7.fhir.<release>/<resourceType>/_search

    code=<coding system>|<code>
    &date=geYYYY-MM-DD
    &date=leYYYY-MM-DD
    &location=urn:iso:std:iso:3166|<country-code>
    &birthdate=geYYYY
    &sex-parameter-for-clinical-use=<birth sex>
    &gender=<administrative gender>
    ```

    For research covering a specific time frame, both start and end dates MUST be specified. If only a start date is provided, the current date will be assumed as the end date.


