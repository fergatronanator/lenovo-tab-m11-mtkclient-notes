# YouTube description — Lenovo Tab M11 mtkclient field notes

Suggested title: **Lenovo Tab M11 + mtkclient — FRP Backup/Erase and Userdata Wipe (Owner-Authorized Field Notes)**

Copy/paste the text below into the YouTube Description field. Adjust the chapter timestamps to match your video.

```text
Owner-authorized field notes from a Lenovo Tab M11 service session: backing up and erasing FRP, then wiping userdata with mtkclient on Windows.

⚠️ Only use this on a device you own or are explicitly authorized to service. Erasing userdata permanently deletes local data. This is not for stolen devices.

Important honesty note: this is a record of what the local logs prove, not a support guide. The logs confirm the FRP backup+erase and the userdata erase. They do NOT include a power-on, setup-wizard, or account-lock check afterward, so this video does not claim the tablet was verified as fully working at the end.

Confirmed in the logs:
- Tool-reported chip string: MT6768/MT6769 (Helio P65/G85, k68v1), HW code 0x707
- Working mode: MediaTek BROM / PreLoader VCOM (PID 0003), not booted Android (PID 2046)
- FRP: 1 MiB backup, then erase at address 0x1688000, length 1048576
- userdata: erase at address 0x30f000000, length 111892463616
- Tooling: Python 3.12, WinFsp, UsbDk, mtkclient from C:\mtkclient

Things that mattered:
- Start mtkclient with the device unplugged, then reconnect in BROM mode.
- Kill stale mtkclient processes and use a single clean watcher.
- Repeated USBError(5) / "Failed to upload da" failures cleared up after a new USB cable was used. Correlation, not proof of cause — but check your cable.
- An "Auth file is required" warning appeared at BROM, yet the later DA runs reported "DA SLA is disabled" and completed. Do not assume another device will behave the same.

Full sanitized notes (identifiers, backups, and session data excluded):
https://github.com/fergatronanator/lenovo-tab-m11-mtkclient-notes

Companion full walkthrough for the onn MT6768 tablet:
https://github.com/fergatronanator/onn-mtkclient-frp-unlock

mtkclient: https://github.com/bkerler/mtkclient

Chapters (adjust timestamps to match your video):
00:00 What this is / safety / honesty note
01:00 Confirmed device facts
02:00 BROM mode and the failures along the way
04:00 FRP backup and erase
06:00 Userdata wipe
08:00 What was not verified / cleanup

#mtkclient #MediaTek #FRP #Lenovo #TabM11
```
