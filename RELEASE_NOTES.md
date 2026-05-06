# AirSpy RX Bridge — Release Notes

## v1.1.2 — DC blocker for zero-IF receivers (2026-05-05)

DC blocker for zero-IF receivers, removes the LO-leakage spike that
FunCube Pro+ V2 / HackRF / RTL-SDR / Pluto / AirSpy leak at the centre
of the spectrum. Per-sample IIR high-pass at the front of both the
on-screen waterfall (FftEngine) and the QMAP wire path (LinradServer);
cutoff = 100 Hz, well below any audio offset Q65 / FT8 cares about.

Toggle in Settings → "DC blocker (zero-IF spike removal)", default ON.
Toggling on also resets the I/Q balance EMA so a stale DC accumulator
from earlier samples doesn't keep subtracting against now-DC-free
input for ~2 s.

Drop-in upgrade from v1.1.1. INI key linrad/dc_block_enabled added
(default true; honours the previous behaviour for anyone who never
opens Settings).

## v1.1.1 — capability-gated IQ rate combo (2026-05-05)

Tightens the IQ-rate combo. Internally adds a per-device capability
list; only sound-card-IQ devices currently restrict their offered
rates -- RF-side bridges (HackRF / RTL-SDR / SDRplay / Pluto / AirSpy)
are unchanged in behaviour and still offer 96 / 128 / 192 / 256 kHz.

Drop-in upgrade from v1.1.0. No INI changes.

## v1.1.0 — UI refresh: fixed window, Settings menu, Linrad rate readout (2026-05-05)

User-visible polish across the bridge UI; no behavioural changes on
the wire (96 kHz IQ format unchanged).

- **Fixed-size 400x640 main window**. Replaces the freely-resizable
  640x540 minimum. Window opens identically every session and
  doesn't drift; conditional banners (manual-freq override,
  transverter IF readout) word-wrap rather than clip.
- **Settings is a top-level menu** in the menu bar (shortcut
  `Ctrl+,`) -- was a button at the bottom of the State group. Frees
  ~40 px of vertical real estate for the waterfall.
- **Linrad rate readout** in the State grid, between the device row
  and RX status. Reads the active LinradServer output rate; matches
  what's persisted in the INI.
- **Settings dialog reflow**: radio gain panel and bridge-wide
  group sit horizontally side-by-side. Was vertical, ran past the
  bottom of 1080p laptops with the deeper panels.
- **New "Linrad IQ rate" combo** in the Settings dialog. Defaults
  to "96 kHz (QMAP Default)" -- same wire format as before; matches
  every shipped QMAP release.
- UDP data port (`50004`) and Linrad host (`127.0.0.1`) /
  TCP port (`49812`) editors gained tooltips explaining their use
  in multi-instance setups.

INI compatible with v1.0.x. Drop-in upgrade.

## v0.99.3 — opt-in CAT server for WSJT-X Doppler tracking (2026-05-04)

Adds the same opt-in CAT pattern as the rest of the family. With
WSJT-X **Rig = Hamlib NET rigctl** at `127.0.0.1:4539`, Doppler
tracking commands corrected freq directly to the bridge.

- Rigctld TCP server on port **4539**, default OFF.
- Toggle in Settings → CAT server (or `--cat`). Restart bridge.
- Auto-detect UDP mute when a CAT client is connected.
- Live source indicator in window title.
- Picks up the bridge-core data-mode-PTT fixes (PKTUSB / PKTLSB).

INI compatible with v0.99.2. Drop-in upgrade.

## v0.99.2 — multi-instance support (multi-band ops) (2026-05-02)

Run two bridges side-by-side — different WSJT-X / QMAP instances,
no shared state.

- New `--instance <name>` CLI flag namespaces the INI file, window
  title, and taskbar entry. `airspy-rx-bridge.exe --instance 1296`
  reads/writes `AirSpy RX Bridge - 1296.ini`.
- New **Settings → "Linrad TCP port"** + **"Linrad UDP port"**
  spinboxes (defaults 49812 / 50004). Increment per bridge instance
  for multi-QMAP setups. CLI: `--linrad-tcp-port`,
  `--linrad-udp-port`. Take effect on next launch.

See RTL-SDR v0.99.8 notes for a full multi-instance walkthrough.
AirSpy-side device-serial picker (for two AirSpy R2s on one machine)
lands in a future version bump.

Drop-in upgrade from v0.99.1.

## v0.99.1 — spectrum waterfall toggle (2026-05-02)

The built-in spectrum / waterfall display can now be turned off from
the **View menu** (or the **Ctrl+W** shortcut). Useful when you don't
need the visual debugging and would rather not pay the CPU cost.

- View → "Show spectrum waterfall" — checkable, default on.
- New CLI flag `--no-waterfall` launches with the display off and
  persists the choice to INI key `gui/waterfall_enabled`.
- When off, three layers of work are skipped: per-IQ-buffer
  `FftEngine::pushIq()` (the per-sample int16→float + Hann window
  multiply at the full 2.5 Msps rate), the widget paint events,
  and the 20 Hz row-poll timer. Roughly 2–5 % of one CPU core saved.
- Default ON for first installs.

Drop-in upgrade from v0.99.0; no INI migration.

## v0.99.0 — first beta (2026-05-01)

First beta release. Initial scope: **AirSpy R2** (the original 24 MHz –
1.8 GHz, 12-bit, 10 Msps receiver).

**What works:**

- **Tunes from WSJT-X dial** via UDP port 2237 — same pattern as the
  HackRF / RTL-SDR / SDRplay siblings. No CAT cable to the bridge.
- **Wideband IQ → QMAP** via the LinradServer UDP path (16-bit, full
  12-bit AirSpy ADC range carries through end-to-end).
- **Demodulated SSB → VB-Cable Line 1** for WSJT-X RX audio, also at
  16-bit width.
- **Three gain modes**: sensitivity preset (0–21, default), linearity
  preset (0–21), or manual LNA + Mixer + VGA (each 0–15). Gain mode is
  switchable live from Settings.
- **AGC** (LNA + Mixer auto-gain together; VGA stays manual).
- **Sample rate**: 2.5 Msps default (the cleanest fit for QMAP's
  96 kHz window). 10 Msps available for power users with sufficient
  USB / CPU headroom.
- **Transverter offset** (CLI: `--transverter-offset <MHz>`; or
  Settings → "Transverter offset"). Operating freq stays at the dial
  the operator types into WSJT-X / QMAP; SDR tunes to dial+offset.
  Same shape as the SDRplay / HackRF / RTL-SDR siblings.
- **Manual SDR freq override** (Settings → "Manual SDR freq"; CLI:
  `--manual-freq <MHz>`). Decouples bridge from WSJT-X dial — useful
  for monitoring scenarios where WSJT-X is at a different freq than
  what you want the SDR pinned to.
- **5-second streaming-stats log line**: `[Stats] RX <N> samples in 5s
  (≈ X sps), peak |IQ|=<X> (<Y> dBFS), last freq update <ms> ms ago`
  for diagnosing "is the SDR streaming?" / "is WSJT-X feeding freq
  updates?" from a single log scrape.

**What's NOT in this version:**

- **AirSpy HF+ Discovery / Dual** support. They share the libairspy
  API and most of the code path, but the HF+'s narrower native rates
  and HF-specific behaviours (bias-T actually doing something, IF
  imaging, lower tuning ranges) need separate testing. Targeted for a
  follow-on v0.99.x or v1.x once an HF+ tester reports in.
- **PPM / frequency correction.** AirSpy R2's Si5351 master clock is
  good to a few ppm out of the box and there's no software-level
  correction call in libairspy. If a tester reports it matters,
  Si5351 register-level writes can be wired in a follow-on version.

**Open issues / things to verify on first hardware:**

- **Sample-rate negotiation**: the bridge sets 2.5 Msps via
  `airspy_set_samplerate`. libairspy reports supported rates only when
  the device is open; the Settings combo populates dynamically. If
  10 Msps is selected and USB throughput is constrained (USB 2.0 host
  controller, hub, long cable), the bridge may underrun — drop back to
  2.5 Msps.
- **Bias-tee toggle is a no-op on R2 hardware** — exposed for forward
  compatibility with HF+. The Settings checkbox is not greyed out so
  the user can toggle it; behaviour on R2 is no observable change.
- **Front-end overload**: AirSpy R2 doesn't expose a "PowerOverload"
  event the way SDRplay does. Watch the `[Stats]` peak |IQ| line
  instead — values >100 (within a few dB of 0 dBFS) suggest the front
  end is being driven hard. Drop the gain.
