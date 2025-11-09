## W12 Material Z-Offset for Creality K2 / K2 Plus (Klipper)

Adds material-specific automatic Z-offsets (PLA, PETG, ASA, ABS, PC, Nylon).
Each filament keeps a persistent offset that is applied automatically at print start.
Fully compatible with OrcaSlicer.

Requirements

Klipper on Creality K2 / K2 Plus

OrcaSlicer (sends temps + filament type to START_PRINT)

A working mesh routine (DO_MESH, DO_REGIONAL_MESH, or BED_MESH_PROFILE)

Mainsail / Fluidd (recommended for file upload)

# 1️⃣ Install the Files

Open **Mainsail/Fluidd**

Drag & drop both files into the config/ folder:

w12_zoffset.cfg

w12_zoffset_save.cfg (can be empty initially)

Include in your printer.cfg:

[save_variables]  
filename: ~/printer_data/config/w12_zoffset_save.cfg

[include w12_zoffset.cfg]

Restart Klipper after saving.

# 2️⃣ OrcaSlicer — Start G-code

Printer Settings → Custom G-code → Start G-code

```bash
START_PRINT EXTRUDER_TEMP=[nozzle_temperature_initial_layer] BED_TEMP=[bed_temperature_initial_layer_single] FILAMENT="{filament_type[0]}"
```

# 3️⃣ Integrate into your existing macros

A) In your START_PRINT (gcode_macro.cfg) — add this

① Top

```bash
[gcode_macro START_PRINT]
variable_prepare: 0
variable_mat_applied: 0  # W12 ZOffset
gcode:
  SET_GCODE_VARIABLE MACRO=START_PRINT VARIABLE=mat_applied VALUE=0 # W12 ZOffset
  ....
```

```bash
{% set BED_TEMP = params.BED_TEMP|default(60)|float %}
{% set EXTRUDER_TEMP = params.EXTRUDER_TEMP|default(220)|float %}
{% set FILAMENT = (params.FILAMENT|default("PLA")).strip() %}  #<------ w12 Z Offset -----
{% set EXTRUDER_WAITTEMP = 140 %}
....
```

② After homing + your leveling mesh — apply the material offset once
(place this right after G28 and your mesh macro)

```bash
; DO_MESH / DO_REGIONAL_MESH / BED_MESH_PROFILE LOAD=default ; keep your own call here

; --- W12 Material Z-Offset (apply once) ---
{% if printer["gcode_macro START_PRINT"].mat_applied|int == 0 %}
{action_respond_info("FILAMENT -> " ~ FILAMENT)}
SET_MATERIAL TYPE="{FILAMENT}"
SET_GCODE_VARIABLE MACRO=START_PRINT VARIABLE=mat_applied VALUE=1
{% else %}
{action_respond_info("Material already applied (skip)")}
{% endif %}
; --- End W12 Z-Offset ---
....
```

# 4️⃣ Default Offsets (edit at the top of w12_zoffset.cfg)

```bash
[gcode_macro MATERIAL_OFFSETS]
variable_pla: 0.000
variable_petg: 0.025
variable_asa: 0.030
variable_abs: 0.030
variable_pc: 0.040
variable_nylon: 0.050
```

⚠️ Important: Printers, plates and filaments vary.
These are starting points. Calibrate your own values and update this block.
They load automatically on restart.

# 🪪 License

MIT License — see the LICENSE file in this repository.
