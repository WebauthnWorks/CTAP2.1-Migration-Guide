# Enterprise Attestation

An enterprise attestation is an attestation that may include uniquely identifying information. 

An enterprise attestation capable authenticator MAY support either or both:

- Vendor-facilitated enterprise attestation

- Platform-managed enterprise attestation 

Example: Platform enterprise attestation

```
REQ:

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
	8: 06e0e04ebbd0a1c73cdfd1d06a39dd4f65e8d6047fbd3981f1b477fcbba8810b, // PinUvAuthParam
    9: 2 // PinUvAuthProtocol
	10: 2 // platform-managed enterprise attestation
}
```
