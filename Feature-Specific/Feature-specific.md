#  Feature-Specific Descriptions and Actions New to CTAP2.1

Listed below are feature-specific platform and authenticator actions, which are new to CTAP2.1. The summaries below are taken from the specs.

<br />

### Enterprise Attestation

An enterprise is some form of organization, often a business entity. An enterprise context is in effect when a device, e.g., a computer, an authenticator, etc., is controlled by an enterprise.

An enterprise attestation is an attestation that may include uniquely identifying information. This is intended for controlled deployments within an enterprise where the organization wishes to tie registrations to specific authenticators. See [Enterprise Attestation](https://fidoalliance.org/specs/fido-v2.1-ps-20210615/fido-client-to-authenticator-protocol-v2.1-ps-20210615.html#sctn-feature-descriptions-enterp-attstn)

<br />

### Always Require User Verification (AlwaysUV)

 
This feature allows the user to protect the credentials on their authenticator to always prompt for user verification, even when the Relying Party does not request it. See [Always Require User Verification](https://fidoalliance.org/specs/fido-v2.1-ps-20210615/fido-client-to-authenticator-protocol-v2.1-ps-20210615.html#sctn-feature-descriptions-alwaysUv)

<br />

### Authenticator Certifications

The certifications member provides a hint to the platform with additional information about certifications that the authenticator has received. See [Authenticator Certifications](https://fidoalliance.org/specs/fido-v2.1-ps-20210615/fido-client-to-authenticator-protocol-v2.1-ps-20210615.html#sctn-feature-descriptions-certifications)

<br />

### Set Minimum PIN Length

This feature allows a Relying Party (e.g., an enterprise) to enforce a minimum pin length policy for authenticators registering credentials by examining the return value of the Minimum PIN Length Extension (minPinLength). See [Set Minimum PIN Length](https://fidoalliance.org/specs/fido-v2.1-ps-20210615/fido-client-to-authenticator-protocol-v2.1-ps-20210615.html#sctn-feature-descriptions-minPinLength)
