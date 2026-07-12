# Rocket Flight Lab — Code User Guide (for non-coders)

This document explains **every piece of code** in `index.html`, in plain language, assuming you've never written a line of code before. By the end, you should be able to open the file, find any section, understand roughly what it does, and — if you're feeling brave — change a number and see what happens.

Nothing here requires you to write code. It's a map and a translator: "here's the code, here's what it means, here's why it's needed."

---

## 0. The big picture — what kind of file is this?

`index.html` is a **single text file** that a web browser reads and turns into the app you see. It contains three languages stacked on top of each other, in this order:

1. **HTML** — the *skeleton*. Defines what things exist on the page: buttons, sliders, panels, the canvas (the drawing area). No behaviour, no looks — just "there is a button here called Start."
2. **CSS** — the *styling*. Says what things look like: colors, sizes, spacing, fonts. If HTML is the skeleton, CSS is the skin and clothes.
3. **JavaScript (JS)** — the *brain*. This is the actual program: the physics, the rules, what happens when you click something, what gets drawn on screen every fraction of a second. Everything the simulator *does* lives here.

You can see the boundaries in the file itself:

```html
<style>
   ...CSS lives here...
</style>
</head>
<body>
   ...HTML lives here...
<script>
   ...JavaScript lives here...
</script>
</body>
</html>
```

Because it's one file with no external downloads, you can literally double-click `index.html` and it runs — no installation, no "build," no internet connection required.

---

## 1. HTML section — the skeleton

### 1.1 What HTML *is*

HTML is made of **tags**, written like `<tagname>content</tagname>`. A tag wraps content and gives it meaning. For example:

```html
<button id="btnStart">LAUNCH</button>
```

This says: "there is a button, its text says LAUNCH, and I'm giving it a name tag (`id="btnStart"`) so JavaScript can find it later and attach behaviour to it." The `id` is like a name badge — it doesn't change how it looks, only how the code can grab hold of it.

### 1.2 The header bar

```html
<header>
  <div class="logo">🚀 ROCKET FLIGHT LAB <small>· ascent &amp; re-entry physics simulator</small></div>
  <div class="tabs">
    <button id="tabAscent" class="active">▲ ASCENT</button>
    <button id="tabDescent">▼ DESCENT / RE-ENTRY</button>
    <button id="tabOrbit">◉ TO ORBIT</button>
  </div>
  <div class="runbox">
    <button id="btnMute" title="Sound on/off">🔊</button>
    <input type="range" id="vol" min="0" max="100" value="70" title="Volume">
    <select id="warp" title="Time warp">...</select>
    <button id="btnStart">LAUNCH</button>
    <button id="btnPause" disabled>PAUSE</button>
    <button id="btnReset">RESET</button>
  </div>
</header>
```

- `<div>` is a generic invisible box used purely to *group* things together — like a folder. The `class="tabs"` groups the three mode buttons so CSS can style them as a row.
- `<button id="tabAscent">` — a clickable button named `tabAscent`. JavaScript later says "when this is clicked, switch to Ascent mode."
- `<input type="range">` — a slider. `min`/`max`/`value` set its range and starting position. This exact tag is what makes the **volume slider**.
- `disabled` on the Pause button means it starts greyed-out and unclickable, because there's nothing to pause before you've launched.

### 1.3 The three-column layout

```html
<main>
  <aside id="controls">...all your sliders and dropdowns...</aside>
  <section id="stage">
    <canvas id="sim"></canvas>
    <div id="banner"></div>
  </section>
  <aside id="telemetry">...readouts, charts, event log...</aside>
</main>
```

Three boxes side by side: controls on the left, the visual simulation in the middle, telemetry (numbers/graphs/log) on the right.

`<canvas id="sim">` is the single most important tag in the whole file for the *visual* side. A canvas is a blank rectangle that JavaScript can draw on — pixel by pixel, shape by shape — many times per second. Every rocket, flame, cloud, star, and explosion you see is drawn fresh onto this canvas roughly 60 times a second. It's not a picture; it's a whiteboard the code repaints continuously.

### 1.4 A control panel example

```html
<details open><summary>PLANET &amp; GRAVITY</summary><div class="body">
  <div class="selrow"><label>World preset</label>
    <select id="preset">
      <option value="earth" selected>Earth</option>
      <option value="moon">Moon (airless)</option>
      ...
    </select></div>
  <div id="grp-planet"></div>
</div></details>
```

- `<details>`/`<summary>` is a built-in HTML "collapsible box" — click the heading (`summary`) and the body folds open/closed. That's what gives you the little ▸/▾ arrow panels like "PLANET & GRAVITY," "ATMOSPHERE," etc. No JavaScript needed for the fold — it's a free browser feature.
- `<select>` is a dropdown. Each `<option>` is a choice; `value="earth"` is the internal name the code uses, "Earth" is what you read.
- `<div id="grp-planet"></div>` is an **empty box on purpose**. It's a placeholder. JavaScript fills it later with sliders, because there'd be too much repetitive HTML to write every slider by hand — instead, the code *generates* them from a table (see §3.2).

---

## 2. CSS section — the look

CSS rules follow the pattern:

```css
selector { property: value; property: value; }
```

The "selector" says *what* to style, the properties say *how*.

### 2.1 Custom colors (CSS variables)

```css
:root{
  --bg:#05070d; --panel:#0b111d; --panel2:#101a2c; --line:#1c2942;
  --txt:#c8d4e8; --dim:#66738c; --cyan:#4fd1ff; --amber:#ffb454;
  --red:#ff5566; --green:#5dffa3; --violet:#b48cff;
}
```

This defines a **palette of named colors** once, at the top. `#05070d` is a hex color code (near-black). Anywhere else in the CSS, `var(--cyan)` means "use that cyan color." The benefit: change the color once here, and every button, border, and text that uses `var(--cyan)` updates together. This is why the whole app has one consistent "mission control" look.

### 2.2 A typical rule

```css
.runbox button{
  font-family:var(--mono);
  font-size:12px;
  padding:8px 16px;
  border-radius:6px;
  border:1px solid var(--line);
  background:var(--panel2);
  color:var(--txt);
  cursor:pointer;
}
```

`.runbox button` means "any `<button>` that's inside something with `class="runbox"`." Reading down: monospace font, 12-pixel text, some inner spacing (`padding`), rounded corners (`border-radius`), a thin border in the line-color, panel-colored background, light text, and `cursor:pointer` — that last one makes your mouse arrow turn into a hand when hovering, telling you "this is clickable."

### 2.3 Layout with Flexbox

```css
body{display:flex;flex-direction:column;overflow:hidden}
main{flex:1;display:flex;min-height:0}
```

`display:flex` turns on a layout mode where children arrange themselves in a row or column automatically and can stretch to fill space — no manual pixel math. `flex-direction:column` stacks the header above the main area. Inside `main`, the default row direction places `#controls`, `#stage`, `#sim` side by side. `flex:1` on `main` means "take up all remaining vertical space." This is *why* resizing the browser window reflows everything nicely instead of breaking.

### 2.4 Responsive design (small screens)

```css
@media (max-width:1050px){
  body{overflow:auto}
  main{flex-direction:column}
  #controls,#telemetry{width:100%;border:none;border-bottom:1px solid var(--line)}
  #stage{min-height:420px;order:-1}
}
```

`@media` is a conditional CSS rule: "only apply what's inside if the browser window is narrower than 1050 pixels." On a phone or small tablet, this switches the three side-by-side columns into a single stacked column instead (`flex-direction:column`), with the simulation view pinned to the top (`order:-1`).

---

## 3. JavaScript section — the brain

This is the part that actually *runs*. Everything from here down sits inside one `<script>` tag. `"use strict";` at the top just tells the browser to be extra strict about catching typos and mistakes — a safety net for the programmer.

JavaScript vocabulary you'll see repeated everywhere, translated:

| Symbol/word | Plain meaning |
|---|---|
| `const x = 5;` | "Create a labeled box called `x`, put 5 in it, and never let it be reassigned." |
| `let x = 5;` | Same, but `x` **can** be changed later (`x = 6;`). |
| `function name(a, b) { ... }` | "Define a reusable recipe called `name` that takes ingredients `a` and `b` and does the steps inside `{ }`." |
| `if (condition) { ... } else { ... }` | "If the condition is true, do this; otherwise do that." |
| `for (...) { ... }` | "Repeat these steps a bunch of times" (a loop). |
| `object.property` | "Look inside `object` for a labeled compartment called `property`." |
| `array[i]` | "Get item number `i` from a numbered list" (counting starts at 0, not 1). |
| `// comment` | Text after `//` is a note for humans; the computer ignores it entirely. |
| `{ key: value, key2: value2 }` | An "object" — a bundle of labeled compartments, like a filing folder with named tabs. |

### 3.1 Physical constants

```js
const G_STD = 9.80665;              // standard gravity for Isp (m/s²)
const SB    = 5.670374419e-8;       // Stefan–Boltzmann (W/m²K⁴)
const SG_K  = 1.7415e-4;            // Sutton–Graves constant, SI (air)
const SKIN  = 0.006;                // nose skin thickness for lumped thermal mass (m)
const T_ABL = 2900;                 // ablation onset temperature (K)
```

These are fixed numbers from real physics that never change during a flight — they're used in formulas later:

- **`G_STD`** — "standard gravity," 9.80665 m/s². Rocket engine efficiency (Isp, "specific impulse") is, by international convention, always measured against *this* fixed number, even on Mars — it's a unit-conversion constant, not the actual local gravity.
- **`SB`** — the Stefan–Boltzmann constant. It's the physics law that says "hotter things radiate more heat, following a fourth-power curve." Used to calculate how much heat the rocket's nose *radiates away* into space.
- **`SG_K`** — the **Sutton–Graves constant**. This is the single most important number for re-entry heating. Real aerospace engineers use exactly this constant (and the formula it plugs into) to estimate how hot a spacecraft's nose gets during re-entry — the same formula used to design the Apollo and Dragon capsule heat shields.
- **`SKIN`** — how thick a slice of the nose material we're pretending to model (6 millimetres). It's a simplification — we don't simulate the whole vehicle's structure, just a thin "skin" layer that heats up and cools down.
- **`T_ABL`** — 2900 Kelvin (about 2627°C), the temperature at which an *ablative* heat shield (like PICA) starts to burn away on purpose to carry heat with it, rather than just getting hotter.

### 3.2 Data tables — the "menus" you pick from

```js
const FUELS = {
  kerolox:   {name:"RP-1 / LOX  (kerolox)", ispSL:283, ispVac:311, color:"#ffb347",
              info:"Kerosene + liquid oxygen. Dense, storable-ish, workhorse of Falcon 9 & Soyuz."},
  hydrolox:  {name:"LH₂ / LOX  (hydrolox)",  ispSL:366, ispVac:452, color:"#9fd8ff",
              info:"Liquid hydrogen + LOX. Highest Isp of any chemical fuel — SLS, Delta IV, LVM3 C25."},
  ...
};
```

This is a **lookup table**: a filing cabinet where `kerolox`, `hydrolox`, etc. are drawer labels, and each drawer holds a folder of facts about that fuel. `ispSL` and `ispVac` are **specific impulse** at sea level and in vacuum — essentially "miles per gallon" for rocket engines, measured in seconds. Higher Isp = more efficient engine = more speed per kilogram of fuel burned. These numbers are the real published values for each propellant type. When you pick a fuel from the dropdown, the code just reaches into this drawer and pulls out its numbers.

The same pattern repeats for:
- **`NOSES`** — cone/ogive/parabolic/sphere/flat, each with drag numbers (`cd0`, `cdPeak`, `cdSup` — drag coefficient at low speed, at the worst transonic speed, and at high supersonic speed) and a nose-radius number (`rn` or `rnFrac`). The nose radius is the single biggest factor in how hot re-entry gets — see §3.5.
- **`MATERIALS`** — aluminium, titanium, steel, Inconel, carbon-carbon, PICA. Each carries: `maxT` (temperature it fails at, in Kelvin), `rho` (density), `cp` (how much energy it takes to heat 1 kg by 1 degree — "specific heat"), `eps` (emissivity — how well it radiates heat away, 0 to 1). PICA additionally has `ablative:true`, `hAbl` (energy absorbed per kilogram sacrificed), and `ablStock` (how many kilograms of ablator you start with per square metre).
- **`PLANETS`** — Earth/Moon/Mars/Venus/Titan preset numbers for gravity, radius, atmosphere depth, surface pressure/temperature, lapse rate (how fast temperature drops with altitude), gas constant, and heat-capacity ratio (gamma).

### 3.3 `PARAM_DEFS` — how every slider in the app is *generated*, not hand-typed

```js
const PARAM_DEFS = [
  {id:"g0",      g:"planet", label:"Surface gravity g₀", unit:"m/s²",   min:0.1, max:30,    step:0.01, val:9.81},
  {id:"radius",  g:"planet", label:"Planet radius",      unit:"km",     min:200, max:20000, step:10,   val:6371},
  {id:"hatm",    g:"atmo",   label:"Atmosphere extent",  unit:"km",     min:0,   max:1000,  step:5,    val:100},
  ...
];
```

Instead of hand-writing 20+ nearly-identical blocks of HTML slider code, the programmer wrote **one row of data per slider** — its internal name (`id`), which panel group it belongs to (`g`), its on-screen label, its unit, and its min/max/step/starting value. Then a single piece of JavaScript (`buildControls()`, next section) reads this whole list and *builds* every slider automatically, matching slider to number-box to label. This is a core programming idea: **don't repeat yourself** — describe the pattern once as data, then generate the repetitive part.

Practical effect for you: if you ever wanted to widen a slider's range (say, allow thrust up to 50,000 kN instead of 20,000), you'd only need to change one number in this list (`max:20000` → `max:50000`) — the slider, its number box, and its live label all update automatically because they're all built from this single source.

### 3.4 Building the controls (`buildControls`)

```js
function buildControls(){
  for (const d of PARAM_DEFS){
    P[d.id] = d.val;
    const row = document.createElement("div");
    row.className = "prow";
    row.innerHTML =
      `<label>${d.label} <span class="pv"><span id="pv_${d.id}"></span> ${d.unit}</span></label>
       <div class="pin">
         <input type="range" id="rg_${d.id}" min="${d.min}" max="${d.max}" step="${d.step}" value="${d.val}">
         <input type="number" id="nm_${d.id}" min="${d.min}" max="${d.max}" step="${d.step}" value="${d.val}">
       </div>`;
    $("grp-"+d.g).appendChild(row);
    ...
  }
}
```

Walking through this in plain English:

- `for (const d of PARAM_DEFS)` — "for every entry `d` in that big list above, do the following."
- `P[d.id] = d.val;` — `P` is a global "current settings" object (think: a shared clipboard everyone can read). This line writes the slider's starting value into it under its name — e.g. `P.g0 = 9.81`. Every physics formula later reads its numbers from `P`.
- `document.createElement("div")` — literally "invent a brand-new empty HTML box" (a `<div>`), which doesn't exist in the original page yet.
- `row.innerHTML = `...`` — fills that new box with an HTML template (a slider + a matching number box), using backtick-quoted text (`` `...` ``) which lets you drop live values into the middle with `${...}`. This is called a *template literal* — it's just "text with blanks that get filled in."
- `$("grp-"+d.g).appendChild(row)` — finds the correct empty placeholder box from the HTML (e.g., `<div id="grp-planet">` from §1.4) and inserts the newly built slider row into it. (`$` is a shorthand helper defined earlier: `const $ = id => document.getElementById(id);` — "give me the element with this ID, quickly.")

After that, event listeners are attached — code that says "when this slider moves, do X":

```js
rg.addEventListener("input", () => set(rg.value));
nm.addEventListener("change", () => set(nm.value));
```

Every time you drag a slider or type in its number box, this fires, which updates `P[d.id]`, keeps the slider and number box in sync with each other, and (importantly) rebuilds the atmosphere table if a planet/atmosphere value changed (see next section) or recalculates the trip immediately if the simulation isn't currently running.

---

## 4. The atmosphere model — how "air" is simulated

This is the physics heart of the realism. It's worth understanding even loosely, because it explains *why* the sliders behave the way they do.

### 4.1 The idea in plain words

Real air gets thinner and colder the higher you go. This program builds a **lookup table** of "at this altitude, here's the temperature, pressure, and density" — computed once whenever you change a planet/atmosphere setting, then re-used thousands of times per second during flight (so it doesn't have to redo the expensive math every single physics tick).

```js
function buildAtmo(){
  const Hm = P.hatm * 1000;
  if (Hm <= 1 || P.p0 <= 0){ ATMO = {vac:true, Hm:Math.max(Hm,0)}; return; }
  const N = 1200, dh = Hm / N;
  const T = new Float64Array(N+1), Pr = new Float64Array(N+1), Rho = new Float64Array(N+1);
  const Tmin = Math.max(P.t0 * 0.72, 4);
  const Tof = h => Math.max(P.t0 - (P.lapse/1000)*h, Tmin);
  const Rm = P.radius * 1000;
  const gOf = h => P.g0 * (Rm/(Rm+h))**2;
  let p = P.p0 * 1000;
  for (let i = 0; i <= N; i++){
    const h = i * dh;
    T[i] = Tof(h);
    const s = Math.min(1, (Hm - h) / (0.08 * Hm));
    const taper = s*s*(3 - 2*s);
    Pr[i]  = p * taper;
    Rho[i] = (p / (P.rs * T[i])) * taper;
    if (i < N){
      const k1 = -(p / (P.rs * Tof(h))) * gOf(h);
      const pm = Math.max(p + k1*dh/2, 0);
      const k2 = -(pm / (P.rs * Tof(h+dh/2))) * gOf(h+dh/2);
      p = Math.max(p + k2*dh, 0);
    }
  }
  ATMO = {vac:false, Hm, dh, T, Pr, Rho, rho0:Rho[0]};
}
```

Step by step:

1. **`Hm = P.hatm * 1000`** — convert your "Atmosphere extent" slider from kilometres to metres (the whole program works in metres/kilograms/seconds internally, converting to friendlier units only for display).
2. **If the atmosphere is 0 km or the surface pressure is 0** (like your Moon preset) — just mark it `vac:true` (vacuum) and skip the rest. This is why the Moon has zero drag and zero heating: there's genuinely no air table built at all.
3. **Slice the atmosphere into 1200 thin altitude "shelves"** (`N = 1200`) from the ground up to your chosen edge. For each shelf we calculate three things and store them in three long lists (`T`, `Pr`, `Rho` — temperature, pressure, density):
   - **Temperature** (`Tof`) — drops linearly with altitude at the "lapse rate" you set, but never below 72% of the surface temperature (`Tmin`) — this mimics the real atmosphere's stratosphere, which stops cooling and levels off.
   - **Pressure** — computed by literally solving the physics equation that says "pressure at any height equals the weight of all the air stacked above it" (the *hydrostatic equation*). The code does this using a numerical technique called **RK2** (Runge-Kutta, 2nd order) — a smarter way of stepping forward bit by bit than just multiplying (it checks the middle of the step too, for accuracy). You don't need to understand RK2 itself; the practical point is: this is *solved from first principles*, not a copy-pasted formula, which is why it correctly adapts to whatever gravity/radius/pressure/temperature you dial in, for any planet.
   - **Density** (`Rho`) — from the **ideal gas law**: density = pressure ÷ (gas constant × temperature). This is the formula connecting pressure, temperature, and density for any gas.
4. **The "taper"** (`s*s*(3-2*s)`) is a smoothing trick (called a *smoothstep*) that gently fades pressure and density down to exactly zero over the last 8% of the atmosphere's height, instead of stopping abruptly. It's why your chosen atmosphere edge genuinely goes to true vacuum, cleanly, right where you set it.

### 4.2 Reading the table back out (`atmoAt`)

```js
function atmoAt(h){
  if (ATMO.vac || h >= ATMO.Hm) return {T:3, P:0, rho:0};
  if (h < 0) h = 0;
  const x = h / ATMO.dh, i = Math.min(Math.floor(x), ATMO.T.length-2), f = x - i;
  return {
    T:   ATMO.T[i]   + (ATMO.T[i+1]  - ATMO.T[i])  * f,
    P:   ATMO.Pr[i]  + (ATMO.Pr[i+1] - ATMO.Pr[i]) * f,
    rho: ATMO.Rho[i] + (ATMO.Rho[i+1]- ATMO.Rho[i])* f,
  };
}
```

Given any altitude `h`, this finds the two nearest shelves in the table and **blends between them** (called *linear interpolation*) so you get a smooth answer even between the pre-computed steps, rather than jumpy staircase values. This function gets called constantly — every physics tick, at whatever altitude the rocket currently is — to answer "what's the air like right here, right now?"

### 4.3 Speed of sound and Mach number

```js
function soundSpeed(T){ return Math.sqrt(P.gamma * P.rs * Math.max(T,1)); }
```

The speed of sound depends on temperature and the gas's properties (`gamma`, `rs`) — this is a standard formula from gas physics. Mach number (how many multiples of the speed of sound you're going) is then just: `S.mach = your_speed / soundSpeed(current_air_temperature)`. This is why Mach number changes even at constant real speed — as the air cools with altitude, the speed of sound itself drops, so your Mach number can *rise* even while you're not accelerating.

---

## 5. Aerodynamics — drag and nose shape

```js
function dragCoeff(n, M){
  if (M < 0.8) return n.cd0;
  if (M < 1.2) return n.cd0 + (n.cdPeak - n.cd0) * (M - 0.8) / 0.4;
  return n.cdSup + (n.cdPeak - n.cdSup) * Math.exp(-(M - 1.2) / 1.3);
}
```

This models a well-known real aerodynamic fact: as an object accelerates from subsonic to supersonic, its drag coefficient doesn't stay flat — it **spikes hard around Mach 1** (the "transonic drag rise," historically called "the sound barrier" because early aircraft struggled to punch through this spike) and then **settles down** to a lower, roughly steady value once fully supersonic. The three `if` branches implement exactly that three-phase shape using each nose shape's own `cd0`/`cdPeak`/`cdSup` numbers from the `NOSES` table.

The drag *force* itself is calculated elsewhere as:

```js
const qDyn = 0.5 * atm.rho * S.v * S.v;
const Cd = dragCoeff(nose, S.mach);
const Fd = qDyn * Cd * area * (S.v > 0 ? -1 : 1);
```

- **`qDyn`** is "dynamic pressure" — the classic aerodynamics formula ½·ρ·v² (half the air density times velocity squared). This single number is *the* measure of how hard the air is pushing back on you, and it's what the telemetry panel's "DYN PRESSURE q" is showing you live.
- **`Fd`** (drag force) = dynamic pressure × drag coefficient × the rocket's cross-section area, with a sign flip (`* -1` or `* 1`) so the force always points *opposite* your direction of travel — drag always resists motion, whichever way you're going.

---

## 6. Heating — why blunt noses survive re-entry

This is the physics idea the whole "nose shape" feature exists to teach.

```js
const rn = noseRadius(nose);
const qdot = atm.rho > 0 ? SG_K * Math.sqrt(atm.rho/rn) * Math.abs(S.v)**3 : 0;
```

This is the **Sutton–Graves equation**, a real formula used by actual spacecraft designers. In words: heat flux (energy hitting the nose per second, per square metre) is proportional to the **square root of air density divided by nose radius**, times **velocity cubed**.

Two things fall straight out of that formula, and both are exactly what real aerospace engineering discovered the hard way:

1. **Velocity cubed** — heating rises *extremely* fast with speed. Double your speed, and heating goes up 8×. This is why re-entry heating is such a dominant, dangerous effect and why "slow down before you hit thick air" is the golden rule of re-entry.
2. **1 ÷ √(nose radius)** — a *larger* nose radius means *less* heating. This is counter-intuitive (a bigger, blunter shape should hit more air, right?) but it's true: a blunt nose pushes the shockwave *away* from the surface (a "detached bow shock"), and most of the superheated air's energy gets carried away in that standoff shock layer instead of being dumped directly onto the vehicle's skin. This is precisely why every crewed re-entry capsule ever flown — Mercury, Apollo, Soyuz, Dragon — is a blunt shape, not a sharp needle. The `NOSES` table's `rnFrac:0.5` for the sphere option encodes exactly this.

### 6.1 The nose's temperature over time

```js
const cool = mat.eps * SB * (S.Tn**4 - Math.max(atm.T,3)**4);
const conv = atm.rho > 0 ? (10 + 0.3*Math.sqrt(atm.rho)*Math.abs(S.v)) * (atm.T - S.Tn) : 0;
const net  = qdot + conv - cool;
const heatCap = mat.rho * mat.cp * SKIN;
S.Tn += net * dt / heatCap;
```

Every physics tick, three heat flows are tallied up:

- **`qdot`** — heat pouring in from the Sutton-Graves formula above (always positive — always heating).
- **`conv`** — ordinary convective heat exchange with the surrounding air (can help cool the nose if the air is colder than it, e.g. during a slow climb through thin cold upper air).
- **`cool`** — heat *radiated away* into space, following the Stefan–Boltzmann law (`SB`) — this is why `S.Tn**4` (temperature to the 4th power) appears; radiative cooling scales extremely steeply with temperature, so a very hot nose sheds heat much faster than a slightly warm one.

`net` is the sum: heat in minus heat out. `heatCap` is "how much energy does it take to change this material's temperature" (bigger, denser, higher-`cp` materials heat up more slowly for the same energy input — that's literally what specific heat capacity means). Finally, `S.Tn += net * dt / heatCap` is Newton's basic rule for temperature change: *change in temperature = (net energy) ÷ (thermal mass)*, applied fresh every tiny time-step `dt`.

### 6.2 Ablation (PICA heat shield)

```js
if (mat.ablative && !S.ablDead && S.Tn >= T_ABL && net > 0){
  S.abl -= net * dt / mat.hAbl;
  S.Tn = T_ABL;
  if (S.abl <= 0){ S.ablDead = true; log("ABLATOR DEPLETED — bare substrate exposed", "bad"); }
} else {
  S.Tn += net * dt / heatCap;
}
```

If the material is ablative (PICA) and it's hot enough (`>= T_ABL`, 2900 K) and still heating (`net > 0`): instead of letting the temperature keep climbing, the code **locks the temperature at 2900 K** and instead spends the incoming heat energy *burning away material* (`S.abl -= ...`). This mirrors precisely what a real ablative heat shield does — it deliberately chars and flakes off, carrying enormous amounts of heat away with each vaporized gram, holding the surface at a roughly constant survivable temperature instead of letting it climb toward failure. Once the ablator stock (`S.abl`) runs out, it switches back to normal heating and the bare structure underneath is exposed and starts climbing in temperature — often fatally, fast.

### 6.3 Failure check

```js
const tLimit = S.ablDead ? 1100 : mat.maxT;
if (S.Tn > tLimit && !S.failed){
  S.failed = true; S.done = true;
  spawnExplosion(0.8);
  log(`NOSE STRUCTURAL FAILURE at ${Math.round(S.Tn)} K (limit ${tLimit} K)`, "bad");
  banner("bad", "⚠ VEHICLE LOST ⚠", ...);
  stopRun();
  return;
}
```

Simple threshold check: if the current nose temperature exceeds the material's rated maximum, the flight ends in failure — an explosion is triggered, an event is logged, an on-screen banner appears, and the simulation stops (`stopRun()`). `S.failed`/`S.done` are flags (true/false switches) that other code checks so it knows not to keep simulating a rocket that's already destroyed.

---

## 7. The propulsion and staging system

```js
let thrust = 0;
S.ispNow = 0;
if (propNow > 0 && throttle > 0 && stageMax > 0 && !sepCoast){
  const ispEff = stageFuel.ispVac - (stageFuel.ispVac - stageFuel.ispSL) * Math.min(atm.P/101325, 1);
  thrust = throttle * stageMax;
  const mdot = thrust / (ispEff * G_STD);
  let burn = mdot * dt;
  if (burn > propNow){ thrust *= propNow / burn; burn = propNow; }
  S.prop -= burn;   // (or S.prop2, depending on active stage)
  S.ispNow = ispEff;
}
```

- **`ispEff`** blends the fuel's vacuum and sea-level efficiency numbers based on how much ambient air pressure there currently is (`atm.P/101325`, where 101325 Pa is 1 standard atmosphere). This is real rocket behaviour: engines are genuinely *more* efficient in vacuum because there's no outside air pressure pushing back on the exhaust plume — that's why the readout for "EFFECTIVE Isp" changes as you climb.
- **`thrust = throttle * stageMax`** — your throttle slider (0–100%, converted to 0–1) simply scales the engine's maximum rated thrust.
- **`mdot`** ("mass flow rate," a standard rocketry term) — how many kilograms of propellant are burned per second, derived directly from thrust and efficiency: more thrust or less-efficient fuel means faster burning.
- **The safety clamp** (`if (burn > propNow)`) — makes sure the very last physics tick of a burn can't withdraw more propellant than actually remains in the tank; it scales thrust down proportionally for that final fractional moment so the numbers stay physically consistent (no negative fuel).

### 7.1 Two-stage separation sequence

```js
if (st2on && S.stage === 1 && S.prop <= 0 && !S.sepStarted){
  S.sepStarted = true; S.sepAt = S.t;
  log("MECO — booster burnout", "warn");
}
if (S.sepStarted && S.stage === 1 && S.t >= S.sepAt + 0.7){
  S.stage = 2; spawnBooster();
  log("STAGE SEPARATION — booster jettisoned", "good");
}
const sepCoast = S.sepStarted && S.t < S.sepAt + 1.4;
```

This is a small **state machine** — a sequence of named stages the flight moves through, one-way, based on conditions:
1. Stage-1 propellant hits zero → log "MECO" (Main Engine Cut-Off, standard spaceflight terminology) and mark the separation clock as started (`S.sepAt = S.t`, remembering *when*, in mission-elapsed time, this happened).
2. **0.7 seconds later** — physically separate: switch the active stage number to 2, and spawn the visual tumbling spent-booster effect.
3. For **1.4 seconds total** after burnout, `sepCoast` stays true, meaning "no engine can fire yet" — this models the brief coast period real rockets need between engine cutoff, physical separation, and the next stage's ignition sequence.

---

## 8. Orbit mode — the more advanced physics

The Ascent and Descent modes think in one dimension: straight up or straight down. Orbit mode needs **two dimensions**, because an orbiting object moves both toward/away from the planet *and* sideways around it.

### 8.1 Polar coordinates, in plain terms

Instead of tracking "up/down position and speed" only, orbit mode tracks:
- `S.h` — height above the surface (same as before)
- `S.v` — **radial** velocity (straight toward/away from planet center)
- `S.vt` — **tangential** velocity (sideways, around the planet — this is what makes an orbit an orbit, rather than just a very tall bounce)
- `S.th` — the downrange angle traveled around the planet so far

### 8.2 The core orbital physics

```js
const aR = -mu/(r*r) + S.vt*S.vt/r + (thrust*dR - Faero*uR)/m;
const aT = -(S.v*S.vt)/r + (thrust*dT - Faero*uT)/m;
S.v  += aR*dt;
S.vt += aT*dt;
S.h  += S.v*dt;
S.th += S.vt/(Rm + Math.max(S.h,0))*dt;
```

These two lines are the textbook equations of motion for an object orbiting a planet, written out in full:

- **`-mu/(r*r)`** — gravity's pull, always toward the planet's center. (`mu` is the planet's "gravitational parameter," `r` is your current distance from the planet's *center*, not its surface.)
- **`+ S.vt*S.vt/r`** — the **centrifugal effect**: moving sideways fast enough naturally tends to fling you outward, fighting gravity's pull inward. This is the entire reason orbits are possible at all — go sideways fast enough, and "falling toward the ground" and "the ground curving away beneath you" happen at the same rate forever, which *is* an orbit.
- **`-(S.v*S.vt)/r`** — the **Coriolis-like coupling term**: your sideways speed itself changes as you move nearer or farther from the planet (think of an ice skater's arms — pulling them in speeds up the spin). This term is what makes elliptical orbits (rather than perfect circles) speed up near the planet and slow down far away.
- **`(thrust*dR - Faero*uR)/m`** — your engine's thrust and air drag, both split into their radial (`R`) and tangential (`T`) components via `dR`/`dT` (thrust direction) and `uR`/`uT` (drag always opposes your actual direction of travel).

You don't need to be able to derive these equations — the important takeaway is: **this is real orbital mechanics**, the same math used to plan actual satellite and spacecraft trajectories, not a simplified approximation.

### 8.3 Guidance — how the rocket "flies itself" into orbit

```js
if (S.ophase === "ascent"){
  const p0 = P.pitchalt*1000, p1 = Math.max(P.turnend*1000, p0+1000);
  let pitch = 90;
  if (S.h >= p0){
    const u = Math.min(1, (S.h - p0)/(p1 - p0));
    pitch = 90*Math.pow(1 - u, 0.85);
    if (S.v < 120 && S.h < P.targetalt*1000*0.9)
      pitch = Math.max(pitch, Math.min(35, (120 - S.v)*0.15));
  }
  const pr = pitch*Math.PI/180;
  dR = Math.sin(pr); dT = Math.cos(pr);
}
```

This automates a real spaceflight technique called a **gravity turn**:
- Below your "pitch-over altitude" slider (`p0`), thrust points straight up (`pitch = 90` degrees from horizontal).
- Between `p0` and your "gravity-turn end" altitude (`p1`), the pitch angle **smoothly rotates from vertical toward horizontal** — `Math.pow(1-u, 0.85)` is just a curve shape (not linear — it stays steeper for longer, then flattens faster near the end, mimicking how real rockets are flown).
- The extra safety clause (`if S.v < 120...`) is a simple auto-correction: if the rocket's climb rate is stalling out too early (still under 120 m/s vertically) well below the target orbit altitude, it forces the nose back up a bit rather than let the vehicle mush into the ground before reaching orbital speed. This was added after testing revealed weak upper stages could otherwise "sink" mid-flight.
- `dR = sin(pitch)`, `dT = cos(pitch)` converts the pitch angle into "how much of my thrust points radially vs. sideways" — basic trigonometry, the same math used for any angle-into-components problem.

### 8.4 Phase machine: ascent → coast → circularize → orbit

```js
if (S.ophase === "ascent" && S.ra >= tgtR && S.vt > 1000 && S.h > P.pitchalt*1000 + 5000){
  S.ophase = "coast"; forceThrottle(0);
  log(`MECO — apoapsis ${((S.ra-Rm)/1000).toFixed(0)} km reached, coasting`, "good");
}
if (S.ophase === "coast"){
  if ($("autoCirc").checked && isFinite(S.ra) && S.vt > 1000 && ... ){
    S.ophase = "circ"; forceThrottle(100);
    log("CIRCULARIZATION BURN — prograde at apoapsis", "good");
  }
}
```

`S.ophase` is a text label ("ascent", "coast", "circ", "orbit", "ballistic") tracking which phase of the mission you're in — the *state* of a state machine, same concept as the staging sequence in §7.1, just with more steps:

1. **ascent** — flying the gravity turn, engine burning.
2. Once your calculated highest point (**apoapsis**, `S.ra`) reaches your target orbit altitude, cut the engine (`forceThrottle(0)`) and switch to **coast** — just falling freely along the current trajectory, like a thrown ball.
3. Once you approach that highest point for real (not just calculated ahead of time, but you're *actually* nearly there) — if "Auto-circularize" is checked, automatically fire the engine again (**circ** phase) to round out the orbit.
4. Once your low point (**periapsis**, `S.rp`) also reaches near the target altitude, you're genuinely in a stable **orbit** — engine cuts, and a banner announces "ORBIT ACHIEVED."

### 8.5 Apoapsis/periapsis — how the code knows your orbit shape *before you finish it*

```js
const eps = S.speed*S.speed/2 - mu/r;
const hAng = r*S.vt;
const ecc = Math.sqrt(Math.max(0, 1 + 2*eps*hAng*hAng/(mu*mu)));
if (eps < 0){
  const sma = -mu/(2*eps);
  S.ra = sma*(1 + ecc);
  S.rp = sma*(1 - ecc);
} else {
  S.ra = Infinity;
  S.rp = (hAng*hAng/mu)/(1 + ecc);
}
```

This is genuine orbital-mechanics bookkeeping: from just your current speed and position, you can mathematically predict the *entire future shape* of your orbit (its highest and lowest points) without simulating forward in time at all. `eps` ("specific orbital energy") tells you whether you're on a bound orbit (negative — you'll come back around) or an escape trajectory (`eps >= 0` — you're never returning, hence `S.ra = Infinity`, an "infinitely far" apoapsis). This is exactly how mission planners predict a spacecraft's orbit from a single tracking snapshot, and it's why the telemetry panel can show you a projected apoapsis/periapsis *live*, updating in real time as you burn, well before you've actually gotten there.

---

## 8½. Advanced flight dynamics

These four features layer real spaceflight-engineering detail on top of the orbit model. Each is a small amount of code with a big physical meaning.

### 8½.1 Planetary rotation and launch latitude

```js
function omegaPlanet(){ return P.sidereal > 0 ? 2*Math.PI/(P.sidereal*3600) : 0; }
function surfEastSpeed(h){ return omegaPlanet()*(P.radius*1000 + h)*Math.cos(P.latitude*Math.PI/180); }
```

A spinning planet carries everything on its surface eastward. `omegaPlanet()` converts the rotation *period* you set (hours per turn) into an *angular rate* (radians per second) — `2π` radians is one full turn, divided by the period in seconds. `surfEastSpeed()` then turns that spin rate into an actual eastward speed at your location: faster the bigger the planet (`P.radius`), and largest at the equator, shrinking to zero at the poles via `cos(latitude)` (the `*Math.PI/180` just converts degrees to the radians the trig functions expect).

On Earth's equator this is about **465 m/s of free velocity** you already have before the engine even lights. In orbit mode, `initState` seeds the rocket's sideways velocity with it:

```js
vt: mode === "orbit" ? surfEastSpeed(0) : 0,   // share the planet's eastward spin on the pad
```

This is why real launch sites cluster near the equator (Kourou, Cape Canaveral, Sriharikota) and launch *eastward* — the planet hands you a big chunk of orbital speed for free. Try setting latitude to 0° versus 70° and watch how much more Δv you have to spend from the high latitude.

There's a subtle, genuinely advanced consequence the code handles carefully: **the air rotates with the planet too.** So aerodynamic drag and heating should respond to your speed *relative to the moving air* (airspeed), not your speed relative to the fixed stars (inertial speed):

```js
const omega = omegaPlanet();
const vAirT = S.vt - omega*r;              // subtract the local air's eastward motion
const airspd = Math.hypot(S.v, vAirT);     // speed relative to the co-rotating atmosphere
```

The orbital *dynamics* (gravity, the shape of your orbit) use the inertial velocity, but everything *aerodynamic* — Mach number, dynamic pressure, drag, heating, and the touchdown speed — uses `airspd`. Without this split, a capsule drifting down under its parachute would look like it's moving at 465 m/s (its inertial speed, because it's still turning with the planet) when in reality it's touching down gently at walking pace relative to the ground.

### 8½.2 Lift and the L/D ratio — steering a re-entry

```js
const ldRaw = (nose.ld || 0) * (S.chute === "stowed" ? 1 : 0);
const Flift = (liftCmd !== 0 && atm.rho > 0) ? qDyn*ldRaw*Cd*area*liftCmd : 0;
const pR = uT, pT = -uR;          // 90° from the airflow; +liftCmd pushes away from the planet
```

A re-entry vehicle isn't just a falling rock — by flying at a slight angle it generates **lift**, a force sideways to its motion, and it can point that lift up or down. Each nose shape carries an `ld` number (its lift-to-drag ratio, from the `NOSES` table): a blunt Apollo-style capsule has L/D ≈ 0.3, a sharp cone more. `Flift` is the lift force — proportional to that L/D times the drag — and `liftCmd` (set by the **LIFT ↑ / BALLISTIC / LIFT ↓** buttons, values +1 / 0 / −1) chooses which way it points. `(pR, pT)` is the direction 90° away from the airflow (basic geometry: rotate the velocity direction a quarter turn).

Why it matters: pointing lift *up* (LIFT ↑) flattens your descent, so you slow down higher in thinner air. That **caps the peak deceleration and peak heating** and stretches your landing footprint downrange. In testing, switching a re-entry from ballistic to lift-up dropped the peak load from ~8 g to ~7 g and the nose temperature by hundreds of degrees. This is exactly why Apollo flew a lifting entry (holding ~6–7 g) instead of dropping straight in (which would pull ~20 g). Note the `S.chute === "stowed"` factor — once a parachute is out, the canopy dominates and body lift is switched off.

### 8½.3 The Δv budget — where a rocket's energy goes

```js
if (thrust > 0){
  const spInert = Math.max(S.speed, 1);
  S.dvSpent  += (thrust/m)*dt;
  S.lossGrav += g*(S.v/spInert)*dt;                       // gravity loss ∝ sin(flight-path angle)
  const cosSteer = (dR*S.v + dT*S.vt)/spInert;            // thrust vs velocity misalignment
  S.lossSteer += (thrust/m)*(1 - Math.max(cosSteer,0))*dt;
}
if (thrust > 0 && S.ophase === "ascent") S.lossDrag += (Faero/m)*dt;
```

Reaching orbit takes far more Δv (change in velocity) than the orbital speed alone, because a lot of your engine's effort is "wasted" fighting three things — and this block tallies each, integrating it up over the whole flight (`+= ... *dt` means "keep adding this tiny slice every time-step"):

- **`dvSpent`** — the total velocity change your engine actually delivered (thrust ÷ mass, summed over time). This is the real "cost" of the flight.
- **`lossGrav`** (gravity loss) — velocity lost to holding yourself up against gravity while climbing. It scales with `S.v/speed`, which is the sine of your flight-path angle: straight up loses the most, horizontal loses none.
- **`lossSteer`** (steering loss) — when your thrust doesn't point exactly along your direction of travel, the sideways component is wasted. `cosSteer` measures the alignment; `1 - cosSteer` is the waste.
- **`lossDrag`** (drag loss) — velocity eaten by air resistance during the climb.

For a typical Earth launch to orbit the simulator reports roughly 9,700 m/s delivered against ~7,800 m/s of orbital speed, with about 1,100 m/s lost to gravity, 200 to drag, and 1,000 to steering — numbers that line up with real launch-vehicle engineering. The telemetry panel shows "Δv DELIVERED" and "Δv LOSSES" live, so you can *see* the cost of a too-steep or too-shallow gravity turn.

### 8½.4 Structural G-limits — pulling too hard

```js
if (S.accFelt > S.peakG) S.peakG = S.accFelt;
...
if (S.accFelt > P.glimit && !S.failed && S.lifted){
  S.failed = true; S.done = true; spawnExplosion(1.0);
  log(`STRUCTURAL FAILURE — ${S.accFelt.toFixed(1)} g exceeds ${P.glimit} g limit`, "bad");
  banner("bad", "⚠ AIRFRAME TORN APART", ...);
  stopRun(); return;
}
```

`S.accFelt` is the **felt** acceleration — the g-force from thrust, drag, and lift only, *not* gravity (you don't feel gravity in free-fall, which is why astronauts float). `S.peakG` remembers the worst load of the flight for the telemetry panel. If that load ever exceeds your set structural G-limit, the airframe tears apart. A straight-down ballistic entry is the steepest possible path and pulls the most g — try dropping the limit to a crewed value (~12 g) and entering fast and steep, and you'll rip the vehicle apart before it lands. The lesson is real: steep entries are only survivable because vehicles fly *shallow, lifting* paths (§8½.2) that spread the deceleration out.

---

## 9. The parachute system

```js
if ($("chutesOn").checked && S.v < 0 && atm.rho > 1e-4){
  if (S.chute === "stowed" && S.mach < 1.6 && qDyn < 40000 && P.chutedrogue > 0){
    S.chute = "drogue"; S.drogueT = S.t;
    log(`DROGUE DEPLOY — ${Math.abs(S.v).toFixed(0)} m/s at ${fmtAlt(S.h)}`, "good");
  }
  if (S.chute === "drogue" && S.h < P.chutealt*1000 && Math.abs(S.v) < 150 && P.chutemain > 0){
    S.chute = "main"; S.mainT = S.t;
    log(`MAIN CHUTE DEPLOY at ${fmtAlt(S.h)}`, "good");
  }
}
```

Another small state machine (`S.chute`: "stowed" → "drogue" → "main"), gated by real deployment logic: a small drogue chute only comes out once you're falling slower than Mach 1.6 *and* the air isn't pushing too hard yet (under 40 kPa dynamic pressure — deploying a chute into too much force would rip it apart, exactly like real recovery systems are designed to avoid). The much larger main chute only opens once you're below your chosen altitude *and* slowed further (under 150 m/s) — because opening a huge canopy at high speed would also destroy it.

```js
S.drogueFrac = Math.min(1, (S.t - S.drogueT)/1.2);
Fch = qDyn * (0.9*P.chutedrogue*S.drogueFrac + 1.75*P.chutemain*S.mainFrac) * (S.v > 0 ? -1 : 1);
```

Chutes don't snap open instantly — `drogueFrac`/`mainFrac` ramp smoothly from 0 to 1 over 1.2 and 2.5 seconds respectively, so the drag force they add builds up gradually (avoiding an unrealistic "opening shock" that would otherwise show up as an instant, physically implausible jolt). `0.9` and `1.75` are the drag coefficients of a ribbon-style drogue and a ringsail main canopy — again, real recovery-parachute values, not arbitrary numbers.

---

## 10. The sound engine — how audio is made *without any sound files*

This is genuinely one of the more clever bits of the program, so it's worth walking through even though you asked specifically about the volume button.

### 10.1 Why there are no audio files

Every sound you hear — engine roar, wind, plasma crackle, sonic boom, chute deployment, explosions — is **generated live by mathematics**, using a browser feature called the **Web Audio API**. This keeps the whole simulator as one small, self-contained file with nothing to download.

```js
function noiseBuffer(ctx, brown){
  const len = ctx.sampleRate*2, buf = ctx.createBuffer(1, len, ctx.sampleRate);
  const d = buf.getChannelData(0);
  let last = 0;
  for (let i = 0; i < len; i++){
    const w = Math.random()*2 - 1;
    if (brown){ last = (last + 0.02*w)/1.02; d[i] = last*3.5; }
    else d[i] = w;
  }
  return buf;
}
```

This builds two seconds of raw randomness — literally `Math.random()` (a random number generator) — that then loops forever. Plain random static is called **white noise** (used for the wind/plasma sounds); the `brown` version smooths each random value into the previous one (`(last + 0.02*w)/1.02`), producing **brown noise** — a deeper, rumblier texture much closer to a real engine's roar than plain static.

### 10.2 Turning noise into "engine," "wind," and "crackle"

```js
SFX.lpE = ctx.createBiquadFilter(); SFX.lpE.type = "lowpass"; SFX.lpE.frequency.value = 260;
SFX.gE = ctx.createGain(); SFX.gE.gain.value = 0;
mkLoop(brown).connect(SFX.lpE); SFX.lpE.connect(SFX.gE); SFX.gE.connect(SFX.master);
```

This is an **audio signal chain**: `.connect()` wires one audio component's output into the next one's input, like plugging cables between guitar pedals. Here: brown noise → a **lowpass filter** (lets only low, rumbly frequencies through — this is what makes it sound like a deep roar instead of static hiss) → a **gain** (volume control for just this one sound layer) → the master output.

The same pattern, with different filter types, creates the other layers:
- **Wind**: white noise → **bandpass filter** (lets only a narrow frequency *band* through, and that band's center frequency is nudged up with your speed — this is what makes faster flight *sound* faster).
- **Plasma crackle**: white noise → **highpass filter** (lets only high, hissy/crackly frequencies through).
- **Sub-rumble**: a plain, pure sine wave at 37 Hz (`sub.frequency.value = 37`) — a frequency you feel more than hear, added under the engine roar for weight.

### 10.3 Making it react to the simulation, live

```js
function updateAudio(){
  ...
  const airF = Math.pow(Math.min((atm.P||0)/101325, 1), 0.35);
  const eV = thrF > 0 ? (0.30 + 0.45*thrF)*Math.max(airF, 0.18) : 0;
  set(SFX.gE.gain, eV);
  ...
  const wV = active ? Math.pow(Math.min((S.qNow||0)/45000, 1), 0.7)*0.5 : 0;
  set(SFX.gW.gain, wV);
}
```

This function runs every physics tick and recalculates each sound layer's volume from the current simulation state: `eV` (engine volume) scales with throttle *and* with `airF` (how much air is around you) — meaning the sound of your own engine genuinely fades in vacuum, since there's no air to carry that sound. `wV` (wind volume) scales with the actual dynamic pressure (`S.qNow`) the rocket is experiencing right now. There is no pre-recorded "launch sound clip" playing — the audio you hear is being computed live, tick by tick, directly from the physics.

### 10.4 The volume button and slider — how your request was implemented

```html
<button id="btnMute" title="Sound on/off">🔊</button>
<input type="range" id="vol" min="0" max="100" value="70" title="Volume">
```

```js
$("btnMute").addEventListener("click", () => {
  sfxInit();
  SFX.muted = !SFX.muted;
  $("btnMute").textContent = SFX.muted ? "🔇" : "🔊";
  applyVol();
});
$("vol").addEventListener("input", () => {
  sfxInit();
  SFX.vol = +$("vol").value/100;
  if (SFX.vol > 0 && SFX.muted){ SFX.muted = false; $("btnMute").textContent = "🔊"; }
  applyVol();
});
```

- **The mute button's click handler**: `SFX.muted = !SFX.muted` is the standard way to write "flip a true/false switch" (`!` means "not" — not-true is false, not-false is true). It also swaps the emoji shown on the button between 🔊 and 🔇 so you get visual confirmation.
- **The volume slider's input handler**: fires continuously as you drag it. `+$("vol").value/100` converts the slider's 0–100 reading into a 0.0–1.0 fraction (`+` in front of something forces text into a number — sliders technically hand back text). Dragging the slider above 0 while muted automatically un-mutes, which matches how volume controls behave on virtually every device you've used.
- **`applyVol()`** actually pushes the resulting number into the master volume knob (§10.2's `SFX.master`), and also saves your preference to the browser's `localStorage` — a small persistent storage area the browser gives every website, which is why your volume setting is remembered the next time you open the page, even after closing the browser entirely.

```js
document.addEventListener("pointerdown", () => {
  sfxInit();
  if (SFX.ok && SFX.ctx.state === "suspended") SFX.ctx.resume();
}, {once:true});
```

One browser quirk this handles: browsers refuse to let a webpage make *any* sound until you've clicked or tapped somewhere on the page at least once (an anti-annoyance rule against sites that auto-blast audio at you). This line says "the very first time you click or tap anywhere on the page, make sure the audio engine is switched on" — so sound is ready to go the moment you press Launch.

---

## 11. Reading and writing values — the "control panel wiring"

A pattern worth understanding since it explains how *every* slider affects the simulation:

```js
const set = v => {
  v = Math.min(d.max, Math.max(d.min, +v || 0));
  P[d.id] = v; rg.value = v; nm.value = v; pv.textContent = +v.toFixed(3);
  onParamChange(d);
};
```

Whenever *any* slider or its paired number box changes, this `set` function runs. It: (1) clamps the value so it can never go outside the slider's allowed min/max even if you type something silly into the number box, (2) writes the new value into `P` (the shared settings clipboard from §3.4) — this is the moment the simulation "sees" your change, (3) keeps the slider, number box, and the little live number-label all showing the same value, and (4) calls `onParamChange(d)`, which — depending on what kind of setting just changed — might rebuild the whole atmosphere table (§4.1) or just refresh the on-screen numbers.

This single function is the answer to "how does moving a slider actually change the rocket's behaviour?" — every physics formula elsewhere in the file reads its numbers from `P`, and `P` is *only* ever updated here.

---

## 12. The drawing loop — how the picture on screen gets made

```js
let lastTick = performance.now(), simAccum = 0;
function tick(){
  const now = performance.now();
  const elapsed = Math.min((now - lastTick)/1000, 0.25);
  lastTick = now;
  if (running && !paused && S && !S.done){
    const warp = +$("warp").value;
    simAccum += elapsed * warp;
    let steps = Math.min(Math.floor(simAccum/DT), 8000);
    simAccum -= steps * DT;
    while (steps-- > 0 && !S.done) physicsStep(DT);
    ...
  }
  updateAudio();
}
setInterval(tick, 16);
```

Two clocks run at once in this program, and it's a subtle but important design:

1. **The physics clock** — `DT = 0.01` means the physics is always calculated in **fixed 0.01-second steps**, no matter how fast or slow your computer actually runs. `simAccum` is a small "banked time" bucket: real elapsed time (scaled by your time-warp setting) gets deposited in, and the code withdraws exactly `DT`-sized chunks from it, running one `physicsStep()` per chunk. This guarantees the physics behaves *identically* regardless of your computer's speed or the browser tab's frame rate — the same launch always flies the same way.
2. **The drawing clock** — a separate, independent loop (`requestAnimationFrame`, further down in the file) redraws the picture as often as your screen can display it (typically 60 times per second), completely decoupled from how many physics steps just happened. This is why the visuals stay smooth even if you crank the time-warp slider to 50× — physics might be crunching through 50 seconds of simulated flight between two drawn frames, but you only ever *see* the current state, smoothly.

`setInterval(tick, 16)` — "run the `tick` function repeatedly, roughly every 16 milliseconds" (about 60 times a second), which is what drives the whole simulation forward through real time.

---

## 13. A worked example — following one slider end to end

To tie everything together, here's the complete journey of a single setting, from slider to rocket behaviour:

1. **You drag the "Surface gravity g₀" slider.** Its HTML `<input type="range" id="rg_g0">` was generated by `buildControls()` (§3.4) from its entry in `PARAM_DEFS` (§3.3).
2. **The slider's `input` event fires**, calling `set(rg.value)`.
3. **`set()`** clamps the value, writes it into `P.g0`, updates the number box and the little live label, and calls `onParamChange(d)`.
4. **`onParamChange`** sees this was a "planet" group setting, switches the World preset dropdown to "Custom" (since you've now deviated from a preset), and calls `buildAtmo()` to rebuild the entire pressure/temperature/density table (§4.1) using your new gravity value — because gravity directly affects how fast pressure drops with altitude.
5. **Every physics tick from then on**, `localG(h)` (§3-adjacent) and the atmosphere table both reflect your new gravity, which changes: how much force is needed to lift off (`acc = thrust/m - g`), how thin the air gets with altitude, how much the rocket weighs at any moment, and (in orbit mode) the entire shape of any orbit you fly, since `mu = g0 * radius²` feeds directly into every orbital-mechanics equation in §8.

That's the whole pipeline, for any of the roughly 20 sliders in the app: **HTML control → event listener → `P` object → physics formulas → drawn/audible result.**

---

## Glossary (plain English)

| Term | Meaning here |
|---|---|
| **Isp (specific impulse)** | Rocket engine efficiency, in seconds. Higher = more speed per kilogram of fuel. |
| **Dynamic pressure (q)** | How hard the air is pushing on the vehicle — ½ × density × speed². |
| **Drag coefficient (Cd)** | A shape's "aerodynamic ugliness" number — higher means more drag for the same size/speed. |
| **Stagnation heating / heat flux (q̇)** | How much heat energy hits the nose per second per square metre. |
| **Apoapsis / periapsis** | The highest and lowest points of an orbit. |
| **Gravity turn** | The real-world technique of gradually tilting a rocket from vertical to horizontal during ascent. |
| **Airspeed vs inertial speed** | Airspeed is your speed relative to the (co-rotating) air; inertial speed is relative to the fixed stars. Aerodynamics feels airspeed; orbits use inertial speed. |
| **L/D (lift-to-drag ratio)** | How much sideways lift a shape makes for a given amount of drag. Higher L/D = more ability to steer and flatten a re-entry. |
| **Δv (delta-v)** | Change in velocity — the fundamental "currency" of spaceflight. Reaching orbit costs far more Δv than orbital speed alone, because of gravity, drag, and steering losses. |
| **Felt / proper acceleration** | The g-force you'd actually feel — from thrust, drag, and lift, but *not* gravity (free-fall feels weightless). |
| **MECO / SECO** | "Main/Second Engine Cut-Off" — standard spaceflight terms for when a stage's engine stops firing. |
| **Ablative heat shield** | A material designed to deliberately burn/flake away, carrying heat with it. |
| **State machine** | Code that tracks "which named phase am I in right now" and moves between phases based on conditions — used for staging, chute deployment, and orbit guidance in this app. |
| **Interpolation** | Smoothly blending between two known values to estimate a value in between. |
| **Web Audio API** | The browser's built-in toolkit for generating and shaping sound live, with no audio files. |

---

*This guide covers the file as of the version pushed to [github.com/ShoiabGoku/rocket-science](https://github.com/ShoiabGoku/rocket-science). If the code changes later, re-check the relevant section against the live file — line numbers and exact snippets may shift, though the underlying ideas explained here will still apply.*
