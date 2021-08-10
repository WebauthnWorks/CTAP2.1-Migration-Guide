# CredProtect 0x06
CREDENTIAL PROTECTION (credProtect)

Relying parties can specify a credential protection policy when creating a credential. 
Authenticators not supporting some form of user verification MUST NOT support this extension. 
Authenticators that include FIDO_2_1 in versions MUST support the credProtect extension if some form of user verification is supported, unless all credentials are implicitly created at credProtect level three (userVerificationRequired).


1. 
Relying party specifies a protection level of the credential to be stored.

```
credProtectPolicies = { 
    userVerificationOptional: 0x01, userVerificationOptionalWithCredentialIDList: 0x02, userVerificationRequired: 0x03 
    }
extensions = {
	'credProtect': credProtectPolicies.userVerificationOptional
}
```


2. 
= If extension is not present in authenticatorMakeCredential request, platform may enforce default. 

- Platform checks policy is a string value as specified in 'credProtectPolices' in (1).
- Platform SHOULD NOT alter policy (unless enterprise policy required, or due to other security requirements) nor create credential in a way that does not implement requested policy.

For non-discoverable credentials, userVerificationOptional and userVerificationOptionalWithCredentialIDList both have the same authenticator behaviour, as relying party must supply an allowList with credentialIDs.

3. 
Call authenticatorMakeCredential with CBOR map including "extensions" field to authenticator (0x06)
( Authenticators for high security environments may be configured to upgrade credential Protection to level 3, userVerificationRequired -- and respond via credProtect extensions result that is set the policy to 3)

**Example 1**
1. Make pinUvAuthToken and pinUvAuthProtocol (https://github.com/WebAuthnWorks/CTAP2.1-Migration-Guide/blob/main/Protocol/PinProtocol/2.md#get-pinuvauthtoken---puat)
```
INPUT:
creds = {
    clientDataHash: '',
    rp: '',
    user: '',
    pubKeyCredParams: '',
}

extensions = {
            'credProtect': credProtectPolicies.userVerificationOptional
        }
options = {
    rk: true // residential key (discoverable credentials)
}
-> Generate Make Credentials using pinUvAuthToken and pinUvAuthProtocol
-> Generate command buffer using credential fields, extensions and options fields set

RESPONSE:
(TO-DO ctap2Response) 
```

**Example 2**
```
allowList = [{
                    type: 'public-key',
                    id: credId
                }
options = [{
                up: false
            }]
-> goodAssertion = generateGoodCTAP2GetAssertion
-> getAssertionBuffer = generateGetAssertionReqCBOR

Send request containing getAssertionBuffer to authenticator

RESPONSE:
statusCode = CTAP1_ERR_SUCCESS(0x00)
[to-do]
```
4. Authenticator persists credProtect value with credential. If no extension was included in request, authenticator SHOULD use default value of 1, userVerificationOptional. 

5. Authenticator returns CBOR output with map entry in "extensions" field of authenticatorData object. I.e
```
authenticatorData: {
    ... // other response fields
    { 
        "credProtect": 0x01" 
    }
}
```
The error code returned is CTAP_SUCCESS(0x00).


**CMD: 0x06**
