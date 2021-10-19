# Enterprise Attestation

An enterprise attestation is an attestation that may include uniquely identifying information. 

This extension is only applicable during the credential creation `MakeCredential(0x01) `

An enterprise attestation capable authenticator MAY support either or both:

- Vendor-facilitated enterprise attestation, enterpriseAttestation = 1

- Platform-managed enterprise attestation, enterpriseAttestation = 2

**Note:** Authenticators wishing to support only vendor-facilitated enterprise attestation MAY treat enterpriseAttestation = 2 the same as enterpriseAttestation = 1.

Example `MakeCredential(0x01)` request with `platform-managed enterprise attestation` extension:
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
REQ: 0x01 ...

CMD: 0x01
PAYLOAD: ...

// verify this 
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
    8: 340ff740ea8c0daf48c5df402c65735315d296cc1982cd1fa184918bf7407c8b, // PinUvAuthParam
    9: 2 // PinUvAuthProtocol
    10: 2 // platform-managed enterprise attestation
}
```

The device then responds with the `CTAP_SUCCESS(0x00)`, attestation response, and epAtt (0x04) in response set to TRUE.
