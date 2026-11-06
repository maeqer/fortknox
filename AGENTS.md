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
- VV groepen: Lounge 4 · Keuken 4 · overige actieve zones 1 elk (nu 14). Later: Grote badkamer 2 · Slaapkamer V2 2. Warmte-aandeel ≈ groepen × klep-duty
- Daikin airco: separate → `climate.*_room_temperature`
- Areas organize; they do not bind sensor↔actuator
- ZW glass (Keuken, Lounge, Grote badkamer): fast solar gains
- Orientation: **ZW** Keuken, Lounge, Grote badkamer · **NO** Slaapkamer Ben, Kantoor, Slaapkamer · **NW** Badkamer Ben, Studio · **ZO** Hal BG · **inner/little sun** Hal V1, Toilet, Badkamer Ben
- WTW / HomeVap: own automations
- Pump with zone valves: Constant Pressure AUTOADAPT preferred
- **VV temp heartbeat (mandatory):** Sonoff SNZB-02P reports temp change-only → VT safety false-trips. Every VV zone needs template `sensor.<kamer>_temp_voor_vv` that depends on both temp + humidity (RH keeps it “alive”), and VT `temperature_sensor_entity_id` must point at that template (not the raw Sonoff temp). Restore humidity + outdoor (`sensor.adam_outdoor_temperature`) when rewriting VT options. Naming: `climate.toilet_hal_vv` → `sensor.toilet_hal_temp_voor_vv`.
- **Zonrem:** `automation.vv_zonrem_zw` — Mon–Sat, if `sensor.zon_zw_vandaag` ≥ `input_number.zonrem_drempel`: 07:00 ZW zones → `input_number.zonrem_temperatuur`; back to comfort when room >19.4 after 10:00, latest 13:00. Only zones with SmartPI `stable` + preset comfort. Zones list: Lounge, Keuken, Grote badkamer. Sunday none. Night stays comfort; FF3 on (SmartPI defaults)
- VT lock off on all VV zones (`lock_users`/`lock_automations` false) — set the same on new zones
- **New VV rooms:** apply the same template + VT rewire immediately; HA cannot create that from stock automations. Vangnet: `automation.vv_safety_bypass_bij_levende_rh_alle_zones` (RH-alive → unlock + long safety delay) and `automation.vv_mist_temp_voor_vv_heartbeat` (notify if template missing).

## Naming
- Device names bare (`Temperatuursensor`, `Klepmotor`); room comes from area
- Entity IDs `<kamer>_<rol>`: `sensor.<kamer>_temperatuur`, `sensor.<kamer>_vochtigheid`, `switch.<kamer>_klepmotor`; diagnostics `<kamer>_temperatuursensor_<x>` / `<kamer>_klepmotor_<x>`
- Areas set `temperature_entity_id` / `humidity_entity_id` to the room sensor

## Zigbee
- ZHA (SkyConnect) for Sonoff/kleppen; Hue Bridge stays for mass Hue lighting

## Dashboards
- `/fort-knox/…`, `/fortknox-hq/…` (Mushroom Shadow)
