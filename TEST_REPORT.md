# Unified VIDAA test report

## Prior live verification on the Mac

The 65U7DS Pro accepted TLS 1.3, Modern XOR initial login, regular PIN
pairing and authenticated MQTT 3.1 and 3.1.1. A separate renderer session
confirmed volume 10 → 12 → 10, both step buttons, mute and restored mute.
Isolated Home Assistant Core 2026.11.2 created one device and confirmed
volume and mute service calls against the actual TV.

## Current regression checks

Before the security fix, a normal PKCS#12 extraction wrote the key in mode
0644. After the fix the regression asserts owner-only permissions.
Before the pairing identity fix, two clients for the same TV produced
the same MQTT client identifier. The regression now verifies separate
client identities and a stable ID across attempts within one client.

Full test suite: `387 passed in 156.62s (0:02:36)`.
Ruff: `All checks passed!`
Home Assistant configuration: `Command completed successfully.`

## Production live tests

The PIN flow completed and the integration registered the TV, a media
player, a remote, session/source/app sensors, MQTT/authentication binary
sensors and repair/maintenance buttons. The TV reported ARC/eARC and
MQTT connected. Direct absolute volume requests returned successfully
but the renderer still reported 47 percent. Remote volume down and
volume up changed the live feedback to 46 and back to 47 percent. The
mute service changed the TV feedback to muted and back to unmuted.
HDMI1 reported as Apple TV, YouTube launched and the Home key restored
the original Home Launcher. The force-reconnect button restored the
runtime MQTT session.

The final suite reported `388 passed in 156.82s (0:02:36)`. Ruff also
passed. Following the final Core restart, the saved pairing reconnected
without another PIN. The additional standalone sensors reported a live
volume level, TV state and firmware. The TV powered off and woke from
standby through Home Assistant. The MQTT port stayed open during standby,
so this exercise did not isolate Wake-on-LAN from the ordinary wake path.
The TV was restored to HDMI1, its Apple TV input. At the final check the
TV was again in standby and the MQTT connection was still online.

No VIDAA errors or critical VIDAA records appeared in the final log
window, and the active repair report had no VIDAA issue. The television
continued advertising an already-paired HomeKit service (`sf=0`).
Apple Home itself was not used as a test client.

Automated coverage is distinct from device verification. Token renewal,
reauthentication, DHCP updates and media command routing have automated
coverage. A real DHCP lease change, expired-token renewal, switching
between SDR/HDR10/Dolby Vision, direct channel and keyboard commands,
media transport in an active Apple TV stream, and legacy hardware
migration were not exercised in production.
