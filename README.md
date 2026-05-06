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

## Latest release — v1.1.3

Download: **[airspy-rx-bridge-1.1.3-setup.exe](https://github.com/n6nu/airspy-rx-bridge/releases/latest/download/airspy-rx-bridge-1.1.3-setup.exe)**

What's new in v1.1.3 (2026-05-06) -- DC blocker default-off for
RF-direct receivers (HackRF / RTL-SDR / SDRplay / Pluto / AirSpy)
since their SDR API already does hardware DC correction. Settings
checkbox is grayed out for those radios. Sound-card-IQ sources
(FunCube etc.) keep DC blocker default-on. Drop-in upgrade.

---
### Previous release — v1.1.2

Download: **[airspy-rx-bridge-1.1.2-setup.exe](https://github.com/n6nu/airspy-rx-bridge/releases/latest/download/airspy-rx-bridge-1.1.2-setup.exe)**

What's new in v1.1.2 (2026-05-05) -- DC blocker for zero-IF
receivers. New "DC blocker (zero-IF spike removal)" checkbox in
Settings (default ON) chews through the LO-leakage spike at the
centre of the on-screen waterfall and the QMAP wideband display.
100 Hz notch, irrelevant for Q65 / FT8 audio. Drop-in upgrade.
### Previous release — v1.1.1

Download: **[airspy-rx-bridge-1.1.1-setup.exe](https://github.com/n6nu/airspy-rx-bridge/releases/latest/download/airspy-rx-bridge-1.1.1-setup.exe)**

What's new in v1.1.1 (2026-05-05) -- internal: per-device IQ-rate
capability gating in the Settings combo. RF-side bridges (this one
included) offer the same 96 / 128 / 192 / 256 kHz options as before;
the sound-card-IQ bridge (iq-rx-bridge) now hides rates above the
sound card's actual rate. Drop-in upgrade.
### Previous release — v1.1.0

Download: **[airspy-rx-bridge-1.1.0-setup.exe](https://github.com/n6nu/airspy-rx-bridge/releases/latest/download/airspy-rx-bridge-1.1.0-setup.exe)**

What's new in v1.1.0 (2026-05-05) -- UI refresh. Main window now
fixed-size 400x640. Settings moved from a button to a top-level
**Settings** menu (`Ctrl+,`). New **Linrad rate** readout in the
State grid. Settings dialog reflowed side-by-side, with a new
**Linrad IQ rate** combo (defaults to "96 kHz (QMAP Default)").
Drop-in upgrade; 96 kHz wire format unchanged.
### Previous release — v0.99.2 — **early alpha**

Download: **[airspy-rx-bridge-0.99.2-setup.exe](airspy-rx-bridge-0.99.2-setup.exe)**

> ⚠ **Status: early alpha.** This is the newest port in the n6nu
> sdr-bridges family and the only one not yet promoted to the
> 1.0 line. Expect rough edges; the AirSpy R2 path has been
> bench-tested but not soaked through extended on-air sessions.
> Sibling bridges (HackRF / RTL-SDR / SDRplay / iq-rx) are at
> v1.0.x stable.

This rebuild also picks up the bridge-core waterfall span fix
(display labels now match the actual IQ rate instead of the
hardcoded 2 MHz default).

### v0.99.2 — Multi-instance support (multi-band ops)

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
