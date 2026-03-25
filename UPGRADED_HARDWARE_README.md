# XLeRobot Upgraded Hardware by MakerMods

An upgraded hardware design for [XLeRobot](https://github.com/XLeRobot), developed by [MakerMods](https://makermods.ai) in collaboration with Vector Wang (original XLeRobot author).

<!-- TODO: Add hero image/photo of the upgraded XLeRobot -->
<!-- ![XLeRobot Upgraded Hardware](assets/hero.jpg) -->

## Buy Pre-Assembled

Want an XLeRobot that works right out of the box? MakerMods offers a fully calibrated and assembled XLeRobot — no assembly required.

**[Buy Now at makermods.ai/xlerobot →](https://makermods.ai/xlerobot)**

---

## What's Upgraded

<!-- TODO: Add more detail / descriptions for each upgrade -->

| Upgrade | Description |
|---------|-------------|
| Redesigned neck | _Description TBD_ |
| Redesigned chassis | _Description TBD_ |
| Powerbank mount | _Description TBD_ |
| USB hub mount | _Description TBD_ |
| Under-arm mounts | _Description TBD_ |
| Control board enclosures | _Description TBD_ |
| Jetson Nano enclosure | 3D printable enclosure for on-robot compute |
| Mac Mini enclosure | 3D printable enclosure for OpenClaw desktop compute |
| Raspberry Pi enclosure | 3D printable enclosure for lightweight on-robot compute |

## Software Compatibility

This hardware is a **drop-in replacement** — no software modifications required.

### LeRobot Integration

Fully integrated with the [Hugging Face LeRobot](https://github.com/huggingface/lerobot) ecosystem. Train and deploy VLA policies, record datasets, and run inference out of the box.

### Supported Control Modes

| Mode | Description |
|------|-------------|
| VR Control | Quest 3 teleoperation via [XLeVR](https://github.com/XLeRobot) |
| Xbox Controller | Wireless Xbox controller teleoperation |
| Switch Joycon | Nintendo Switch Joycon control |
| Keyboard | Direct keyboard teleoperation |
| Simulation | Full MuJoCo simulation support |
| Web Control | Browser-based control interface |

All modes work with the existing [XLeRobot codebase](https://github.com/XLeRobot) — just plug and play.

### MakerMods App

The [MakerMods App](https://makermods.ai) provides a desktop interface for teleoperation, dataset recording, and VLA model debugging — designed to work seamlessly with the XLeRobot upgraded hardware.

<!-- TODO: Isaac to expand this section — features, supported platforms, screenshots, download link -->

## Bill of Materials

<!-- TODO: Fill in actual BOM -->

| Part | Qty | Link | Unit Price |
|------|-----|------|------------|
| _Servo motor (STS3215)_ | _6_ | _[Link]()_ | _$12.00_ |
| _3D printed parts kit_ | _1_ | _See below_ | _—_ |
| _USB-C hub_ | _1_ | _[Link]()_ | _$8.00_ |

> **Estimated total: $XXX** (excluding 3D printed parts)

## Assembly Guide

<!-- TODO: Link to or embed the assembly guide -->

> Assembly guide coming soon. In the meantime, [buy pre-assembled](https://makermods.ai/xlerobot) to skip this step entirely.

### 3D Printed Parts

All files are provided in both STL and 3MF formats, verified for **Bambu Lab** printers (H2S, H2D, P2S, P1S).

**Recommended print settings:**
- Material: **eSUN PLA+**
- Layer height: 0.2 mm
- Infill: 40%+
- Supports: Yes (where noted)

Files are located in the [`/stl`](./stl) directory.

#### Compute Enclosures

We include 3D printable enclosures for the following compute platforms:

| Enclosure | Use Case |
|-----------|----------|
| NVIDIA Jetson Nano | On-robot inference and control |
| Mac Mini | Desktop compute for [OpenClaw](https://github.com/) integration |
| Raspberry Pi | Lightweight on-robot control |

<!-- TODO: Add full table of printed parts with filenames and notes -->

## Getting Started

1. **Assemble the hardware** — follow the [Assembly Guide](#assembly-guide) or [buy pre-assembled](https://makermods.ai/xlerobot).
2. **Install the XLeRobot software** — follow the [XLeRobot setup instructions](https://github.com/XLeRobot).
3. **Calibrate** — run the calibration script (auto-calibration supported).
4. **Teleoperate or train** — pick a [control mode](#supported-control-modes) and start collecting data.

## About MakerMods

[MakerMods](https://makermods.ai) builds tools that make robotics and embodied AI accessible to everyone — from pre-assembled robots to modular hardware components and training software.

## Acknowledgments

- **Ryan Chan** — Directed the overall hardware upgrade design and designed the USB hub mount
- **Thomas Lei** — Lead CAD contributor. Designed the neck, chassis, powerbank mount, and Jetson Nano enclosure
- **许运城 (Yuncheng Xu)** — Designed the under-arm mounts and control board enclosures
- [Vector Wang](https://github.com/) — Original XLeRobot creator and collaborator on this upgrade
- **Isaac Sin** — Hardware upgrade suggestions and [MakerMods App](https://makermods.ai) development
- **刘琦 (LiuQi)** — Hardware upgrade suggestions and feedback
- [XLeRobot](https://github.com/XLeRobot) — The foundation this project is built upon
- [LeRobot (Hugging Face)](https://github.com/huggingface/lerobot) — Robotics framework
- [SO-101](https://github.com/) — Arm design used by XLeRobot

## License

<!-- TODO: Choose a license — MIT, Apache 2.0, or CC-BY-SA for hardware -->

This project is licensed under the [MIT License](./LICENSE).

## Contact

- **Website:** [makermods.ai](https://makermods.ai)
- **Email:** [contact@makermods.ai](mailto:contact@makermods.ai)
- **Discord:** [Join our community]()
- **X (Twitter):** [@MakerMods](https://x.com/MakerMods)