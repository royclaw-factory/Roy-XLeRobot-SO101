# XLeRobot Upgraded Hardware by MakerMods

An upgraded hardware design for [XLeRobot](https://github.com/Vector-Wangel/XLeRobot), developed by [MakerMods](https://makermods.ai) in collaboration with Vector Wang (original XLeRobot author).

## Demos

**3rd Place Winner at 2025 Seeed × NVIDIA × LeRobot Hackathon**

<div align="center">
  <a href="assets/gifs/hackthon-whole.gif" target="_blank">
    <img src="assets/gifs/hackthon-whole.gif" alt="Watch demo GIF" width="80%"/>
  </a>
  <p><em>Click the GIF to view it full-size</em></p>
</div>

**System demonstrations (autonomous)**

<div align="center">
  <table>
    <tr>
      <td align="center" width="33%">
        <img src="assets/gifs/bimanual-manipulation.gif" alt="Bimanual manipulation demo" width="100%"/>
        <p><em>Bimanual manipulation</em></p>
      </td>
      <td align="center" width="33%">
        <img src="assets/gifs/side-view.gif" alt="Side view demonstration" width="100%"/>
        <p><em>Side view</em></p>
      </td>
      <td align="center" width="33%">
        <img src="assets/gifs/unzip-bagger.gif" alt="Task execution demo" width="100%"/>
        <p><em>Task execution</em></p>
      </td>
    </tr>
  </table>
</div>


## Buy Pre-Assembled

Want an XLeRobot that works right out of the box? MakerMods offers a fully calibrated and assembled XLeRobot — no assembly required. 

**[Buy Now at makermods.ai/xlerobot →](https://makermods.ai/xlerobot)**

---

## What's Upgraded


| Upgrade                  | Description                                             |
| ------------------------ | ------------------------------------------------------- |
| Redesigned neck          | *Description TBD*                                       |
| Redesigned chassis       | *Description TBD*                                       |
| Powerbank mount          | *Description TBD*                                       |
| USB hub mount            | *Description TBD*                                       |
| Under-arm mounts         | *Description TBD*                                       |
| Control board enclosures | *Description TBD*                                       |
| Jetson Nano enclosure    | 3D printable enclosure for on-robot compute             |
| Mac Mini enclosure       | 3D printable enclosure for OpenClaw desktop compute     |
| Raspberry Pi enclosure   | 3D printable enclosure for lightweight on-robot compute |


## Software Compatibility

This hardware is a **drop-in replacement** — no software modifications required.

### LeRobot Integration

Fully integrated with the [Hugging Face LeRobot](https://github.com/huggingface/lerobot) ecosystem. Train and deploy VLA policies, record datasets, and run inference out of the box.

### Supported Control Modes


| Mode            | Description                                                                                                         |
| --------------- | ------------------------------------------------------------------------------------------------------------------- |
| VR Control      | Quest 3 teleoperation via [XLeVR](https://xlerobot.readthedocs.io/en/latest/simulation/getting_started/vr_sim.html) |
| Xbox Controller | Wireless Xbox controller teleoperation                                                                              |
| Switch Joycon   | Nintendo Switch Joycon control                                                                                      |
| Keyboard        | Direct keyboard teleoperation                                                                                       |
| Simulation      | Full MuJoCo simulation support                                                                                      |
| Web Control     | Browser-based control interface                                                                                     |


All modes work with the existing [XLeRobot codebase](https://github.com/Vector-Wangel/XLeRobot) — just plug and play.

### MakerMods App

The MakerMods App provides a desktop interface for teleoperation, dataset recording, and VLA model debugging — designed to work seamlessly with the XLeRobot upgraded hardware.

## Bill of Materials

Hardware components list (excluding compute and shipping).  
`Actual Used Qty` is what is consumed in one build. `Minimum Buyable Qty` is the smallest practical purchase amount.

| Item | Actual Used Qty | Minimum Buyable Qty |
| --- | --- | --- |
| PETG HF filament | 3900 g | 6 rolls PETG |
| TPU 90A filament | 40 g | 1 roll TPU |
| 3M grip material | 256 mm | 1000 mm |
| 12V servo motors | 17 units | 17 units |
| 7.4V servo motors | 12 units | 12 units |
| Lenovo ThinkPlus power bank (140W fast-charging) | 2 units | 2 units |
| Bus servo driver board | 4 units | 4 units |
| 100 cm servo 3-pin cable | 2 units | 2 units |
| USB hub | 2 units | 2 units |
| USB-C cable | 4 cables | 4 cables |
| 12V 3A Type-C to PD trigger cable | 3 cables | 3 cables |
| 12V DC power cable | 2 cables | 2 cables |
| 7.4V DC power cable | 2 cables | 2 cables |
| Wrist-mounted camera module (including camera connection cable) | 1 set | 1 set |
| Top camera module (including camera connection cable) | 1 set | 1 set |
| IKEA utility cart (large) | 1 set | 1 set |
| Caster wheels (without coupler) | 3 units | 3 units |
| Screws and nuts (M3 series) | 1 set | 1 set |

## Assembly Guide

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


| Enclosure          | Use Case                                                        |
| ------------------ | --------------------------------------------------------------- |
| NVIDIA Jetson Nano | On-robot inference and control                                  |
| Mac Mini           | Desktop compute for [OpenClaw](https://github.com/openclaw/openclaw) integration |
| Raspberry Pi       | Lightweight on-robot control                                    |


## Getting Started

1. **Assemble the hardware** — follow the [Assembly Guide](#assembly-guide) or [buy pre-assembled](https://makermods.ai/xlerobot).
2. **Install the XLeRobot software** — follow the [XLeRobot setup instructions](https://github.com/Vector-Wangel/XLeRobot).
3. **Calibrate** — run the calibration script (auto-calibration supported).
4. **Teleoperate or train** — pick a [control mode](#supported-control-modes) and start collecting data.

## About MakerMods

[MakerMods](https://makermods.ai) builds tools that make robotics and embodied AI accessible to everyone — from pre-assembled robots to modular hardware components and training software.

## Acknowledgments

- [Ryan Chan](https://x.com/Ryan_Resolution) — Directed the overall hardware upgrade design and designed the USB hub mount
- **Thomas Lei** — Lead CAD contributor. Designed the neck, chassis, powerbank mount, and Jetson Nano enclosure
- **Yuncheng Xu** — Designed the under-arm mounts and control board enclosures
- [Vector Wang](https://github.com/Vector-Wangel) — Original XLeRobot creator and collaborator on this upgrade
- [Isaac Sin](https://github.com/IsaacSinn) ([X](https://x.com/IsaacSin12)) — Hardware upgrade suggestions and MakerMods App development
- [Liu Qi](https://github.com/Lakesenberg) — Hardware upgrade suggestions and feedback
- [XLeRobot](https://github.com/Vector-Wangel/XLeRobot) — The foundation this project is built upon
- [LeRobot (Hugging Face)](https://github.com/huggingface/lerobot) — Robotics framework
- [SO-101](https://github.com/TheRobotStudio/SO-ARM100) — Arm design used by XLeRobot

## License

This project is licensed under the [MIT License](./LICENSE).

## Contact

- **Website:** [makermods.ai](https://makermods.ai)
- **Email:** [ryan@makermods.ai](mailto:ryan@makermods.ai), [isaac@makermods.ai](mailto:isaac@makermods.ai)
- **Discord:** [Join our community](https://discord.gg/EP9MFcSc)
- **X (Twitter):** [@Ryan_Resolution](https://x.com/Ryan_Resolution), [@IsaacSin12](https://x.com/IsaacSin12)

