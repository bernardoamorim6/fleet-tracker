# KiCad + JLCPCB PCB Design Guide — Truck Telematics

This guide covers designing the truck telematics PCB in KiCad for manufacture and assembly by JLCPCB. It assumes you have KiCad installed (version 8 or newer) and a working knowledge of the schematic you already have.

The design philosophy:
- **All SMD parts** except a handful where through-hole is mechanically necessary or where the best available part is only through-hole (terminal block, fuse holder, female headers for LilyGo, buck converter module).
- **JLCPCB machine-assembles the SMD side** on one board face. For the through-hole parts, either use JLCPCB's through-hole assembly service (small extra fee per insertion) or hand-solder them after receiving the boards.
- **Design around JLCPCB's assembly library** so every SMD part is a part they can actually load into their pick-and-place machines.
- **Verify every part against jlcpcb.com/parts live** during the schematic entry phase — stock changes, part numbers get reassigned, and Chinese clones are common. See Part 2's "How to verify an LCSC listing" section.

Total time from starting KiCad to sending files to JLCPCB, for a first-time PCB designer: roughly **3–5 working days** (schematic 1 day, layout 2–3 days, verification and file export half a day).

---

## Part 1 — KiCad setup for JLCPCB workflow

### Install KiCad

Download KiCad 8 (or newer) from kicad.org. Install with default options. Launch KiCad — you'll see the project manager with buttons for the schematic editor, PCB editor, symbol/footprint editors, etc.

### Install the kicad-jlcpcb-tools plugin

There's a KiCad plugin called **kicad-jlcpcb-tools** (by Bouni; also sometimes listed simply as "JLCPCB Tools") that will help you at the PCB-layout stage. It does not help during schematic entry — that limitation matters, so plan accordingly (see Part 2).

What the plugin does, and when:
- **Runs only in the PCB editor**, not the schematic editor
- Reads the LCSC part numbers you tagged onto components during schematic entry
- Verifies each LCSC number against JLCPCB's current stock and library
- Flags parts that are out of stock or no longer assembly-eligible
- Generates the correct BOM CSV and CPL (pick-and-place) files in JLCPCB's expected format
- Provides a JLCPCB-integrated library browser for searching parts by LCSC number

To install:
1. In KiCad, open the PCB editor (open any PCB file, or create an empty one if you don't have one yet).
2. Go to `Tools → Plugin and Content Manager`.
3. Click "Plugins" tab, find "JLCPCB Tools" or "kicad-jlcpcb-tools" (by Bouni), install it.
4. Restart KiCad.
5. You'll see a new icon in the PCB editor toolbar — not the schematic editor.

**What this means for the schematic-entry phase (Part 2):** you'll verify parts manually against JLCPCB's website while placing symbols, tagging each one with its LCSC number as a custom field. The plugin then takes over once you're in PCB layout. This is a slightly annoying workflow gap in the KiCad ecosystem — the plugin doesn't do schematic-time verification — but it's manageable if you know about it upfront.

### Configure JLCPCB design rules

You set these once, in the PCB editor, before you start routing. You don't need to do this on Day 1 — the schematic editor doesn't need these settings — but it's a useful thing to know about now so you can do it on the day you create the PCB (Day 3 of the timeline below).

When you open the PCB editor for the first time:

- `File → Board Setup → Design Rules → Constraints`
- Set **minimum track width**: 0.15 mm (JLCPCB's default capability is 0.127 mm but 0.15 mm gives you margin)
- Set **minimum via diameter**: 0.4 mm
- Set **minimum via drill**: 0.2 mm
- Set **minimum clearance**: 0.15 mm
- Set **minimum hole to hole clearance**: 0.5 mm

These match JLCPCB's standard capabilities. Anything below these costs extra or isn't offered.

---

## Part 2 — Schematic entry

You already have a schematic in KiCad from the earlier work. If you're starting fresh or need to redraw:

### Create the project

`File → New → Project`. Give it a sensible name like `truck_telematics_v1`. This creates a folder with the schematic (.kicad_sch) and PCB (.kicad_pcb) files.

### How to verify an LCSC listing — the critical skill

Before diving into component placement, learn to read a JLCPCB parts listing carefully. This is the single most important skill for this project — we've already had multiple cases (during the design of this guide) where the "obvious" listing was a clone or wrong variant.

**When you search jlcpcb.com/parts by manufacturer part number, you'll typically see multiple results. Check these five things on every candidate:**

**1. Manufacturer field (biggest red flag if wrong)**

The manufacturer field must match the real manufacturer of the part you're searching for.

- If you search `TJA1051T/3` and see one result manufactured by "NXP Semicon" and another by "Tokmas" — the Tokmas one is either a clone or a relabel. Skip it, even if it's cheaper.
- If you search `R-78E5.0-1.0` and see results by "YLPTEC" or "EVISUN" — those are Chinese clones of Recom's design. The real Recom listing will say "Recom Power" or "Recom".
- Trust genuine listings only: NXP Semicon, onsemi, TI (Texas Instruments), TDK, Murata, Nichicon, Recom Power, Traco Power, Panasonic, Omron, Nexperia, Bourns, Diotec, Littelfuse, Vishay, Yageo, Samsung, etc.
- Chinese-brand-but-legitimate-industrial: MornSun, XLSEMI, Silergy, Chipsea. These are real manufacturers with their own IP, not clone shops. Fine to use.
- Red flag names: "original brand", generic-sounding manufacturer names, or a manufacturer whose website you can't find with a quick search.

**2. Manufacturer Part Number (MPN) consistency**

The MPN in the listing header, the MPN in the details table, and the MPN you searched for should all match exactly. If they don't:
- The listing might be lazily copy-pasted from a template (suggests the vendor doesn't really know what they're shipping)
- The listing might be a relabel (they're selling one part under another part's number)

We saw this earlier with a NUP2105L listing that had "SMBJ200A" in the MPN field — the seller had populated their listing sloppily, and there's no way to know what actually ships.

**3. Stock quantity**

- **>1000 units**: safe to use, unlikely to run out during your order processing
- **100–1000 units**: acceptable but pay attention; large stock swings can happen
- **<100 units**: risky for a 5-board prototype run. JLCPCB might not be able to fulfill your order and will either delay it or substitute. Only use if there are no better alternatives.
- **"Consign Parts" with 0 stock**: don't use. This means someone consigned inventory to JLCPCB but it's currently empty.

**4. Basic vs Extended**

- **Basic parts** are always loaded in JLCPCB's machines, no setup fee. Common passives (many 0603/0805 resistors, X7R caps) fall in this category.
- **Extended parts** require a one-time ~€3 setup fee per unique extended part (they load the reel into a machine for your job). Most of your BOM will be Extended.
- Neither category is "better" — Extended is fine. Just budget for the setup fees (~€30-40 total for a design like yours).

**5. Package matches your KiCad footprint**

If your KiCad footprint expects SOIC-8 and the listing shows SOP-8, verify these are the same physical package (they usually are — SOIC and SOP are used interchangeably). But watch for:
- SOIC-8 vs SOIC-8-150mil vs SOIC-8-208mil (different body widths)
- SOT-23 vs SOT-23-3 vs SOT-23-6 (different pin counts)
- 0603 vs 0603 (imperial) vs 0603 (metric) — imperial 0603 = metric 1608, imperial 0402 = metric 1005; KiCad uses metric names, most datasheets use imperial

**Always download the datasheet PDF from the listing and verify the mechanical drawing before committing to a KiCad footprint.**

### Verify footprint dimensions against the datasheet

This is a bigger deal than it sounds. Every time you assign a KiCad footprint to a component, you must verify that footprint's pad dimensions and spacing match the actual part.

**Why this matters:**
- KiCad's built-in footprints are generic best-guesses. They usually match common parts but not always exactly.
- Two parts with the same package name can have subtly different footprints (e.g., different fuse holders both called "5×20 mm" can have pin spacings of 22 mm or 25 mm).
- A footprint mismatch means either the part won't fit the board, or JLCPCB's machine will place it in the wrong position and short something.

**How to verify:**
1. Download the part's datasheet PDF from the LCSC/JLCPCB listing.
2. Find the "Recommended Land Pattern" or "PCB Footprint" section in the datasheet — this shows the exact pad sizes and positions the manufacturer recommends.
3. In KiCad, open the footprint editor and view the footprint you plan to use.
4. Check that pad positions and sizes match. Pay particular attention to:
   - **Pin spacing** (pitch)
   - **Pad size** (rectangular pads have length × width)
   - **Overall footprint bounding box** (does the part fit where you expect it to?)
   - **Silkscreen outline** (should match the actual part's body outline)

**When they don't match:** you have three options:
- Modify the existing footprint in the footprint editor to match the datasheet
- Create a new footprint from scratch using the datasheet's land pattern
- Search for a different footprint variant that already matches

**Common gotchas per component type:**
- **Fuse holders:** pin spacing varies a lot (22-25 mm) even for "5×20 mm" holders. Always verify.
- **Terminal blocks:** verify pin pitch (2.54, 3.5, 3.81, 5.0, 5.08 mm all exist) AND the number of pins.
- **DC-DC modules:** pad size and spacing varies between manufacturers even for the same functional part. A Traco TSR footprint is NOT interchangeable with a Recom R-78 footprint.
- **Common-mode chokes:** the 4-pad SMD footprint varies by manufacturer. Verify against the datasheet's land pattern drawing.
- **Passives (0402, 0603, 0805, 1206):** these are standardized; KiCad's built-in footprints match. But make sure you're using the metric name (1608 for imperial 0603, etc.) consistently.

**Order the physical part before finalizing the footprint if in doubt.** For a €0.50 part, waiting a week for it to arrive so you can measure with calipers is worth it compared to a €400 wasted board respin.

### Component-by-component schematic guide

For each part on your BOM, here's what to place and any specific notes:

**U1 — LilyGo T-SIM7080G-S3 socket footprint (not the LilyGo itself)**
- What you place in the schematic: two 1×16 pin header symbols. Place them from your `telematics` library (fetched from LCSC C350305 via easyeda2kicad) — symbol and footprint come pre-paired. If for some reason you want to use KiCad's built-in generic symbol instead, it's `Connector_Generic:Conn_01x16`, but then you'd have to manually assign the footprint and LCSC field. Prefer the telematics library version.
- Label the pins matching the LilyGo's actual pinout (GPIO16, GPIO17, GPIO18, 5V, 3.3V, GND, etc.).
- These represent the female headers on your board that the LilyGo plugs into.
- The LilyGo itself is not a component on your PCB — it's an off-board module.

### Sourcing KiCad symbols and footprints

**The unified approach: fetch every part from easyeda2kicad and use those for everything.**

Every component you need on your BOM has an LCSC part number. easyeda2kicad's `--full` flag fetches:
- **Schematic symbol** (for the schematic editor)
- **PCB footprint** (for the PCB editor)
- **3D model** (for the 3D viewer)
- **LCSC part number pre-tagged as a custom field** (for BOM export)

All matched to the exact part JLCPCB will install. This means when you place a symbol in your schematic:
- The correct footprint is already associated
- The LCSC field is already populated
- The pin numbering matches the physical part
- Everything is ready for BOM/CPL export

**Don't mix easyeda2kicad parts with KiCad's built-in symbols.** It's tempting because KiCad's built-ins are visually prettier, but:
- You'd have to manually assign a footprint to every built-in symbol
- You'd have to manually add the LCSC field to every built-in symbol
- Pin numbering could mismatch between built-in symbols and easyeda2kicad footprints
- Verification during BOM export becomes harder

**Use easyeda2kicad for every part.** Even generic resistors and capacitors. It's less work and less error-prone.

#### Installation on NixOS

`easyeda2kicad` is packaged in nixpkgs. Run:

```bash
nix-shell -p easyeda2kicad --run "easyeda2kicad --help"
```

If that works, you're set. No need to install anything permanently.

#### Fetching your BOM

Once you have your final BOM with confirmed LCSC C-numbers, fetch them all in one command:

```bash
mkdir -p ~/kicad-parts

LCSC_IDS="C1591 C89632 C78420 C10756 C14486 C3131 C88056 C2912579 C98220 C350305 C38695 C5346721"

nix-shell -p easyeda2kicad --run "easyeda2kicad --lcsc_id $LCSC_IDS --full --output ~/kicad-parts/telematics --use-cache"
```

This creates:
- `~/kicad-parts/telematics/telematics.kicad_sym` (the symbol library)
- `~/kicad-parts/telematics/telematics.pretty/` (the footprint library)
- `~/kicad-parts/telematics/telematics.3dshapes/` (the 3D models)

**Register these in KiCad** (see instructions below):
- Symbol library: `Preferences → Manage Symbol Libraries → +`
- Footprint library: `Preferences → Manage Footprint Libraries → +`

After registration, every part on your BOM is available in KiCad's symbol chooser under the `telematics` library, with the correct footprint pre-associated and LCSC field pre-populated.

#### Verifying each footprint against the datasheet

Even though easyeda2kicad usually produces correct output, **always verify each footprint against the manufacturer's datasheet** before finalizing your PCB. Occasional issues:
- Wrong pin numbering (rare but possible)
- Slightly off pad sizes
- Missing silkscreen outlines

Five minutes per part × 12 parts = one hour of verification, saves you from a €300+ prototype respin.

**The parts most worth checking carefully:**
- **L1 (ACT45B choke)** — 4-pad SMD footprint, uncommon package
- **U3 (Traco buck)** — through-hole SIP, verify pin spacing
- **J1 (DORABO terminal block)** — verify 5.08 mm pitch
- **F1 holder** — verify pin spacing (varies by holder)

For standardized packages (0603, 1210, SOIC-8, SOT-23) easyeda2kicad's output is almost always correct because the geometry is industry-standardized.

#### Fallback if easyeda2kicad fails for a specific part

If easyeda2kicad can't fetch a specific part (occasionally happens if LCSC's EasyEDA model is missing or malformed), your fallbacks in order:

1. **SnapEDA** (https://www.snapeda.com) — free with login, high-quality symbols for well-known Western brands
2. **Ultra Librarian** (https://ultralibrarian.com) — similar, sometimes has parts SnapEDA doesn't
3. **KiCad built-in libraries** — for truly standardized packages (0603, SOIC-8, etc.), the built-in symbol works. Manually associate it with the easyeda2kicad footprint and add the LCSC field.
4. **Create manually in KiCad's Symbol/Footprint Editor** — last resort, but doable in 15-30 minutes per part following the datasheet's recommended land pattern

### Component-by-component schematic guide (final BOM)

The BOM below is what's been confirmed available and correct at LCSC/JLCPCB. For each part, use the specific LCSC C-number listed — this is what you'll fetch with easyeda2kicad and what appears on the BOM you send JLCPCB. **Verify each listing's manufacturer field before ordering** (see the counterfeit-detection guidance in the previous section).

**Since you're using easyeda2kicad, the schematic symbol AND footprint come pre-paired for every part.** You don't need to manually assign footprints or search KiCad's built-in libraries. Just place the symbol from your `telematics` library in the schematic; the correct footprint is already associated.

The KiCad footprint names I mentioned in previous versions of this guide (e.g., `Package_SO:SOIC-8_3.9x4.9mm_P1.27mm`) are informational only — they're what the easyeda2kicad-fetched footprint should look like, useful as a sanity check when you verify the footprint against the datasheet. But you don't need to manually assign them.

---

**U1 — LilyGo T-SIM7080G-S3 (external module, plugged into sockets on your PCB)**
- Not a component in your BOM — it's a full module, not JLCPCB-assembled
- Order 3 LilyGo boards from LilyGo's official AliExpress store for measurement and prototyping
- Represented in your schematic by J2 and J3 (the two female headers it plugs into — see below)

**J2, J3 — 1×16 female pin headers, 2.54 mm pitch (the LilyGo socket)**
- LCSC: **C350305** (or verify current stock at LCSC search: `2.54mm 1x16 female pin header single row through hole`)
- Fetch with easyeda2kicad — symbol and footprint come paired
- Quantity: 6 total (2 headers per PCB × 3 PCBs)
- **Basic part** (widely stocked)
- This is what the LilyGo actually plugs into. JLCPCB installs these onto your PCB; you plug the LilyGo into them by hand when boards arrive.
- Physical layout note: place J2 and J3 on your PCB at the LilyGo's exact row-to-row spacing (verify with calipers when your LilyGo arrives, expected ~22.86 mm)

**U2 — TJA1051T/3/1J CAN transceiver**
- MPN: **TJA1051T/3/1J**
- LCSC: **C38695** (NXP Semicon, SOIC-8-150mil)
- Alternative if primary is out of stock: `C58988` (TJA1051T/3,118, NXP Semicon)
- Expected footprint from easyeda2kicad: SOIC-8, ~3.9×4.9 mm body, 1.27 mm pitch
- Extended part
- **Verify manufacturer field shows "NXP Semicon"** — reject Tokmas or other manufacturers.
- Quantity: 3

**U3 — Traco TSR 1-2450E buck regulator (through-hole SIP)**
- MPN: **TSR 1-2450E**
- LCSC: **C5346721** (TRACO POWER)
- Package: SIP through-hole module, 11.6 × 7.5 mm, 3 pins (VIN, GND, VOUT)
- Input: 7-36V, Output: 5V, Output current: 1A, Efficiency: 92%
- Expected footprint from easyeda2kicad: 3-pin SIP with ~5 mm pin spacing. **Verify pin spacing against the datasheet.**
- Extended part
- Through-hole — enable JLCPCB's through-hole assembly service when ordering (see Part 7)
- **Verify manufacturer field shows "TRACO POWER"** — reject YLPTEC/EVISUN clones.
- Quantity: 3

**J1 — DORABO DB128V-5.08-5P-GN-S terminal block (through-hole)**
- MPN: **DB128V-5.08-5P-GN-S**
- Manufacturer: DORABO — legitimate for mechanical parts, fine to use
- 5 positions, 5.08 mm pitch, 300V/16A rated (massively over-spec), M2.5 screws, wire gauge 12-22 AWG
- Expected footprint from easyeda2kicad: 5-position screw terminal, 5.08 mm pitch, through-hole. **Verify pin position and hole diameter against the DORABO datasheet.**
- Through-hole
- Note: this exposes only the 5 FMS pins your device uses (GND, CAN_L, CAN_H, IGN, +24V_perm). The 12-pin FMS-side connector lives on your external wire harness — see "Truck-side installation topology" section.
- Quantity: 1 per board

**F1 — 5×20 mm cartridge fuse holder + 1A fuse (through-hole)**
- Holder MPN: **5×20 BLX-A Type Fuse holder XC-7** (Xucheng Elec)
- Fuse: 5×20 mm glass cartridge, 1A fast-blow, 250V — sold separately (search LCSC for `5x20mm glass fuse 1A fast`)
- Expected footprint from easyeda2kicad: 2 through-holes ~22-25 mm apart. **Verify pin spacing against Xucheng Elec datasheet.**
- Manufacturer: Xucheng Elec is fine for this mechanical part
- Order 10 spare fuses since bring-up may blow some
- Quantity: 1 holder per board, 10 fuses total (for prototypes + spares)

**D1 — SMBJ33CA (main TVS on 24V input)**
- MPN: **SMBJ33CA**
- LCSC: search `SMBJ33CA` — verify manufacturer is Littelfuse, Diotec, or similar reputable brand
- Package: SMB (DO-214AA)
- Bidirectional TVS, 33V standoff, 400W peak power
- Expected footprint from easyeda2kicad: DO-214AA / SMB
- Extended part
- Quantity: 10 (order extras; TVS on truck input will absorb spikes and may eventually degrade)

**D2 — SMAJ5.0CA (GPIO TVS on ignition sense pin)**
- MPN: **SMAJ5.0CA**
- LCSC: search `SMAJ5.0CA` — verify manufacturer is a reputable brand
- Package: SMA (DO-214AC)
- Bidirectional TVS, 5.0V standoff, ~9.2V clamp voltage
- Expected footprint from easyeda2kicad: DO-214AC / SMA
- Extended part
- **Note on the change from SMAJ3.3CA:** the SMAJ5.0CA has a slightly higher standoff (5.0V vs 3.3V) and clamps at ~9.2V (vs ~8V). Both are safe protection for the ESP32 GPIO because the voltage divider keeps normal operation well below either threshold. The SMAJ5.0CA is often more available at LCSC than the SMAJ3.3CA.
- Quantity: 10

**D3 — NUP2105LT1G (CAN bus TVS array)**
- MPN: **NUP2105LT1G**
- LCSC: search `NUP2105LT1G onsemi` — verify manufacturer is "onsemi"
- Package: SOT-23 (3-pin)
- Bidirectional 2-channel CAN TVS
- Expected footprint from easyeda2kicad: SOT-23, 3-pin
- Extended part
- Quantity: 5

**L1 — ACT45B-101-2P-TL003 (CAN common-mode choke)**
- MPN: **ACT45B-101-2P-TL003**
- LCSC: search `ACT45B-101-2P-TL003 TDK` — verify manufacturer is "TDK"
- 4-pad SMD, 4.5×3.2 mm, 100 µH common-mode inductance, AEC-Q200 automotive-grade
- Expected footprint from easyeda2kicad: 4 pads in a 1.6×3.4 mm rectangle. **Verify footprint dimensions carefully against the ACT45B datasheet** — the 4-pad SMD package is uncommon and worth double-checking.
- Extended part
- Quantity: 5

**C1, C2 — 100 nF decoupling capacitors (SMD, 0603)**
- MPN: **CL10B104KB8NNNC** (Samsung Electro-Mechanics, X7R, 50V)
- LCSC: **C1591**
- Expected footprint from easyeda2kicad: 0603 (1608 metric)
- Basic part (no setup fee)
- Quantity: 50 (comes in strips; used for TJA1051 decoupling)

**C3, C4 — 10 µF capacitors (SMD, 1210)**
- MPN: **CL31B106KBHNNNE** (Samsung Electro-Mechanics, X7R, 50V, 1210 package)
- LCSC: **C89632**
- Expected footprint from easyeda2kicad: 1210 (3225 metric) — note this is larger than the 1206 you may have originally planned
- 50V rating is critical for C3 (buck input, sees truck transients). C4 (output) could use lower voltage but 50V keeps the BOM simple.
- Extended part
- Quantity: 20

**R1 — 100 kΩ resistor (SMD, 0603)**
- MPN: **HoAR0603-1/10W-100KR-1%-TCR25**
- LCSC: **C2912579**
- Alternative: `RC0603FR-07100KL` (Yageo) if the primary is out of stock
- Expected footprint from easyeda2kicad: 0603 (1608 metric)
- Basic part
- Quantity: 50

**R2 — 10 kΩ resistor (SMD, 0603)**
- MPN: **RC0603FR-0710KL** (Yageo)
- LCSC: **C98220**
- Expected footprint from easyeda2kicad: 0603 (1608 metric)
- Basic part
- Quantity: 100

**J_BAT1 — LiPo battery connector (NOT needed on your PCB)**
- The LilyGo has its own JST-PH 2.0 mm connector for the LiPo. The LiPo connects directly to the LilyGo, not to your board.
- **Skip this on your PCB.** No BOM entry needed.

---

## Part 2b — What connects to where (schematic wiring)

This section spells out every connection in the schematic — what pin on each part connects to what pin on which other part. Use this as your reference when drawing the schematic in KiCad.

### Nets (named signals) used in this design

Give each of these a clear net name in KiCad so they're easy to trace:

| Net name | Description | Voltage |
|---|---|---|
| `GND` | Common ground reference | 0 V |
| `+24V_TRUCK` | Raw truck power from FMS pin 12 | 24 V nominal (12-30 V with spikes) |
| `+24V_FUSED` | Truck power downstream of F1 fuse | 24 V nominal |
| `+5V` | Regulated 5V from buck output | 5.0 V |
| `+3V3` | 3.3V rail sourced from LilyGo's 3V3 output pin | 3.3 V |
| `IGN_RAW` | Raw ignition signal from FMS pin 10 | 0 V or 24 V |
| `IGN_SENSE` | Divided ignition signal to GPIO | 0 V or ~2.2 V |
| `CAN_H_RAW` | CAN high from FMS pin 7 | Differential signal |
| `CAN_L_RAW` | CAN low from FMS pin 6 | Differential signal |
| `CAN_H_FILT` | CAN high after common-mode choke | Differential signal |
| `CAN_L_FILT` | CAN low after common-mode choke | Differential signal |
| `CAN_TXD` | UART TX from ESP32 to TJA1051 (unused but must be connected) | 3.3V logic |
| `CAN_RXD` | UART RX from TJA1051 to ESP32 | 3.3V logic |

### Detailed connection list — every wire in the schematic

#### Power input chain (truck power → 5V rail)

| From | To | Net |
|---|---|---|
| J1 pin 1 (labeled GND on your terminal block) | GND net | `GND` |
| J1 pin 5 (labeled +24V) | F1 fuse holder, pin 1 | `+24V_TRUCK` |
| F1 fuse holder pin 2 | D1 (SMBJ33CA) pin 1, C3 (10µF) pin 1, U3 (Traco TSR) VIN pin | `+24V_FUSED` |
| D1 (SMBJ33CA) pin 2 | GND | `GND` |
| C3 (10µF) pin 2 | GND | `GND` |
| U3 (Traco TSR) GND pin | GND | `GND` |
| U3 (Traco TSR) VOUT pin | C4 (10µF) pin 1, LilyGo VSYS/5V input pin | `+5V` |
| C4 (10µF) pin 2 | GND | `GND` |

**Layout notes:**
- D1, C3 must be physically adjacent to U3's input pin. C3 within 5mm of VIN. D1 can be slightly further.
- C4 must be physically adjacent to U3's output pin. Within 5mm.
- All GND connections join the ground pour.

#### Ignition sense chain (24V pin → 3.3V GPIO)

| From | To | Net |
|---|---|---|
| J1 pin 4 (labeled IGN) | R1 (100kΩ) pin 1 | `IGN_RAW` |
| R1 (100kΩ) pin 2 | R2 (10kΩ) pin 1, D2 (SMAJ5.0CA) pin 1, LilyGo ignition-sense GPIO pin | `IGN_SENSE` |
| R2 (10kΩ) pin 2 | GND | `GND` |
| D2 (SMAJ5.0CA) pin 2 | GND | `GND` |

**Divider math check:** with R1=100kΩ, R2=10kΩ, IGN_RAW at 24V → IGN_SENSE = 24 × 10/(100+10) = 2.18V. Safe for ESP32 GPIO. If IGN_RAW spikes to 33V → IGN_SENSE = 3.0V (still under 3.3V limit).

**Layout notes:**
- D2 should be physically adjacent to the LilyGo GPIO pin, not near the divider — it protects the pin at its point of entry.
- R1 and R2 can be anywhere in between.

#### CAN bus chain (FMS CAN pins → TJA1051 UART)

Signal path: **J1 → D3 (TVS protection) → L1 (common-mode filter) → U2 (transceiver)**

| From | To | Net |
|---|---|---|
| J1 pin 3 (labeled CAN_H) | D3 (NUP2105L) pin 3, L1 (ACT45B) pin 1 | `CAN_H_RAW` |
| J1 pin 2 (labeled CAN_L) | D3 (NUP2105L) pin 1, L1 (ACT45B) pin 2 | `CAN_L_RAW` |
| D3 (NUP2105L) pin 2 | GND | `GND` |
| L1 (ACT45B) pin 4 (winding 1 out) | U2 (TJA1051) pin 7 (CANH) | `CAN_H_FILT` |
| L1 (ACT45B) pin 3 (winding 2 out) | U2 (TJA1051) pin 6 (CANL) | `CAN_L_FILT` |

**CRITICAL — ACT45B pin mapping:**
- Pin 1 = CAN_H input side (winding 1 start)
- Pin 4 = CAN_H output side (winding 1 end)
- Pin 2 = CAN_L input side (winding 2 start)
- Pin 3 = CAN_L output side (winding 2 end)

The two windings (1↔4 and 2↔3) must each carry ONE signal only. Crossing them (e.g., CAN_H entering pin 1 but exiting pin 3) means the choke doesn't work — verify carefully against the ACT45B datasheet circuit diagram before finalizing.

**Layout notes:**
- Route CAN_H and CAN_L as a **differential pair** — parallel traces, 0.2 mm gap, same length. From J1 → D3 → L1 → U2, keep them tightly coupled.
- **D3 sits as close to J1 as physically possible** — its job is to absorb transients before they can damage anything else, including L1
- L1 sits between D3 and U2, filtering the (now-safe-voltage) signal on its way to the transceiver

#### TJA1051 transceiver — full pinout wiring

| U2 pin | Name | Connect to | Net |
|---|---|---|---|
| 1 | TXD | LilyGo GPIO pin (CAN_TXD, e.g., GPIO18) | `CAN_TXD` |
| 2 | GND | Ground pour | `GND` |
| 3 | VCC | C1 (100nF) pin 1, +5V rail | `+5V` |
| 4 | RXD | LilyGo GPIO pin (CAN_RXD, e.g., GPIO17) | `CAN_RXD` |
| 5 | VIO | C2 (100nF) pin 1, S pin (pin 8), LilyGo 3V3 output pin | `+3V3` |
| 6 | CANL | L1 output side pin 3, D3 pin 1 | `CAN_L_FILT` |
| 7 | CANH | L1 output side pin 4, D3 pin 3 | `CAN_H_FILT` |
| 8 | S (Silent) | Tied to VIO net (`+3V3`) | `+3V3` |

**Decoupling caps:**
- C1 (100nF) sits across pin 3 (VCC) and pin 2 (GND) — must be within 2 mm of the pins
- C2 (100nF) sits across pin 5 (VIO) and pin 2 (GND) — must be within 2 mm of the pins

**Note on Silent (S) pin:** tying pin 8 to VIO (3.3V) enables listen-only mode. The transceiver physically cannot transmit — this is exactly what you want since you're passively observing the truck's CAN bus.

**Note on TXD pin:** even though we're in listen-only mode, pin 1 must be connected to a defined logic level (a GPIO configured as an output). The chip's internal design expects both TXD and RXD wired.

#### LilyGo socket connections (which header pins connect to your PCB)

You'll socket the LilyGo via two 1×16 female headers. Only some of the header pins carry signals from your PCB; the rest are unconnected (the LilyGo brings out lots of GPIOs you don't use, and that's fine).

**LilyGo pins you connect on your PCB:**

| LilyGo header pin | Your PCB net | Purpose |
|---|---|---|
| VSYS (see note) | `+5V` | Power input to LilyGo — from your buck's 5V rail |
| GND (any of several GND pins) | `GND` | Common ground |
| 3V3 (output pin) | `+3V3` | Sourced FROM the LilyGo; supplies TJA1051 VIO |
| GPIO16 (ignition sense) | `IGN_SENSE` | Ignition-sense input |
| GPIO17 (CAN RXD) | `CAN_RXD` | From TJA1051 pin 4 |
| GPIO18 (CAN TXD) | `CAN_TXD` | To TJA1051 pin 1 |

**All other LilyGo header pins on your PCB: leave unconnected** (no trace, no via). The LilyGo will still boot and function normally — those pins just have nothing to do on your board.

**Note on VSYS as power input:** based on your GitHub issue to LilyGo, this may or may not be supported. Design the PCB with two options:
1. **Primary:** VSYS pin on header connects to `+5V` net
2. **Fallback:** a solder pad on your PCB labeled "5V TO LILYGO" that connects to `+5V`, positioned near the LilyGo footprint. If VSYS turns out not to work, you can bodge a wire from this pad to the H606 solder point on the LilyGo (per LilyGo's Tip #18).

Once you get the response from LilyGo's GitHub, you can commit to one approach and remove the fallback. Until then, having both means you're safe either way.

**Note on GPIO choice:** these three GPIOs (16, 17, 18) are camera pins on the LilyGo. Since you're not using a camera, they're free per LilyGo's Tip #11: *"If the camera is not connected, all IO ports are available."* GPIO16 is VSYNC, GPIO17 is HREF, GPIO18 is Reset — all input-capable and unused when the camera isn't populated.

#### LiPo battery connection

The LiPo battery plugs directly into the LilyGo's onboard JST-PH connector — nothing on your PCB. The AXP2101 PMU on the LilyGo handles charging, switchover, and low-voltage cutoff automatically.

**No wiring needed on your PCB for the battery** other than the +5V input (which the PMU uses to charge the LiPo).

---

## Part 2c — Truck-side installation topology

Your PCB has a 5-position terminal block. The truck's FMS port has 12 pins. The mapping happens in the external harness:

**Harness wiring (truck side to your terminal block):**

| FMS pin | Signal | → | Your J1 pin |
|---|---|---|---|
| Pin 1 | GND | → | J1 pin 1 |
| Pin 6 | CAN_L | → | J1 pin 2 |
| Pin 7 | CAN_H | → | J1 pin 3 |
| Pin 10 | Ignition | → | J1 pin 4 |
| Pin 12 | +24V permanent | → | J1 pin 5 |

Pins 2, 3, 4, 5, 8, 9, 11 of the FMS connector are unused (they carry tachograph driver-card data etc. that you don't need).

### Coexisting with Webfleet (existing tracker on the same truck)

If a Webfleet device is already installed on the truck's FMS port, your device needs to coexist without disturbing it. Since CAN is a shared bus and your TJA1051 is in listen-only mode, this works fine — you just need a physical way to tap into the same wires as Webfleet.

**Recommended: Y-splitter harness.** A cable that plugs into the truck's FMS port and has two branches on the other end:
- One branch → Webfleet's connector
- One branch → 5 bare wires terminating into your PCB's J1 terminal block

Search AliExpress for: `FMS Y splitter harness` or `truck 12-pin CAN Y cable`.

**Critical rule regardless of Y-splitter or direct tap:** your PCB must NOT include a 120 Ω CAN termination resistor. The truck's bus already has proper termination at its endpoints; adding more terminators degrades the bus for all devices. Your listen-only design is correct — high-impedance passive tap.

---

## Part 2d — Building the schematic in KiCad (step by step)

This section walks you through wiring the full schematic in KiCad. Follow it top-to-bottom — parts are placed and wired in the order they'd physically be assembled: power input first, then power conversion, then signal processing, then the microcontroller socket.

**Before you start:**
- Your `telematics` library is registered in KiCad (symbol + footprint)
- You have the schematic editor open on a fresh sheet
- Keep this guide open on a second screen or printed out — you'll reference it constantly

**Conventions used below:**
- Every wire is described as **"part X pin Y → part Z pin W"** — no assumed connections
- All wires are drawn using KiCad's Wire tool (hotkey `W`) unless noted otherwise
- Net labels (green tags with names like `+5V`, `GND`) are added using the Net Label tool (hotkey `L`)
- Global power symbols (like the GND triangle or `+5V` arrow) come from the Add Power Port tool (hotkey `P`)

---

### Step 1 — Set up the sheet

1. Open the schematic editor for your project
2. `File → Page Settings` — fill in Title ("Truck Telematics v1"), Date, Rev ("1.0"), your Company name
3. Sheet size: A4 landscape is fine

You now have an empty schematic. Everything below places parts and wires them on this sheet.

---

### Step 2 — Place J1 (truck-side terminal block)

**What it is:** the 5-position screw terminal that accepts the FMS harness wires.

1. Press `A` to open the symbol chooser
2. Search for the terminal block from your `telematics` library (LCSC C-number for DORABO DB128V-5.08-5P-GN-S — the one you fetched)
3. Click to place it on the left side of the sheet
4. Press `Escape` to stop placing

**Label J1's pins** (right-click the symbol → Properties → edit pin names, OR use the symbol editor to rename pins in your library, whichever is easier):

| J1 Pin # | Pin name | Purpose |
|---|---|---|
| 1 | `GND` | Ground from truck |
| 2 | `CAN_L_RAW` | CAN low from FMS pin 6 |
| 3 | `CAN_H_RAW` | CAN high from FMS pin 7 |
| 4 | `IGN_RAW` | Ignition from FMS pin 10 (24V or 0V) |
| 5 | `+24V_TRUCK` | +24V permanent from FMS pin 12 |

These are just internal names for your convenience — they don't correspond to any KiCad electrical net yet.

**Now add net labels** (hotkey `L`) on wires coming out of each pin:

1. Draw a short horizontal wire (hotkey `W`) from J1 pin 1 (~5 mm long)
2. Press `L`, type `GND`, place the label on the wire's end
3. Repeat for the other four pins: `CAN_L_RAW`, `CAN_H_RAW`, `IGN_RAW`, `+24V_TRUCK`

After this step: J1 is placed, each of its 5 pins has a short wire with a net label. Those labels will connect to matching labels elsewhere on the sheet as we place more parts.

**Pin electrical type:** all 5 pins on J1 should be **Passive** (a terminal block is just a pass-through).

---

### Step 3 — Add the GND power symbol

Every ground connection on the schematic will use the KiCad global GND symbol so all grounds automatically connect together.

1. Press `P` (Add Power Port)
2. Search for `GND`, place one somewhere near J1
3. Draw a wire from J1's `GND` wire endpoint to this GND symbol

Now `GND` is a global net. Anywhere else on the schematic where you place a GND symbol, it's the same net.

---

### Step 4 — Place F1 fuse holder + fuse

**What it is:** the 5×20 mm cartridge fuse holder (Xucheng Elec) that holds a 1A glass fuse. Protects downstream circuitry from short-circuit fault current.

1. Press `A`, search for the fuse holder from your `telematics` library
2. Place it to the right of J1
3. **Wire F1 pin 1 → J1's `+24V_TRUCK` wire.** Either draw a wire directly between them, or draw a short wire from F1 pin 1 and label it `+24V_TRUCK` (KiCad automatically connects matching net labels)
4. Draw a wire from F1 pin 2 to the right, label the wire `+24V_FUSED` (this is a new net downstream of the fuse)

**Pin electrical type:** both pins **Passive**.

After this step: `+24V_TRUCK` enters F1, `+24V_FUSED` exits.

---

### Step 5 — Place D1 (main TVS diode)

**What it is:** the SMBJ33CA TVS on the truck power input. Clamps voltage spikes above 33V to ground.

1. Press `A`, search for SMBJ33CA (LCSC C78420) from your `telematics` library
2. Place it below or next to F1
3. **Wire D1 pin 1 → `+24V_FUSED` net** (draw a wire from D1 to F1's output, or use net labels)
4. **Wire D1 pin 2 → GND** (place a GND symbol below D1 and wire to it)

**Note on polarity:** SMBJ33CA is bidirectional (the "CA" suffix), so pin 1 vs pin 2 doesn't matter electrically. But by convention, put the cathode-band side toward the positive rail.

**Pin electrical type:** both pins **Passive**.

---

### Step 6 — Place C3 (buck input capacitor)

**What it is:** 10 µF ceramic capacitor smoothing the +24V rail at the buck's input.

1. Press `A`, search for the 10 µF cap from your `telematics` library (LCSC C89632 — the Samsung 1210)
2. Place it near where U3 (the buck) will go — you'll place U3 next, so put C3 immediately to the left of where U3 will be
3. **Wire C3 pin 1 → `+24V_FUSED` net**
4. **Wire C3 pin 2 → GND** (place a GND symbol, or extend GND from D1)

**Pin electrical type:** both pins **Passive**.

**Layout note (for later PCB stage):** C3 must be physically within 5 mm of U3's VIN pin on the actual PCB. On the schematic, put it right next to U3 so you don't forget this.

---

### Step 7 — Place U3 (Traco TSR 1-2450E buck)

**What it is:** the switching regulator that converts +24V from the truck down to +5V for your electronics.

1. Press `A`, search for TSR 1-2450E (LCSC C5346721) from your `telematics` library
2. Place it to the right of C3
3. Identify U3's three pins (VIN, GND, VOUT) — the exact pin numbers depend on how easyeda2kicad drew the symbol; check the datasheet if unclear
4. **Wire U3 VIN pin → `+24V_FUSED` net** (this ties it to F1's output and C3's positive side)
5. **Wire U3 GND pin → GND** (place a GND symbol below U3)
6. **Wire U3 VOUT pin → a new wire going right**, label the wire `+5V` (this is a new global power net)

**Pin electrical type:**
- VIN: **Power input** (consumes power)
- GND: **Power input** (or Passive)
- VOUT: **Power output** (produces power — this satisfies ERC's "who drives +5V?" check)

---

### Step 8 — Place C4 (buck output capacitor)

**What it is:** 10 µF ceramic capacitor smoothing the +5V rail at the buck's output.

1. Press `A`, search for the 10 µF cap again (same as C3)
2. Place it immediately to the right of U3
3. **Wire C4 pin 1 → `+5V` net** (should be U3's VOUT output)
4. **Wire C4 pin 2 → GND**

**Pin electrical type:** both **Passive**.

---

### Step 9 — Add the +5V power port

The `+5V` net is your regulated 5V rail. Make it easy to reference elsewhere by adding a global power port:

1. Press `P`, search for `+5V`, place one next to C4
2. Wire this `+5V` power port to the `+5V` net (should already be there from Step 7)

Also add a `PWR_FLAG` on the `+5V` net:
1. Press `A`, search for `PWR_FLAG` (in the `power` library)
2. Place it next to `+5V`, wire it to `+5V`
3. This tells ERC "yes, this net is powered by something" — silences the "no driver" warning

---

### Sanity check: run ERC now

Before moving on, run `Inspect → Electrical Rules Checker` (or click the ladybug icon in the toolbar).

Expected result: probably clean or with just "unconnected pin" warnings on the parts we haven't wired yet (like J1's CAN pins). No errors related to power should appear.

If you see "no driver on `+24V_FUSED`": that's expected for now — the truck harness will drive it, and we haven't added a PWR_FLAG for that net yet (we'll skip it since J1 is where truck power enters, and ERC treats terminal blocks as external inputs).

If you see other errors, stop and fix them before continuing. Common issues:
- Wire not actually connecting (looks connected but isn't — junction dot missing)
- Two different net labels near each other, both trying to name the same wire
- A pin left floating without a No Connect flag

---

### Step 10 — Place R1 and R2 (voltage divider for ignition sense)

**What they are:** two resistors forming a 10:1 voltage divider. Scales the 24V ignition signal down to ~2.2V for the ESP32 GPIO.

1. Press `A`, search for the 100 kΩ resistor (LCSC C2912579) from your `telematics` library
2. Place R1 vertically, somewhere below the J1 terminal block
3. Search for the 10 kΩ resistor (LCSC C98220) from your `telematics` library
4. Place R2 vertically, directly below R1

Wiring:

5. **Wire R1 pin 1 → `IGN_RAW` net** (comes from J1 pin 4)
6. **Wire R1 pin 2 → R2 pin 1** (this is the divider midpoint)
7. **Wire R2 pin 2 → GND**
8. **Draw a wire from the midpoint (between R1 and R2), label it `IGN_SENSE`** — this is the divided-down signal that'll go to the LilyGo GPIO

**Pin electrical type:** all pins on both resistors: **Passive**.

**Divider check:** with 24V on `IGN_RAW`, `IGN_SENSE` = 24 × (10/(100+10)) = 2.18V. Safe for ESP32.

---

### Step 11 — Place D2 (GPIO protection TVS)

**What it is:** SMAJ5.0CA bidirectional TVS. Clamps any voltage transient on the GPIO pin to ~9V, protecting the ESP32 from spikes.

1. Press `A`, search for SMAJ5.0CA (LCSC C10756) from your `telematics` library
2. Place it near the midpoint between R1 and R2 (on the `IGN_SENSE` net)
3. **Wire D2 pin 1 → `IGN_SENSE` net**
4. **Wire D2 pin 2 → GND**

**Pin electrical type:** both **Passive**.

After this step: your ignition-sense chain is complete. `IGN_RAW` (24V) → R1 → midpoint (with D2 to ground) → R2 → GND. The midpoint is `IGN_SENSE`, which will go to the LilyGo later.

---

### Step 12 — Place D3 (CAN TVS protection — at the point of entry)

**What it is:** ONsemi NUP2105LT1G, a 2-channel bidirectional TVS array specifically designed for CAN bus protection. This sits **as close to J1 as possible** so it can absorb any transient the moment it enters the board — before the transient reaches L1 or any other components.

**Why D3 before L1:** the standard automotive CAN protection topology is `connector → TVS → choke → transceiver`. The TVS goes first because its job is to shunt transient energy away *before* it damages anything downstream. If you put the choke first, big transients (like a 87V alternator load-dump) could damage the choke's windings before they reach the TVS. Every TI, NXP, and ON Semi CAN application note follows this ordering.

1. Press `A`, search for NUP2105LT1G (LCSC C14486) from your `telematics` library
2. Place it immediately to the right of J1's CAN pins (as close to the connector as visually possible)

Wiring (verify NUP2105LT1G pin numbers against the datasheet — pin numbering matters here):

3. **Wire D3 pin 1 → `CAN_L_RAW` net** (from J1 pin 2, directly)
4. **Wire D3 pin 2 → GND**
5. **Wire D3 pin 3 → `CAN_H_RAW` net** (from J1 pin 3, directly)

**Pin electrical type:** all 3 pins **Passive**.

After this step: D3 is sitting on the CAN_H_RAW and CAN_L_RAW nets, ready to clamp any transient that arrives from the truck harness.

---

### Step 13 — Place L1 (CAN common-mode choke — after the TVS)

**What it is:** TDK ACT45B-101-2P-TL003 common-mode choke. Filters out common-mode noise on the CAN bus while passing the differential CAN signals cleanly. Sits between D3 and U2 so it filters the (already-clamped, safe-voltage) signal on its way to the transceiver.

1. Press `A`, search for ACT45B-101-2P-TL003 (LCSC C88056) from your `telematics` library
2. Place it to the right of D3, in the signal path toward U2
3. Identify L1's four pins by looking at the datasheet. **Critical:** the two windings are pins 1↔4 (one winding) and pins 2↔3 (the other winding). Each winding must carry ONE signal only — don't cross them.

Wiring:

4. **Wire L1 pin 1 → `CAN_H_RAW` net** (same net that D3 pin 3 is on)
5. **Wire L1 pin 2 → `CAN_L_RAW` net** (same net that D3 pin 1 is on)
6. **From L1 pin 4, draw a wire, label it `CAN_H_FILT`** (this is CAN_H after common-mode filtering — this is what goes to U2)
7. **From L1 pin 3, draw a wire, label it `CAN_L_FILT`** (CAN_L after common-mode filtering)

**Pin electrical type:** all 4 pins **Passive**.

After this step: the CAN chain is `J1 → D3 (protects) → L1 (filters) → CAN_H_FILT/CAN_L_FILT → U2`. Ready to connect to U2 in the next step.

---

### Step 14 — Place U2 (TJA1051 CAN transceiver)

**What it is:** NXP TJA1051T/3/1J, the CAN transceiver chip that converts between the CAN bus differential signals and the ESP32's UART logic-level signals.

1. Press `A`, search for TJA1051T/3/1J (LCSC C38695) from your `telematics` library
2. Place it in the middle of the schematic, to the right of L1

U2 has 8 pins. Wire each one as follows:

| U2 Pin # | Pin Name | Connect to | Notes |
|---|---|---|---|
| 1 | TXD | New wire, label it `CAN_TXD` | ESP32 → transceiver direction |
| 2 | GND | GND | |
| 3 | VCC | `+5V` net | Chip power (5V side) |
| 4 | RXD | New wire, label it `CAN_RXD` | Transceiver → ESP32 direction |
| 5 | VIO | New wire, label it `+3V3` | Chip's logic-side power (3.3V) — will be driven by LilyGo |
| 6 | CANL | `CAN_L_FILT` net | Filtered CAN low |
| 7 | CANH | `CAN_H_FILT` net | Filtered CAN high |
| 8 | S (Silent mode) | `+3V3` net (same as VIO) | Tied high = listen-only mode |

For each row above:
1. Draw a wire from the pin
2. Either connect it to the named net directly, or add a net label with the correct name

**Pin electrical types** (these are the datasheet-defined roles):
- Pin 1 (TXD): Input (chip receives data from ESP32)
- Pin 2 (GND): Power input
- Pin 3 (VCC): Power input
- Pin 4 (RXD): Output (chip sends data to ESP32)
- Pin 5 (VIO): Power input
- Pin 6 (CANL): Bidirectional
- Pin 7 (CANH): Bidirectional
- Pin 8 (S): Input

easyeda2kicad's TJA1051 symbol probably has these set correctly already — verify by double-clicking the symbol and checking the pin table.

---

### Step 15 — Place C1 (TJA1051 VCC decoupling)

**What it is:** 100 nF ceramic capacitor decoupling the TJA1051's VCC pin.

1. Press `A`, search for the 100 nF cap (LCSC C1591) from your `telematics` library
2. Place C1 immediately next to U2 pin 3 (VCC)
3. **Wire C1 pin 1 → `+5V` net** (same as U2 pin 3)
4. **Wire C1 pin 2 → GND**

**Pin electrical type:** both **Passive**.

**Layout note (for PCB stage):** C1 must be physically within 2 mm of U2's VCC pin on the actual PCB. On the schematic, put it as close as possible visually.

---

### Step 16 — Place C2 (TJA1051 VIO decoupling)

**What it is:** 100 nF ceramic capacitor decoupling the TJA1051's VIO pin.

1. Press `A`, search for the 100 nF cap again
2. Place C2 immediately next to U2 pin 5 (VIO)
3. **Wire C2 pin 1 → `+3V3` net** (same as U2 pin 5)
4. **Wire C2 pin 2 → GND**

**Pin electrical type:** both **Passive**.

After this step: the whole CAN section is done. J1 CAN pins → L1 → D3 → U2 → CAN_TXD/CAN_RXD nets ready for LilyGo, with proper decoupling on U2's power pins.

---

### Step 17 — Place J2 (LilyGo left header socket)

**What it is:** the left 1×16 female pin header on your PCB. The LilyGo's left row of male header pins plugs into this. JLCPCB installs the empty header; you install the LilyGo when boards arrive.

1. Press `A`, search for the 1×16 female pin header (LCSC C350305) from your `telematics` library
2. Place it on the right side of the schematic sheet

**Now rename each of J2's 16 pins to match the LilyGo's left column pinout.** You can do this two ways:
- **In the symbol editor:** open the C350305 symbol, edit each pin's name — this makes the names stick with every future use of the symbol
- **Per-instance:** double-click the placed symbol, edit pin names in the Properties dialog — only affects this one instance

For a one-off design, per-instance is fine.

**J2 pin naming (LilyGo left header, top to bottom per the pinout image):**

| J2 Pin # | Pin Name | Type | Wire it to |
|---|---|---|---|
| 1 | `3V3_ALT` | Power output | (unused — see below) |
| 2 | `GND_L1` | Power input | GND |
| 3 | `GPIO16` | Bidirectional | `IGN_SENSE` net |
| 4 | `GPIO17` | Bidirectional | `CAN_RXD` net |
| 5 | `GPIO18` | Bidirectional | `CAN_TXD` net |
| 6 | `GPIO8` | Bidirectional | (unused — camera XCLK) |
| 7 | `GPIO3` | Bidirectional | (unused — camera SIOD) |
| 8 | `GPIO46` | Bidirectional | (unused — strapping pin) |
| 9 | `GPIO9` | Bidirectional | (unused — camera D9) |
| 10 | `GPIO10` | Bidirectional | (unused — camera D8) |
| 11 | `GPIO11` | Bidirectional | (unused — camera D7) |
| 12 | `GPIO12` | Bidirectional | (unused — camera D6) |
| 13 | `GPIO13` | Bidirectional | (unused — camera D5) |
| 14 | `GPIO14` | Bidirectional | (unused — camera D2) |
| 15 | `GND_L2` | Power input | GND |
| 16 | `DC5` | Power input | (unused — solar input) |

Note on the two 3V3 pins across J2 and J3: the LilyGo has 3V3 on both headers. Only wire one of them to your `+3V3` net (I recommend the J3 side — see next step). Leave the J2 3V3_ALT pin as No Connect. Wiring both would create a redundant path; not harmful, but simpler to leave one alone.

**Wiring for J2** (do these one at a time):

1. **J2 pin 2 (GND_L1) → GND** — draw a wire, add a GND symbol
2. **J2 pin 3 (GPIO16) → `IGN_SENSE` net** — draw a wire, add a net label saying `IGN_SENSE`
3. **J2 pin 4 (GPIO17) → `CAN_RXD` net**
4. **J2 pin 5 (GPIO18) → `CAN_TXD` net**
5. **J2 pin 15 (GND_L2) → GND**
6. **All other J2 pins (1, 6, 7, 8, 9, 10, 11, 12, 13, 14, 16):** press `Q` on each pin to place a "No Connect" flag (the X symbol). This tells KiCad you're intentionally leaving these floating.

---

### Step 18 — Place J3 (LilyGo right header socket)

**What it is:** the right 1×16 female pin header on your PCB. Symmetric mirror of J2.

1. Press `A`, search for the same 1×16 female pin header (LCSC C350305) again
2. Place it to the right of J2, spaced at the LilyGo's row-to-row distance (~22.86 mm — verify when your LilyGo arrives)

**J3 pin naming (LilyGo right header, top to bottom per the pinout image):**

| J3 Pin # | Pin Name | Type | Wire it to |
|---|---|---|---|
| 1 | `3V3` | Power output | `+3V3` net (drives it) |
| 2 | `GPIO1` | Bidirectional | (unused — camera SCL) |
| 3 | `GPIO2` | Bidirectional | (unused — camera SDA) |
| 4 | `TXD_43` | Bidirectional | (unused — UART0 programming pin, don't touch) |
| 5 | `RXD_44` | Bidirectional | (unused — UART0 programming pin, don't touch) |
| 6 | `GPIO37` | Bidirectional | (unused — PSRAM, do NOT connect) |
| 7 | `GPIO36` | Bidirectional | (unused — PSRAM, do NOT connect) |
| 8 | `GPIO35` | Bidirectional | (unused — PSRAM, do NOT connect) |
| 9 | `GPIO0` | Bidirectional | (unused — BOOT strapping pin) |
| 10 | `GPIO45` | Bidirectional | (unused — strapping pin) |
| 11 | `GPIO48` | Bidirectional | (unused — camera D4) |
| 12 | `GPIO47` | Bidirectional | (unused — camera D3) |
| 13 | `GPIO21` | Bidirectional | (unused — camera VSYNC) |
| 14 | `GND_R1` | Power input | GND |
| 15 | `VSYS` | Power input | `+5V` net (this powers the LilyGo!) |
| 16 | (not populated on some LilyGos) | — | No Connect |

**Wiring for J3** (do these one at a time):

1. **J3 pin 1 (3V3) → `+3V3` net** — draw a wire, add the `+3V3` net label
2. **J3 pin 14 (GND_R1) → GND** — draw a wire, add a GND symbol
3. **J3 pin 15 (VSYS) → `+5V` net** — draw a wire, add the `+5V` net label (this is where your buck's 5V feeds into the LilyGo)
4. **All other J3 pins (2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 16):** press `Q` on each to place a No Connect flag

---

### Step 19 — Add a PWR_FLAG on +3V3

The `+3V3` net is driven by the LilyGo's 3V3 output pin (J3 pin 1). ERC might not automatically recognize this as a "driver" unless the pin type is set to Power output.

To be safe, add an explicit PWR_FLAG:

1. Press `A`, search for `PWR_FLAG` in the `power` library
2. Place it near the +3V3 net somewhere
3. Wire it to the `+3V3` net

This tells ERC "yes, +3V3 is powered by something, stop complaining."

---

### Step 20 — Add the 5V-to-LilyGo fallback pad (design safety)

Until LilyGo confirms whether feeding VSYS via the header is supported (per your GitHub issue), add a backup path:

1. Place another `+5V` power port near J3
2. Draw a short wire from it to nowhere in particular — this becomes an isolated `+5V` net segment
3. On the PCB layout stage (Part 4), you'll add a solder pad here that connects to +5V and place it physically near where the LilyGo's H606 pad will land when plugged in

This gives you a bodging point if VSYS doesn't work. Not strictly needed in the schematic — you could add it directly in the PCB layout — but including it here as a placeholder is cleaner.

---

### Step 21 — Final ERC check

Run `Inspect → Electrical Rules Checker`. You should see **zero errors**. Warnings are OK if you understand them.

**Common leftover issues:**
- **"Pin not connected on U1 pin X"** — did you forget a No Connect flag? Every unused pin needs one.
- **"Multiple drivers for +5V"** — you have two `Power output` pins on the same net. This is expected if the buck's VOUT and a PWR_FLAG both drive `+5V`. Change one to `Power input`, or accept the warning.
- **"Multiple drivers for +3V3"** — same as above but for +3V3.
- **"Pin conflict"** — a pin type mismatch. Usually fixed by changing a pin to Bidirectional or Passive.

If ERC is clean, save the schematic.

---

### Step 22 — Annotate and assign footprints

1. `Tools → Annotate Schematic` → clicks "Annotate" to auto-assign final reference designators (R1, R2, C1, C2, etc.)
2. `Tools → Assign Footprints` — verify that every symbol has a footprint from your `telematics` library. easyeda2kicad already assigned them, but this is the check step.

If any symbol shows "no footprint assigned," it's because either:
- easyeda2kicad's fetch failed for that part → re-run the fetch, or manually assign a footprint
- The library wasn't properly registered → check `Preferences → Manage Footprint Libraries`

---

### What you should have now

- A complete schematic with all 12 BOM parts placed and wired
- 5 external inputs from J1 (from truck)
- One 5V regulated rail powering U2 and the LilyGo (via J3 VSYS)
- One 3.3V rail driven by the LilyGo, powering U2's VIO
- Ignition sense chain feeding LilyGo GPIO16
- CAN bus chain feeding LilyGo GPIO17 (RX) and GPIO18 (TX)
- Common ground
- Every pin either wired, labeled with No Connect, or acknowledged as unused
- ERC clean

You're ready to move to PCB layout (Part 4).

---
---


## Part 3 — Alternatives if a part isn't available

JLCPCB's stock fluctuates. Verify each part's availability at jlcpcb.com/parts during schematic entry (manual check), and again via the plugin once you're in PCB layout. If something is out of stock or no longer assembly-eligible:

### TJA1051T/3 — alternatives
- **TJA1042T/3** — very similar NXP part, drop-in compatible for most applications. Also has 3.3V VIO. LCSC: `C81271`.
- **MCP2551-I/SN** — Microchip's competitor; **but no 3.3V VIO pin**, requires 5V logic — do NOT substitute directly.
- **ISO1050** — TI, isolated CAN transceiver; overkill and different pinout, not a direct swap.
- **Best alternative: TJA1042T/3** — same NXP family, near-identical behaviour, always in stock.

### NUP2105LT1G — alternatives
- **PESD1CAN** (Nexperia) — CAN-specific TVS array, drop-in equivalent
- **CDSOT23-T03LC** (Bourns) — similar spec, drop-in
- **PESD2CAN** — 2-channel version, same package
- Any of these work; verify the SOT-23 pinout matches (some CAN TVS arrays have different pin orders — check the datasheet).

### SMBJ33CA — alternatives
- Any bidirectional TVS in SMB package with 33V standoff works
- **P6SMB33CA** — larger power rating (600W), same package
- **SMBJ30CA** — 30V standoff (clamps a bit lower; still safe for the buck)
- **SMBJ36CA** — 36V standoff (clamps a bit higher; still under buck's 40V max)
- Prefer 33V±3V standoff, bidirectional, SMB package

### SMAJ5.0CA (D2) — alternatives
- **SMAJ3.3CA** — 3.3V standoff (clamps around 8V), tighter clamp than SMAJ5.0CA, less commonly stocked
- **PESD3V3L2BT** — Nexperia ESD diode, SOD-323, lower capacitance if you're worried about high-speed
- **ESD5Z3.3T1G** — onsemi ESD diode, SOD-523, faster response than TVS
- For a GPIO input at DC-ish signal (ignition sense is essentially a slow-changing signal), any of these work

### ACT45B-101-2P-TL003 — alternatives (this is the most likely to have stock issues)
- **744232101** (Würth) — 100 µH CAN common-mode choke, similar spec, sometimes more available
- **DLW5BTN102TQ2** (Murata) — 1 kΩ at 100 MHz CAN choke, drop-in for CAN applications
- **B82789C0103H002** (TDK/EPCOS) — 10 µH per line CAN choke (lower inductance but still works for CAN)
- Verify the footprint before substituting — CAN chokes vary in size (some are 5×4 mm, some 4.5×3.2 mm, some smaller)

### Traco TSR 1-2450E (U3 buck) — alternatives

If the TSR 1-2450E is out of stock or you find a better option when you're building the BOM, here's the landscape of what to consider — and what to avoid — based on real LCSC availability at the time this guide was written.

**Genuine industrial-grade alternatives (in order of preference):**

1. **Traco TSR 1-2450SM** — SMD variant of the same part, LCSC `C5157926`. Stock is often low and lead time can be long, but if it's available it's the best option: SMD, single-part, genuine Traco. Check stock at order time.

2. **Recom R-78S5.0-1.0** — Recom's SMD variant of the R-78 family. Search LCSC for `R-78S` filtered by manufacturer = "Recom Power". Only use if the listing genuinely shows "Recom Power" as manufacturer. In practice, Recom parts often show 0 stock at LCSC — verify before committing.

3. **Murata OKI-78SR-5/1.5-W36-C** — 1.5A output, Japanese-brand pedigree. Search LCSC filtered by manufacturer = "Murata". If in stock, a solid choice; typically more expensive.

4. **Other Traco variants** — TSR 1-2450 (standard), TSR-2-2450 (2A version). Ensure the listing says "TRACO POWER" as manufacturer.

**What to avoid — clones and relabels:**

Based on searches during this guide's writing, watch out for:

- **"R-78E5.0-1.0" listings by YLPTEC, EVISUN, or manufacturers other than "Recom Power"** — these are clones of the Recom design. Same functional purpose, but no quality control, no guaranteed lifetime, and no accountability if they fail in a truck. Not appropriate for automotive deployment.
- **"TSR 1-2450" listings by YLPTEC** — clone of Traco's design. Same reasoning.
- **The general pattern:** if the LCSC listing shows a well-known Western brand's part number but a Chinese manufacturer name in the manufacturer field, it's almost always a clone. Skip these.

**Chinese manufacturers that ARE legitimate:**

- **MornSun** — legitimate Chinese industrial power supply manufacturer with their own designs. If they have a K7805 series SMD buck module in stock, it's a real product (I recommended this earlier in the guide but couldn't confirm current stock at write time — check LCSC for `MornSun K7805` yourself). Not to be confused with the "K7805" name showing up on clone parts.
- **XLSEMI** — makes clone/equivalent buck IC designs (like the XL1509, an LM2596 equivalent). Legitimate as a chip manufacturer. But these are **discrete buck ICs, not modules** — you'd need to add an external inductor, freewheeling diode, and output cap. Only viable if you're comfortable with switching-converter layout, which is a lot to ask of a first PCB.
- **Silergy, Chipsea** — legitimate Chinese semiconductor manufacturers making integrated buck modules. Worth searching if you want an SMD single-part solution.

**Discrete buck ICs — if you must:**

If nothing else is in stock and you're forced to design a discrete buck, the XLSEMI XL1509-5.0E1 (LCSC `C61063`, hundreds of thousands in stock) works well and has an LM2596-compatible reference design. But this requires:
- Adding a ~100 µH power inductor (search LCSC for `100uH SMD power inductor 2A saturation`, choose a shielded part)
- Adding an SS14 Schottky freewheeling diode (LCSC `C2480`)
- Following the datasheet's application circuit precisely
- Careful layout of the switching node

Not recommended for a first PCB unless you're comfortable with switching-converter design and have access to an oscilloscope for debugging noise/ripple issues.

**Verification checklist regardless of which buck you pick:**

- **Manufacturer field shows a real industrial brand** (Recom, Traco, Murata, MornSun, TDK-Lambda, TI, etc.) — not a generic Chinese distributor name
- **Input voltage range covers 12V to at least 32V** (truck systems spike to ~28V during alternator load)
- **Output current rating ≥ 1A** (you'll use ~200-500 mA but headroom matters for reliability and heat management)
- **Efficiency > 85% at your typical load**
- **Available for JLCPCB PCBA** (check stock at jlcpcb.com/parts — for SMD parts, "Assembly" status must be shown; for through-hole, you'll pay JLCPCB's small through-hole assembly fee)
- **KiCad footprint available or easy to create from the datasheet** — remember to verify pin/pad dimensions carefully

### Passives — alternatives
- Any 0603 or 0805 X7R resistor/capacitor with the right value and voltage works. LCSC has hundreds of equivalents; just pick whichever is in stock and preferably a Basic part.

---

## Part 4 — PCB layout

Open the PCB editor from the project manager. Update the PCB from the schematic (`Tools → Update PCB from Schematic`, or F8 in the PCB editor).

**Now is the time to install the kicad-jlcpcb-tools plugin** if you haven't already. Follow the install steps from Part 1. Once installed, open the plugin panel (click the JLCPCB icon in the PCB editor toolbar) and let it scan your project — it will list every part, look up each LCSC number you tagged during schematic entry, and flag any stock or assembly issues. Fix these before proceeding with layout.

Also, set the design rules per Part 1 (`File → Board Setup → Design Rules → Constraints`) if you haven't yet.

### Board outline

Decide on board size first. A reasonable target for this design: **80 × 60 mm**. Big enough for comfortable routing, small enough to fit anywhere. Draw the outline on the `Edge.Cuts` layer using the rectangle tool.

If you want mounting holes, place 4× 3.2 mm holes at the corners (M3 clearance) inset ~5 mm from the edges.

### Layer stackup

For a design this simple, **2 layers** is sufficient. JLCPCB's default 2-layer FR4 is ~€5–10 for 5 boards.

Top layer: signal traces and ground fills
Bottom layer: signal traces, main ground pour

### Component placement

Some rules of thumb specific to your design:

**1. Group by function.** Truck-side connector and power protection on one side of the board, LilyGo footprint and signal processing on the other. Physically separate the "hostile" 24V side from the "sensitive" 3.3V side.

**2. Buck converter loop is critical.** The Traco TSR 1-2450E (U3) must have C3 (input cap) *within 5 mm* of its VIN and GND pins, and C4 (output cap) *within 5 mm* of its VOUT and GND pins. This is the #1 layout mistake in switch-mode designs. Long traces here cause the buck to oscillate, generate noise, or fail entirely. Because the Traco is through-hole SIP-3, place C3 and C4 as SMD parts directly underneath or immediately adjacent to the Traco's through-hole pins on the bottom side of the board — this gives you the shortest possible loop. (If you use an SMD buck module instead, this becomes easier — C3 and C4 can sit right next to the module's pads on the same board layer.)

**3. TJA1051 decoupling caps.** C1 (across VCC and GND) and C2 (across VIO and GND) must be within 2 mm of the chip's power pins. Not "in the same neighbourhood" — literally right at the pins.

**4. CAN bus routing.** CAN_H and CAN_L must be routed as a **tight differential pair** — parallel traces, close together (0.2 mm gap is fine), same length. This maintains the 120 Ω differential impedance. From the terminal block → common-mode choke → NUP2105L → TJA1051, keep the two traces tightly coupled throughout.

**5. Ground plane.** Fill the bottom layer with a solid ground pour (right-click on the bottom layer in the layers panel → "Add Filled Zone" → Net: GND, Layer: B.Cu). This gives every component a low-impedance return path. Also add a ground pour on the top layer in unused areas.

**6. LilyGo footprint.** The two 16-pin female header rows must match the LilyGo's exact pin spacing. See Part 5 below.

**7. Thermal considerations.** The Traco TSR 1-2450E dissipates ~200 mW at your typical load — barely warm, no heatsink or thermal via needed. The module has an internal metal case that acts as its own small heatsink. Leave a small copper pour around it for margin, especially if the enclosure gets hot in a truck cabin in summer.

### Routing

Route power traces wider than signal traces. Suggested widths:

- **24V input, 5V rail**: 0.5 mm minimum, 0.8 mm preferred
- **Ground return traces**: same or wider (but with a ground pour, this is mostly handled)
- **Signal traces** (CAN, UART, GPIO): 0.2–0.3 mm is plenty
- **Between the buck and its caps**: as wide as physically possible — 1 mm+ if you can fit it

Use vias to jump between layers only where necessary. Every via adds inductance and slightly disrupts the ground plane.

### DRC (design rule check)

Before exporting, run `Inspect → Design Rules Checker` in the PCB editor. Fix all errors. Warnings are OK to ignore in most cases, but read them to understand what they're flagging.

Also run the ERC (Electrical Rules Check) in the schematic editor to catch unconnected pins or floating nets.

---

## Part 5 — The LilyGo footprint (the one thing you can't finalize without the physical board)

**Order 3 LilyGo boards from LilyGo's official AliExpress store today.** They ship in 1–2 weeks. In the meantime, do all other PCB design work.

When the LilyGo arrives, measure with digital calipers:

1. **Pin-to-pin spacing within a row**: should be exactly 2.54 mm. Verify — if wrong, don't use this board.
2. **Row-to-row spacing** (centre of one row of 16 pins to centre of the other row): measure precisely, expect roughly **22.86 mm** based on LilyGo documentation but *verify with calipers*.
3. **Overall board length and width**: for enclosure design, not critical for PCB layout unless you want the LilyGo to sit flush.
4. **Position of the USB-C port, buttons, and antenna connectors relative to the pin headers**: so you know which side of your PCB should have the "front" cutout for those.
5. **Height of the LilyGo above its pin headers** (the LilyGo sits on top of the female headers you install on your board): typically ~10 mm. Ensures no components on your PCB directly under the LilyGo footprint are too tall.

In KiCad, create your LilyGo footprint:

- Open the Footprint Editor from KiCad's main window
- Create a new footprint called `LilyGo_T-SIM7080G-S3_Socket`
- Add 32 through-hole pads (2 rows × 16 pins) with 2.54 mm within-row spacing and your measured row-to-row spacing
- Pad size: 1.7 mm hole with 3.0 mm annular ring (standard for 2.54 mm through-hole)
- Add a silkscreen outline showing the LilyGo's board outline (from your measurements)
- **Label each pad on the silkscreen with the LilyGo pin name**, so wiring is unambiguous. Reference the LilyGo pinout diagram to identify each pin's physical position. Pins you specifically care about labeling:
  - **VSYS** (potential 5V input — verify support via LilyGo GitHub issue before relying on it)
  - **GND** pins (multiple exist; label them all)
  - **3V3** (output pin — supplies TJA1051 VIO)
  - **GPIO16** (your ignition sense input)
  - **GPIO17** (your CAN RXD from TJA1051)
  - **GPIO18** (your CAN TXD to TJA1051)
- Optionally label all other pins too, or just mark them as "NC" (not connected) — makes debugging much easier later
- **Also add a solder pad on the PCB** labeled "5V TO LILYGO" positioned near where the LilyGo's H606 pad will land when plugged in. This is your fallback if VSYS-as-input doesn't work — you can bodge a wire from this pad to the H606 point.

Save the footprint and associate it with your schematic's two 1×16 headers (U1 pins 1-16 and U1 pins 17-32, or however you organized it).

---

## Part 6 — Generating manufacturing files

Once the layout is complete and DRC-clean:

### How the three files fit together (understand this before generating them)

When JLCPCB assembles your board, they only see three artifacts you send them:

1. **Gerber files** — the physical PCB itself (traces, holes, pads, silkscreen, board outline). This tells JLCPCB's fabrication machines how to make the bare PCB. **JLCPCB never sees your KiCad schematic**, and they don't need to — everything they need is in the Gerbers.

2. **BOM (Bill of Materials) CSV** — for each designator on your board (C1, R1, U2, etc.), which LCSC part to install. This tells JLCPCB "put LCSC part C1591 at every position marked C1 or C2 on the board."

3. **CPL (Component Placement / Pick-and-Place) CSV** — for each designator, exact X/Y coordinates and rotation. This tells the pick-and-place machine "put C1 at position (25.4, 30.5) mm rotated 0°, put C2 at (27.9, 30.5) mm rotated 0°, put U2 at (40.0, 25.0) mm rotated 90°..."

**The key mental model:** your schematic and its symbols exist only in your KiCad project — for you to visualize and debug the design. Once you generate the manufacturing files, symbols are irrelevant. What survives to manufacturing is:

- The **footprints** on your PCB (which live in your Gerbers as pad shapes)
- The **LCSC part numbers** in your BOM (linking designators to physical parts)
- The **X/Y positions** in your CPL (telling machines where to place them)

This means: if your schematic symbol for the LilyGo socket is one big custom rectangle vs. two smaller headers doesn't matter to JLCPCB. What matters is that your BOM correctly identifies each physical component and each has a proper footprint on the board.

### About the LilyGo — it's NOT machine-assembled

Here's a subtle but important point: **the LilyGo T-SIM7080G-S3 itself is not something JLCPCB will assemble.** It's a full module (a whole PCB in its own right), not a component in their assembly library.

What JLCPCB will assemble on your board:
- All your SMD parts (TJA1051, choke, TVS diodes, resistors, caps)
- Your through-hole parts (terminal block, fuse holder, buck, and **the female pin headers that receive the LilyGo**)

What you'll do yourself when the boards arrive:
- **Plug the LilyGo into the female pin headers** (5 seconds per board, no soldering — just push the LilyGo's male header pins into the female sockets on your PCB)
- Insert the fuse into the fuse holder
- Connect the LiPo to the LilyGo's onboard connector
- Connect antennas

So on your BOM, U1 doesn't appear as a LilyGo — U1 (or a similar designator) appears as **two 1×16 female pin headers** (LCSC part like C350305 or whatever you chose). These are what JLCPCB installs. The LilyGo isn't in the BOM at all; you install it manually.

### Files JLCPCB needs

1. **Gerber files** (the board fabrication data)
2. **Drill file** (where to drill holes)
3. **BOM CSV** (bill of materials with LCSC part numbers)
4. **CPL / pick-and-place CSV** (where each component goes on the board)

### Using the JLCPCB Tools plugin (easy way)

Click the plugin's icon → "Fabrication → Generate all files". It creates a ZIP with the Gerbers and the BOM/CPL in JLCPCB's expected format. Upload the ZIP to JLCPCB's order page and you're done.

### Manual way (if the plugin fails or you want to understand the outputs)

**Gerbers (from PCB editor):**
- `File → Plot`
- Output directory: your project folder or a subdirectory
- Output format: **Gerber**
- Layers to plot (check these): `F.Cu`, `B.Cu`, `F.Paste`, `B.Paste`, `F.Silkscreen`, `B.Silkscreen`, `F.Mask`, `B.Mask`, `Edge.Cuts`
- Check "Plot border and title block": unchecked
- Check "Use Protel filename extensions": can be either; JLCPCB accepts both
- Click "Plot" then "Generate Drill Files" — save the drill files (.drl) to the same directory
- Zip up all resulting `.gbr` and `.drl` files (and any drill map files) into one archive to upload to JLCPCB

**BOM CSV (from schematic editor, in KiCad 8):**
- In the schematic editor: `Tools → Generate Legacy Bill of Materials...` (older KiCad menu wording) or `File → Export → Bill of Materials...` (newer wording — the exact menu label depends on your KiCad version)
- Select the "BOM CSV" or equivalent CSV export
- Configure columns to include: `Reference`, `Value`, `Footprint`, `MPN`, `LCSC` (these last two come from the custom fields you added during schematic entry)
- Save as CSV

The BOM CSV file for JLCPCB needs specific column headers matching their template. If manual export doesn't produce the exact format they want, the plugin's output is more reliable — that's the main reason the plugin is worth using despite this workflow gap.

**CPL / pick-and-place file (from PCB editor):**
- `File → Fabrication Outputs → Component Placement (.pos) File...`
- Format: **CSV** (not the default .pos ASCII format — JLCPCB wants CSV)
- Units: mm
- Files: Single file (both sides in one)
- Include board edge reference: default
- Save to your project folder

Note: KiCad's default CPL column order differs from JLCPCB's expected order. If you export manually you'll need to either rename columns to match JLCPCB's template (`Designator`, `Val`, `Package`, `Mid X`, `Mid Y`, `Rotation`, `Layer`) or upload as-is and use JLCPCB's column-mapping interface during order review. The plugin handles this remapping automatically.

---

## Part 7 — Ordering from JLCPCB

Go to jlcpcb.com and start a new order:

**PCB tab:**
- Upload your Gerber ZIP
- Quantity: 5 (minimum, and you want spares)
- Layers: 2
- Thickness: 1.6 mm (standard)
- Material: FR-4
- Surface finish: HASL (cheapest) or ENIG (nicer, more expensive; better for fine-pitch parts, but you don't have any)
- Silkscreen colour: white (or whatever you like)
- Solder mask: green is default and cheapest; black looks nicer

**PCB Assembly tab:**
- Enable PCBA (this is JLCPCB's SMT assembly service, machine pick-and-place)
- Assembly side: Top only (default; cheaper)
- Tooling holes: Added by JLCPCB (default)
- **Enable "Through-Hole Components Assembly"** (or "PTH Assembly", exact naming varies) — this is a separate service that JLCPCB provides for hand-installing through-hole components. Cost is roughly €0.30-0.50 per insertion plus small setup fees. For your design with 4 unique through-hole parts across 5 boards, this adds roughly €15-25 total. Absolutely worth it — otherwise you'd receive boards with only the SMD parts installed and would have to hand-solder the rest yourself.
- Confirm parts placement based on your CPL

**What JLCPCB will assemble on your boards:**

Machine-assembled (SMT):
- C1, C2 (100 nF caps)
- C3, C4 (10 µF caps)
- D1 (SMBJ33CA)
- D2 (SMAJ5.0CA)
- D3 (NUP2105L)
- L1 (ACT45B choke)
- R1, R2 (resistors)
- U2 (TJA1051 CAN transceiver)

Hand-assembled by JLCPCB technicians (through-hole):
- J1 (DORABO terminal block)
- F1 holder (fuse holder)
- U3 (Traco TSR 1-2450E buck)
- J2, J3 (1×16 female pin headers — the LilyGo socket)

**What JLCPCB will NOT install** (you do these when boards arrive):
- **U1 — the LilyGo T-SIM7080G-S3 itself** — this is a complete module, not a component in JLCPCB's assembly library. You plug it into the female headers (J2, J3) by hand when boards arrive. Takes 5 seconds per board, no soldering.
- **The 1A glass fuse** — you insert it into the fuse holder by hand.
- **The LiPo battery** — you plug it into the LilyGo's onboard JST-PH connector.
- **The antennas** — you screw them onto the LilyGo's IPEX/uFL connectors.
- **The FMS truck harness** — you land the wires into the terminal block.

**Upload BOM and CPL** when prompted. JLCPCB's system parses your BOM, matches each line to their stock, and shows any parts they don't have. If a part is missing, use the alternatives list in Part 3 to swap it in.

**Confirm the parts placement preview.** JLCPCB shows you a rendering of your board with parts placed based on the CPL. Verify each part is oriented correctly (especially polarised parts like the TVS diodes and the TJA1051 — pin 1 in the right corner).

**Order flow:**
- PCB fabrication: 1–2 days
- Assembly: 3–5 days
- Shipping to Portugal: 5–7 days (DHL) or slower for standard air mail

**Total: about 2–3 weeks from placing the order to boards in hand.**

Expected cost for 5 boards:
- PCB fabrication: €10–15
- SMD assembly setup + per-board: ~€60–80 (setup fees for extended parts + per-board assembly)
- Through-hole assembly (4 parts × 5 boards): ~€6–12
- Components: ~€30–50 per board (sourced by JLCPCB from LCSC)
- Shipping: €15–40
- **Total: €280–450 for 5 fully assembled boards** (SMD + through-hole)

Plus separately: 3× LilyGo boards from AliExpress (~€75), LiPo batteries, FMS harnesses, and the through-hole parts if you didn't include them in JLCPCB's assembly service.

---

## Part 8 — On design review

You expressed reluctance to post to Reddit for review, understandable. My honest read is that the PCB layout of this device isn't strong IP — it's a fairly standard telematics reference design pattern. The real IP is your firmware, backend, and business relationships, not the arrangement of a buck converter next to a CAN transceiver.

That said, if you're not comfortable posting, alternatives:

1. **Paid freelance review** — post on Upwork with title "Review 2-layer KiCad PCB (small telematics board)". Expect to pay €100–200 for a thorough review by someone experienced. Turnaround 3–5 days. Sign a simple NDA before sharing files.
2. **Anonymize before posting** — remove company name, silkscreen labels, and any identifying marks. Post to r/PrintedCircuitBoard as "generic CAN telematics board" — nobody will connect it to your business.
3. **Ask a local electronics engineer** — via LinkedIn or engineering school (FEUP, ISEP in Porto). Often faster and easier to sign an NDA with a local individual.

**I strongly recommend getting a review before ordering.** A first-time PCB layout benefits enormously from expert eyes. Common issues a reviewer catches: buck converter loop too large, decoupling caps too far from pins, ground plane split awkwardly, differential pair not routed as a pair, silkscreen colliding with pads. Cost of a review: €100–200. Cost of a wasted board respin: €400 + 3 weeks. Skip this step at your peril.

---

## Part 9 — Timeline

**Day 1**: KiCad setup, project creation, install easyeda2kicad via nix-shell, fetch all component libraries. Register libraries in KiCad.
**Day 2**: Create custom LilyGo socket symbol with correctly named pins and electrical types. Start schematic assembly per Part 2d — Phases 1 through 3 (sheet setup, power chain, ignition sense chain).
**Day 3**: Continue schematic — Phases 4 and 5 (CAN bus chain, LilyGo socket wiring). Add PWR_FLAGs, No Connect flags, run ERC. Address all errors.
**Day 4**: Install kicad-jlcpcb-tools plugin in PCB editor. Board outline, component placement, initial routing.
**Day 5**: Complete routing, ground pours, DRC clean-up.
**Day 6**: When LilyGo arrives — measure with calipers, finalize the LilyGo footprint, final DRC, run the plugin's stock verification, generate manufacturing files.
**Day 6 (evening)**: Send to a reviewer.
**Days 7–9**: Address review feedback.
**Day 9**: Order from JLCPCB.
**Days 9–28**: Wait for boards.
**Day 28+**: Boards arrive fully assembled. Plug in the LilyGos (5 seconds each), insert fuses, connect LiPos and antennas, land the FMS harness wires into the terminal block. Total: ~5 minutes per board of manual assembly. Then bench-test and validate.

So: roughly **1 month from starting the KiCad work to holding a working prototype in your hand.** Faster if the LilyGo arrives quickly and the review is fast; slower if the design has issues or JLCPCB flags stock problems.

---

## Summary

- Design in KiCad, using the kicad-jlcpcb-tools plugin at the PCB-layout stage only (not schematic)
- During schematic entry: verify each part manually against jlcpcb.com/parts, checking manufacturer field, stock, and MPN — watch for clones (see "How to verify an LCSC listing" in Part 2)
- Tag each part with `LCSC` and `MPN` custom fields
- **Use two separate 1×16 female header symbols (J2, J3) for the LilyGo socket** — not one custom combined symbol. This keeps BOM export clean.
- During PCB layout: the plugin verifies stock, checks assembly eligibility, and generates BOM/CPL files
- **Always verify KiCad footprint dimensions against the manufacturer's datasheet before ordering** — mismatched footprints are the most common cause of failed prototype runs
- **Enable both SMT assembly AND through-hole assembly on the JLCPCB order.** Through-hole assembly costs ~€15-25 total across 5 boards and means JLCPCB installs the terminal block, fuse holder, buck, and LilyGo socket headers for you. Absolutely worth enabling.
- **The LilyGo itself is NOT installed by JLCPCB** — it's a module you plug into the socket headers by hand when boards arrive (5 seconds per board, no soldering)
- Order LilyGo today for physical measurements — don't rely on published dimensions
- Get a design review before ordering (freelance reviewer or anonymized Reddit post)
- Expect 3–4 weeks total elapsed time from starting design to working boards
- Expect €300–450 for 5 fully-assembled prototype boards from JLCPCB (SMT + through-hole assembly), plus ~€100 for the LilyGos, batteries, and harnesses. When boards arrive, ~5 minutes per board to plug in the LilyGos, fuses, antennas, and LiPos.
