# Extension: LargeBlobKey

LargeBlobKey extension allows authenticator to generate an opaque random, 32 byte array associated with the credential. LargeBlobsKey can be used together with the LargeBlobs to encrypt/decrypt data.

The key MUST not be reused from other extensions such as [Hmac-Secret](../HmacSecret.md)

## Feature detection

Authenticator advertises support for this extension by returning `largeBlobKey` in the `GetInfo.extensions(0x02)`.

Authenticator that supports `largeBlobKey` must also support `LargeBlobs` option.

## Create/MakeCredential

To enable extension the Platform will send MakeCredential with extensions map containing `largeBlobKey` set to true.

```
MakeCredential_Request
{
    1: fe87bb84f9ee...,    // clientDataHash
    2: {...},              // rp
    3: {...},              // user
    4: [...],              // pubKeyCredParams
    5: [...],              // excludeList
    6: {                   // extensions
        "largeBlobKey": true
    },
    8: 51e971f093509e4..., // pinUvAuthParam
    9: 2                   // pinUvAuthProtocol
}
```

The the authenticator will respond with MakeCredential result, `largeBlobKey(0x05)` set to new key.

```
MakeCredential_Result
{
    1: "packed",             // fmt
    2: 9569088f1ecee32329... // authData
    3: {...},                // attStmt
    // largeBlobKey 32 byte long
    5: 60d4ebc791530405738f00b18d0d1dec63726906e2ef692169e183ad7e6b39d2
}
```

## Get/GetAssertion

The platform can retrieve `largeBlobKey` for the existing credential by sending GetAssertion request with the extension containing `largeBlobKey` set to true:

```
GetAssertion_Request
{
    1: "webauthn.works",   // rpid
    1: 39fcc878ea01...,    // clientDataHash
    2: [...],              // allowList
    4: {                   // extensions
        "largeBlobKey": true
    },
    6: 5544d1bf5f49563..., // pinUvAuthParam
    7: 2                   // pinUvAuthProtocol
}
```

The the authenticator will respond with GetAssertion result, `largeBlobKey(0x07)` set to the credential largeBlobKey key.

```
GetAssertion_Result
{
    1: {...},                // credential
    2: 824fc6f45e0d2ee82f... // authData
    3: 2a7eebb66f64b46f13... // signature
    // largeBlobKey 32 byte long
    7: 60d4ebc791530405738f00b18d0d1dec63726906e2ef692169e183ad7e6b39d2
}
```

## Credential Management API

The LargeBlobKey can be accessed as well via [CredManAPI](../Protocol/CredentialManagement.md) credential enumeration response `largeBlobKey(0x0B)`.