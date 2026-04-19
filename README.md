# Mira Liquid Handler

A DIY automated liquid handling system built from a Creality Ender 3 V3 SE 3D printer and Ependorf 1 mL pipette. 
Robot is controlled via custom Python GUI, that acts as a G-code graphical interface.

![Liquid Handler Demo](assets/liquid_handler_matrix.mp4) <!-- Add your demo video/GIF here -->

## Overview

This project transforms a standard FDM 3D printer into a precision liquid handling robot capable of performing common laboratory workflows including liquid transfers, dilutions, pooling, and aliquoting. The system uses the printer's XYZ motion system for positioning and the extruder stepper motor to drive a pipette mechanism.

### Why Build This?

- **Cost-effective**: Repurpose an inexpensive 3D printer instead of buying a $10k+ commercial liquid handler
- **Customizable**: Open-source software allows you to modify protocols and add new functionality
- **Compact**: Fits on a standard lab bench
- **User-friendly**: GUI-based operation - no coding required to run protocols

---

## What It Can Do

### Supported Protocols

| Protocol | Description |
|----------|-------------|
| **Liquid Transfer** | Move liquids between any modules (well-to-well, tube-to-plate, etc.) |
| **Pooling** | Combine multiple samples into a single destination |
| **Dilutions** | Create serial dilutions across 96-well plates |
| **Aliquoting** | Distribute samples from a source to multiple destinations |
| **Dilution + Aliquots** | Combined workflow for complex sample preparation |

### Supported Labware Modules

- **Vials** - Up to 3x96-well plates, 50 and 15 mL Falcon tubes, 4 mL vials, 1.5 mL HPLC vials, Eppendorf tubes, PCR vials
- **Wash Station** - 2-position cleaning/waste station
- **Tip Rack** - 7×5 grid (35 positions) for 1000 µL pipette tips
- **Tip Eject Station** - Tip disposal area

### Features

- **Pause & Resume** - Safely interrupt and continue long protocols
- **Soft Stop (Abort)** - Emergency stop without losing position
- **Live Position Tracking** - Real-time X/Y/Z coordinates and pipette volume display
- **Tip Inventory Management** - Visual grid showing available/used tips
- **Calibration Wizard** - Step-by-step module calibration with precision jog controls
- **Sequence Timer** - Estimated completion time with live countdown
- **Dry Run Mode** - Test protocols without physical execution
- **5 Preset Slots per Protocol** - Save and recall common configurations
- **Automatic Logging** - All actions logged to timestamped files

---

## What It Cannot Do

### Hardware Limitations

- **Single-channel only** - No multi-channel pipetting
- **Fixed pipette** - 10 µL to 1000 µL per transfer (configurable in code)
- **No liquid level detection** - Cannot auto-detect liquid heights in containers
- **No tip detection** - System trusts user-reported tip status; no physical confirmation
- **Fixed labware orientations** - Modules must be positioned at pre-calibrated locations

### Software Limitations

- **No plate reader integration** - Cannot read absorbance/fluorescence
- **No heated/cooled positions** - No temperature control for reactions
- **No barcode scanning** - Samples must be tracked manually
- **No cloud connectivity** - Local operation only

### Safety Limitations

- **No collision detection** - Will attempt to move to any valid coordinates
- **No spill detection** - Cannot detect or respond to liquid spills
- **No biosafety containment** - Not suitable for BSL-2+ without external containment

---

## Hardware Requirements

### Required Components

| Component | Specification                             | Notes |
|-----------|-------------------------------------------|-------|
| 3D Printer | Marlin-compatible (Ender 3, CR-10, etc.)  | XY accuracy ±0.1mm minimum |
| Controller Board | 32-bit recommended (SKR, Creality v4.2.7) | Better stepper resolution |
| Pipette Mechanism | 100-1000 µL air displacement              | Modified manual pipette or custom build |
| Pipette Tips | Universal 100-1000 µL tips                | Tip rack dimensions matter |
| USB Cable | A to B or Micro-USB                       | For computer connection |
| Power Supply | Printer's stock PSU                       | 24V or 12V depending on printer |

### Mechanical Modifications

1. **Pipette Mount**: Design or adapt a bracket to hold the pipette on the print head
2. **E-axis Coupling**: Connect pipette plunger to extruder stepper via Bowden cable or direct drive
3. **Bed Removal**: Remove heated bed; replace with flat deck for labware
4. **Labware Holders**: 3D print or machine holders for each module type

---

## Software Requirements

### Dependencies

```bash
pip install pyserial
```

- Python 3.8 or higher
- tkinter (usually included with Python)
- pyserial 3.5+

### Supported Operating Systems

- Windows 10/11
- macOS 10.14+
- Linux (Ubuntu 20.04+ recommended)

---

## Installation & Setup

### 1. Printer Firmware Configuration

Ensure your printer is running Marlin firmware with these settings:

```gcode
M302 S0      ; Allow cold extrusion (required for pipette operation)
M906 E150    ; Pipette motor current (adjust as needed)
M84 E S3     ; Disable E stepper after 3s idle
```

### 2. Software Installation

```bash
# Clone or download this repository
git clone https://github.com/yourusername/liquid-handler.git
cd liquid-handler

# Install dependencies
pip install -r requirements.txt
```

### 3. First Launch

```bash
python main.py
```

### 4. Initial Calibration

1. **Connect** to your printer via USB (115200 baud default)
2. **Home All** axes from the Init tab
3. **Absolute Calibration**: Use the Calib. tab with the calibration pin
4. **Module Calibration**: Calibrate each module's corner positions
5. **Save Configuration**: Settings auto-save to `config.json`

---

## Configuration

All settings are stored in `config.json` and can be edited:

```json
{
  "CALIBRATION_PIN": {"PIN_X": 109.5, "PIN_Y": 111.4, "PIN_Z": 133.7},
  "PIPETTE": {"STEPS_PER_UL": 0.3019, "MIN_VOL": 100, "MAX_VOL": 1000},
  "TIP_RACK": {"A1_X": 69.4, "A1_Y": 31.4, ...},
  "PLATE": {"A1_X": -104.1, "A1_Y": 112.8, ...}
}
```

**⚠️ Backup your config.json after calibration!** Losing calibration means re-teaching all positions.

---

## Safety & Warnings

### ⚠️ Critical Safety Information

- **Never leave running unattended** - Always monitor operation
- **Keep emergency stop accessible** - Know your printer's reset button location
- **Verify clearances before homing** - Ensure no obstacles in XYZ path
- **Use appropriate PPE** - Gloves, goggles when handling hazardous liquids
- **Secure loose clothing/hair** - Can entangle in moving parts

### Operational Warnings

- **Always home after power cycle** - Position is lost when power is removed
- **Check tip attachment** - Loose tips cause volume errors and spills
- **Mind the Z-height** - Wrong Z calibration crashes pipette into labware
- **Watch for bubbles** - Affects accuracy; pre-wet tips for viscous liquids

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| "Port not found" | Check USB connection; try different cable/port |
| "No response from printer" | Verify baud rate (115200 default); check printer is on |
| Tips not picking up | Increase Z-pick depth in calibration; check tip fit |
| Volume inaccurate | Recalibrate `STEPS_PER_UL`; check for air bubbles |
| Serial timeout | Reduce polling interval in config; check cable shielding |
| Lost position | Home all axes; check for missed steps |

---

## Project Structure

```
liquid-handler/
├── .log/             # Operation logs (auto-created)
├── .gitignore        # Excludes logs and config from git
├── stable/           # Version history (main_v1.py - main_v29.py)
├── main.py           # Main application (single file ~300KB)
├── config.json       # Calibration & settings (auto-generated)
└── README.md         # This file
```

---

## License

MIT License - Feel free to use, modify, and distribute. Attribution appreciated.

---

## Acknowledgments

- Built using the Marlin firmware project
- Inspired by the open-source lab automation community
- Thanks to the 3D printing community for hardware hacks

---

**⚠️ DISCLAIMER: This is experimental equipment. Use at your own risk. Always verify critical volumes independently. Not for clinical diagnostic use.**
