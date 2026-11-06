# switchmanager_config

Home Assistant [Switch Manager](https://github.com/macpit/Home-Assistant-Switch-Manager) configuration for the Sintra property.

## File

| File | Description |
|---|---|
| `switch_manager` | `.storage/switch_manager` export: 37 managed switches, blueprint `oxrs-4-button-switch` |

## Switch naming convention

`INT-<panel-number>-<area-code>`, e.g. `INT-16-COR-Q3` is panel 16 controlling Corredor Q3.

## Button logic (standard since v1.3)

Single press, hold and release are identical on every switch. A switch's behaviour is set only by its variables `area_id_1` and `area_id_2`.

| Button | OXRS channel | Single press | Hold | Release |
|---|---|---|---|---|
| B1 top-left | 1 | `light.turn_on` L | ramp up (`dim_dir = 1`) on L | stop (`dim = 0`), pin AL |
| B2 top-right | 3 | `light.turn_on` R | ramp up (`dim_dir = 1`) on R | stop (`dim = 0`), pin AL |
| B3 bottom-left | 2 | `light.turn_off` L | ramp down (`dim_dir = -1`) on L | stop (`dim = 0`), pin AL |
| B4 bottom-right | 4 | `light.turn_off` R | ramp down (`dim_dir = -1`) on R | stop (`dim = 0`), pin AL |

- L = `{{ data.variables.area_id_1 }}`
- R = `{{ data.variables.area_id_2 or data.variables.area_id_1 }}`: `area_id_2` on the 15 switches that set it (14 two-area, plus `INT-03-Q3` where it equals `area_id_1`); mirrors `area_id_1` on the other 22 (`area_id_2` = 0)

Hold: `switch_manager.set_variables` (`switch_id` is required) sets `dim = 1` and `dim_dir`, then `light.turn_on` with `brightness_step: 10 * dim_dir` (transition 0.2 s) repeats every 200 ms until release sets `dim = 0`. A ramp up from off starts the light at the lowest step.

Release (mode `parallel`): sets `dim = 0` first, so the ramp stops at once. After 2 s it calls `adaptive_lighting.set_manual_control` (`manual_control: true`) on the lights of the same area that are on ("pin AL"), so Adaptive Lighting's next periodic update does not undo the hold. Without this, a ramp up from off was not detected as manual control and Adaptive Lighting faded the lights to its own brightness about a minute later. Adaptive Lighting keeps one manual-control list for all zones, so one call naming `switch.adaptive_lighting_kasa` covers lights in the Jantar and Escadas-Exterior-Cobertura zones too; lights in no zone are ignored. The flag clears the next time the light turns on, or with the `Restore Adaptive Lighting` script.

Press 2x-5x slots are not standardised. Variables `cct_dir` and `scene_idx` are unused leftovers of the holds removed in v1.3.

---

## Changelog

### v1.4 - 2026-11-06
- Release on all four buttons of all 37 switches (148 actions): after stopping the ramp, mark the area's lit lights as manually controlled in Adaptive Lighting, so a hold is no longer overridden about a minute later
- Release mode `single` -> `parallel`, so a quick second release is never dropped (the first one now waits 2 s)
- Synced with changes made in the Switch Manager UI after v1.3:
  - `INT-18-ISQ1` -> `INT-18-Q1-ISQ1`: left `quarto_1` (`cover.blind_q1`), right `isq1` (`cover.blind_isq1`); B2/B4 press 2x open/close/stop `cover_entity_2`
  - `INT-19-ISQ2` -> `INT-19-Q2-ISQ2`: left `quarto_2` (`cover.blind_q2`), right `isq2` (`cover.blind_is_q2`); B2/B4 press 2x open/close/stop `cover_entity_2`
  - `INT-06-SAL` `area_id_2`: `0` -> `jantar`
  - `INT-12-COZ` `area_id_2`: `0` -> `sanca`

### v1.3 - 2026-10-02
- Single, hold and release standardised on all four buttons of all 37 switches; button config is identical fleet-wide
- Right column (B2/B4): on/off + ramp on `area_id_2` for two-area switches, mirrors `area_id_1` on single-area switches
- Removed B2 CCT-step hold (`color_temp_step` is not a `light.turn_on` field) and B4 scene-cycle hold (`color_temp` removed in HA 2026.3); both were failing
- Fixed `INT-16-COR-Q3` B2/B4 hold/release from v1.2 (missing required `switch_id`, non-standard loop); filled empty B2 hold/release on `INT-20-Q1`
- `INT-22-ENT` single presses use `area_id_1` instead of hardcoded `entrada`
- `INT-04-ISQ3` `area_id_2`: `q3` (area does not exist) -> `quarto_3`
- `INT-04-B3-WC` `area_id_2`: `quarto_3` -> `b3`

### v1.2 - 2026-09-28
- `INT-16-COR-Q3` converted to two-area: B1/B3 (left) -> `corredor_q3`, B2/B4 (right) -> `cozinha`
- Note: B2/B4 hold/release were malformed (missing `switch_id`); fixed in v1.3

### v1.1 - 2026-09-28
- `INT-16-COZ` renamed to `INT-16-COR-Q3`; area reassigned from `cozinha` to `corredor_q3`

### v1.0 - 2026-09-11
- Replaced the alternating `dim_dir` toggle with fixed `1` (B1/B2) and `-1` (B3/B4) on all hold slots (79)
- `INT-22-ENT`: `light.toggle` replaced by `light.turn_on` (B1/B2) and `light.turn_off` (B3/B4) on single press
- `INT-02-DES`: removed explicit `loop`/`loop_interval` keys (Switch Manager's native loop settings at their defaults; no behaviour change)

### v0.0 - 2026-09-11
- Baseline as exported from Home Assistant Switch Manager
