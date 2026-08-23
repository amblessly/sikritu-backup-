# Project DELTA - Development Update Documentation

**Student Health Dashboard + Raspberry Pi Touchscreen Deployment**

Date: August 23, 2026
Prepared for: Project Manager review

---

## 1. The Short Version

We now have a working student health dashboard that displays live vitals,
calculates a Heat Stress percentage using our approved formula, interprets
sweat pH strips using the Bromocresol Green color scale, and gives automatic
health recommendations.

The same dashboard can now run on a Raspberry Pi with the 7-inch touchscreen.
When the Pi powers on, it boots straight into the dashboard fullscreen - no
browser window, no typing, no manual steps. It behaves like a dedicated
health monitoring device.

Nothing about the visual design was compromised along the way - every change
kept the existing look and feel intact.

---

## 2. What the Dashboard Shows

The dashboard is one screen divided into clear sections:

**Header** - Project DELTA branding on the left, live clock on the right
(clock is green to match the monitoring theme).

**Vital Signs row** - three cards showing Electrolytes, Hydration, and Stress,
each with its current value, a status label (GOOD / LOW / CRITICAL, etc.),
and a colored progress bar.

**Electrolyte Breakdown card** - one shared card containing Sodium and
Lactate displayed side by side, each with its own status pill and bar.

**Heat Stress panel** - shows the computed Heat Stress percentage with a
LOW / MODERATE / HIGH classification, plus the normalized input values
(Temperature, Electrolytes, Lactate) used in the calculation.

**Sweat pH Strip panel** - a visual replica of our actual Bromocresol Green
test strip. A moving marker sits on the strip's yellow-to-blue gradient and
shows exactly where the current reading falls, alongside the zone name
(BLUE, GREEN, YELLOW-GREEN, BRIGHT YELLOW) and pH range.

**Stress and Temperature pair** - two standalone cards beside each other,
each with a circular icon badge next to its label (brain icon for stress,
thermometer for temperature).

**Recommendations card** - an AI-style feed that generates up to three
recommendation cards based on current readings. Each card has its own color
theme (yellow for hydration-related, red for stress-related) and follows the
guidance language from our project documents.

---

## 3. How Heat Stress Is Calculated

This follows the formula agreed in the project specification:

```
Heat Stress % = (0.55 x Temperature) + (0.25 x Electrolytes) + (0.20 x Lactate)
```

Because the three sensors report in different units (degrees C, %, mmol/L),
each raw reading is first converted to a common 0-100 risk scale:

| Input | Normalization | Example |
|---|---|---|
| Temperature | 36.5 C counts as 0%, 39.5 C (heat emergency) counts as 100% | 37.4 C becomes 30 |
| Electrolytes | Already a 0-100 index matching our sodium-loss bands | 74 stays 74 |
| Lactate | 1.0 mmol/L (resting) is 0%, 5.0 mmol/L (severe) is 100% | 2.8 becomes 44 |

Worked example: 0.55(30) + 0.25(74) + 0.20(44) = **44.9% -> MODERATE**

Classification bands: 0-33% Low, 34-66% Moderate, 67-100% High.

A full derivation document exists separately (`HEAT-STRESS.md`) if anyone
wants the deeper explanation, including why temperature carries the heaviest
weight.

---

## 4. The Sweat pH Strip Interpretation

Our strip uses Bromocresol Green (BCG), which changes color across pH 3.8 to
5.4: yellow when very acidic, blue when not acidic, green in between. This is
backed by two studies we cited (Al Tamimi et al. 2023 and Zhou et al. 2025),
which validated reading these exact strips through camera image analysis -
the same approach our device uses.

The dashboard maps the strip colors to exertion zones:

| Strip color | pH range | Meaning |
|---|---|---|
| Bright Yellow | 3.8 and below | Extreme exhaustion, high lactate, high salt loss |
| Yellow-Green | 3.9 - 4.5 | Heavy exertion |
| Green | 4.6 - 5.3 | Mild exertion |
| Blue | 5.4 and above | Resting / default state |

Right now the dashboard simulates realistic sensor values so the team can
demo and refine the UI without hardware. When the real sensors are ready,
only the data source file needs to change - the entire UI, formulas, and
display logic carry over unchanged. This was a deliberate design decision
made early on.

---

## 5. Fixes Worth Knowing About

Two issues came up during development that the PM should be aware were
caught and resolved:

1. **Garbled characters in recommendation text** ("aE" appearing mid-sentence).
   Cause: a file encoding mishap during editing corrupted the dash symbols
   in text strings. Fixed by cleaning all four frontend files to pure
   standard characters. Verified clean - zero stray bytes remain.

2. **Duplicate code name collision** that temporarily stopped the dashboard
   from loading after the pH feature was added. Caught through automated
   browser testing, fixed immediately. All features verified working after.

Both fixes are covered by the automated browser checks we now run after
every change (the dashboard is loaded headlessly and every panel is asserted
for correct values before anything is declared done).

---

## 6. Raspberry Pi Deployment

### As a web dashboard (current setup)

The dashboard is bundled inside our existing Node.js server project
(`videototext`). One server on the Pi serves both the transcription tool and
the dashboard at `http://<pi-address>:3000/dashboard/`. Setup instructions
are in `RPI-SETUP.md`.

### As a dedicated touchscreen device (new)

For the 7-inch 1024x600 touchscreen, we added a kiosk layer:

- On power-up, Linux starts two small background services automatically.
- Service 1 runs the local server.
- Service 2 opens the dashboard in a fullscreen browser view with all
  browser elements hidden - no address bar, no tabs, no menus, no pinch
  zoom, no accidental navigation gestures.
- If either service ever crashes, the system restarts it on its own.

To the athlete or nurse standing in front of it, it is simply a health
monitoring device that turns on and works. Setup takes one command on the Pi
(`setup-pi.sh`); the full guide including hardware list, installation,
updating, and a testing checklist is in `RPI-KIOSK.md`.

### Screen adaptation

The dashboard gained a dedicated layout mode that activates only on the Pi's
1024x600 screen: sections rearrange into a two-column arrangement that fills
the physical display edge to edge, with larger numbers and finger-friendly
touch targets. Importantly, the original phone-style and desktop layouts are
completely untouched outside of that screen size - we verified both side by
side.

---

## 7. Where Everything Lives

```
Documents/project/
|
+-- HumanDetectionPrototype/health-dashboard/     <- dashboard source
|     index.html            page structure
|     styles.css            all styling incl. Pi screen mode
|     main.js               rendering + interactions
|     data.js               sensor data layer + formulas + simulation
|     HEAT-STRESS.md        heat stress math + pH strip documentation
|
+-- videototext/                                  <- deployment package
      public/dashboard/     copy of the dashboard served by the server
      raspberry-pi/
        kiosk.sh            fullscreen launcher (self-restarting)
        videototext.service system service: the server
        delta-kiosk.service system service: the touchscreen shell
        setup-pi.sh         one-command installer for the Pi
      RPI-SETUP.md          basic Pi deployment guide
      RPI-KIOSK.md          full touchscreen device guide + checklist
```

---

## 8. Quick Test Guide

On any computer:

1. Open a terminal in `videototext`, run `npm start`.
2. Visit `http://localhost:3000/dashboard/`.
3. Watch values move, recommendations appear, and the pH marker travel
   along the strip over a minute or two.

On the Raspberry Pi (after running `setup-pi.sh`):

1. `sudo reboot`
2. The dashboard should appear fullscreen on the touchscreen by itself.
3. Tap around - cards respond, nothing should scroll sideways or clip.
4. Unplug/replug power - it should come back to the same screen.

---

## 9. Current Status and Next Steps

**Done:** dashboard complete, heat stress engine implemented per spec, pH
strip interpretation implemented per spec, Pi touchscreen deployment ready,
all changes documented and tested.

**Next (needs the hardware team):**

- Replace the simulated data source with real sensor readings. The code has
  a designated swap point for this; no UI work is required.
- Confirm the pH-to-exertion thresholds against actual strip trials once
  sample data is collected.
- Optional polish: boot splash logo while the Pi starts up.

---

*All automated checks passed as of this writing: no console errors, no
layout overflow at target resolutions, all panels updating correctly.*
