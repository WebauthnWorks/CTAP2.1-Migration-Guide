# AuthenticatorSelection 0x0B

AuthenticatorSelection is used to help user select which device they would like to use. For example:

During the device configuration, user may have multiple authenticators plugged in. Then platform will send AuthenticatorSelection(0x0B) to the devices causing them to provide visual indication that they are waiting for user presence like blinking light. As soon as the user touches the authenticator, it will return CTAP_OK(0x00).


**Generating request**
```
REQ: 0x0B
```

On success the authenticator must return `CTAP_SUCCESS(0x00)`.

On timeout the authenticator will return `CTAP2_ERR_USER_ACTION_TIMEOUT(0x00)`.

If user has explicitly declined request, like BLE authenticator with a screen, then device must return `CTAP2_ERR_OPERATION_DENIED(0x27)`.