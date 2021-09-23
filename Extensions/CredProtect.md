# Extension: CredProtect

SUPPORT OF CREDPROTECT IS MANDATORY FOR ALL CTAP2.1 AUTHENTICATORS

Credential Protection extension provides enterprise platform ability to enforce stricter user verification requirements for the specific credentials.

This extension is only applicable during the credential creation `MakeCredential(0x01)`

There are three modes for credentials:

- User Verification Optional - DEFAULT VALUE IF CREDPROTECT IS MISSING. Platform may request assertion for a credential, with UP false, and UV false, or in a silent mode. This applicable for both, roaming credential (with credentialIDl), and for discoverable(resident) credential. 
- User Verification Optional With CredentialID List - If credentialID was specified, platform may obtain assertion without user verification, silently. It is still mandatory to obtain user verification for the discoverable(resident) credential.
- User Verification Required - In any case, with or without credential list, platform must always obtain explicit user verification before obtaining assertion.


CredProtect modes ENUM:

```
userVerificationOptional: 0x01
userVerificationOptionalWithCredentialIDList: 0x02
userVerificationRequired: 0x03
```

Example `MakeCredential(0x01)` request with `CredProtect` extension:

```
subCommand       = 0x04 
subCommandParams = {
    0x01: a379a6f6eeafb9a55e378c118034e2751e682fab9f2d30ab13d2125586ce1947 // rpIDHash of example.com
} 
                 = a1015820a379a6f6eeafb9a55e378c118034e2751e682fab9f2d30ab13d2125586ce1947

message = 0x04 || a1015820a379a6f6eeafb9a55e378c118034e2751e682fab9f2d30ab13d2125586ce1947

pinUvAuthParam  = HMAC-SHA-256(key = puat, message = message)
                => 340ff740ea8c0daf48c5df402c65735315d296cc1982cd1fa184918bf7407c8b
```

```
REQ: 0101a70158206905d67f4c0fe55e6a39cd3f74c0e31e3ba86ccf8949d21c7dfbd1f0babf2d4502a26269646b6578616d706c652e636f6d646e616d657829546865204578616d706c6520436f72706f726174696f6e20776974682066616b6520646f6d61696e2103a362696458208520a240e8b44e31e13e5387aaef7af301d0a97661120fb58783dea341c1ab2c646e616d65782b646f6e657474616c756b617369657769637a406d61676e69666963656e7477617465726d656c6f6e2e6e676b646973706c61794e616d6573446f6e65747461204c756b617369657769637a0481a263616c672664747970656a7075626c69632d6b657906a16b6372656450726f746563740308582006e0e04ebbd0a1c73cdfd1d06a39dd4f65e8d6047fbd3981f1b477fcbba8810b0902

CMD: 0x01
PAYLOAD: 01a70158206905d67f4c0fe55e6a39cd3f74c0e31e3ba86ccf8949d21c7dfbd1f0babf2d4502a26269646b6578616d706c652e636f6d646e616d657829546865204578616d706c6520436f72706f726174696f6e20776974682066616b6520646f6d61696e2103a362696458208520a240e8b44e31e13e5387aaef7af301d0a97661120fb58783dea341c1ab2c646e616d65782b646f6e657474616c756b617369657769637a406d61676e69666963656e7477617465726d656c6f6e2e6e676b646973706c61794e616d6573446f6e65747461204c756b617369657769637a0481a263616c672664747970656a7075626c69632d6b657906a16b6372656450726f746563740308582006e0e04ebbd0a1c73cdfd1d06a39dd4f65e8d6047fbd3981f1b477fcbba8810b0902

DECODED:
{
    1: 6905d67f4c0fe55e6a39cd3f74c0e31e3ba86ccf8949d21c7dfbd1f0babf2d45,// ClientDataHash
    2: { // RP
        id: "example.com",
        name: "The Example Inc"
    },
    3: { // User
        id: 8520a240e8b44e31e13e5387aaef7af301d0a97661120fb58783dea341c1ab2c, 
        name: "donettalukasiewicz@magnificentwatermelon.ng",
        displayName: "Donetta Lukasiewicz"
    },
    4: [ // PublicKeyCredParams
        {
            alg: -7,
            type: "public-key"
        }
    ],
    6: {
        credProtect: 3 // userVerificationRequired
    },
    8: 06e0e04ebbd0a1c73cdfd1d06a39dd4f65e8d6047fbd3981f1b477fcbba8810b, // PinUvAuthParam
    9: 2 // PinUvAuthProtocol
}
```

The device then responds with the `CTAP_SUCCESS(0x00)` and attestation response. When Platform tries to call GetAssertion with `uv` set to false, the authenticator will return `CTAP2_ERR_CREDENTIAL_EXCLUDED(0x19)`.