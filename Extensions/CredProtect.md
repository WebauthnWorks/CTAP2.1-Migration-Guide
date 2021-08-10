# CredProtect 0x06
CREDENTIAL PROTECTION (credProtect)

Relying parties can specify a credential protection policy when creating a credential. 
Authenticators not supporting some form of user verification MUST NOT support this extension. 
Authenticators that include FIDO_2_1 in versions MUST support the credProtect extension if some form of user verification is supported, unless all credentials are implicitly created at credProtect level three (userVerificationRequired).


1. 
Relying party specifies a protection level of the credential to be stored.

```
credProtectPolicies = { userVerificationOptional: 0x01, userVerificationOptionalWithCredentialIDList: 0x02, userVerificationRequired: 0x03 }
extensions = {
	'credProtect': credProtectPolicies.userVerificationOptional
}
```


2. 
(a) If extension is not present in authenticatorMakeCredential request, platform may enforce default. 

(b)
(i) Platform checks policy is a string value as specified in 'credProtectPolices' in (1).
(ii) Platform SHOULD NOT alter policy (unless enterprise policy required, or due to other security requirements) nor create credential in a way that does not implement requested policy.

For non-discoverable credentials, userVerificationOptional and userVerificationOptionalWithCredentialIDList both have the same authenticator behaviour, as relying party must supply an allowList with credentialIDs.

3. 
Call authenticatorMakeCredential with CBOR map including "extensions" field to authenticator (0x06)
( Authenticators for high security environments may be configured to upgrade credential Protection to level 3, userVerificationRequired -- and respond via credProtect extensions result that is set the policy to 3)

e.g.,
```
INPUT:
extensions = {
            'credProtect': credProtectPolicies.userVerificationOptionalWithCredentialIDList
        }
-> Generate buffer and make request to authenticator
RESPONSE:
[to-do]
```
4. Authenticator persists credProtect value with credential. If no extension was included in request, authenticator SHOULD use default value of 1, userVerificationOptional. 

5. Authenticator returns CBOR output with map entry in "extensions" field of authenticatorData object. I.e
{ "credProtect": 0x01" } . The error code returned is CTAP_SUCCESS(0x00).


CMD: 0x06