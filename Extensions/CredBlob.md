# CredBlob Extension

This extension enables RPs to provide a small amount of extra credential configuration information (credBlob value) to the authenticator when a credential is made.

This extension is only applicable with `authenticatorMakeCredential` or `authenticatorGetAssertion` 


Example `MakeCredential(0x01)` request with `credBlob` extension:

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
    6: {
        credBlob:"foo" // 2263726564426c6f62223a22666f6f22
    },
    8: 340ff740ea8c0daf48c5df402c65735315d296cc1982cd1fa184918bf7407c8b, // PinUvAuthParam
    9: 2 // PinUvAuthProtocol
}
```

The device then responds with the `CTAP_SUCCESS(0x00)` and makeCredential response, WITH credBlob = true in extensions field
### elaborate

