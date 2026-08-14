# IPTSD

This repository is a downstream fork of the userspace touch processing daemon
for Microsoft Surface devices using Intel Precise Touch technology. Its lineage
is:

```text
linux-surface/iptsd
  -> alex-lentz/iptsd
  -> validated base commit 3663e96e758145801c3cb1ce7c72f362d0d4a5f5
  -> andreasbjorshol/iptsd
  -> sl7-0c77-fixes
```

The daemon reads HID reports containing raw capacitive touch data, stylus
coordinates, and DFT pen measurements, then creates standard input events using
uinput devices.

## Downstream changes

The validated base includes Alex Lentz's work on the Surface Laptop 7 Intel
haptic touchpad, on top of the upstream linux-surface project:

- **Physical click on Sensel haptic touchpads** — Parses IPTS report frame type
  `0x94`, which carries touchpad button state on devices such as the Surface
  Laptop 7 Intel, and routes the physical button state through the existing
  `on_button` path so `BTN_LEFT` is emitted correctly.
- **Recovery after sleep when a hidraw device persists** — Adds a
  `systemd-sleep` hook that re-fires udev `add` events for hidraw devices after
  resume, and retries `iptsd@.service` after transient startup failures.

Branch `sl7-0c77-fixes` adds exactly two further lifecycle fixes for the
Snapdragon Surface Laptop 7 configuration:

- **Preserve singletouch state across unstable frames** — A selected, valid
  contact reported with `stable=false` is treated as an unstable measurement,
  not as a physical lift. This prevents premature `BTN_LEFT` release during
  physical click-and-drag.
- **Release touch state on daemon shutdown** — `Daemon::on_stop()` calls
  `TouchDevice::disable()` so touch and button state are explicitly released
  before the uinput device is destroyed.

The detector, tracker, stabilizer, palm detection, and button debounce behavior
are unchanged and are not part of these patches. The detailed technical
explanation is in [docs/sl7-touchpad-fixes.md](docs/sl7-touchpad-fixes.md).

These downstream changes are not part of upstream
[linux-surface/iptsd](https://github.com/linux-surface/iptsd).

### Target hardware and validation

The additional fixes target the physically validated:

- Microsoft Surface Laptop 7 13.8"
- Snapdragon X Elite / X1E80100
- ARM64 Linux
- Touchpad VID:PID `045E:0C77`

Physical validation results:

- pointer: PASS
- physical click: PASS
- drag-and-drop: PASS
- long-held drag: PASS
- direction changes while dragging: PASS
- two-finger scrolling: PASS
- clean daemon shutdown: PASS
- touchpad click after daemon replacement: PASS
- external mouse click after daemon replacement: PASS

## Credits

IPTSD is developed and maintained by the
[linux-surface](https://github.com/linux-surface) team, primarily
[Maximilian Luz](https://github.com/quo) and
[Dorian Stoll](https://github.com/StollD). All core functionality originates
from the upstream project. The inherited Surface Laptop 7 Intel patches were
authored by Alex Lentz with assistance from [Claude](https://claude.ai).

## Installing

The downstream patches in this repository are not part of the linux-surface
package repository. Installing IPTSD through the normal linux-surface packages
will therefore install upstream IPTSD without these changes.

This fork does not publish prebuilt packages or add installation automation.
For the Snapdragon Surface Laptop 7 configuration, build and run the standalone
binary as described below. If these downstream changes are not needed, use
[upstream linux-surface/iptsd](https://github.com/linux-surface/iptsd#installing)
and its official packages instead.

**Important:** Support on Debian-based distributions only goes back to Debian 11
/ Ubuntu 22.04.

## Building

To build IPTSD from source, you need:

 * A C++ compiler
 * meson
 * ninja
 * CLI11
 * Eigen3
 * fmt
 * inih / INIReader
 * gsl
 * spdlog
 * cmake, because some dependencies do not ship pkgconfig files

To build the plotting tools for visualizing data, you also need cairomm and
SDL2. Most dependencies can be downloaded and included automatically by meson
if they are not provided by the distribution.

Clone this downstream repository and select the validated branch:

```bash
git clone https://github.com/andreasbjorshol/iptsd.git
cd iptsd
git switch sl7-0c77-fixes
meson setup build
ninja -C build src/iptsd
```

For the validated Surface Laptop 7 configuration, run the standalone daemon
with the Chromium HID-over-SPI touchpad device:

```bash
sudo ./build/src/iptsd /dev/surface-touchpad
```

This workflow does not install the binary over `/usr/local/bin/iptsd` or modify
the system configuration.
