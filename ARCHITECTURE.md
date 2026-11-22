# Unified VIDAA architecture

The `vidaa` config entry owns one device and one supervised MQTT session.
The config flow has a short-lived session for initial pairing or reauthentication.
The pairing session closes before the runtime session takes ownership.

| Concern | Implementation |
| --- | --- |
| Discovery and config flows | `flow/discovery.py`, `flow/user.py`, `flow/certs.py`, `flow/reauth.py`, `flow/options.py` |
| TLS and initial authentication | `protocol/certs.py`, `protocol/crypto.py`, `protocol/pairing.py` |
| Session, dispatch and renewal | `protocol/connection.py`, `protocol/dispatcher.py`, `protocol/auth_lifecycle.py`, `tv/tokens.py` |
| Commands, state, capabilities | `tv/actions_*.py`, `tv/state.py`, `tv/settings.py`, `tv/callbacks.py` |
| Home Assistant platforms | `media_player/`, `remote.py`, `button.py`, `sensor/`, `binary_sensor.py`, `select.py`, `number.py`, `switch.py` |
| Direct TV observations | `protocol/upnp.py` reads RenderingControl and AVTransport. The known ports are 38400 and 18400. |

MQTT owns power, source, app and volume state. A fresh DLNA mute reading
can inform a mute action before sending the MQTT toggle. DLNA confirms
volume changes independently. The 65U7DS Pro returned HTTP 200 for direct
DLNA volume writes without changing its volume, so it uses MQTT for writes.

The modern Android certificate is provided at deployment time. PKCS#12
extraction writes owner-only PEMs and falls back to a private temporary
directory when the selected location is unwritable. Git contains neither
the supplied certificate bundle nor its derived key or active tokens.

The TV MAC is reserved for discovery, DHCP tracking and Wake-on-LAN.
A new pairing client uses its own synthetic identity for the lifetime of
the pairing attempt. The resulting MQTT identifier is stored with the
issued tokens and reused for runtime connections.

The media player dynamically advertises absolute volume for the TV
speaker route. For the TV's ARC/eARC volume type it publishes the
relative key capability and mute, which were effective in production.

The volume, TV state and hardware sensors subscribe to the same client's
events as the media player. A missing firmware response is represented
as unknown.
