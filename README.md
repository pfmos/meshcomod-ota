# meshcomod OTA artifacts

Firmware images staged here solely so a MeshCore repeater can fetch them over
the air. `meshcoreHttpOtaUrlAllowed()` in meshcomod only permits OTA downloads
from `raw.githubusercontent.com`, GitHub `/raw/` URLs, and the meshcomod
domains — hence a public repo rather than a local HTTP server.

## maxwell-heltec-v4-repeater-tcp-mqtt.bin

App-only image for **Heltec V4 (OLED)**, env `heltec_v4_repeater_tcp_mqtt`,
built from meshcomod `work/1.17.1`. Flash at `0x10000` only when the partition
table and bootloader already match.

Adds, on top of `heltec_v4_repeater_tcp`:

- **`MqttBridge`** — a publish-only MC2MQTT observer feed. RX is taken from
  `logRxRaw()` (the only hook carrying SNR/RSSI, fired pre-dedup); TX from
  `logTx()`, which is the point: a half-duplex radio never hears itself, so a
  repeater's own relays are invisible to aggregators without it. Idle unless
  `mqtt.host`, `mqtt.port` and `mqtt.iata` are all set.
- **An OTA rollback guard** — overrides Arduino's weak `verifyRollbackLater()`
  so a freshly-flashed image stays `PENDING_VERIFY` and only confirms itself
  after 10 minutes of healthy uptime. Without it, Arduino marks the image valid
  before `setup()` runs and a crash bootloops forever.

Contains no credentials, keys, or node identity — identity lives in SPIFFS and
all MQTT settings are runtime prefs.

## maxwell-heltec-v4-repeater-tcp-mqtt-status-stats-20260906.bin

`heltec_v4_repeater_tcp_mqtt` with the MQTT `/status` document fixed to match the
MC2MQTT contract the rest of the observer fleet publishes:

- adds the `stats` block (battery, uptime, packet counters, queue, noise floor,
  RSSI/SNR, air time, recv errors, heap), pushed in from `MyMesh` — it was never
  implemented, which is why Maxwell's `/status` carried no telemetry
- `timestamp` is now an ISO8601 string, not a bare integer
- status is published **retained**, so a late subscriber sees current state
- registers a retained MQTT last will, so an ungraceful death reports `offline`
  instead of the node silently vanishing
- adds the missing `client_version`

Validated on a bench Heltec V3 before release. The previous image
(`maxwell-heltec-v4-repeater-tcp-mqtt.bin`) is retained here as the rollback target.
