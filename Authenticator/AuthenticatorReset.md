# AuthenticatorReset (0x07)

This command is part of the authenticator API and is new to CTAP2.1. The following has been taken directly from the [original specs](https://fidoalliance.org/specs/fido-v2.1-ps-20210615/fido-client-to-authenticator-protocol-v2.1-ps-20210615.html#authenticatorReset).
<br></br>

Resetting an authenticator is a potentially destructive operation. Authenticators MAY thus choose, for each transport they support, whether this command will be supported when received on that transport. For example, an authenticator may choose not to support this command over NFC, fearing that coincidentally nearby readers may send malicious reset commands.

However this command MUST be supported on at least one transport. If the USB HID transport is supported then this command MUST be supported on that transport.

This method is used by the client to reset an authenticator back to a factory default state. Specifically this action at least:

- Invalidates all generated credentials, including those created over CTAP1/U2F.

- Erases all discoverable credentials.

- Resets the serialized large-blob array storage, if any, to the initial serialized large-blob array value.

- Disables those features that are denoted as being subject to disablement by authenticatorReset:

  - Enterprise attestation

- Resets those features that are denoted as being subject to reset by authenticatorReset:

  - Always Require User Verification

  - Set Minimum PIN Length


Additionally:

  - In order to prevent an accidental triggering of this mechanism, evidence of user interaction is required.

- In case of authenticators with no display, request MUST have come to the authenticator within 10 seconds of powering up of the authenticator.

If all conditions are met, authenticator returns **0x00 CTAP2_OK**. If this command is disabled for the transport used, the authenticator returns **0x27 CTAP2_ERR_OPERATION_DENIED**. If user presence is explicitly denied, the authenticator returns **0x27 CTAP2_ERR_OPERATION_DENIED**. If a user action timeout occurs, the authenticator returns **0x2F CTAP2_ERR_USER_ACTION_TIMEOUT**. If the request comes after 10 seconds of powering up, the authenticator returns **0x30 CTAP2_ERR_NOT_ALLOWED**.

[Status codes](https://fidoalliance.org/specs/fido-v2.1-ps-20210615/fido-client-to-authenticator-protocol-v2.1-ps-20210615.html#error-responses)

