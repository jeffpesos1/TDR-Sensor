# Web page reference

Every entity on the device's own web page (`http://<name>.local`), section by section, in the order they appear. For each one: what it means, the range you should expect to see, and what an odd value usually tells you.

This is a reference, not a walkthrough. For the calibration procedure itself, see [CALIBRATION.md](CALIBRATION.md). For wiring, see [WIRING.md](WIRING.md).

## Live Readings

The four numbers you actually watch day to day.

| Entity | What it means | Typical range | What it implies |
|---|---|---|---|
| VWC | Volumetric Water Content, the share of the substrate's volume that is water right now. 0% is bone dry, 100% is fully saturated. | Rockwool 0-80%, Coco/Coir 0-75%, Peat 0-65%, mineral soil 10-45% | Freshly irrigated media sits near the top of your substrate's range; a steady decline after that is normal dryback. Pinned near 0% almost always means the rods are not actually in substrate, not that the substrate is dry. |
| Pore EC | Salt concentration in the water the roots can actually reach, derived from bulk EC and water content (blends the mass-balance and Hilhorst models, see below). | Rockwool 2-10 dS/m, Coco/Coir 1-8 dS/m, Peat 0.5-5 dS/m | A slow rise across a crop cycle with no change in feed is expected, salts concentrate as the plant drinks. A sudden jump usually means underwatering. A sudden drop usually means a diluted or missed feed. |
| Substrate Temperature | Root-zone temperature, not air temperature. | 18-24 C for most greenhouse crops | Below 15 C nutrient uptake slows down. Above 28 C roots are stressed and root disease becomes more likely. |
| Bulk EC @25C | The raw electrical conductivity of substrate + water + salts together, temperature-normalised. Not the same as Pore EC, this is the input to it, not the root-zone number. | Depends on feed EC and VWC, generally lower than Pore EC in wet media | Mainly useful as a diagnostic if Pore EC looks wrong, since Pore EC is derived from this and VWC. |

## Crop Steering Analytics

Everything here is derived on-device from the VWC stream, no other input needed.

| Entity | What it means | What it implies |
|---|---|---|
| Peak VWC | The last confirmed high point after an irrigation plateaus. | Your reference point for dryback. Updates only once a rise is confirmed as a real shot, not just noise. |
| Trough VWC | The lowest VWC seen since the last irrigation. | The bottom of the current dryback. |
| Dryback | Peak VWC minus current VWC, in points. | How far the substrate has dried since the last shot. |
| Dryback Percent | Dryback expressed as a percent of Peak VWC, rather than raw points. | Lets you compare dryback across substrates with different field capacities. |
| Dryback Rate | Slope of the drying curve over the last rolling 60 minutes, in %/hour. | A rising rate late in the day with no matching irrigation can mean root demand is increasing, worth watching alongside plant behaviour. |
| Time Since Irrigation | Minutes since the last detected shot. | Sanity-checks your irrigation controller is actually firing on schedule. |
| Irrigations Today | Count of detected shots since midnight, resets at local midnight. | Compare against what your controller says it fired, a mismatch means shots are too small or too slow to trigger detection (see Rise Threshold in Analytics Tuning). |
| Last Shot Size | How much VWC rose during the most recently confirmed irrigation. | A shrinking shot size over several days with the same run time can mean drippers are clogging or pressure is dropping. |
| Max Dryback Today | The largest dryback point reached since midnight. | Your daily deepest point, useful for comparing generative/vegetative steering day to day. |
| Overnight Dryback | VWC drop between your configured lights-off and lights-on hours (set in Analytics Tuning). | Overnight dryback should be small and stable, a rise in it usually flags a leak or a root-zone problem rather than plant water use, since transpiration is near zero in the dark. |
| Saturation | Current VWC as a percent of Field Capacity. | Just irrigated should read close to 100%. Above 100% (up to 120%, clamped) means still draining or actively being watered. |
| Field Capacity (learned) | The highest daily peak VWC over the last 7 days, auto-tracked. Diagnostic only, does not overwrite your manual Field Capacity setting. | Compare it to your manual Field Capacity. If they agree after a week of normal irrigation, your manual number is solid. |
| Pore EC at Field Capacity | Pore EC captured at the top of the most recent confirmed shot. | The baseline EC Stacking measures against. |
| EC Stacking | Current Pore EC minus Pore EC at Field Capacity. | How much salts have concentrated since the last irrigation. Rising through the day is expected, a big jump right after a shot usually means the feed EC itself was high. |
| VWC 24h Min / Max / Average | Rolling 24 hour statistics on VWC. | The Average feeds the Steering Index headroom calculation. Min/Max diagnose how wide your daily swing actually is. |
| Pore EC 24h Min / Max / Average | Same, for Pore EC. | Useful for spotting a slow drift in feed strength that a single reading would not show. |
| Irrigating | On while a rise is in progress and has not yet plateaued into a confirmed peak. | Turns off automatically once the plateau is confirmed (see Plateau Confirm Drop / Time in Analytics Tuning). |
| Dryback Target Reached | On once Dryback (in points) reaches your Dryback Target (set in Analytics Tuning). | A trigger point for an automation, e.g. fire irrigation when this turns on. |
| Peak Time / Trough Time | Local time of day the last peak/trough happened. | Lets you see at a glance whether irrigation timing lines up with when you actually expect shots. |

## Steering Detection

| Entity | What it means | What it implies |
|---|---|---|
| Steering Mode | Vegetative, Balanced, Generative, or Learning. Derived from Steering Index against the two threshold anchors in Analytics Tuning. | "Learning" shows until at least 3 full irrigation cycles have completed, the index is not meaningful before that. |
| Steering Index | -1 (fully vegetative) to +1 (fully generative), blending dryback size, shots per day, EC stacking, and VWC headroom against field capacity. | The direction and magnitude tell you which way the plant is currently being pushed by your irrigation strategy, not whether that is right or wrong for your goal. |
| Steering Confidence | 0-100%, how far the index sits from zero, scaled against your threshold anchors. | Low confidence near a mode boundary means small changes in irrigation could flip the reported mode, do not over-react to a single day near the line. |

## Calibration

Field meanings and expected ranges. For the actual step-by-step procedure, see [CALIBRATION.md](CALIBRATION.md).

| Entity | What it means | Typical range | Notes |
|---|---|---|---|
| Substrate Profile | Rockwool, Coco, Peat, Mineral Soil, or Custom. Loads starting defaults for Field Capacity, Hilhorst e0, and the Pore EC blend window. | — | Set this first. Changing it later reloads those three defaults and undoes any manual tuning you did on them. |
| VWC Gain | Multiplier applied to the raw VWC polynomial output. | 0.25-2.0, default 1.0 | Set automatically by Apply VWC Calibration, rarely typed by hand. |
| VWC Offset | Additive shift applied after the gain, in VWC points. | -40 to 40 | Same, set by the calibration buttons. |
| Saturated Reference | The VWC% you calculated for your block at true saturation (gravimetric method, see CALIBRATION.md). | 20-95%, commonly 50-70% depending on substrate | This is a measurement you feed in, not something the firmware guesses. |
| Field Capacity | The VWC% you expect right after a normal irrigation and drain. Anchors every dryback and steering number. | Rockwool 60-70%, Coco 50-60%, Peat similar to Coco, mineral soil 25-40% | Re-check every couple of weeks, root mass changes how the block holds water as a crop matures. |
| Temperature Offset | Manual correction added to the raw probe temperature. | -5 to 5 C, default 0 | Only touch this if you have cross-checked against a real thermometer and found a consistent gap. |
| EC Gain | Multiplier applied to raw bulk EC. | 0.25-2.0, default 1.0 | Set automatically by Calibrate EC to Reference. |
| EC Offset | Additive shift applied to raw bulk EC, in µS/cm. | -1000 to 1000 | Same, set by the calibration button. |
| EC Reference Solution | The true EC of the calibration standard you dip the probe in. | 0.1-20 dS/m, commonly 1.413 or 2.76 | Type this in before pressing Calibrate EC to Reference. |
| EC Temp Coefficient | Manual temperature compensation slope for bulk EC. | 0-0.03 (1/C), default 0 | Leave at 0 for the MT22, it already normalises to 25C internally. Only relevant if you swap in a probe that reports raw, uncompensated EC. |
| Hilhorst e0 | Dielectric-offset constant in the Hilhorst pore-EC model. | 0-7, default 4.1 | 4.1 is correct for almost every substrate, do not change this without a specific reason. |
| Pore EC Blend Low | VWC% below which Pore EC uses the mass-balance method only. | 10-60%, default 40% | Lower this if Pore EC reads too high while the media is still fairly wet. |
| Pore EC Blend High | VWC% above which Pore EC uses the Hilhorst method only. Between Low and High, the two are blended. | 30-90%, default 60% | Raise this if Pore EC swings unrealistically as VWC crosses through the blend window. |
| Custom Poly a / b / c / d | Coefficients of your own `theta = a*R^3 + b*R^2 + c*R + d` VWC polynomial, R = raw counts. Hidden unless you enable them. | — | Only used when Substrate Profile is set to Custom. Defaults match the built-in TEROS-12 soilless curve, a starting point to tweak from, not a blank slate. |

Buttons in this section (Capture Dry Point, Capture Saturated Point, Apply VWC Calibration, Calibrate EC to Reference, Reset to Substrate Defaults, Reset VWC/EC Calibration) are covered step by step in [CALIBRATION.md](CALIBRATION.md).

## Analytics Tuning

Knobs that shape how the irrigation/dryback engine and steering detection behave. Defaults work for a normal drip-to-waste rockwool or coco setup, most people never touch these.

| Entity | What it means | Typical range |
|---|---|---|
| Rise Threshold | Minimum VWC rise, in points, before a rise counts as the start of an irrigation shot rather than noise. | 0.3-10%, default 1.5% |
| Plateau Confirm Drop | How far VWC has to fall back from its candidate peak before that peak is confirmed. | 0.1-5%, default 0.8% |
| Plateau Confirm Time | How long the candidate peak must hold without a new high before it is confirmed, even without a drop. | 2-60 min, default 10 min |
| Irrigation Rise Window | How long a rise has to complete within to count as a real shot rather than slow drift. | 5-120 min, default 20 min |
| Dryback Target | The dryback (in points) that trips the Dryback Target Reached flag. | 3-40%, default 15% |
| Lights On Hour / Lights Off Hour | Local hour (0-23) used to snapshot VWC for Overnight Dryback. | — |
| Fault Freeze Minutes | How long raw counts can sit frozen before Sensor Fault trips. | 2-120 min, default 15 min |
| Steering Gen Threshold / Steering Veg Threshold | Steering Index cutoffs (positive/negative) above/below which Steering Mode reports Generative/Vegetative instead of Balanced. | 0.05-0.9 / -0.9 to -0.05 |
| Steer Dryback Veg Anchor / Steer Dryback Gen Anchor | Dryback size (points) treated as the vegetative and generative ends of the Steering Index scale. | 1-20% / 5-40% |
| Steer Shots Gen Anchor / Steer Shots Veg Anchor | Shots-per-day treated as the generative and vegetative ends of the Steering Index scale (fewer shots reads more generative). | 1-20 / 2-40 |

Reset Analytics clears every rolling counter (peak, trough, dryback, shots, learned field capacity) back to a fresh start. Reset Daily Counters only clears today's shot count and max dryback, use it if you want to zero the day without losing the longer-running numbers.

## Network

| Entity | What it means |
|---|---|
| WiFi Signal | Signal strength in dBm. Above -70 dBm is comfortable, below -80 dBm expect drops. |
| IP Address | Current address on your LAN. Also shown in the boot log. See [CONFIG.md](CONFIG.md#giving-a-sensor-a-fixed-local-address) for pinning it with a DHCP reservation. |
| WiFi SSID | The network currently joined. Hidden by default. |
| MAC Address | The chip's factory-burned hardware address, stable across reboots and reflashes. Use this for a router DHCP reservation, or to tell two otherwise-identical boards apart. |

## Diagnostics

Mostly for troubleshooting, most of these are hidden by default and safe to ignore day to day.

| Entity | What it means |
|---|---|
| Raw Counts | The unconverted number coming off the probe over SDI-12, before any calibration math. If this is frozen or NaN, the probe is not answering, see [TROUBLESHOOTING.md](TROUBLESHOOTING.md). |
| Sensor Fault | On when raw counts are NaN, or have not changed at all for longer than Fault Freeze Minutes. A real, moving reading clears it automatically. |
| VWC Uncalibrated | The VWC polynomial's raw output before your gain/offset are applied. Hidden by default. Useful while running the two-point calibration, since the capture buttons record this value, not the calibrated VWC. |
| Bulk EC | Bulk EC in dS/m before the Hilhorst/mass-balance pore-EC derivation. |
| Permittivity | Apparent dielectric permittivity, the intermediate value the Hilhorst model is built on. Diagnostic only. |
| Pore EC (mass-balance model) / Pore EC (Hilhorst model) | The two individual pore-EC estimates before they get blended into the single Pore EC reading on Live Readings. Hidden by default, useful for understanding why Pore EC is behaving a certain way at the edges of the blend window. |
| Active Substrate | Text readout of the currently selected Substrate Profile. |
| Status LED Enable | Toggles the onboard status LED (blue = irrigating, green = drying, orange = dryback target reached). Atom Lite / Atom PoE only. |
| Uptime | Seconds since last boot. A number that keeps resetting on its own points at a power or WiFi stability problem. |
| Debug Logging | Flips the serial/web log to DEBUG level live, no reflash needed, so you can see raw SDI-12 bus traffic. Turn it off again once done, left on permanently it is noisy. |
| Restart / Restart (Safe Mode) | Reboots the device. Safe Mode restart is hidden by default, forces the next boot into ESPHome's safe/recovery mode. |
| ESPHome Version | The firmware's ESPHome version. Hidden by default. |
