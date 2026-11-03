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
| V2 | Slaapkamer |

People: Mathijs, Patricia.

## Climate
- Floor heat: **Versatile Thermostat + SmartPI** on klepmotor switches → `climate.<kamer>_vv`
- Daikin airco: separate → `climate.*_room_temperature`
- Areas organize; they do not bind sensor↔actuator
- ZW glass (Keuken, Lounge, Badkamer Ben): fast solar gains
- WTW / HomeVap: own automations
- Pump with zone valves: Constant Pressure AUTOADAPT preferred

## Zigbee
- ZHA (SkyConnect) for Sonoff/kleppen; Hue Bridge stays for mass Hue lighting

## Dashboards
- `/fort-knox/…`, `/fortknox-hq/…` (Mushroom Shadow)
