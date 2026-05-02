# AirSpy RX Bridge — Beta Releases

Windows installer downloads for the **AirSpy RX Bridge** — a Qt6 C++
application that adds wideband Q65 (QMAP) reception via an **AirSpy R2**
receiver to a station already running a real radio (IC-905, IC-705,
FT-991A, etc.) for TX. Sibling to the HackRF, RTL-SDR, and SDRplay RX
Bridges, with a 12-bit-ADC AirSpy R2 front end and 24 MHz – 1.75 GHz
tuning range — covering 2 m, 70 cm, and just clipping the bottom of
23 cm.

The bridge listens to **WSJT-X UDP** for the dial frequency, tunes the
AirSpy to match, demodulates SSB to **VB-Cable Line 1** for WSJT-X RX
audio, and streams **96 kHz IQ to QMAP** for wideband Q65 decode. Your
real rig keeps doing TX (and narrowband RX, if you prefer). No
interference with your existing CAT / audio setup.

Author: **Andreas Junge, N6NU** &lt;<n6nu@arrl.net>&gt;.

---

## Latest release — v0.99.2 (early alpha)

Download: **[airspy-rx-bridge-0.99.2-setup.exe](airspy-rx-bridge-0.99.2-setup.exe)**

**Multi-instance support.** Run two bridges side-by-side — different
WSJT-X / QMAP instances, no shared state. New `--instance <name>`
CLI flag namespaces the INI / window title; new **Settings → "Linrad
TCP port"** / **"Linrad UDP port"** rows let two bridges feed two
QMAPs without colliding. See RELEASE_NOTES.md for the full
multi-instance walkthrough.

Drop-in upgrade from v0.99.1.

---

## v0.99.1 — Spectrum waterfall toggle

**Spectrum waterfall is now optional.** The built-in waterfall display
can be turned off when you don't need the visual debug info.

- **View → "Show spectrum waterfall"** (or **Ctrl+W**) toggles it.
- New CLI flag `--no-waterfall` launches with the display already off.
- Default ON — same look as v0.99.0 unless you turn it off.
- When off, the bridge skips the per-IQ-buffer FFT compute work, paint
  events, and 20 Hz row-poll timer. Roughly 2–5 % of one CPU core
  saved at 2.5 Msps; more on slower boxes.

Drop-in upgrade from v0.99.0; no INI migration.

### v0.99.0 — first beta

Initial scope: AirSpy R2 (the original 24 MHz – 1.8 GHz, 12-bit, up to
10 Msps receiver). The bridge bundles `airspy.dll` so no separate
AirSpy SDK install is needed.

**What works:**

- **Tunes from WSJT-X dial** via UDP port 2237 — same pattern as the
  HackRF / RTL-SDR / SDRplay siblings. No CAT cable to the bridge.
- **Wideband IQ → QMAP** via the LinradServer UDP path, 16-bit on the
  wire, full 12-bit AirSpy ADC range carries through end-to-end.
- **Demodulated SSB → VB-Cable Line 1** for WSJT-X RX audio, also at
  16-bit width.
- **Three gain modes**: sensitivity preset (0–21, default), linearity
  preset (0–21), or manual LNA + Mixer + VGA (each 0–15). Switchable
  live from Settings.
- **AGC** (LNA + Mixer auto-gain together; VGA stays manual).
- **Sample rate**: 2.5 Msps default (the cleanest fit for QMAP's
  96 kHz window). 10 Msps available for power users with sufficient
  USB / CPU headroom.
- **Transverter offset** (CLI: `--transverter-offset <MHz>`; or
  Settings → "Transverter offset"). Operating freq stays at the dial
  the operator types into WSJT-X / QMAP; SDR tunes to dial+offset.
- **Manual SDR freq override** (Settings → "Manual SDR freq"; CLI:
  `--manual-freq <MHz>`).
- **5-second streaming-stats log line** for diagnosing "is the SDR
  streaming?" / "is WSJT-X feeding freq updates?" at a glance.

**First-install notes:**

- **WinUSB driver via Zadig.** On a fresh install, you may need to
  run [Zadig](https://zadig.akeo.ie) once to bind the **WinUSB** driver
  to the AirSpy R2. If the bridge logs
  `airspy_open() failed: AIRSPY_ERROR_NOT_FOUND` with the device
  plugged in, that's the cause. Same step that HackRF and RTL-SDR
  testers do.
- **VB-Audio Virtual Cable must be 16-bit.** Right-click VB-Cable in
  Windows Sound Properties → Advanced → set both Recording and
  Playback to **16 bit, 48000 Hz**. 24-bit causes the bridge audio
  path to fall back to int32 and WSJT-X then sees silence/garbage.

**What's NOT in this version:**

- AirSpy HF+ Discovery / Dual support (separate testing needed).
- Software PPM / frequency correction (R2's master clock is good to
  a few ppm out of the box).

**Reporting:** to <n6nu@arrl.net>. Please include:

- Does `airspy_open()` succeed and what serial number is logged?
- Does the `[Stats]` line show ~2,500,000 sps with peak |IQ| moving
  as expected when an antenna is connected?
- Does WSJT-X dial → SDR retune work?
- Does QMAP show signal in its wideband panel?
- Any errors from libairspy in the log? Run with `--console` on
  Windows to see the live log.
