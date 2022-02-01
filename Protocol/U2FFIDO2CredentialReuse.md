# U2F with CTAP2.1

It is generally advised for manufacturers to support U2F to ensure global platform compatibility.

If manufacturer supports U2F, it must specify `U2F_V2` in the `GetInfo.versions(0x01)` list.

FIDO2.1 devices supporting U2F, must support usage of U2F created credentials in CTAP2

This means that if credential was created using U2F protocol, it must be accessible if specified in CTAP2.

The credential must not be discoverable.

In contrary, the credentials created using CTAP2 protocol, can not be accessible over U2F.


Ref: https://fidoalliance.org/specs/fido-v2.1-ps-20210615/fido-client-to-authenticator-protocol-v2.1-ps-20210615.html#cross-version-credentials