# Lenovo Tab M11 — mtkclient FRP backup/erase and userdata wipe (sanitized field notes)

Owner-authorized field notes from a September 2026 service session on a **Lenovo Tab M11**. This records what the local logs actually show: an FRP backup and erase, followed by a `userdata` wipe, using `mtkclient` on Windows.

> **Only do this on a device you own or are explicitly authorized to service.** Erasing `userdata` permanently deletes local data.

> This is a **research record, not a product support guide**. It deliberately does not claim more than the evidence proves. For a fuller beginner walkthrough of the toolchain and driver setup, see the companion writeup: [onn MT6768 FRP removal and userdata wipe](https://github.com/fergatronanator/onn-mtkclient-frp-unlock).

> Video text: [YOUTUBE_DESCRIPTION.md](YOUTUBE_DESCRIPTION.md)

---

## 0. Read this first: what is confirmed and what is not

### Confirmed by local logs

- The device was identified by the owner as a **Lenovo Tab M11**.
- It communicated with `mtkclient` in **MediaTek BROM / PreLoader VCOM** mode.
- `frp` was backed up (1 MiB) and erased successfully.
- `userdata` was erased successfully.
- WinFsp, UsbDk, Python 3.12, and `C:\mtkclient` were used.

### Not confirmed

- No exact model number (for example `TB330FU`, `TB328`, or `TB331`) was confirmed.
- No firmware version, Android version, or build number was confirmed.
- No IMEI, serial number, Wi-Fi detail, or other personal identifier was confirmed or is published here.
- **No power-on, setup-wizard, or account-lock verification was logged after the wipe.** The operations succeeded; the end result was not observed in the retained evidence. The owner reports unplugging the tablet immediately after the FRP work, which explains why no follow-up check was captured.
- An early “Helio G88” label was an assistant inference. The tool only logged the broader string `MT6768/MT6769 (Helio P65/G85, k68v1)`.
- A firmware/flashing-tool rescue guide was pasted into the session as background context. It is **not** locally observed evidence and is intentionally excluded from this repo.

---

## 1. Evidence basis

This note is built only from sanitized excerpts of:

- an OpenCode desktop session titled **“Setup and run MTKClient on Windows”** — the M11 portion begins 2026-09-28 around 20:23 local time;
- small script files under `C:\Temp` (`frp_script.txt`, `userdata_script.txt`, `mtk_watch.ps1`);
- redirected `mtkclient` output in `C:\Temp\mtk_watch.log` and `C:\Temp\mtk_run.log`;
- file metadata confirming that the 1 MiB FRP backup was created at 20:41.

No raw session database, raw tool output, backup binary, screenshot, or hardware-metadata file was copied into this repository.

---

## 2. Toolchain used

| Item | Detail |
|---|---|
| OS | Windows |
| Python | 3.12, invoked explicitly as `py -3.12` |
| mtkclient | run from `C:\mtkclient` via `C:\mtkclient\mtk.py` |
| Low-level USB | UsbDk (64-bit), confirmed running |
| Filesystem support | WinFsp, confirmed installed |
| Device transport | MediaTek PreLoader VCOM / BROM, USB PID `0003` |

Booted Android exposed **PID `2046`** and was **not** usable for this flow. The working mode was BROM/PreLoader with **PID `0003`** (hardware ID `USB\VID_0E8D&PID_0003`, Windows port name `MediaTek PreLoader USB VCOM Port (COMxx)`).

The exact button combination used on this unit is not independently preserved in the sanitized notes. mtkclient's own hint is:

> Power off the phone before connecting. For brom mode, press and hold vol up, vol dwn, or all hw buttons and connect usb. For preloader mode, don't press any hw button and connect usb.

The general “unplug → start mtkclient → wait for `Waiting for PreLoader VCOM` → reconnect in BROM” sequence is documented in the [companion onn guide](https://github.com/fergatronanator/onn-mtkclient-frp-unlock).

---

## 3. What failed before it worked

These are log-confirmed failures from the M11 session, in order:

1. The device initially stayed in normal Android mode (PID `2046`), so no BROM handshake occurred.
2. Two stale `mtkclient` processes were found and killed; a single clean watcher was started.
3. First BROM detection succeeded, but the Download Agent upload failed with `USBError(5)` / `Failed to upload da.`
4. Further attempts produced `Error, payload answered instead` and `DaHandler - Please disconnect, start mtkclient and reconnect.`
5. USB selective suspend on AC power was disabled (`0x00000000`), and different ports/cables were tried.
6. After the owner supplied a **new USB cable**, the DA loaded and both operations completed.

The new cable correlates with success; the logs prove correlation, not causation. A damaged or data-incapable cable is a reasonable thing to check when you see repeated `USBError(5)` failures.

At BROM detection, mtkclient also printed:

```text
Auth file is required. Use --auth option.
```

No auth file was found or used locally. Later successful DA runs reported `DA SLA is disabled`, so this warning did not prevent the backup and erase on this unit. Do not assume the same will be true for a different device or security configuration.

---

## 4. Confirmed successful operations

### 4.1 FRP backup and erase

Script used (two lines, run through mtkclient's script mode):

```text
r frp <frp-backup-name>.bin
e frp
```

Result:

| Field | Value |
|---|---|
| Partition | `frp` |
| Sector | `46144` |
| Address | `0x1688000` |
| Length | `1048576` bytes (1 MiB) |
| Backup size confirmed | `1,048,576` bytes, created at 20:41 |

The backup file is **private**. Keep it local and never upload it — it can contain device-specific lock data.

### 4.2 Userdata wipe

Script used:

```text
e userdata
```

Result:

| Field | Value |
|---|---|
| Partition | `userdata` |
| Address | `0x30f000000` |
| Length | `111892463616` bytes |
| Sector | `25657344` |
| Sector count | `218539968` |

Sanitized success lines from the run:

```text
DA SLA is disabled.
Successsfully formatted addr 0x30f000000 with length 111892463616.
Formatted sector 25657344 with sector count 218539968.
```

No logged operations were performed on `nvram`, `proinfo`, `persist`, or `metadata`.

---

## 5. Device facts observed in the logs

| Item | Value |
|---|---|
| Owner-identified device | Lenovo Tab M11 |
| Tool-reported SoC string | `MT6768/MT6769 (Helio P65/G85, k68v1)` |
| HW code | `0x707` |
| Target config | `0xe7` |
| BROM security flags reported | SBC, SLA, DAA, SWJTAG, memory read auth, memory write auth, blocked command `0xC8` |
| eMMC identifier class | `MMC128` |
| eMMC Boot1 / Boot2 | `0x400000` each |
| eMMC RPMB | `0x1000000` |
| eMMC USER region | `0x1d20000000` |
| eMMC GP1–GP4 | `0x0` |

The eMMC CID and all similarly identifying values were present in the raw output and are **deliberately omitted** here.

---

## 6. What was deliberately excluded

This repo does **not** contain:

- FRP backup binaries for any device;
- auth files, resign-DA files, or any firmware/flash-tool package;
- the third-party rescue guide pasted into the session;
- SP Flash Tool, ADB/debloat, MotoRoot, or other-device material;
- the OpenCode session database, raw tool-output files, `.state`, or `hwparam.json`;
- screenshots or device photos;
- ME_ID, SOC_ID, eMMC CID, serials, IMEIs, Wi-Fi details, certificate thumbprints;
- COM numbers, VCOM instance IDs, or USB hardware suffixes;
- personal user-profile paths.

---

## 7. Cleanup

If Windows Test Mode was enabled for unsigned driver installation on your machine, turn it back off and reboot when you are finished. Never keep a test-signed driver configuration as your normal daily state.

---

## 8. Related

- [onn MT6768 FRP removal and userdata wipe — full walkthrough](https://github.com/fergatronanator/onn-mtkclient-frp-unlock)
- [mtkclient](https://github.com/bkerler/mtkclient)
