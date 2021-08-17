# AuthenticatorSelection (0x0B)

This command is part of the authenticator API and is new to CTAP2.1. It allows the platform to let a user select a certain authenticator by asking for user presence. The following has been taken directly from the [original specs](https://fidoalliance.org/specs/fido-v2.1-ps-20210615/fido-client-to-authenticator-protocol-v2.1-ps-20210615.html#authenticatorSelection).
<br /><br />

The command has no input parameters.

When the authenticatorSelection command is received, the authenticator will ask for user presence:

- If User Presence is received, the authenticator will return **CTAP2_OK**.

- If User Presence is explicitly denied by the user, the authenticator will return **CTAP2_ERR_OPERATION_DENIED**. The platform SHOULD NOT repeat the command for this authenticator.

- If a user action timeout occurs, the authenticator will return **CTAP2_ERR_USER_ACTION_TIMEOUT**. The platform MAY repeat the command for this authenticator.

If an authenticator is selected, the platform SHOULD send a cancel to all other authenticators.
