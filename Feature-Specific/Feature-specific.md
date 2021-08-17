#  Feature-Specific Descriptions and Actions New to CTAP2.1

Listed below are feature-specific platform and authenticator actions, which are new to CTAP2.1.
<br />

## Enterprise Attestation

Reference: [Enterprise Attestation](https://fidoalliance.org/specs/fido-v2.1-ps-20210615/fido-client-to-authenticator-protocol-v2.1-ps-20210615.html#sctn-feature-descriptions-enterp-attstn)

<br />

## Always Require User Verification (AlwaysUV)

 
This feature allows the user to protect the credentials on their authenticator to always prompt for user verification, even when the Relying Party does not request it.

Reference: [Always Require User Verification](https://fidoalliance.org/specs/fido-v2.1-ps-20210615/fido-client-to-authenticator-protocol-v2.1-ps-20210615.html#sctn-feature-descriptions-alwaysUv)
<br />

## Authenticator Certifications

The certifications member provides a hint to the platform with additional information about certifications that the authenticator has received

Reference: [Authenticator Certifications](https://fidoalliance.org/specs/fido-v2.1-ps-20210615/fido-client-to-authenticator-protocol-v2.1-ps-20210615.html#sctn-feature-descriptions-certifications)
<br />

## Set Minimum PIN Length

This feature allows a Relying Party (e.g., an enterprise) to enforce a minimum pin length policy for authenticators registering credentials by examining the return value of the Minimum PIN Length Extension (minPinLength).

Reference: [Authenticator Certifications](https://fidoalliance.org/specs/fido-v2.1-ps-20210615/fido-client-to-authenticator-protocol-v2.1-ps-20210615.html#sctn-feature-descriptions-minPinLength)
