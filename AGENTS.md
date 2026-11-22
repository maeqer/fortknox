# Fort Knox

Standing Home Assistant context for agents. Same source of truth as `.cursor/rules/ha-fortknox.mdc` — update both when a decision changes.

## Purpose
Configure the house Home Assistant from Cursor via MCP. Prefer live HA queries over memory dumps.

## Access
- HA: `http://192.168.1.130:8123`
- MCP servers in `.cursor/mcp.json` (gitignored): builtin `home-assistant` + community `ha-mcp`
- Token: Windows user env `HA_TOKEN`
- Config work → **ha-mcp** + `ha_get_skill_guide` first

## Layout
| Floor | Areas |
|-------|--------|
| BG | Meterkast, Hal BG, Toilet (`area_id: entree`), Keuken, Studio, Berging |
| V1 | Kantoor, Hal V1, Slaapkamer Ben, Badkamer Ben, Lounge |
| V2 | Slaapkamer, Grote badkamer |

People: Mathijs, Patricia.

## Climate
- Floor heat: **Versatile Thermostat + SmartPI** on klepmotor switches → `climate.<kamer>_vv`
- VV groepen: Lounge 4 · Keuken 4 · Grote badkamer 2 · Slaapkamer 2 · overige zones 1 elk (=18). Warmte-aandeel ≈ groepen × klep-duty
- Daikin airco: separate → `climate.*_room_temperature`
- Areas organize; they do not bind sensor↔actuator
- ZW glass (Keuken, Lounge, Grote badkamer): fast solar gains
- Orientation: **ZW** Keuken, Lounge, Grote badkamer · **NO** Slaapkamer Ben, Kantoor, Slaapkamer · **NW** Badkamer Ben, Studio · **ZO** Hal BG · **inner/little sun** Hal V1, Toilet, Badkamer Ben
- WTW vocht: `automation.wtw_vocht_aan` → `script.wtw_vocht_boost_naar_stand3` (`met_boost` true/false) + `automation.wtw_vocht_uit` (max **trigger-ruimtes** &lt; `wtw_rh_herstel_drempel` **55%** voor 15′). **Drie paden:** (1) snelle stijging in **natte zones** (grote badkamer, badkamer Ben, keuken, toilet): Δ≥`wtw_vocht_stap_drempel` + RH≥`wtw_rh_stijging_vloer` **50%** → één boost 15′ daarna stand 3; (2) `sensor.woonruimte_max_vochtigheid` ≥ `wtw_rh_hoog_drempel` **60%** / 2′ → stand 3; (3) ≥ `wtw_rh_extreem_drempel` **65%** / 2′ → één boost. Geen herhaalde boost als RH hoog blijft — stand 3 blijft tot herstel. **Vals alarm:** (a) trigger-max &lt; 50% voor 2′ → `automation.wtw_vocht_vals_alarm`; (b) 10′ na sessie-start nog &lt; 55% → `automation.wtw_vocht_vals_alarm_geen_hoogte`. Beide stoppen boost/script + vorige stand. Woonruimte-max = alle zones incl. berging (was). **Vlag mag niet blokkeren** (script `restart`). Trigger-ruimtes: `input_text.wtw_vocht_trigger_ruimtes` → `sensor.wtw_vocht_trigger_max`. **Sessie vrijgeven:** echte daling fanstand ≥3 → ≤2 → `automation.wtw_vocht_sessie_vrij_bij_stand2` (negeert Smarty unavailable→66). HomeVap: eigen automations
- Pump with zone valves: Constant Pressure AUTOADAPT preferred
- **VV temp heartbeat (mandatory):** Sonoff SNZB-02P reports temp change-only → VT safety false-trips. Every VV zone needs template `sensor.<kamer>_temp_voor_vv` that depends on both temp + humidity (RH keeps it “alive”), and VT `temperature_sensor_entity_id` must point at that template (not the raw Sonoff temp). Restore humidity + outdoor (`sensor.adam_outdoor_temperature`) when rewriting VT options. Naming: `climate.toilet_hal_vv` → `sensor.toilet_hal_temp_voor_vv`.
- **Zonrem:** `automation.vv_zonrem_zw` — Mon–Sat, if `sensor.zon_zw_vandaag` ≥ `input_number.zonrem_drempel`: 07:00 ZW zones → `input_number.zonrem_temperatuur`; back to comfort when room >19.4 after 10:00, latest 13:00. Only zones with SmartPI `stable` + preset comfort. Zones list: Lounge, Keuken, Grote badkamer. Sunday none. Night stays comfort; FF3 on (SmartPI defaults)
- VT lock off on all VV zones (`lock_users`/`lock_automations` false) — set the same on new zones
- **New VV rooms:** apply the same template + VT rewire immediately; HA cannot create that from stock automations. Vangnet: `automation.vv_safety_bypass_bij_levende_rh_alle_zones` (RH-alive → unlock + long safety delay) and `automation.vv_mist_temp_voor_vv_heartbeat` (notify if template missing).

## Naming
- Device names bare (`Temperatuursensor`, `Klepmotor`); room comes from area
- Entity IDs `<kamer>_<rol>`: `sensor.<kamer>_temperatuur`, `sensor.<kamer>_vochtigheid`, `switch.<kamer>_klepmotor`; diagnostics `<kamer>_temperatuursensor_<x>` / `<kamer>_klepmotor_<x>`
- Areas set `temperature_entity_id` / `humidity_entity_id` to the room sensor

## Zigbee / Thread / Matter
- ZHA on ZBT-1 `…6a7142…` (Sonoff/kleppen) — niet flashen voor Thread
- OTBR on tweede ZBT-1 `…7abedcf…`; Thread `ha-thread-0434`; URL `http://core-openthread-border-router:8081`
- Matter Server + OTBR blijven beschikbaar; **Nuki Smart Lock Pro 4** → **MQTT over wifi** (niet Matter). Mosquitto + MQTT-integratie actief
- Hue Bridge blijft voor mass Hue lighting

## CCTV (Hikvision)
- NVR: **DS-7608NXI-K2/8P** @ `192.168.1.193` (8-ch AcuSense PoE)
- Camera (o.a. Voortuin): **DS-2CD2387G2H-LIU** zwart 2.8mm (AcuSense / ColorVu hybrid)
- Channels: 1 Voortuin · 2 Zijgevel voor · 3 Zijgevel achter · 4 Achtertuin · 5 Parkeerplaats · **6 Deurbel** (`192.168.1.194`, manual)
- Deurbel: **DS-KV6124-WBE1** @ `192.168.1.194` — NVR ch6 (continue opname) + Frigate `camera.deurbel` direct (RTSP + ISAPI 2-weg); zone `stoep`. Hik-Connect gebonden. Nuki Pro 4: MQTT (wifi) i.p.v. Matter
- **Detectie-pad: Frigate Full Access** (`ccab4aaf_frigate-fa` 0.18) + HA-integratie — Hikvision smart events uit; NVR blijft recorder + RTSP-bron (camera’s 1–5)
- Streams: main ~4K H.265 (record); sub ~640×480 **H.264** (Frigate detect). OpenVINO **GPU** + VAAPI via Proxmox iGPU PCI passthrough (`0000:00:02.0` → HA VM 100; `/dev/dri/renderD128`). Protection mode Frigate FA staat **uit**.
- Cameras: `camera.voortuin` · `zijgevel_voor` · `zijgevel_achter` · `achtertuin` · `parkeerplaats` · `deurbel`
- Zones (alerts): Voortuin `terrein` · Zij voor `pad` · Zij achter `erf` · Achtertuin `terras` · Parkeerplaats `parkeren` · Deurbel `stoep`. Detect fps 5. Face + LPR aan (LPR actief op Voortuin + Parkeerplaats).
- Meldingen: `automation.frigate_persoon_melding` + `automation.frigate_auto_melding` (nacht óf niemand thuis). Legacy `automation.camera_line_crossing_melding` staat uit.
- Camera-UI: alleen **`/fort-knox/camera-s`** — deurbel bovenaan + `picture-entity` (camera_view auto, 16:9 cover) + detectie-tiles; geen ACC-grid (te zwaar/instabiel op mobiel). Tik camera → more-info voor live.

## Dashboards
- **Actief (enige):** `/fort-knox/…` — alle dashboard-wijzigingen hier
- `/fortknox-hq/…` — **niet in gebruik**; niet onderhouden tenzij Mathijs anders vraagt
