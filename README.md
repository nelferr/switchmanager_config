# switchmanager_config

Home Assistant [Switch Manager](https://github.com/macpit/Home-Assistant-Switch-Manager) configuration for the Sintra property.

## File

| File | Description |
|---|---|
| `switch_manager` | JSON export of all managed switches (37 switches, OXRS 4-button blueprint) |

## Switch naming convention

`INT-<panel-number>-<area-code>` — e.g. `INT-16-COR-Q3` is panel 16 controlling Corredor Q3.

## Button logic

All switches follow the same 4-button layout:

| Position | Single press | Hold | Release |
|---|---|---|---|
| B1 (top-left) | `light.turn_on` area 1 | Brightness ramp **up** (`dim_dir = +1`) | Stop ramp (`dim = 0`) |
| B2 (top-right) | `light.turn_on` area 2 | Brightness ramp **up** (`dim_dir = +1`) | Stop ramp (`dim = 0`) |
| B3 (bottom-left) | `light.turn_off` area 1 | Brightness ramp **down** (`dim_dir = −1`) | Stop ramp (`dim = 0`) |
| B4 (bottom-right) | `light.turn_off` area 2 | Brightness ramp **down** (`dim_dir = −1`) | Stop ramp (`dim = 0`) |

Single-area switches (Type A, 32 switches) use only `area_id_1`; B2/B4 may be used for CCT step or scene cycling.  
Two-area switches (Type B, 5 switches) use `area_id_1` (left) and `area_id_2` (right).

---

## Changelog

### v1.2 — 2025-09-28
- `INT-16-COR-Q3` converted to Type B (two-area): B1/B3 (left) → `corredor_q3`, B2/B4 (right) → `cozinha`
- B2: single `turn_on`, hold dim-up (`dim_dir = +1`), release; B4: single `turn_off`, hold dim-down (`dim_dir = −1`), release

### v1.1 — 2025-09-28
- `INT-16-COZ` renamed to `INT-16-COR-Q3`; area reassigned from `cozinha` to `corredor_q3`

### v1.0 — 2025-09-28
- **Fix**: Replace bidirectional `dim_dir` toggle with deterministic `+1` (top buttons B1/B2) and `−1` (bottom buttons B3/B4) across all 37 switches (79 hold slots)
- **Fix**: `INT-22-ENT` — replace `light.toggle` with `light.turn_on` (B1/B2) and `light.turn_off` (B3/B4) on single press
- **Cleanup**: `INT-02-DES` — remove stale `loop`/`loop_interval` keys from all 28 action slots

### v0.0 — baseline
- Initial state as imported from Home Assistant Switch Manager
