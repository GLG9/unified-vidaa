# VIDAA-Integration für Home Assistant

Verglichen wurden V4n1X/ha_hisense_tv bei `ee8b2ba` und stevene1919/vidaa bei `603b5ef`. Die Matrix beschreibt implementierte Funktionen. Die Verfügbarkeit hängt vom TV ab.

| Bereich | V4n1X | stevene1919 | Ziel |
| --- | --- | --- | --- |
| Verbindung | aiomqtt, MQTT 3.1, statische Anmeldung, TCP/TLS, Reconnect | aiomqtt, MQTT 3.1.1, statische und dynamische Profile, mTLS und Token-Lebenszyklus | Eine Sitzung pro TV, Profil und Protokoll pro Gerät |
| Zertifikate und Kopplung | RemoteNOW-PEM, PIN, Reauth | Benutzerdateien als PKCS#12 oder PEM, PIN und Token-Erneuerung | Externe Identität, PIN und Cache über HA, öffentliche Diagnose ohne Geheimnisse |
| Discovery und Verfügbarkeit | SSDP, DHCP, WOL, IP-Reparatur, Registry-Metadaten | UPnP, Fingerprint, Profilwahl, WOL, Standby-Modell | IP-Wechsel und WOL am selben stabilen Gerät |
| Media Player | Lautstärke, Mute, HDMI, Apps, Wiedergabe, Ein/Aus | Dieselben Kernfunktionen sowie differenzierte Medien- und Menüfunktionen | Ein Media Player, verfügbare Features aus bestätigten Protokollfähigkeiten |
| Fernbedienung und Dienste | Remote-Tasten, konfigurierbare Kurztasten | Remote, Tastenfolgen, Kanalwahl, App-Start, Text, Bild-/Tonaktionen | Ein Remote, Kurztasten optional, gemeinsame Befehlslogik |
| Sensoren und Diagnostik | Lautstärke, Quelle, TV-Zustand, Firmware/Plattform | Aktive App und Quelle, Sitzungsstatus, gemeldeter TV-Name und Uhrabweichung. Binärsensoren für MQTT, Nutzung und Authentifizierungsfehler | Ein Zustand pro Gerät. Lautstärke und Betriebszustand im Media Player. Bestehende Sensor-IDs bei einer Migration berücksichtigen |
| Automationen | Standardaktionen der HA-Entitäten | Zusätzlich eigene Dienste und ein Einschalt-Trigger mit Legacy-Kompatibilität | Referenz-Trigger und Dienste erhalten, Ziele der bestehenden Automationen bei Migration prüfen |
| Bild, Ton und Wartung | Grundfunktionen über Media Player und Tasten | Bild-/Tonmodus, Helligkeit, Kontrast, Hintergrundbeleuchtung, Audio-only, Reconnect, Uhr, Probe, Reauth | Nur validierte Menüs und Bedienelemente aktivieren |
| DLNA | Ergänzender Mute-Leser, SSDP und feste Renderer-Ports | MQTT als Schwerpunkt | Optionaler Lesepfad für bestätigte Werte. Schreiben erst nach unabhängig bestätigter Wirkung |

## Architekturentscheidung

Das Referenzprojekt ist die bessere Grundlage für den Protokollkern. Es trennt Sitzung, Authentifizierung, Pairing, Tokenpflege und Capability-Funktionen. Der bestehende V4n1X-Branch bleibt Ausgangspunkt für die Diagnose und ein möglicher Kompatibilitätspfad für bestehende HA-Einträge. Zwei aktive Clients für denselben TV würden konkurrierende PIN-Dialoge und widersprüchliche Zustände erzeugen.

## Vertikale Umsetzung

1. Zertifikatsmetadaten, Schlüsselzuordnung und unabhängigen CONNACK ermitteln. Danach reguläre PIN-Kopplung und Token-Ausgabe am TV nachweisen.
2. Einträge über eine Laufzeitinstanz anbinden. IP-Wechsel, Wiederverbindung und Token-Erneuerung testen. Statische Altgeräte über Tests mit unverändertem PIN- und MQTT-3.1-Pfad abdecken.
3. Einen Gerätezustand und einen Media Player durchgängig anbinden. Schreibbefehle anhand nachfolgender TV-Rückmeldung prüfen. Fehlende Werte gezielt per DLNA ergänzen und ihre Herkunft berücksichtigen.
4. Dienste, Kurztasten und Diagnosefunktionen ergänzen. Erweiterte Menüs erst anhand verfügbarer Werte und bestätigter Lese- und Schreiboperationen anbieten. Keine Menü- oder Kalibrierwerte blind setzen.
5. Eine Registry-Migration für vorhandene Einträge mit Testdaten validieren. Bisherige IDs soweit möglich übernehmen. Bestehendes DLNA und HomeKit bleiben in the production HA host eigenständig, bis eine spätere Änderung ausdrücklich vereinbart ist.
6. Beide MIT-Lizenztexte und Copyright-Hinweise für übernommene Dateien erhalten. Zertifikate, Schlüssel und Token bleiben außerhalb von Git. Vor Veröffentlichung und produktivem HA-Einsatz getrennte Freigaben und Verifikation einhalten.

**GO:** Diagnose, PIN- und Funktionsprüfung auf dem Mac. **Freigabekriterium für die Zusammenführung:** live bestätigte MQTT-Paarung inklusive Token, Zustand und Wiederverbindung. Produktive Installation erst nach vollständigen Tests, Sicherung und Konfigurationsprüfung.

## Messung am 65U7DS Pro

`El.p12` enthält ein passendes RSA-2048-Schlüsselpaar mit Zertifikat
`VidaaAppAndroidV01`, ausgestellt von `RemoteCA`, gültig von Juni 2024
bis Juni 2034. Der Referenzclient meldete TLS 1.3 und
`Modern XOR Dynamic Auth: ACCEPTED (rc=0)`. Standard Dynamic, Middle
Dynamic und Legacy Static meldeten jeweils `rc=135`. Bei der Messung
zeigte der TV keine Abweichung seiner Systemuhr. Der Referenzclient hat
danach die reguläre PIN-Anforderung des Fernsehers empfangen.

Die PIN wurde akzeptiert, und der TV stellte Access- und Refresh-Token aus.
Ein erneuter Verbindungsaufbau mit Access-Token lieferte Live-Lautstärkedaten.
Der Fernseher meldete den Home Launcher, vier HDMI-Eingänge und 46 Apps.
Mit dem Access-Token akzeptierte der Broker MQTT 3.1 und 3.1.1.
MQTT-Lautstärke 10 → 12 → 10 und Stummschaltung wurden unabhängig am
TV-Renderer zurückgelesen. DLNA dient in diesem Ablauf nur als Lesekanal.

## Erster Integrationsstand

Der lokale Branch `fix/unified-vidaa` im Checkout `work/vidaa` übernimmt
die MQTT- und Token-Architektur der Referenz und führt die ersten Funktionen
zusammen. Er ergänzt auswählbare Fernbedienungs-Kurztasten für Lauter,
Leiser und Stumm. Die Tasten verwenden denselben Konfigurationseintrag,
dieselbe Gerätedefinition und dieselbe MQTT-Sitzung wie der Media Player.
Die Einrichtung bricht vor der Kopplung ab, wenn für dieselbe Adresse
bereits ein aktiver VIDAA- oder V4n1X-Eintrag vorliegt. Eine DHCP-Meldung
für die gespeicherte primäre MAC kann die IP eines vorhandenen VIDAA-
Eintrags aktualisieren und dieselbe Sitzung am neuen Ziel neu laden.
Dafür liegen automatisierte Tests vor.

Der DLNA-Lesepfad verwendet bei Bedarf den zweiten beobachteten
Renderer-Port und merkt sich den erfolgreichen Port. Vor einem MQTT-Mute-
Toggle liest der Media Player den tatsächlich gemeldeten Mute-Zustand.
Das verhindert eine unnötige Umschaltung bei einer wiederholten Aktion.
Ein isolierter Home Assistant Core 2026.11.2 lud den Branch, legte einen
Media Player für genau ein Gerät an und stellte die Lautstärke über
`media_player.volume_set` erfolgreich auf 12 Prozent. Auch die HA-Aktion
für Stummschaltung änderte den TV-Zustand. Lautstärke und Mute wurden
anschließend zurückgesetzt, der Eintrag entladen und MQTT getrennt.

## Spätere Installation auf the production HA host

1. Home-Assistant-Sicherung anlegen. Vorhandene HomeKit- und DLNA-Einträge
   bestehen lassen. Falls für diesen TV bereits ein aktiver V4n1X-MQTT-Eintrag
   existiert, die Übernahme seiner Entitäten vor der Umstellung planen.
2. Den Ordner `custom_components/vidaa` aus `fix/unified-vidaa` über Samba
   unter `/config/custom_components/vidaa` ablegen. Zertifikat und Schlüssel
   getrennt vom Integrationsordner unter einem geschützten HA-Pfad ablegen.
   Den vorhandenen PKCS#12-Container oder das verifizierte PEM-Paar verwenden.
3. `ha core check` ausführen. Erst bei erfolgreicher Prüfung neu starten.
4. **VIDAA Smart TV** hinzufügen, `192.0.2.10` und das moderne Profil
   auswählen. Im Zertifikatsschritt die Pfade des PEM-Paars angeben und die
   neue PIN am Fernseher ablesen. Die Test-Token bleiben lokal beim Test.
5. Gerät, Media Player und Rückmeldung für Lautstärke und Mute in HA prüfen.

Offen bleiben die automatische Übernahme vorhandener V4n1X-Entitäts-IDs,
zusätzliche V4n1X-Kurztasten und der Hardware-Nachweis für alte RemoteNOW-
Geräte. Der DHCP-Adresswechsel wurde simuliert. Die exakte Verwendung von
Bild-/Tonmenüs, Quellenwechseln sowie Standby/WOL auf dem aktuellen
Gerät wurde in diesem Schritt nicht verifiziert. Leseabfragen für Bild- und
Tonmenüs lieferten keine Menüeinträge. HDMI 1 bis 4 waren in der Quellenliste
sichtbar. Während der Messung war der Home Launcher aktiv.

## Code-level coverage

| Function | Unified implementation | 65U7DS Pro evidence |
| --- | --- | --- |
| Power, standby, Wake-on-LAN | `tv/actions_power.py`, media player controls, Wake-on-LAN configuration | Power-state read. Wake-on-LAN and boot from full standby require a real power-cycle test |
| Volume, step keys, mute | `tv/actions_media.py`, `media_player/controls.py`, `button.py` | Absolute volume, step keys and mute confirmed by independent renderer reads |
| Playback, pause, stop, previous and next | `media_player/controls.py`, MQTT key dispatch | Depends on the active app. No transport test against every app |
| HDMI, app IDs and app labels | `tv/callbacks.py`, `tv/media.py`, `tv/actions_input.py` | Four HDMI IDs and 46 apps read. Availability depends on current source |
| Navigation, key sequences, channel, keyboard | `remote.py`, `services/`, `tv/actions_input.py` | Routing covered by tests. Channel, text and navigation depend on the active screen |
| Picture mode, brightness, contrast, backlight | `select.py`, `number.py`, `tv/settings.py` | No menu response on the tested Home Launcher context. Defaults leave controls disabled |
| Saturation, sharpness, color temperature, dimming, HDR brightness, dynamic contrast and motion | Discovery-based generic picture service in `services/` and `tv/settings.py` | No validated menu IDs or separate SDR, HDR10 and Dolby Vision write routes |
| Sound mode and audio-only | `select.py`, `switch.py`, `tv/settings.py` | No sound-menu response. External output and ARC/eARC are not identified by a confirmed MQTT response |
| Power, screen, playback, source, app and audio state | `tv/state.py`, `media_player/`, `sensor/`, `binary_sensor.py` | Use native media player attributes first. Source and app sensors provide independent automation targets |
| Connection, authentication, token expiry, clock | `binary_sensor.py`, `sensor/diagnostic.py`, `diagnostics.py` | Connection and token renewal tracked without showing secrets |
| Firmware, model, manufacturer, MAC and network | `entity.py`, `tv/fingerprint.py`, device registry, diagnostics | Values appear when advertised by the device. An empty field means it was not returned |
| DHCP, device identity, old integrations | `flow/discovery.py`, `flow/user.py`, `__init__.py` | Primary-MAC DHCP changes tested with simulated events. No old MQTT entry to migrate on the production HA host |

## Production 65U7DS Pro

ARC/eARC was reported in live HA attributes. Both volume keys and the
mute action changed TV feedback. Absolute level changes on this route
were ignored. The integration exposes volume step and mute for ARC/eARC.
The absolute level remains available for the TV speaker route.

HDMI1 selection, YouTube launch and restoration of the original Home
Launcher were verified on the production HA host. The picture and sound menu controls
are disabled for this installation. Exact HDR mode parameters, sound
profiles, transport keys by app, channel/text input, older hardware and
migration of old V4n1X entity IDs have no production verification.

The standalone V4n1X-style volume, TV state and firmware/hardware sensors
are also provided. They read the existing MQTT client's cache rather than
opening a second session. Firmware or platform remains unknown until the
TV supplies it.
