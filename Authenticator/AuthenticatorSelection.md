# AuthenticatorSelection (0x0B)

This command is lets a user select a certain authenticator by asking for user presence.

When the command is received, the authenticator will ask for user presence:

- If received, authenticator will return `CTAP2_OK`.

- If explicitly denied by the user, authenticator will return `CTAP2_ERR_OPERATION_DENIED`.

- If user action timeout occurs, the authenticator will return `CTAP2_ERR_USER_ACTION_TIMEOUT`.

If an authenticator is selected, the platform SHOULD send a cancel to all other authenticators.
