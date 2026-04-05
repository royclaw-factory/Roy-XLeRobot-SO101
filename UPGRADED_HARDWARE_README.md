# XLeRobot Upgraded Hardware by MakerMods

**Redesigned 3D-printed structure, stronger wheel base, quick-release powerbank mounts, and organized electronics** for the [XLeRobot](https://github.com/Vector-Wangel/XLeRobot) bimanual mobile robot -- drop-in compatible with [LeRobot](https://github.com/huggingface/lerobot), no software changes needed.

Built by [MakerMods](https://makermods.ai) in collaboration with Vector Wang (original XLeRobot author). Available **pre-assembled** or as a **curated kit** (coming soon) at [makermods.ai](https://makermods.ai).

## MakerMods XLeRobot Hardware Upgrade

<div align="center">
  <img src="assets/images/hero_shot_17_04_02_18.png" alt="MakerMods XLeRobot Hardware Upgrade" width="85%"/>
</div>

### At a glance

- **Redesigned structural parts** -- two-part neck, bearing-supported wheel modules, quick-release power bank mount, organized hub and driver board enclosures. Only the head and SO-101 arm servo mounts are stock.
- **Print-tested `.3mf`** -- one Bambu Studio project (H2S / P2S, Esun PLA+) with labeled plates, supports, and settings -- the same file we use internally.
- **Hackathon-proven** -- deployed **25 upgraded XLeRobots** for contestants at the Shenzhen MakerMods Physical AI x OpenClaw Hackathon; all units ran successfully throughout the event.
- **Pre-assembled or DIY** -- **pre-built** skips ~two full days of assembly; you run our **auto-calibration script** at setup. **Kit** with MakerMods-sourced parts **coming soon** -- see [Buy Pre-Assembled](#buy-pre-assembled).
- **Works directly with [XLeRobot](https://github.com/Vector-Wangel/XLeRobot) and [LeRobot](https://github.com/huggingface/lerobot)** -- clone the existing repos and follow their setup guides as-is; no fork, no patches, no custom firmware.
- **Updates** -- [makermods.ai](https://makermods.ai) and [Discord](https://discord.gg/EP9MFcSc) for kit news and build support.

## Buy Pre-Assembled

Want your XLeRobot **running without ~two full days of mechanical assembly**? MakerMods sells an **assembled** unit so you can get to bring-up, software, and experiments much faster than a from-scratch DIY build.

**Calibration:** We **do not** advertise "fully calibrated out of the box." After unboxing, you run our **auto-calibration** workflow (script) -- it's designed to be **quick and repeatable** on your bench.

**[Buy pre-assembled at makermods.ai/xlerobot -->](https://makermods.ai/xlerobot)**

> **Kit (coming soon):** If you **enjoy the assembly**, we're launching a **kit** on [makermods.ai](https://makermods.ai) soon. It uses **MakerMods-curated** components so we can **stand behind part quality** and **save you the time** of sourcing every BOM line yourself versus retail-hunting each item.

---

## Demos

**Full walkthrough video:** *[Add YouTube (or hosted) URL here when published -- this is what travels best on social.]*

**Hackathon-Proven: 25 Upgraded XLeRobots at the Shenzhen Physical AI X OpenClaw Hackathon**

<div align="center">
  <a href="assets/gifs/hackthon-whole.gif" target="_blank">
    <img src="assets/gifs/hackthon-whole.gif" alt="Watch demo GIF" width="80%"/>
  </a>
  <p><em>Click the GIF to view it full-size</em></p>
</div>

**System demonstration**

<div align="center">
  <table>
    <tr>
      <td align="center" width="33%">
        <img src="assets/gifs/candy_demo.gif" alt="Bimanual manipulation demo" width="100%"/>
        <p><em>Bimanual manipulation</em></p>
      </td>
      <td align="center" width="33%">
        <img src="assets/gifs/side-view.gif" alt="Side view demonstration" width="100%"/>
        <p><em>Autonomous execution (side view)</em></p>
      </td>
      <td align="center" width="33%">
        <img src="assets/gifs/unzip-bagger.gif" alt="Task execution demo" width="100%"/>
        <p><em>Autonomous execution</em></p>
      </td>
    </tr>
  </table>
</div>

---

## Mechanical design upgrades

This section covers the **hardware changes**: what we re-designed in CAD vs a typical stock XLeRobot-style build, and **why it is nicer to print, mount, and service**. Everything here is **mechanical** (printed parts, mounts, enclosures). You use the [XLeRobot](https://github.com/Vector-Wangel/XLeRobot) and [LeRobot](https://github.com/huggingface/lerobot) repositories directly -- no modifications needed on the software side.

**Stock vs redesigned plastic:** Only the **servo mounts on the head** and the **servo mounts on the two SO-101 arms** remain **stock** (upstream SO-101 / XLeRobot-style parts). **All other structural prints** in our release -- neck, IKEA cart integration, drivable wheel base, hub and driver mounts, **quick-release dual power bank mount** (below the neck), enclosures, and the rest of the mechanical stack -- are **MakerMods redesigned**.

### Two-part neck (printability)

We **split the neck into two printed parts** so each piece is **easier to orient and print successfully** on common FDM printers (fewer tall single-piece jobs).

<div align="center">
  <img src="assets/mechanical/neck-two-part.jpg" alt="Two-part neck" width="60%"/>
</div>

### SO-101 arm mounts, IKEA cart, and nut traps

The **large IKEA utility cart** is the robot's **main rolling structure** (frame + shelves) -- think of it as the **body** you bolt everything to. The **SO-101 arm mounting** is reworked so the arms **attach to that cart cleanly** -- less fussing with alignment and bracketry than ad-hoc prints.

**Nut traps:** every **hexagon nut trap is replaced with a square nut trap**. Nuts **slide in from the side** into their pockets. When you **disassemble** a joint or bracket, **captured square nuts do not fall out** the way loose or bottom-dropping hex nuts often do -- faster iteration when you are tuning wiring, swapping parts, or shipping the robot.

<div align="center">
  <img src="assets/mechanical/so101-arm-mounts.jpg" alt="SO-101 arm mounts on IKEA cart" width="60%"/>
</div>

### Drivable base at the bottom of the cart (wheel modules + bearings)

The **bottom of the IKEA cart** is where **locomotion** lives -- this is **not** the same thing as the cart's metal frame above; we call that region the **drivable base**.

**What was wrong with the stock setup:** the original IKEA configuration uses the **LeKewi** assembly as the base reference (per our BOM naming). In practice each wheel was effectively **supported on one side only**: the **servo / drive side** carried the load, while the **other side** was just the **wheel and its hub circle** with **no opposing support**. There was **nothing on the far side of the wheel** to react side loads, so the assembly could **tilt and rock** -- especially under the **full weight of an XLeRobot**. That asymmetry shows up as **wobble**, uneven contact, and a base that feels **under-constrained**.

**What we changed:** we treat **each wheel as its own module**. On one side you still have the **servo** that drives the wheel; on the **opposite side** we add a **bearing** so the axle is **supported from both sides**. Load is shared across **servo + bearing**, the wheel stays **square to the ground**, and the cart **handles XLeRobot mass** without the old **single-sided cantilever** behavior.

<div align="center">
  <img src="assets/mechanical/drivable-base-wheels.jpg" alt="Drivable base with bearing-supported wheel modules" width="60%"/>
</div>

### USB hub mounts

Dedicated **USB hub mounts** put hubs where the **cable runs stay short** and **snag less** than zip-tying hubs to random surfaces. Hubs stay in the same place build after build, which matters when you are debugging USB dropouts or packing the robot for travel.

<div align="center">
  <img src="assets/mechanical/usb-hub-mount.jpg" alt="USB hub mount installed" width="60%"/>
</div>

### Control board mounts & driver enclosures

**Bus servo driver boards** get **proper mounts and enclosures**: mechanical support, **strain relief** for harnesses, and a degree of **protection** (shorts, dust, bumps) vs bare PCBs floating in the volume. Together with the hub mounts, this is how we keep **power + USB + servo wiring** organized on a **rolling IKEA cart**.

<div align="center">
  <img src="assets/mechanical/control-board-enclosure.jpg" alt="Control board mount and driver enclosure" width="60%"/>
</div>

### Dual power bank mount (below the neck, quick release)

The structure **directly under the neck** includes a **quick-release cradle for two Lenovo Fluxo 140 W USB-PD power banks** (per [BOM](#bill-of-materials)). Banks **drop into place** and are held by a **compliant push tab** that acts as the **latch** -- no screws to chase when you need to **swap or charge** a pack in the field. It keeps both units **aligned** with the **harness plan** so USB-C bends and connector strain stay under control.

<div align="center">
  <img src="assets/mechanical/power-bank-mount.jpg" alt="Dual power bank quick-release mount" width="60%"/>
</div>

### Compute enclosure (Jetson Nano)

Printable shell for **on-robot** compute (**NVIDIA Jetson Nano**) so the board is not bare and sliding around the cart.

<div align="center">
  <img src="assets/mechanical/compute-enclosure-jetson.jpg" alt="Jetson Nano compute enclosure" width="60%"/>
</div>

### Upgrade checklist (quick scan)

| Area | What changed (one line) |
| ---- | ------------------------ |
| Neck | **Two printed parts** for easier FDM printing |
| SO-101 + IKEA cart | **Arm mounts** for the **cart frame**; **square side-loaded nut traps** (no falling hex nuts) |
| Cart bottom / wheels | **Per-wheel modules**: **servo + opposing bearing** (replaces asymmetric stock **LeKewi**-style support) |
| USB hubs | **Dedicated mounts**, shorter / safer cable paths |
| Driver boards | **Mounts + enclosures**, cleaner wiring |
| Power | **Quick-release** mount **below neck** for **2x Lenovo Fluxo 140 W**; **compliant tab latch** |
| Compute | **Jetson Nano** printable enclosure |

## Stock vs this repository

| Topic | Upstream XLeRobot (baseline) | This repository (MakerMods hardware refresh) |
| ----- | ---------------------------- | --------------------------------------------- |
| What you get | Reference robot design, software, and community BOM paths | **Replacement / add-on printed parts**, mounts, and enclosures for the same software stack |
| Printed parts scope | Mostly upstream STLs | **Stock:** head + **both SO-101 arm** servo mounts only. **Redesigned:** all other structural parts -- plus a **single print-tested `.3mf`** (H2S/P2S, Esun PLA+) with **labeled plates** |
| Mechanical packaging | Baseline community CAD | **Two-part neck** (printability); **SO-101 mounts** on **IKEA cart**; **square side-loaded nut traps**; **printed upper stack** on cart; **drivable base** with **bearing-supported wheel modules** (vs stock **LeKewi** asymmetry); **hub + driver** mounts/enclosures |
| Field power | Per upstream BOM choices | **2× Lenovo Fluxo 140 W** in a **quick-release mount below the neck** (compliant **latch tab**) + cabling aligned to the **printed layout** on the cart |
| USB / drivers | User-dependent layout | **Hub + driver mounts/enclosures** for a repeatable install |
| Compute | Bring your own | **Jetson Nano** printable enclosure |

## Specs (hardware configuration for this BOM)

Figures below are **for this repo's BOM snapshot**; arm kinematics, payload, and exact DOF follow upstream XLeRobot -- see upstream docs for arm-specific numbers.

| Item | Details |
| ---- | ------- |
| Software compatibility | Use [XLeRobot](https://github.com/Vector-Wangel/XLeRobot) and [LeRobot](https://github.com/huggingface/lerobot) directly -- no fork or modifications needed |
| Cameras | **Same camera module** mounted at **wrist** and **head** (two units). Using one camera type in both places **reduces cost** versus using two different cameras; **perception quality remains strong** in our builds. |
| Servos | **17x** 12 V + **12x** 7.4 V bus servos (per BOM) |
| Power | **2x** Lenovo **Fluxo** **140 W** USB-PD power banks (per BOM) |
| Mobile base | IKEA **large** utility cart + casters (per BOM) |
| DOF / reach / payload | Same as upstream XLeRobot configuration you mirror -- confirm in [XLeRobot documentation](https://xlerobot.readthedocs.io/) |

## Software Compatibility

This hardware is a **drop-in replacement** for the stock printed/mechanical stack. You install and run the [XLeRobot](https://github.com/Vector-Wangel/XLeRobot) and [LeRobot](https://github.com/huggingface/lerobot) repositories exactly as documented -- same setup steps, same commands, same configs. No fork, no patches, no environment changes.

### LeRobot Integration

Use the [Hugging Face LeRobot](https://github.com/huggingface/lerobot) repo directly: train and deploy VLA policies, record datasets, and run inference with the same workflows and scripts as a stock XLeRobot build.

> **Auto-calibration PR (pending review):** We have an open pull request on LeRobot that adds **automatic calibration for all SO-101-class arms**: [huggingface/lerobot#3282](https://github.com/huggingface/lerobot/pull/3282). If you want to try it with **LeRobot v0.5.0**, you can check out that branch directly. Once merged, SO-101 users -- including XLeRobot-style bimanual setups -- get a **cleaner, more repeatable calibration workflow** inside the official framework.

### Supported Control Modes

| Mode | Description |
| ---- | ----------- |
| VR Control | Quest 3 teleoperation via [XLeVR](https://xlerobot.readthedocs.io/en/latest/simulation/getting_started/vr_sim.html) |
| Xbox Controller | Wireless Xbox controller teleoperation |
| Switch Joy-Con | Nintendo Switch Joy-Con control |
| Keyboard | Direct keyboard teleoperation |
| Simulation | Full MuJoCo simulation support |
| Web Control | Browser-based control interface |

All modes work directly with the existing [XLeRobot](https://github.com/Vector-Wangel/XLeRobot) and [LeRobot](https://github.com/huggingface/lerobot) repositories -- no extra configuration required.

## Getting Started (first ~30 minutes)

Goal: end with **teleoperation or simulation running**, not just a cloned repo.

1. **Choose your path**
   - **DIY:** slice and print from the **master `.3mf`** in [`/stl`](./stl) (Bambu **H2S / P2S**, **Esun PLA+** -- see [3D Printed Parts](#3d-printed-parts)), source the [BOM](#bill-of-materials), and assemble (interim guidance under [Assembly Guide](#assembly-guide)).
   - **Fast path:** [buy pre-assembled](https://makermods.ai/xlerobot) and skip roughly **two days** of assembly; then run **auto-calibration** with our script.
2. **Prepare a workstation** -- Ubuntu or macOS per upstream docs; GPU optional for sim, required for some local training paths.
3. **Install XLeRobot software** -- clone the [XLeRobot repository](https://github.com/Vector-Wangel/XLeRobot) and follow its [documentation](https://xlerobot.readthedocs.io/) for environment setup and dependencies. No extra steps are needed for the upgraded hardware -- the existing install instructions apply as-is.
4. **Connect hardware (physical robot)** -- power, USB hubs, and servo buses as per your wiring checklist; fix any cable strain before first motion.
5. **Calibration** -- run the upstream calibration / teleop entry flow (auto range-of-motion style calibration is supported in the stack -- see [teleop docs](https://xlerobot.readthedocs.io/en/latest/software/getting_started/XLeRobot_teleop.html) and project README). For **SO-101 + LeRobot**, see the pending upstream PR described under [LeRobot integration](#lerobot-integration).
6. **First motion** -- start **simulation** or **hardware teleop** using a [supported control mode](#supported-control-modes). You should see **joints move under command** or **sim state updating** within one session.
7. **Optional** -- join [Discord](https://discord.gg/EP9MFcSc) for build help and MakerMods updates.

## Bill of Materials

Hardware components list (**excluding compute and shipping**).
`Actual Used Qty` is consumed in one build. `Minimum Buyable Qty` is the smallest practical purchase unit.

**Cameras:** The **head** and **wrist** use the **same camera module** -- you need **two** units total. That keeps the BOM simpler and cheaper than using different cameras for each mount, and we have **not seen practical issues** in use.

| Item | Actual Used Qty | Minimum Buyable Qty |
| ---- | --------------- | ------------------- |
| Esun PLA+ filament | 3900 g | 4 rolls of PLA+ |
| TPU 90A filament | 40 g | 1 roll TPU |
| 3M grip material | 256 mm | 1000 mm |
| 12V servo motors | 17 units | 17 units |
| 7.4V servo motors | 12 units | 12 units |
| Lenovo Fluxo power bank (140 W USB-PD) | 2 units | 2 units |
| Bus servo driver board | 4 units | 4 units |
| 100 cm servo 3-pin cable | 2 units | 2 units |
| USB hub | 2 units | 2 units |
| USB-C cable | 4 cables | 4 cables |
| 12V 3A Type-C to PD trigger cable | 3 cables | 3 cables |
| 12V DC power cable | 2 cables | 2 cables |
| 7.4V DC power cable | 2 cables | 2 cables |
| Camera module (same type for wrist and head; each includes connection cable) | 2 units | 2 units |
| IKEA utility cart (large) | 1 set | 1 set |
| Caster wheels (without coupler) | 3 units | 3 units |
| Screws and nuts (M3 series) | 1 set | 1 set |

**Estimated DIY materials total:** *Add a single headline number when you have a defensible retail sum (same exclusions: compute, tools, shipping). Until then, total the table above at your suppliers.*

## Assembly Guide

> **Full step-by-step guide:** coming soon -- it's the biggest adoption unlock for DIY builders.

**Interim assets (please add when ready):**

- Exploded view or **labeled major subassemblies** photo.
- **5-8 step** photo strip for "day one" assembly milestones.

Until those ship, lean on the [BOM](#bill-of-materials), the **master `.3mf`** and any STLs in [`/stl`](./stl), and upstream XLeRobot mechanical references -- or [buy pre-assembled](https://makermods.ai/xlerobot) to skip multi-day assembly (then auto-calibrate).

### 3D Printed Parts

**Use the master `.3mf` (recommended).** We ship **one Bambu Studio project** in [`/stl`](./stl) that is **validated on Bambu Lab H2S and P2S** -- the **same file we use internally** when we print parts for our own XLeRobots. It contains **every redesigned structural part** plus the **stock head and SO-101 arm servo mounts**, organized as **labeled build plates**, with **support settings and process parameters** already set. The project is **print-tested** end to end; you should not need to guess plate layouts or support toggles for the core robot.

**Filament:** **Esun PLA+** (same as the [BOM](#bill-of-materials)) is what we recommend for that `.3mf`.

**Other formats:** Individual **STL** files are also available in [`/stl`](./stl) if you slice on another printer or workflow -- then you must choose your own layer height, infill, and supports. For **H2S / P2S**, the `.3mf` is the supported path.

**Printers:** The bundled `.3mf` targets **H2S and P2S**. Other Bambu models or third-party printers are **use STLs at your own risk** unless we document them later.

#### Compute Enclosures

| Enclosure | Use Case |
| --------- | -------- |
| NVIDIA Jetson Nano | On-robot inference and control |

## Contributing

We welcome **PRs** and issues that improve real builds:

- Assembly documentation, wiring diagrams, and printable tweaks.
- New **enclosure** variants or mount plates for common peripherals.
- Integration notes (cameras, hubs, alternate carts) with test evidence.

Start a thread in **[Discord](https://discord.gg/EP9MFcSc)** for larger proposals, or open a focused issue/PR in this repo.

## About MakerMods

[MakerMods](https://makermods.ai) builds tools that make robotics and embodied AI accessible -- from **pre-assembled robots** and (soon) **curated kits** to **software** and **community** (see [Discord](https://discord.gg/EP9MFcSc)).

## Acknowledgments

- [Ryan Chan](https://github.com/RyanResolutions) ([X](https://x.com/Ryan_Resolution)) -- Directed the overall hardware upgrade design; USB hub mount.
- **Thomas Lei** -- *Past* CAD contributor: neck, **IKEA cart integration** (upper prints + related layout), power bank mount, and Jetson Nano enclosure *(historical credit; not active maintenance).*
- **Yuncheng Xu** -- Under-arm mounts and control board enclosures.
- [Vector Wang](https://github.com/Vector-Wangel) -- Original XLeRobot creator; collaborator on this upgrade.
- [Isaac Sin](https://github.com/IsaacSinn) ([X](https://x.com/IsaacSin12)) -- Hardware input and MakerMods software development.
- [Liu Qi](https://github.com/Lakesenberg) -- Hardware feedback.
- [XLeRobot](https://github.com/Vector-Wangel/XLeRobot) -- Software and robot foundation.
- [LeRobot (Hugging Face)](https://github.com/huggingface/lerobot) -- Robotics framework.
- [SO-101](https://github.com/TheRobotStudio/SO-ARM100) -- Arm lineage used by XLeRobot.

### Maintainers

Active maintenance and product questions: **Ryan Chan** and **Isaac Sin** -- see [Contact](#contact) and [Discord](https://discord.gg/EP9MFcSc).

## Pre-publish checklist (maintainers)

Run through this **before** a major public push (homepage, launch post, press). Several URLs in this file are easy to get wrong or leave as placeholders.

| Check | What to verify / replace |
| ----- | ------------------------- |
| **Discord invite** | Every community link currently uses `https://discord.gg/EP9MFcSc`. **Confirm this is the correct, permanent invite** (not expired, not a test server). If it changes, **search the repo** for `discord.gg` / `EP9MFcSc` and **update all occurrences** so At a glance, Getting Started, Contributing, About, Maintainers, and Contact stay in sync. |
| **Demo video** | [Demos](#demos): replace the **YouTube / hosted URL** placeholder with the real link. |
| **LeRobot calibration PR** | [LeRobot integration](#lerobot-integration): **paste the real PR URL** when you want visibility (and remove or shorten the italic note). |
| **Hero still** | [MakerMods XLeRobot Hardware Upgrade](#makermods-xlerobot-hardware-upgrade): add `assets/hero.jpg` when ready (or update the caption). |
| **BOM headline price** | [Bill of Materials](#bill-of-materials): fill in **estimated DIY total** when you have a defensible number. |
| **Kit / buy copy** | [Buy Pre-Assembled](#buy-pre-assembled): add **kit** pricing or URL lines when the product page exists. |
| **Master `.3mf` path** | [3D Printed Parts](#3d-printed-parts): ensure the **filename** in [`/stl`](./stl) matches what you tell builders (update README if the file is renamed). |
| **Mechanical photos** | Optional: drop images under [`assets/mechanical/`](./assets/mechanical/) per the suggested filenames in [Mechanical design upgrades](#mechanical-design-upgrades). |
| **Link sanity** | Click-test **makermods.ai**, **xlerobot** docs, **buy** link, and **Discord** from an incognito window. |

## License

This project is licensed under the [MIT License](./LICENSE).

## Contact

- **Website:** [makermods.ai](https://makermods.ai)
- **Email:** [ryan@makermods.ai](mailto:ryan@makermods.ai), [isaac@makermods.ai](mailto:isaac@makermods.ai)
- **Discord:** [Join our community](https://discord.gg/EP9MFcSc)
- **X (Twitter):** [@Ryan_Resolution](https://x.com/Ryan_Resolution), [@IsaacSin12](https://x.com/IsaacSin12)
