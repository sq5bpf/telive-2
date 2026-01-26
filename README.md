telive - Tetra Live Monitor
(c) 2014-2026 Jacek Lipkowski <sq5bpf@lipkowski.org>

telive is a program which can be used to display information like
signalling, calls etc from a Tetra network. It is also possible to
log the signalling information, listen to the audio in realtime and
to record the audio. Playing the audio and recompressing it into ogg
is done via external scripts.

Note: this is a test version that will be used with osmo-tetra-sq5bpf-2:
https://github.com/sq5bpf/osmo-tetra-sq5bpf-2

For comprehensive documentation, please read telive_doc.pdf:
https://github.com/sq5bpf/telive/raw/master/telive_doc.pdf


## Quick Start Guide

### Architecture Overview

```
┌─────────────┐    UDP     ┌─────────────────┐           ┌─────────────┐    UDP     ┌────────┐
│  GnuRadio   │  :42001    │  receiver1udp   │           │  tetra-rx   │  :7379     │ telive │
│  (RTL-SDR)  │ ─────────> │  + simdemod3    │ ───────>  │  (decoder)  │ ─────────> │  (UI)  │
│  RF Input   │   IQ data  │  (demodulator)  │   bits    │             │  decoded   │        │
└─────────────┘            └─────────────────┘           └─────────────┘            └────────┘
```

**What each component does:**
- **GnuRadio**: Receives RF signal from RTL-SDR dongle, outputs IQ samples via UDP
- **receiver1udp**: Wrapper script that chains together:
  - `socat`: Receives UDP data from GnuRadio
  - `simdemod3_telive.py`: CQPSK demodulator (converts IQ to bits)
  - `tetra-rx`: TETRA protocol decoder (extracts signaling, voice, SDS)
- **telive**: ncurses UI that displays decoded information and plays audio

### Prerequisites

- RTL-SDR dongle (or other GnuRadio-supported SDR)
- GnuRadio 3.8+ with gr-osmosdr
- libosmocore and libosmocore-dev
- TETRA codecs in `/tetra/bin/` (installed via install-tetra-codec)
- osmo-tetra-sq5bpf-2 compiled

### Installation

Use the install script for supported distributions:
```bash
./scripts/install_telive.sh
```

Or install manually - see the FAQ section in telive_doc.pdf.

### Running telive (Simple 1-Channel Setup)

You need **3 terminals** (or use the xterm from the applications menu):

**Terminal 1 - Start the demodulator/decoder:**
```bash
cd ~/tetra/osmo-tetra-sq5bpf-2/src
./receiver1udp 1
```
This listens on UDP port 42001 for IQ data from GnuRadio and sends decoded data to telive on port 7379.

**Terminal 2 - Start telive (requires 203x60 terminal):**
```bash
xterm -geometry 203x60 &
# In the new xterm:
cd ~/tetra/telive-2
./rxx
```
Or use the "Telive xterm 203x60" entry from your applications menu.

**Terminal 3 - Start the GnuRadio receiver:**

With GUI (recommended for tuning):
```bash
cd ~/tetra/telive-2/gnuradio-companion/python3_based_gnuradio
python3 telive_1ch_simple_gr310_udp_xmlrpc.py
```

Or headless (lower CPU usage):
```bash
python3 telive_1ch_gr310_udp_xmlrpc_headless.py
```

**Tune to a TETRA frequency** in the GnuRadio GUI:
- Set baseband frequency (e.g., `435M` for 435 MHz)
- Adjust PPM for your dongle's frequency correction
- The spectrum should show the TETRA signal

### Multi-Channel Setup

For 6 channels (to capture all traffic in a Location Area):

```bash
# Start 6 demodulators
cd ~/tetra/osmo-tetra-sq5bpf-2/src
for i in $(seq 1 6); do xterm -T "RX$i" -e ./receiver1udp $i & done

# Start telive
cd ~/tetra/telive-2
./rxx

# Start 6-channel GnuRadio receiver
cd ~/tetra/telive-2/gnuradio-companion/python3_based_gnuradio
python3 telive_6ch_gr310_udp_xmlrpc_headless.py
```

### Telive Keyboard Commands

| Key | Function |
|-----|----------|
| `?` | Show help |
| `R` | Toggle recording |
| `L` | Toggle logging |
| `M` | Toggle mute |
| `m` | Toggle mutessi (filter traffic without SSI - hides encrypted) |
| `a` | Toggle alldump (show all signaling) |
| `s` | Stop current playback |
| `t` | Toggle between main window and frequency window |
| `V/v` | Increase/decrease verbosity |
| `f` | Toggle SSI filter |

### Output Files

- **Recordings**: `/tetra/in/` (raw ACELP), `/tetra/out/` (converted to OGG by tetrad)
- **Log file**: `telive.log` (signaling, SDS messages)
- **KML file**: Location information for Google Earth (if configured)

### Troubleshooting

- **No audio**: Check that codecs exist in `/tetra/bin/` and test with:
  ```bash
  /tetra/bin/tplay < testfile.acelp
  ```
- **Garbled display**: Ensure terminal is exactly 203x60 characters
- **No data in telive**: Verify receiver1udp is receiving data (should show scrolling text)
- **receiver1udp errors**: Check GnuRadio is running and sending to correct UDP port


## Disclaimer

The program is licenced under GPL v3 (license text is also included in the
file LICENSE). I may not be held responsible for anything associated with
the use of this tool.

This was a quick hack written in my spare time, and for my own pleasure,
and because of this the code is really ugly. The code is also based on wrong
assumptions - using the usage identifier as the key is not a good way of
following calls (but it works most of the time). Maybe one day this will
be rewritten to look better (or not).

---


