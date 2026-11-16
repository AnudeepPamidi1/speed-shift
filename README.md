# Speed Shift — speed limiter, M-style shift lights and live data on the BMW instrument cluster

**By AnudeepPamidi.** Version 1.5.6. For iDrive 7 / MGU (tested target: a
2024 BMW X3 G01 xDrive30i, cluster 1920×720).

A row of OEM-style M shift-light segments at the very top of the instrument
cluster — green, then yellow, then red as the rpm rises, flashing red at
the shift point — plus a speed block with the live speed AND the speed
limit, both BIG, a drive block with the 0-100 km/h time, the fuel range
and level together, and the engine torque, a G-force ball, the current
street name, and a small M badge. Everything draws on the cluster panel,
so it stays visible on **both the Waze map and the stock BMW map**, and it
never hides the media, the BC functions, or the date/time and outside
temperature. It fixes the evangoyuk Shift Lights 1.6.0 problem: that app
drew on the cluster "panel" slot and fought the stock HMI for it, so the
bar flickered and disappeared in Map view — and while it held the panel it
hid the media display and the BC (board computer) functions.

## What it does

- **LED shift-light segments (OEM M style)** — a horizontal row of 15
  individual rectangular LED segments at the very top of the cluster, like a factory
  M car. The segments light up one after another as the rpm rises (first
  green, then yellow, then red as you approach the shift point), and at
  the shift point the whole row turns red and flashes. The segments are
  **always visible**: at idle they show as dim/unlit outlines, so you
  can always see the row is there and working. The red is (235,0,15)
  and the yellow is (255,199,10) — proven bright on
  this cluster. On an upshift the row resets at once to
  the new level (a gear change drops the rpm in one step, so the row
  follows it at once instead of easing down for a second).
- **Speed block — BOTH numbers BIG** — the live speed (80 px digits) and
  the speed limit as a round sign, **sign-board big** (radius 40, like a
  real road sign, number in 34/44 px), in the map's top-left corner. When the
  speed goes over the limit plus the `overspeed` offset (default +10
  km/h, changeable 0/5/10/15/20), the **current speed turns red and
  flashes** (the digits only, at 4 Hz) — the limit number stays plain
  black and the sign stays steady. The next speed limit shows small under
  the sign ("next 60"). No limit yet: a dim ring with "--". The gear
  (D1–D8, or P/R/N) and the live boost pressure (bar) are below.
- **Drive block** — three rows at the **top-right** of the map: the 0-100
  km/h time (measured from the live speed, 20 samples a second) and the
  engine torque (Nm); the fuel range and the fuel level **together**
  ("329 km • 32.0 L"); and one conditional line that only appears when
  relevant — a tyre low ("TYRE LOW FL 1.8", red), an oil/coolant
  overheat ("OIL HOT 125°C", red), a low fuel ("LOW FUEL", red), a
  warming-up ("WARMING UP", amber), or the average fuel use ("avg 12.3
  L/100km", dim). When parked, the drive block switches to a brief trip
  summary (trip distance, average speed, fuel used) from the BC trip
  data. It sits at the top-right (v1.4.0 — it was at the bottom-left,
  where the stock BC menu appears when the BC button on the stalk is
  pressed, so pressing BC showed no change and the menu looked hidden).
- **G-force ball** — a small circle at the bottom-right of the map with a
  dot that moves with the G-force: up when accelerating, down when
  braking, sideways in a corner. The dot turns red past 0.8 g, and the
  magnitude ("0.5 g") is written under the ball.
- **Street name** — the current road name ("Koala Ct") at the bottom
  centre, exactly where the stock street name sits, shown only while Waze
  is showing (the stock street name is hidden then; it already shows
  whenever BMW Maps is the active map, so this never duplicates it).
- **M badge** — a small BMW M badge (the M tricolour stripes + a bold
  white "M") in the top strip, just left of the dots, shown only while
  Waze is showing (the stock M badge from Cluster Layouts' "m" cluster is
  hidden then; it already shows whenever BMW Maps is the active map).
- **No black boxes** — the speed and drive blocks have TRANSPARENT
  backgrounds: the numbers float directly on the map with only a subtle
  shadow/glow behind them, so they stay readable with no solid black
  rectangles.
- **Colours follow the drive mode** — the accent colours (gear text,
  G-ball ring and dot, drive-block label, dots' unlit ring tint) follow
  the car's drive mode live, from the FES signal: SPORT = red, COMFORT =
  blue, ECO PRO = green, ADAPTIVE = violet. The dots' lit RPM colours
  stay OEM green/yellow/red.
- **Menu screen** — the tyre pressures (FL/FR/RL/RR, bar, with the
  target) and temperatures (°C), the gearbox oil temperature, the cruise
  set speed (or "off"), the oil and coolant temperatures, the turn
  indicators and the ambient colour slot, the board-computer trip data
  (trip distance, time, average speed, average consumption, fuel used,
  odometer), the service-due, the speed limit with the next limit and the
  warn threshold ("speed turns red over 35 km/h"), the street name, the
  best 0-100 time, and the position of the selected overlay.
- **Adjustable positions (the Shift Lights way)** — every overlay can be
  moved: position settings (posX/posY/size for the dots, speedX/speedY,
  driveX/driveY, gX/gY, streetX/streetY, mX/mY) plus Move buttons in the
  app (Up/Down/Left/Right 20 px a press, Bigger/Smaller in the SIZES
  steps, Centre, Reset positions). The `move` setting picks which overlay
  the buttons move. After a move the dots show lit for 5 s so you can see
  where the row now is. Every position is clamped to the panel and above
  the bottom strip — no overlay can ever be moved over the date/time or
  the outside temperature.
- **Stays visible on Waze AND BMW Maps** — it draws on the cluster
  panel (see "How it stays visible"), so the overlays show on whichever
  map is active. No flicker.
- **Starts with the head unit** — autostart is baked into the package.
  No opening needed after a restart.
- **Changeable in the car** — start/shift rpm, flash, the speed block,
  the drive block, the G-ball, the speeding-warning offset, the positions,
  and an on/off switch, all in the app's own settings screen.
- **Read-only and safe to leave running always** — it only reads car
  signals and draws pixels. See "Safety".

## The speed limit — the real fix (new in 1.3.0)

The speed limit signal was **found on the car** by running Cluster
Layouts' `signal_finder` against the car's own catalog
(`/etc/oap/someip/someip-interfaces.json`):

- Interface: `de.bmw.dass.driverassistanceapplications.TrafficSignDisplayControl`
- Field: `displaySpeedLimitAndNoPassingRestriction` — present in the
  catalog, subscribable. It carries `displaySpeedLimit` (the current
  limit), `displaySpeedLimitUnit` (KM_H/MPH), and `displaySpeedLimitWarning`
  (the SLI warning flag). This is the same value the car itself displays.
- Also present: FASuS `statusNextSpeedLimitRecognition` (the NEXT limit),
  shown small under the sign.
- **Not** in the catalog: FASuS `statusActualSpeedLimitRecognition` — it
  is not used.

How the warning works: the speed turns red and flashes when
`speed > limit + overspeed`. The `overspeed` setting (0/5/10/15/20,
default +10 km/h) is the threshold you set — the menu states it plainly
("speed turns red over 35 km/h" for a 30 limit with +5). The car's own
SLI warning flag is an additional trigger. If the car has not sent a limit
yet, the sign shows "--" and the warning stays off — nothing is faked.

**No chime.** The kit has no audio API, and the only sound path MGU apps
have (the YouTube app's smartx audio) rebooted this head unit, so a chime
is NOT added — the red flashing speed IS the warning. **No LIM control:**
the app stays read-only; it never sets or changes the car's speed limiter
(LIM) max speed — the car's own LIM button already does that.

## Phone calls — only what covers the call hides (new in 1.3.0)

When a phone call comes in, BMW shows the call (caller name/number +
call status) in the cluster's central display area — the same window the
map sits in (x 416-1504, y 120-600). The overlays that overlap that area
hide while the call is on and come back when it ends; the rest keep
showing. At their default places the speed block, the drive block and
the G-ball sit inside the central area and hide, while the LED dots, the
street name and the M badge sit outside it (top strip / bottom strip)
and stay visible. The overlap is computed from each overlay's CURRENT
rectangle, so an overlay the user moved only hides when it really does
cover the call display. The hiding is done two ways at once, so it
cannot flicker or fight the call: the call-state handler hides the
overlapping layers the moment a call starts, and the draw loop refuses
to show one again until the call ends (the tick runs 20×/s, so a hide
or a restore lands within a frame or two).

The call state comes from the car's own telephone interface:
`de.bmw.infotainment.Telephone`, field `callState`, instance 1 — the
interface and the field are in the head unit's own interface catalog
(the `TelephoneDefinition` in the unit's own `app.js` lists `callState`
among its values). It is subscribed, not read (`"s"` in app.json), and
marked `_optional`, so a unit without it still installs the app and the
feature simply does nothing. The exact enum names were NOT in the kit,
so the value is read defensively — a name (`INCOMING`, `ACTIVE`, …) or
a wire number (0 = no call, anything else = a call is on) — and the raw
value is logged, so the live car confirms which values mean "a call is
on". Until the car sends a value, no call is assumed and nothing hides.
The menu status line shows the call state (`Call ACTIVE` / `Call idle` /
`Call --`), so the live-car check can see what the car sent.

## What it does NOT do (honest limits — nothing faked)

These were all asked for, checked against the SDK / the on-car catalog /
a live read, and found **not available to any MGU app**. The app does not
show them and does not fake them. Where a value is missing, the app shows
"--" or leaves it out.

| Wanted | Available? | What happens instead |
|---|---|---|
| **Next-turn arrow + distance** | **No.** No navigation interface exists in the kit, and Waze on cluster's status has no route data (only showing/target). | Waze itself already shows the next turn and the distance in its own banner at the top of the map ("Start on Koala..."). The app cannot extract that data to draw a bigger arrow. BMW's own nav route data exists, but only while BMW nav guides — the user navigates with Waze, so it is not used. |
| **ETA / arrival time** | **No.** Same reason — no nav route data reaches MGU apps. | Waze shows the ETA in its own display on the map while navigating. |
| **Accelerator pedal angle (throttle)** | **No.** The interface is in the catalog but it is NOT proven on this car — no app on this car declares it. | The **engine torque** (Nm) is shown instead — that is the engine load, and it is verified live. |
| **Electronic oil level, 12V battery, fuel injection volume** | **No.** In the catalog but unproven on this car. | Left out. |
| **Check-control messages** | **No.** `KombiInterface_2 ccList` / `ccListRiskClass` have empty members (arrays, not parseable); `KombiIF statusActiveCCMessage` / `standardCCMessage` / `extendedCCMessage` carry message numbers/IDs only, no human-readable text and no lookup table. | Left out. The service-due (cbsDataSBA) IS shown — that one has real text. |
| **Power split** | **No.** His X3 is petrol (xDrive30i, not a hybrid). The xDrive front/rear torque split is NOT available on this car (Cluster Info removed it — the car sends no usable reading), and the KombiIF power-distribution signal is for hybrid/electric power, which this car does not have. | Left out. |
| **Night mode (dim at night)** | **No.** The kit has no day/night signal, and no exact interface/field was found for it. | Left out. |
| **Bigger gear in manual mode** | **No.** The kit's gear handling exposes the stage only as P/R/N/D — there is NO manual ('M') stage or manual-mode flag, and no verified manual-mode signal was found. | Left out. The gear is shown BIG in the speed block in every mode. |
| **Front collision warning / auto-braking readout** | **No.** No front-collision-warning / AEB trigger flag was found in the available references (Cluster Layouts' app.json lists only assisted-driving-view inputs — the cars ahead and beside — and no FCW/AEB flag). Deriving a warning from the object list would be guesswork and could show false warnings for a safety system. | Left out. Only READ was ever contemplated — never write to or interfere with the braking/safety system. A future check can re-run signal_finder on the car for the `de.bmw.dass` front-collision-warning interfaces when the catalog is available again. |
| **Blind spot indicator** | **No.** His X3 has no Driving Assistant package, so it almost certainly has no blind-spot-monitoring hardware (no rear radar sensors, no mirror warning lights) — there is nothing to read. No verified readable blind-spot signal was found. | Left out. |
| **Lane assist at low speed** | **No — and it cannot be done by an app.** The lane-assist minimum speed (it works at around 70 km/h on his car) is a SAFETY setting of the driver-assistance system. An MGU app is read-only and cannot write to safety systems. | Left out of Speed Shift entirely. Whether BimmerCode can change it is not verified — it would be a coding job to investigate separately (KAFAS module), not part of this app. Do not expect an app to change it. |
| **Steering-wheel vibration readout** | **No.** No verified readable signal for the lane-departure vibration trigger was found. | Left out. |
| **Chime when speeding** | **No safe way.** See "The speed limit" above. | The red flashing speed IS the warning. |
| **LIM (speed limiter) control** | **No — read-only app.** | The car's own LIM button sets the max speed. |

What **is** available (all live from the car, see "Signals"):

| Value | Signal (SDK) | Notes |
|---|---|---|
| RPM | CDS `statusEngineSpeed2` | Same as Cluster Info / M Vitals |
| Speed (km/h) | Vehicle2 `statusCarSpeed2` | Live, km/h — shown BIG |
| Speed limit (km/h) | TrafficSignDisplayControl `displaySpeedLimitAndNoPassingRestriction` | Found on the car; "--" until the car sends one; MPH converted |
| Next speed limit (km/h) | FASuS `statusNextSpeedLimitRecognition` | Small under the sign + on the menu |
| Gear (D1–D8, P/R/N) | VehicleInformation `statusGearboxDrive` + `DataPowerTrain2` | With the kit's GearGuess fallback |
| Boost (bar) | FES `statusSwitchDriveDynamicSport2`, `displayBoostPressure` | Shown as "--" when stale (>5 s) or invalid |
| Torque (Nm) | FES `statusActualValueTorque2` | Crankshaft torque — the engine load |
| Drive mode | FES `statusFESVehicleDynamicSwitchHMI` | SPORT / COMFORT / ECO PRO / ADAPTIVE — the accent colours follow it |
| G-force long/lat (g) | CDS2 `statusLongitudinalAccelerationMassCenter` / `statusLateralAccelerationMassCenter` | Shown as a ball on the map |
| 0-100 km/h time (s) + best | Derived from the live speed | Measured from the speed samples (20/s, with the samples' own timestamps) — a real measured time; the best is on the menu |
| Fuel range (km) + fuel level (L) | KombiIF `statusRangeTankStatus` (parsed by field name, as Cluster Layouts does it) + VehicleInformation `Range` as a second source | Shown TOGETHER ("329 km • 32.0 L"); "--" until the car has sent a value |
| Fuel use (L/100km) | KombiIF `statusBordcomputerAverage` (average consumption) | The BC average, converted to L/100km; on the drive block row 3 and the menu |
| BC trip data | KombiIF `statusBordcomputerAverage` + `statusBordcomputerTravelData` | Total km (odometer, from `statusRangeTankStatus.mileageKilometre`), trip distance, trip time, average speed, average consumption, fuel used — confirmed by a live read on the car (2026-11-14): 21.7 km/h, 8.1 km/L, 747 km, 36h 24m, 329 km range, 32 L, odo 43629 km |
| Service-due | KombiInterface_2 `cbsDataSBA` | Pre-formatted text from the car ("Service due in ---", "14000 km", "_10/2027", GREEN) — confirmed live |
| Oil / coolant temps (°C) | CDS `statusEngineTemperature` | x1 − 48 °C; on the menu (already on the cluster via Cluster Info) |
| Tyre pressures (bar) + temperatures (°C), FL/FR/RL/RR + target | TPMS `statusTyrePressure` / `statusTyreTemperature` / `statusTargetTyrePressure` | Polled every 10 s, as M Vitals does it; on the menu; a low tyre (< 2.0 bar or < 80% of target) shows a red warning on the drive block |
| Gearbox oil temp (°C) | BasicDataSet `basicDataSet`, `valueTemperatureTransmissionOil` | On the menu |
| Cruise set speed (km/h) | DisplayDataServer `functionalLibraryData`, `cruiseCtrlSpeedHUD` | 0 when the cruise is off; on the menu |
| Turn indicators | `de.bmw.body.Light` `StatusFunctionsIndicators` | The ONLY exterior-light signal in the kit; on the menu (the stock cluster already shows the arrows) |
| Ambient colour slot | `de.bmw.infotainment.misc.ComfortFunctionIuK` `statusAmbientLightInteriorColour` | The car reports only the slot number (1–12), no RGB; on the menu |
| Street name | NavigationStatus `currentLocation.address.street.text` | Best effort: the BMW nav service sleeps while Waze is the active map (confirmed unavailable for 30 s on the car), so the street name only shows if the nav service wakes up (e.g. while driving). If it stays asleep, the street name simply does not show — never faked. The stock street name returns whenever BMW Maps is the active map. |

Deliberately NOT duplicated on the cluster (already there): the outside
temperature (in the bottom strip — it must stay visible there, never
duplicated on the map), the time/date (bottom strip), the drive mode and
the range (bottom strip / drive block), and the steering angle / yaw rate
(niche — verified available, but no clear use on the cluster).

## Layout — designed around the real cluster photo

The layout is designed around the user's actual cluster photo (Waze on,
parked): the map is LARGE and central between the two dials; the left dial
is the speedometer; the right dial is the rpm with the media ("103.3 FM")
on the right side; the bottom strip shows the time, SPORT INDIVIDUAL, P
and the temperature. Waze's own next-turn banner ("Start on Koala...")
sits at the top centre of the map.

```
1920x720 cluster (the whole panel)
+--------------------------------------------------------------+
| top strip (stock status bar)                  y 0-120        |
|  [M badge 648,26]   [LED dots 720,14  480x72]                |
|                                                              |
|  +-- left dial --+  +----------------------+  +-- right --+  |
|  | speedometer   |  |   WAZE MAP           |  | rpm dial   |  |
|  | 0-330 km/h    |  |   x 416-1504         |  | 0-7 x1000  |  |
|  | digital speed |  |   y 120-600          |  | 103.3 FM   |  |
|  | fuel, 131 km  |  |   (1088x480)         |  | (media)    |  |
|  +---------------+  +----------------------+  +------------+  |
|                                                              |
| bottom strip: 7:31 am · SPORT INDIVIDUAL · P · +7.5 C        |
| (stock BC / media strip)                      y 660-720      |
+--------------------------------------------------------------+

Panel coords (1920x720) — all six overlays, klass overlay, z 20, NO contend
+--------------------------------------------------------------+
| LED dots    x 720-1200, y 14-86   (480x72)  — very top centre |
| M badge     x 648-712,  y 26-62   (64x36)   — top strip, left |
|                                              of the dots    |
| speed block x 428-692,  y 132-302 (264x170) — top-left of    |
|                                              the map         |
| Cluster Info panel x 838-1082, y 152-224 — its own app,      |
|                                              untouched       |
| Waze turn banner — top centre of the map (Waze), untouched    |
|                                                              |
|              map centre: car arrow, turn info, route         |
|              x 668-1252, y 302-524 — CLEAR, no overlay       |
|                                                              |
| drive block x 1148-1388, y 132-262 (240x130) — top-right     |
| G-ball      x 1396-1492, y 492-588 (96x96)  — bottom-right   |
| street name x 760-1160, y 665-705 (400x40)  — bottom strip   |
|                                              centre ONLY     |
+--------------------------------------------------------------+
```

- **Nothing overlaps.** The dots row is at the very top, above the map
  and above Cluster Info's panel. The M badge sits left of the dots.
  The speed block is 146 px left of the Cluster Info panel and on the
  opposite side from the media (right). The drive block is at the
  top-right, the G-ball at the bottom-right. The map's centre (car arrow,
  turn info, route) and Waze's turn banner (top centre) are empty. The
  drive block sits at the top-right so it never covers the stock BC menu
  (the BC button on the stalk cycles the central display through the trip
  computer, mileage, media and performance views — the drive block used
  to sit at the bottom-left, on top of it).
- **The bottom strip is a hard rule.** No overlay ever covers the bottom
  strip (y 660–720) — the date/time and the outside temperature stay
  visible at all times, like the media and the BC functions. Every
  overlay's bottom edge is clamped to y ≤ 660. The ONLY thing in the
  strip is the street name, in the strip's CENTRE (x 416–1104), exactly
  where the stock street name sits, and only while Waze is showing — it
  never touches the time (left) or the temperature (right).
- **The media stays visible.** The media / song info shows on the RIGHT
  side ("103.3 FM" inside the right dial). The speed block is at the
  TOP-LEFT — the far side from the media — and the drive block, the
  G-ball, the dots and the M badge are at the top or the bottom. Nothing
  this app draws ever goes near the right side or the media area.
- **The overlays cover only road.** They sit on the map's edges (top
  strip, top-left corner, bottom edge) and cover a sliver of road there
  — never the route, the car arrow, the turn banner, the dials, the
  digital speed, the media, or the bottom strip. The dots are always
  visible as dim outlines at idle (never fully transparent), so the user
  can always see the row is there.
- **Adjustable, but never over anything.** The positions can be moved
  (settings + Move buttons), but every position is clamped to the panel
  and above the bottom strip, so no overlay can ever be moved over the
  date/time, the outside temperature, the media, or the BC functions.

## How it stays visible (the mechanism)

The dots, the speed block, the drive block, the G-ball, the street name
and the M badge are **image layers on the PANEL target (the whole
1920×720 cluster)**, with class `overlay`, z 20, and **no `contend`**.
The panel is always shown, so overlay layers on it with no contend draw
above whatever the stock HMI shows — the Waze map or the stock BMW map —
in the same scene, without taking a slot. That is why the overlays show on
both Waze and BMW Maps, and why they never fight the stock HMI. The Waze
map's showing state is read from `/run/waze-on-cluster/status.json` (as
Cluster Info does) only to decide when to show the street name and the M
badge (which must not duplicate the stock ones on BMW Maps).

This is the opposite of the evangoyuk Shift Lights mistake: that app drew
an **effect layer on the `panel` target with `contend: true`**, so it
fought the stock HMI for the panel slot (the flicker), and its map
fallback also used contend and got "slot locked". Speed Shift never contends
— it rides the panel the stock HMI already holds.

What Speed Shift **does** borrow from Shift Lights (the user asked for the
developer's way, since he did it correctly): the **red rendering** — the
exact red (235,0,15) and yellow (255,199,10) from his shader, the 4 Hz
flash (`fract(u_time * 4.0)` → a 125 ms toggle), all dots turning red at
the shift point — and the **position handling** — posX/posY/size settings,
20 px Move buttons, Bigger/Smaller in the SIZES steps, Centre, Reset
positions, and a 5 s lit preview after a move. Only the rendering and the
position handling are borrowed; the overlay mechanism (panel, no contend)
is Speed Shift's own, so there is no panel fight.

## How it differs from evangoyuk Shift Lights (mistakes avoided)

| evangoyuk Shift Lights 1.6.0 | Speed Shift |
|---|---|
| Panel target + contend → flicker in Map view | Panel target, overlay class, no contend → stays visible on Waze AND BMW Maps |
| Held the panel → hid the media display and BC functions | Never takes a slot; fixed-size rects on the panel only, placed so they never sit where the BC menu appears (the drive block moved to the top-right in v1.4.0 — it had sat at the bottom-left, on top of the BC menu) → media/BC/bottom strip never hidden |
| No autostart in the built package → "not working" after a restart | `startType: autostart` in app.json, `mguAppStartType: autostart` baked into the package → starts with the head unit |
| Settings in a file on the car, hard to change | Settings in the app's own settings screen + Move buttons, changeable any time |
| Segmented bar that fills (a scale) | OEM M-style individual round LED dots at the top |
| Bar fully transparent below the start rpm → invisible at idle | Dots always visible as dim outlines at idle |
| Solid look, no mode colours | Transparent backgrounds; accent colours follow the drive mode |

## Design rule (standing rule)

Nothing this app draws may hide or cover any other screen function —
media, BC options, Waze map, Cluster Info, the date/time, the outside
temperature, anything. Every BMW function must remain visible. The
overlays fit around them, not on top of them. Their rectangles are fixed
in size — they never grow — and their positions are clamped so their
bottom edge stays above the bottom strip, so they can never cover anything
else, not even for a moment while the dots or the boost activate. The only
thing that changes is the pixels INSIDE those rectangles.

## Never changes the layout

This app draws **ON TOP of the layout he already has**. It does NOT change
or replace the cluster layout. His Cluster Layouts setup (Modern Wide /
Analog Sport, Waze map in the centre) stays exactly as it is. The dots,
the speed, the gear and the boost are overlays drawn on top of the current
layout, in their own spots. This app never writes Cluster Layouts'
settings, never changes its layout, centre or show setting, and never
touches any other app's files. It only reads the Waze map's status (to
know when to show the street name and the M badge) and draws its own
layers on the panel target. Turning it off or uninstalling it leaves the
layout exactly as it was.

## Safety — read-only, no permanent change, easy revert

- **Read-only.** This app only READS car signals and draws pixels on the
  cluster. It sends no command to the car, changes no car setting, touches
  no other ECU or function, and does not interfere with any other app
  (Waze on cluster, Cluster Layouts / Info / Centre, Ambient). It never
  writes to a safety system (no lane-assist, braking, or limiter changes).
  Safe to leave running always.
- **Live data only.** Every value shown is real, live data from the car.
  No value is hardcoded and no value is made up. When a reading is not
  available the app shows "--" instead of inventing a number. The only
  demo is the "Test the bar on the cluster" button: a clearly labelled
  6-second preview, OFF by default (it only runs when tapped), and it
  never affects the live readings.
- **Stock speed limit — not touched.** The car already has BMW's own
  speed limit (Speed Limit Info): the KOMBI itself draws the small
  speed-limit sign from the KAFAS camera and the nav data. This app never
  writes to the KOMBI, never changes any speed-limit setting, and never
  sends any command — so the stock speed-limit display and function keep
  working exactly as before, untouched. The app only READS the same limit
  value and shows it BIG with a warning on top.
- **No permanent change.** The app writes nothing except its own settings
  file (inside its own app folder) and its own layer state. It modifies
  no BMW system file and no other app's files.
- **Off switch.** `mode` Off in the app's settings hides all six layers
  completely and stops them drawing — instantly, from the car.
- **Clean uninstall — the car goes back to exactly how it was.** One
  command (over SSH, or use the Store's uninstall):

```
mgu-deploy uninstall muse_shift_bar --yes
```

That removes the app, its settings and its layers. No leftovers, no
changed system files, no leftover settings that break other apps. After
removal the cluster is exactly as it was before: the Waze map, Cluster
Info, Cluster Layouts and the stock displays are untouched (they are
separate apps and this app never wrote to them). If the head unit is
restarted after the uninstall, the overlays are simply gone — nothing else
changes.

## Install

Needs: **Cluster renderer** 0.4.2 or newer (the car has 0.5.12) and
**Waze on cluster** running. Both are already installed.

**From the MGU Store (after publishing — see RELEASE.md):** add the source
`AnudeepPamidi1/speed-shift`, install
**Speed Shift 1.5.6**.

**Without the Store (mgu-deploy, over SSH):**

```
mgu-deploy install /path/to/muse_shift_bar@1.5.6.mgu --yes
```

Then restart the head unit (or start the app once from the Store).
Autostart is baked into the package, so it starts on every boot after that.

## Test (parked)

1. Open Waze on CarPlay so the map shows on the cluster.
2. Open the **Speed Shift** app on the centre screen. The status line shows
   the live rpm, speed and gear, the speed limit and the warn threshold,
   and whether the overlays are on the cluster. Below it: the tyre
   pressures and temperatures, the gearbox oil temperature and cruise set
   speed, the oil and coolant temperatures, the turn indicators and the
   ambient slot, the BC trip data, the service-due, and the street name.
3. Tap **Test the bar on the cluster** — the dots sweep up to the shift
   point and the speed block shows a demo (speed, gear, boost) for
   6 seconds. This is a clearly labelled preview; the live readings
   resume after it.
4. Drive (or rev parked, carefully): at idle the dots show as dim
   outlines; as the rpm rises they light up one after another (green,
   then yellow, then red); at the shift point the whole row turns red and
   flashes. On an upshift the row drops at once to the new rpm — it does
   not hang above it.
5. Drive past a speed-limit sign: the limit sign shows the limit BIG in
   the speed block; go over the limit + the overspeed offset and the
   speed digits turn red and flash (the limit stays plain black).
6. Launch from a standstill: the drive block times the 0-100 km/h run
   and keeps the time on screen until the next run; the best time is on
   the menu screen.
7. Park the car: the drive block switches to the trip summary (trip
   distance, average speed, fuel used).
8. Change the numbers in the app's settings any time — they take effect
   immediately. Move an overlay with the Move buttons (pick it first with
   the "Move buttons move" setting) — the dots light up for 5 s so you
   can see where it went.

## Settings (in the app, on the car)

| Setting | What it does | Default |
|---|---|---|
| Speed Shift (mode) | On / Off — the off switch. Off hides everything and stops drawing | On |
| Lights start at | The rpm the first dot lights | 700 rpm |
| Shift point | The rpm the row goes red and flashes | 4000 rpm |
| Flash at the shift point | Flash the row at the shift point (4 Hz) | On |
| Speed, gear, boost and limit on the map | Show/hide the speed block (top-left) | On |
| 0-100, fuel range and torque on the map | Show/hide the drive block (top-right) | On |
| G-force ball on the map | Show/hide the G-ball (bottom-right) | On |
| Speeding warning: over the limit by | Warn when speed is over the limit by this much (km/h) | +5 km/h |
| Dots: left / top / size | The LED dots row's position and size | 720 px / 14 px / 100% |
| Speed block: left / top | The speed block's position | 428 px / 132 px |
| Drive block: left / top | The drive block's position | 428 px / 458 px |
| G-force ball: left / top | The G-ball's position | 1396 px / 492 px |
| Street name: left / top | The street name's position (centre of the bottom strip only) | 760 px / 665 px |
| M badge: left / top | The M badge's position | 648 px / 26 px |
| Move buttons move | Which overlay the Move buttons move (dots / speed / drive / G-ball / street / M) | dots |

## Files in this project

```
shift-bar/
  README.md                 this file
  RELEASE.md                how to publish it to GitHub + install from the Store
  LICENSE                   Proprietary — All Rights Reserved (AnudeepPamidi)
  mgu-repo.json             repo manifest (format 1) for the MGU Store source
  apps/muse_shift_bar/      the app source (app.json, main.js, cluster_render_client.js)
  kit/                      the idrive7-apps kit (build tools + runtime)
  oap-sdk/                  the OAP SDK (from the head unit, via mgu-deploy sdk-donor)
  dist/
    muse_shift_bar@1.5.6.mgu   the installable package
    muse_shift_bar@1.5.6.tgz   the same package as a tarball
  test/smoke.js             headless smoke test (node test/smoke.js — 56 checks)
```

Build it again after a change with:

```
python3 kit/tools/build.py muse_shift_bar
```

## Developer

**AnudeepPamidi** — in `app.json` (`vendor`), in this README, in the
release files, and on the app's menu screen ("Made by AnudeepPamidi").

Built on the idrive7-apps kit (MIT) and robertnisipeanu's cluster-render
client (MPL-2.0); the dots' look follows BMW's own M shift light, as the
stock cluster draws it, and the red rendering + position handling follow
evangoyuk's Shift Lights 1.6.0 (MIT), which did those two things
correctly.

## Version history

- **1.5.6** — the working build (2026-11-15): the 1.5.5 package would not
  start on the car — its JavaScript used newer syntax the head unit's
  Node 8 cannot read. 1.5.6 rebuilds it for Node 8 and is verified on the
  car itself (installed and running, all signals subscribed). Everything
  from 1.5.5 included: his saved positions/sizes as defaults; street
  name fixed; limit layer hides when no limit; lane redesigned; car
  ahead with gap bar and lorry rule; BC menu button ("BC menu — hide
  overlays") that hides all overlays for 20 s so the stock BC menu can
  show — note: the car's BC button press sends no signal any app can
  see, so the menu button is the way to see the BC menu while Speed
  Shift runs. Smoke test: 138 checks.
- **1.5.5** — your layout as the design (2026-11-15): his saved positions/sizes are the
  defaults, so a fresh install lands exactly on his cluster; street
  name fixed above the bottom strip (never cut off); the limit layer
  hides when there is no limit (no empty "--" box — the white "--" box
  in his photo is BMW's own stock box, which no app can hide); lane
  redesigned clean and OEM (no card, thin glowing lines, subtle fill,
  drift direction unmistakable — car off-centre + near line thicker and
  brighter); car ahead with a big 30 px distance number and a 6-segment
  Audi-style gap bar, lorry image when the vehicle is wide (width >=
  20 dm) as well as when the type is a truck; BC menu fix (all 13
  overlays hide for 15 s on a Kombi selection change and 20 s from the
  menu's "BC menu" button, every layer yields, never contends, so the
  whole right-side BC segment stays clear); smooth animation everywhere
  (limit number cross-fades, ring colour blends, lane and car-ahead
  glide on low-pass smoothing, never snap); TEST auto-off above
  5 km/h; packaged JavaScript minified + obfuscated; LICENSE
  proprietary, source never published. Smoke test: 138 checks.
- **1.5.0** — the working speed limit + sign boards + lane display + car ahead (2026-11-15): the speed limit now comes from the car's fused DASS traffic-sign recognition (presentClassLimitSpeed × 5 = km/h — class 6 = 30, class 10 = 50, proven by the drive logs; the old display signal never publishes on this car and is kept only as a fallback). A small road sign board shows next to the speed block when the KAFAS camera recognises a sign (STOP, speed limit, yield, traffic-light-ahead), shaped like the real sign (~48 px, with a "8 m" / "30 · 7 m" distance caption), fading in over ~300 ms and out when passed; it hides when no sign is recognised. A small lane display at the lower-left shows the lane lines (the REAL line type — dashed, solid, curb, double line — from the broadcast's properties) with a small rear-view car in the middle, the line spacing scaled to the real lane width; the side the car drifts toward blinks amber slowly (~1 Hz), the side it crosses blinks red fast (~3 Hz), and the correct side is guaranteed by the verified sign convention (+y = RIGHT, offset = −(yRight+yLeft)/2 — the 05:11:42 UTC right-side crossing proves it). A small car-ahead indicator at the bottom of the map shows the vehicle ahead as a realistic white/silver rear view (a sleek car, a tall boxy truck, or a generic van), with the distance in bold white whole metres underneath, a type + status tag (Car/Truck · moving/closing/stopped) and a thin vertical gap bar beside it that fills upward and turns red from the bottom up when close — the icon itself NEVER changes colour. Also in 1.5.0: the speeding offset's default is 10 km/h (was 5, still changeable 0/5/10/15/20); the fast 4 Hz speed flash is replaced by a slow pulse (the speed digits turn red steady, the limit sign gently pulses ~1 Hz — ring thickens, sign scales up ~10%); the next limit is a small round sign under the main sign (not "next 50" text); BMW's own SLI warning flag (controlSLIWarning = 1) gets a red-bordered triangle with a "!" badge at the limit sign's top-right, pulsing gently; and the BC safe zone (x 600–1100, y 300–550, where BMW's own Board Computer menu appears) is now a hard constraint — no overlay may overlap it, the Move buttons clamp to it, and a saved position inside it is moved out on startup. The traffic light is unchanged (raw KAFAS + displayForesight, hidden when no value — it still has no live colour on this car). Smoke test: 100+ checks.
- **1.4.0** — traffic light on the cluster (2026-11-14): a real traffic
  signal box on its OWN movable layer (36x68, default next to the
  speed-limit sign at 560,144): a dark rectangular housing with three
  round lamps stacked vertically (red on top, yellow, green), only the
  current colour lit BRIGHT with a glow and the others dim, fading
  smoothly between colours over ~300 ms like a real signal. When the
  light is a green arrow (17/18), a BIG, BOLD white arrow is drawn
  inside the lit green lamp (upper-right for 17, lower-right for 18).
  THE TURN SIGNAL decides whether the arrow applies to him: indicator
  on (left/right) = the arrow is bright (it applies to him); indicator
  off (going straight) = the green lamp is NOT lit bright and a small
  DIM green arrow is drawn instead, so he cannot mistake it for a
  bright green go-signal while his light is red. A plain green/red/
  yellow applies to all directions. NO SIGNAL, NO COLOUR: the box shows
  a colour ONLY when the signal reports an active light state
  (12/13/14/15/16/17/18); with no signal (0/10/11/unknown/no value)
  the box is hidden — it never fakes a colour. The light comes from the
  RAW KAFAS traffic-light camera feed (the primary source — BMW service
  0x3055/UCC is not offered on this car, so displayForesight usually
  never publishes), with displayForesight as the fallback and the arrow
  source; a foresight arrow wins even when the raw feed says red (red
  for straight, green arrow for turning). The box is movable like the
  other overlays (Move buttons, 'Traffic light', tlX/tlY) and hides
  during a phone call when it overlaps the call display. Also in 1.4.0:
  the LED dots are now blue / cyan / white by RPM (never green/red —
  those are the traffic light's colours) and flash white at the shift
  point; the speed-limit sign is sign-board big (radius 40); the drive
  block moved from the bottom-left to the top-right of the map
  (1148,132) so the stock BC menu (BC button on the stalk) is never
  covered; a moved position is saved at once and survives a restart
  (a backup copy, positions.json, is restored if the settings are ever
  reset). The live traffic-light value is NOT yet confirmed on this car
  — it needs a drive to a traffic light. Smoke test: 70 checks.
- **1.3.1** — live-car fixes from the user's own testing (2026-11-14):
  the LED dots are now neutral grey outlines at idle and green/yellow/red
  by RPM only — the drive-mode accent no longer tints them, so the whole
  row no longer looks red at idle in Sport mode (exactly like Shift
  Lights); the speed turns red ONLY when it is over the limit + offset —
  BMW's own SLI warning flag is no longer used, so it no longer turns red
  under the limit; the parked trip summary no longer shows the average
  speed, so the speed shows once on the cluster (the average speed stays
  on the menu screen). Smoke test: 56 checks (2 new: the grey dots in
  Sport mode, the warning flag not turning the speed red).
- **1.3.0** — OEM LED dots at the top (always visible, Shift Lights red
  + 4 Hz flash, adjustable positions); speed limit BIG with the speeding
  warning (speed turns red and flashes over limit + offset, next limit
  small); transparent backgrounds; colours follow the drive mode; fuel
  litres next to the range; drive block's conditional line (tyre low /
  overheat / low fuel / warming up / fuel use) + trip summary on park;
  street name + M badge while Waze is showing (bottom strip hard rule);
  KOMBI signals (BC trip data, service-due, oil/coolant, indicators,
  ambient slot); overlays on the panel so they show on Waze AND BMW Maps;
  best 0-100 on the menu; phone calls hide only the overlays that overlap
  BMW's call display (speed / drive / G-ball at their default places —
  the dots, street name and M badge keep showing) and bring them back
  when the call ends. NOT added (checked and excluded): check-control
  messages (unparseable), power split (petrol car), night mode, bigger
  gear in manual, FCW/AEB, blind spot, lane-assist low speed, chime, LIM
  control.
- **1.2.0** — drive block (0-100 timer, fuel range and level, torque)
  and G-force ball on the cluster; tyre pressures/temperatures, gearbox
  oil temperature and cruise set speed on the menu screen; `drive` and
  `gforce` settings; gear-change snap (the bar resets at once on an
  upshift).
- **1.1.0** — speed block (BIG speed, gear, boost), `speed` and
  `overspeed` settings, the settled-frame fix (the bar fully clears when
  the rpm drops), developer name AnudeepPamidi.
- **1.0.0** — first build (bar only).
