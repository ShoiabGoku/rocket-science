# 🚀 Rocket Flight Lab — Ascent & Re-entry Physics Simulator

A single-file, zero-dependency web simulator of a rocket **ascending from** or **re-entering toward** a planetary surface, with a real atmosphere model, aerodynamic heating, and configurable everything — gravity, atmosphere depth, thrust, propellant chemistry, nose geometry, and thermal-protection material.

**Run it:** open `index.html` in any browser — no build step, no server needed.
(Or serve locally: `python -m http.server` and visit `http://localhost:8000`.)

---

## Two flight modes

| Mode | What happens |
|---|---|
| **▲ ASCENT** | Ignition on the pad → Max-Q → MECO → apogee. If the rocket isn't fast enough to escape, it falls back and re-enters — the full flight is simulated, including the return heating. |
| **▼ DESCENT / RE-ENTRY** | Released nose-first at your chosen entry altitude and speed. Ride the plasma, then use the throttle as **retro-thrust** for a propulsive landing burn. Touch down under 8 m/s for a soft landing. |

## What you can configure

- **Planet & gravity** — surface gravity g₀, planet radius (gravity falls off as inverse-square with altitude), or pick a preset: **Earth, Moon, Mars, Venus, Titan**
- **Atmosphere** — how far it extends (the air genuinely ends at your chosen edge), surface pressure & temperature, lapse rate, specific gas constant R, heat-capacity ratio γ
- **Rocket** — dry mass, propellant mass, max thrust, body diameter, live throttle
- **Propellant** (sets sea-level & vacuum specific impulse):
  RP-1/LOX · LH₂/LOX · CH₄/LOX · N₂O₄/UDMH · APCP solid
- **Nose shape** (sets the Mach-dependent drag curve *and* the stagnation-point radius):
  conical · ogive · parabolic · spherical-blunt · flat
- **Nose material** (thermal mass, emissivity, failure temperature):
  Aluminium 2024 · Ti-6Al-4V · Stainless 310 · Inconel X-750 · Carbon-Carbon · **PICA ablative** (consumes ablator stock above 2900 K)

Full telemetry: altitude, velocity, Mach, felt g-load, dynamic pressure, heat flux, nose temperature vs material limit, effective Isp, TWR, local air ρ/P/T — plus strip-chart graphs and a time-stamped flight event log (LIFTOFF, MAX-Q, MECO, APOGEE, ENTRY INTERFACE, PEAK HEATING, TOUCHDOWN / CRASH / ESCAPE / VEHICLE LOST).

## Physics models

| Effect | Model |
|---|---|
| Gravity | Inverse-square: g(h) = g₀ · (R / (R+h))² |
| Atmosphere | Hydrostatic equation dP/dh = −ρ·g(h) integrated numerically (RK2) with T(h) = max(T₀ − L·h, 0.72 T₀); density from the ideal-gas law ρ = P/(R_s T); smooth taper to exactly zero at the user-set edge |
| Drag | F = ½ ρ v² C_d(M) A, with a per-nose-shape drag curve (subsonic plateau → transonic rise → supersonic decay) |
| Speed of sound | a = √(γ R_s T), so Mach number responds to the local air temperature |
| Stagnation heating | Sutton–Graves: q̇ = 1.7415×10⁻⁴ √(ρ/rₙ) v³ — blunt noses (large rₙ) genuinely run cooler, which is why capsules are blunt |
| Nose thermal balance | Lumped skin: heat in (q̇ + convection) minus radiative cooling εσ(T⁴ − T_air⁴); exceed the material limit and the vehicle is lost |
| Ablation | PICA holds ~2900 K while sacrificing ablator mass (q̇/h_abl); run out and the bare substrate fails fast |
| Propulsion | Effective Isp blends vacuum ↔ sea-level values with ambient pressure; propellant drains at ṁ = F/(Isp·g₀); MECO when dry |
| Escape | Flags an escape trajectory when specific orbital energy v²/2 − μ/(R+h) ≥ 0 outside the atmosphere |
| Integration | Semi-implicit Euler at 100 Hz with time-warp 0.5×–50×; physics on a wall-clock timer so background tabs don't freeze the flight |

The Earth preset reproduces the US Standard Atmosphere closely (10 km: 223 K, 26.4 kPa, 0.413 kg/m³ vs the standard's 223.3 K, 26.5 kPa, 0.414 kg/m³).

## Things to try

1. **Why capsules are blunt** — re-enter at 3 km/s with a *conical* aluminium nose (vehicle lost in seconds), then switch to *spherical* + PICA and watch peak heat flux drop by an order of magnitude.
2. **Suicide burn** — descent mode, 4 m blunt body, ~2 t of propellant: coast through entry heating, then throttle up below ~3 km and try to touch down under 8 m/s.
3. **Moon hop** — Moon preset: no drag, no Max-Q, no heating, and a TWR of 12 on the same rocket. Escape velocity comes embarrassingly quickly.
4. **Venus is hell** — 92 bar and 737 K at the surface. Watch what the atmosphere does to your ascent.
5. **Throttle discipline** — full throttle from the pad melts an aluminium nose in the dense lower atmosphere; real rockets throttle down through Max-Q for a reason.

## Controls

`↑ / ↓` throttle · `Space` pause · `R` reset — plus the on-screen sliders for everything else.

---

Built with vanilla HTML/CSS/JS Canvas — no frameworks, no dependencies.
Educational 1-DoF (vertical) model; not a mission-design tool.

*References: Sutton & Graves (1971) stagnation heating correlation; U.S. Standard Atmosphere 1976; Anderson, "Hypersonic and High-Temperature Gas Dynamics".*
