# the production HA host deployment

Home Assistant Core 2026.11.2. The pre-change Supervisor configuration
backup is `the local backup` and excludes the database. The previously installed
VIDAA directory was retained as `/config/vidaa-before-20261122`.
There were no configured VIDAA or V4n1X MQTT entries. The existing
Hisense DLNA entry, Apple TV integrations and HomeKit entries were retained.

Files were staged and read back over signed Samba before activation.
The certificate bundle and PEM pair are under `/ssl/vidaa-unified`,
with directory permissions 0700 and credential files 0600. The persistent
PEM paths were selected because the runtime's /ssl mount was read-only
during PKCS#12 conversion. `ha core check` succeeded before each restart.
An auto-discovered flow for this same TV was dismissed so the explicit
modern-profile setup could proceed.

Rollback uses the Supervisor backup or, for code-only rollback, disables
the new config entry, restores the previous component directory through
Samba, checks the HA configuration and restarts Core. It does not
remove any previous HomeKit or DLNA entries.

The production PIN completed after the client identity fix. The new
entry `the TV` loaded and registered
`media_player.example_tv` and `remote.example_tv`. The existing
`media_player.example_dlna` belongs to the old native DLNA entry and remains
in place. It does not open a competing MQTT connection.

The final restart used the saved pairing. One VIDAA device was online,
with separate volume, TV-state, firmware/hardware, app and source
sensors. The two additional diagnostic sensors for the reported name
and clock drift were enabled through the entity registry. The native
DLNA and Apple TV integrations and HomeKit pairing remain in place.
