# VIDAA on 65U7DS Pro

- Symptom. Cold RenderingControl calls time out on the default port.
  Cause. This TV listens on 18400. Rule. Retry cold reads on the other known
  renderer port and retain the successful port.
- Symptom. A repeated mute request can toggle the TV when local MQTT state
  is stale. Cause. The key is a toggle and local state can lag behind the
  device. Rule. Query the live renderer before issuing the MQTT toggle.
- Symptom. The CLI state snapshot raises AttributeError despite a successful
  MQTT connection. Cause. It uses obsolete source/app property names.
  Rule. Read the current state properties from the client.
- Symptom. Adding another entry can start pairing while the same TV already
  has an MQTT client. Cause. A MAC check alone does not catch entries added
  under another integration or without a MAC. Rule. Check the host of an
  enabled existing VIDAA or V4n1X entry before pairing.
- Symptom. An existing MQTT session uses the old address after a DHCP lease
  changes. Cause. Discovery by IP cannot track a device that has moved. Rule.
  Match the registered primary MAC and reload the same entry after updating
  its address. Avoid cycling between leases on different interfaces.
- Symptom. Production pairing reached CONNACK but had no PIN response.
  Cause isolated for testing. The HA flow reused the TV hardware MAC as
  the mobile client ID input. Rule. Keep the pairing client identity
  separate from the TV hardware MAC.
- Symptom. PKCS#12 extraction wrote a world-readable key with a typical
  umask and could fall back to a shared predictable directory. Cause.
  Direct regular file opens used default permissions. Rule. Use exclusive
  owner-only temporary files and a private fallback directory.
- Symptom. Absolute MQTT volume requests received service acknowledgement
  but did not move the TV while ARC/eARC was active. The volume keys
  changed the reported volume and Mute also changed state. Rule. Advertise
  volume step and Mute for the ARC route. Keep absolute setting for the TV
  speaker route.
