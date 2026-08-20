# PhantomSDR-Plus WebSDR - Archive

**Maintained by SV1BTL.** The current version is 3.7.0, and the project lives at
**https://github.com/sv1btl/PhantomSDR-Plus** — that is the one to clone, link to
and report issues against.

## Note: Tested on Debian 12 (Bookworm), Debian 13 (Trixie), Ubuntu 22.04, Ubuntu 24.04.


**New in v.3.7.0**

* **A CPU over-temperature guard now protects the machine on its own.** It watches the CPU temperature and responds in stages — a warning, then a lower CPU clock, then stopping the server, then restarting it once the machine has cooled — with each stage held for a sustain time so a brief spike during a compile never trips it. Nothing is hard-coded: the thresholds are derived from the trip points your own CPU publishes, landing near 95 °C on an Intel with a 100 °C limit, 90 °C on a Ryzen and 80 °C on a Raspberry Pi. It works with **any** start/stop method, because while the CPU is too hot it re-issues the stop on every 2-second check, so a watchdog, a systemd unit or a cron job that revives the server is undone until the machine is cool. **It ships inert** — the mode starts at `log`, recording in `crash.log` what it *would* have done, so you can watch your own machine for a week before letting it act. Full sysop manual: **[docs/THERMAL_GUARD.md](docs/THERMAL_GUARD.md)**, in all six translations.
* **The guard also runs without the admin panel.** `thermal_guard.py` is a standalone program with `--mode`, `--config` and `--once` flags — no JSON file is required to get protection — and `thermal-guard.service` is included for installations that do not use the panel.
* **The throttle stage no longer needs root.** `./setup-cpufreq-perms.sh` grants a `cpufreq` group write access to the CPU frequency ceiling and installs a `tmpfiles.d` rule so the permission survives a reboot, instead of running the panel — and every start/stop script it launches — as root. `--revoke` undoes it.
* **The admin panel and proxy can run under systemd.** Two ready units, `phantomsdr-admin.service` and `phantomsdr-proxy.service`, start both at boot and restart them within five seconds of a crash — which matters because the thermal guard lives inside the panel, so a panel that dies at 03:00 takes the protection with it. Installing the units is **strongly recommended** and is what `setup_admin.sh` now offers by default. `manage_admin.sh` remains fully supported for anyone who prefers to start things by hand, and for machines with no systemd at all; pick one method and stay with it, as both are described in [the setup guide](docs/ADMIN_PANEL_SETUP.md#step-3--start--stop--restart).
* **`setup_admin.sh` now sets up the whole panel, not just the ports.** It asks which scripts start and stop your receiver — the panel and the thermal guard both act through these — then how far the temperature guard may go on its own, and offers to grant the throttle permission and to install the two systemd units. It also installs `tomli-w`, which the panel needs to save `.toml` files.
* **The installers now set up the whole receiver on their own.** `install.sh` and the five distro variants install the **admin panel**, the **FreeDV RADE decoder** and the **statistics server** by default (answer `n` to skip any of them), install the **RX888 udev rules** when you pick that receiver, decide about **OpenCL from the hardware they find** instead of asking you to guess, patch the two **websocketpp headers** websdr.org registration needs, and finish with one full `recompile.sh`. Nothing on the machine has to be prepared by hand first — see [What the installer does](#what-the-installer-does--read-this-before-you-start).
* **The RADE installer runs on any of the four package managers.** It installs its dependencies through apt, pacman, dnf or zypper — whichever the machine has — falling back to pip for a module a distribution does not ship, and it upgrades `websockets` itself when the distro version is older than the 11.0 the sidecar needs. It also still picks the right variant for your system: Ubuntu 22.04 needs `install_rade_ubuntu22.sh`, and each script reads `/etc/os-release` and offers to hand over to the other when it does not match.
* **A new `smeter_theme.sh` tool** sets which analog S-meter face is the default for new visitors, and offers to recompile the frontend for you afterwards.
* **The magic-eye indicator now behaves like a real EM84 tube**, following the F1NSK design more closely.
* **Mobile fixes.** The `/mobile` page follows the band plan as you tune, its digital S-meter reads the same level as the desktop one, and the meter gained an extra segment.

**New in v.3.6.1**

* **The S-meter / layout variants now switch without reloading the page.** The ⚙️ menu at the top right used to navigate to a separate build of the site (`/analog`, `/digital`, `/v2-analog`, `/v2-digital`), so every switch reloaded everything: the audio stopped, the waterfall cleared and any running decoder was lost. Since the four `App__*_smeter_.svelte` files were merged into one `App.svelte` those "versions" are just two properties of the same page, so the menu now changes them **in place** — the switch is silent and listening continues uninterrupted. Each visitor's choice is remembered in their own browser, and the variant picked in `./recompile.sh` is the **starting** variant a first-time visitor sees.
* **One desktop build instead of five.** With the variants switchable at runtime, the four extra builds were four identical copies of the same application, so they are gone: `build-all.sh` now produces the desktop page and `/mobile`, and a full rebuild takes **about half as long again** (roughly 29 s against 53 s here). The per-variant scripts `build-analog.sh`, `build-digital.sh`, `build-v2-analog.sh`, `build-v2-digital.sh` and the obsolete `switch-version.sh` have been removed, and the `./recompile.sh` build menu is now simply *all* / *desktop only* / *mobile only*. Old bookmarks to the four retired URLs are not broken — each keeps a small page that redirects to `/`.
* **The simplified mobile page now follows the band plan as you tune.** On `/mobile` the mode used to change only when you pressed a band button — typing a frequency, or stepping to one, kept whatever mode was selected, so a QSY from 40 m to a broadcast station stayed in LSB. The mode now follows `bands-config.js` whenever the dial moves into a different band or a different mode segment, exactly as the full interface has always done. A mode you choose by hand still sticks while you move about inside that segment; outside every defined band the mode is left alone; and while **RADEL/RADEU** are running they keep the receiver, so tuning does not drop the RADE decoder back into a listening mode.
* **Your frequency now follows you between the two mobile views.** `/mobile` and the extended mobile view are separate pages, so switching between them reloaded the site and dropped you back on the receiver's default frequency. Both switch buttons now carry the current frequency and mode in the link, and both pages read it on arrival, so you stay on the signal you were listening to. The address bar keeps up with your tuning as well, which means reloading the page, bookmarking it, or sending the link to someone else all return to that exact frequency. The **mode follows the band plan in `bands-config.js`**: your own mode travels with you, but where the other view has no equivalent for it the band plan decides — a broadcast frequency arrives in AM, 40 m in LSB, a CW segment in CW. `SAM` on `/mobile` maps to AM with the synchronous detector in the extended view and back again, and a frequency outside the receiver's coverage is pulled back to the nearest edge.

**New in v.3.6.0**

* All variants of App.svelte have been merged in one file,
* A new QRSS Grabber implementation has been added.
* A new SSTV decoder implementation has been added.
* A new FAX decoder implementation has been added.
* A new PSK31 decoder implementation has been added.
* A new Olivia decoder implementation has been added.
* All frequency schedules have been fully updated. Inactive frequencies have been removed, current information has been added, and the transmitting station is now displayed in the decoder window.
* The frontend/GUI AGC system has been redesigned.
* A new simplified mobile interface has been added as a standalone `.svelte` component. It is available at:  `http://your_server:PORT/mobile` This interface does not include a waterfall display, but it supports all operating modes, including RADE. The previous mobile interface remains available for users who require access to additional information and functionality. The `./recompile.sh` script has also been updated to support the new mobile GUI. The interface has been tested on both Android and iOS.
* The main GUI has been redesigned with more realistic 3D visual effects throughout the interface. All `.svelte` files have been updated accordingly.
* A **TUNE** button has been added to the **Users** tab, allowing the user to tune directly to another listener’s frequency.
* The analog and digital S-meters have been calibrated to provide consistent signal-level readings. A signal generator was used during the calibration process.
* The decoders are no longer reached through the Bandwidth row, which has been removed. A **Decoders** button row now sits on the main panel, directly under the Modes selector — one button per decoder (FT8, FT4, FT2, CW, WSPR, FAX, SSTV, NAVTEX, RTTY) — while **RADEL** and **RADEU** sit beside the Modes selector itself and are repeated inside the Modes and Bands pop-ups. One press starts the decoder and opens its window; a second press stops it.
* The Admin Panel has a new **Graphs** page plotting CPU frequency, CPU load, CPU temperature and users online on one shared time axis, over **15 MIN / 1 HOUR / 4 HOURS / 12 HOURS / 24 HOURS**. Sampling runs every 2 seconds in memory only (nothing is written to disk) and is kept in two tiers — 1 hour at full 2-second resolution plus 24 hours of 30-second averages — so a full day stays readable instead of becoming ~43000 points. Hovering the plot draws a crosshair with all four values at that instant.
* The frontend now rebuilds in about **half the time**. `build-all.sh` used to build the same sources five times one after another, because the S-meter variant was written into `src/main.js` before each build and only one build could own that file at a time. The variant is now passed to Vite as a build-time value, so every build reads the same unmodified sources and they run **in parallel** — a full `build-all.sh`, including `/mobile` and the title/favicon pass, drops by about **50%**. Nothing changes in how you use `./recompile.sh`. The number of builds running at once defaults to 3, which keeps the CPU cool on a laptop; raise it with `PHANTOM_BUILD_JOBS=5 ./build-all.sh` if your machine has the cores and the cooling for it. The variant chosen for the site root (`/`) is now remembered in `frontend/variant.json` — previously that choice could be overwritten by `build-default.sh`, so selecting, for example, the Digital S-meter as default still served the analog page at `/`.
* Two new modes have been added to the **FSK / RTTY** decoder window: **PSK31** and **Olivia**. Both sit in the same window's Variant dropdown alongside the three existing FSK variants, and the panel adapts to whichever is selected — the shift, baud, framing, encoding and invert controls are hidden for the two new modes, which have no mark/space tone pair. PSK31 corrects its own tuning error over about ±25 Hz and reports the recovered carrier and the transmitter's IMD; Olivia offers the four common configurations (**8/250** — the default the panel opens on — plus **16/500, 32/1000, 16/1000**), searches blindly for synchronisation because the mode sends no preamble, and has a **Squelch** slider controlling how strong the error-correction match must be before text is printed. See the [decoder documentation](docs/DECODERS.md).
* **A running decoder now owns the mode and the passband.** The mode used to follow the band plan in `bands-config.js` on every retune, which quietly undid whatever a decoder had set up: FT8 on 40 m flipped the receiver to LSB as soon as the dial moved, and the narrow decoder passbands widened back to the full SSB filter. A running decoder now keeps the mode and passband it needs across retunes, including a jump to another band, and the band's own mode returns when the decoder is switched off. Mode buttons still win — a deliberate choice by the operator is never overridden — and the CW decoder is unaffected, since it decodes in whatever mode you are listening in.
* The **QRSS grabber** has a **Band** list of the QRSS/MEPT windows from 2200 m to 6 m. Picking one and pressing **Tune** goes to that window in **CW** with a passband sized for the mode, so the dial reads the true QRSS frequency and the trace lands on the centre line of the display. Bands where two conventions are in use (40, 80, 160 and 10 m) list both windows. While a window is tuned the grabber holds the receiver the same way a decoder does, and the passband follows the dial as you hunt along the band. See the [decoder documentation](docs/DECODERS.md).
* The **Users** list marks your own session **you** when `users.html` is opened as a page of its own, not only inside the pop-up window. The page has no receiver of its own, so it asks whichever tab of the interface does — over the page's parent frame in the pop-up, and over a same-origin broadcast channel when standalone. With no interface tab open on that address there is no session to mark, and the list is shown unmarked as before.
* The four start scripts — `start-rx888mk2.sh`, `start-airspyhf.sh`, `start-rtl.sh` and `start-rsp1a.sh` — now **tell you what they are doing** instead of detaching in silence. Each one mirrors the start-up log to your terminal while the server comes up and then prints a summary saying whether the receiver, `spectrumserver`, the **RADE sidecar** and **websdr.org registration** are all up, before handing back the prompt; `Ctrl-C` stops only the mirroring, never the server. Add `-q` for the previous two-line output. The **RADE sidecar is now built into the start scripts** — it starts once the server is confirmed running, is restarted on its own if it exits (without restarting the server), is stopped by `stop-websdr.sh`, and can be skipped with `RADE_ENABLED=0`; where RADE is not installed the server starts normally and the summary just says the sidecar was not activated. Each script also reads your `.toml` and reports whether **[websdr.org] enabled** and **[websdr] register_online** are switched on, then waits for websdr.org to answer so the registration can be seen succeeding. Finally, `spectrumserver`'s own output — which was previously discarded — is kept in **`spectrumserver.log`** (rotating at 10 MB), with the `[WebSDROrg]` lines also copied into `logwebsdr.txt` — minus the routine `/~~orgstatus` and keep-alive-ping chatter, so what you read there is the registration story and any errors, not a running tally.
* The Admin Panel's Spot Reporting page now keeps **two separate counters**. The **SPOTS UPLOADED PER DECODER** tiles (FT8 / FT4 / WSPR) count uploads **since the daemon last started**, with the decode count and queue depth under each; a decoder whose destination is switched off shows `reporting off` instead of a bare `0`. The number beside each **BANDS & MODES** checkbox is that band+mode's **all-time** upload total, stored in `autorun-totals.json` so it survives restarts — a leading dot (`·123`) means the slot has decoded but not yet uploaded. 

**New in v. 3.5.0**

- **Automatic spot reporting to PSK Reporter and WSPRnet** — a built-in *autorun* engine decodes signals off-air locally and uploads your spots to the reporting networks, with no extra software required:
  - **FT8 / FT4 → [PSK Reporter](https://pskreporter.info/)**, over the native IPFIX/UDP protocol, with spots batched and flushed on the recommended ≥5-minute cycle,
  - **WSPR → [WSPRnet](https://wsprnet.org/)**, via the standard upload endpoint (WSPR is sent *only* to WSPRnet, to avoid duplicate spots),
  - Runs several band/mode slots at once (e.g. `20m:ft8`, `40m:ft8`), independently of connected users,
  - Each network can be enabled or disabled separately, and your identity (callsign + Maidenhead grid) is taken from your site configuration,
  - A **dry-run** mode logs the spots it *would* send without actually uploading them, for safe testing.
- **All decoders moved off the main thread** — SSTV, HF FAX, NAVTEX, FSK/RTTY and CW each run in their own Web Worker, so decoding no longer competes with audio playback:
  - No audio gap or dropout when a decoder is switched on or off,
  - The waterfall and GUI stay smooth while a picture or page is decoding,
  - NAVTEX and FSK/RTTY can now run at the same time (they previously shared one set of internal state),
  - FAX no longer discards your LPM / IOC / shift selection when the decoder is activated,
  - HF FAX and NAVTEX now switch the receiver to USB *before* starting, so the retune is never fed into the decoder, and SSTV picks the sideband from the band (LSB below 10 MHz, USB above) while still honouring a manual change.
- **SSTV no longer triggers on noise** — a static crash, a spark or mains hash used to be enough to start a decode. Every detection gate now has an absolute reference: a tonality test on the sync and VIS tones, a confirmation pass that must predict the next lines before anything is drawn, and a signal-presence gate. Forced modes are verified the same way instead of painting noise immediately.

**New in v. 3.4.0**

- Minimize as possible the CPU and RAM load for the CLIENT.
- Spectrum and waterfall in reverse is the default GUI for all varriants. Manually can work separately,
- All divs have now the same with 1380 px,
- Compressor, with manual settings and reset is added,
- Equalizer, with manual settings and reset is added,
- Record not only Audio, but Video + Audio as well, either full waterfall or selected area,
- 'Magic Eye' indicator tool is added, emulating EM84 tubes in older radios,
- Dark and light needle S-meter, dark is the default,
- geo-location users under the waterfall, not showing the flag in all browsers except Mozilla, is now fixed,
- Total new FreeDV Reporter tool. Now there is not an embedded external web page, but we get directly data from the FreeDV json and we build our own tool. We gain some data traffic this way, as we did with DX Cluster. 
- C-quam is auto Opus now, offering a much better sound. All the other modes are in Flac. In .toml file FLAC remains. When a C-quam station is traced, then the button text turns to green
- When in AM mode click again the button the Sam is activated on AM carrier and the button text turns to yellow and shown SAM. If the button is pressed again then return to AM.
- Noise Suppression is redesigned, OFF by default,
- Auto Adjust is redesigned from scratch,
- Emoticons in the chat are more stable now.
- 'Screws" in four angles in all divs, just for fun... 

**New in v. 3.3.3**

- All the install scripts are redesigned.
- Now change the frequency in the GUI with just hover the mouse and drag up/down wheel,
- In the admin panel is added crash.log file, if for any reason spectrumserver failed. You can also read the file from the root directory
- Optimized mobile GUI
- Redesigned all color schemes for the waterfall, new ;PhantomSDR' color scheme is added
- Audio background noise suppression algorithm is added and Noise Gate presets are redesigned
- Emoticons are added for use in the chat window
- Total bugs clean for server's stability.

**New in v. 3.2.1**

- users and stats now include maps,
- new admin panel with "kick user" option,
- debug tool,
- new sstv, Robot 36 decoder is added, band-pass filter 1100–2400 Hz and improvements to all sstv decoders,
- new rade_install and update script,
- Total bugs clean for server's stability.

**New in v. 3.2.0**

- Users list in real time with Geo location, tuned frequency, start connected time, total time connected.
- Statistic data about visitors, most visited bands and frequencies, hourly visitors data, long term statistics,
- Minor cosmetic changes in GUI,
- New AGC approach,
- New colored spectrum, with waterfall colour scale,
- CTCSS is now working, tracing subtones and opening the mute when a signal with subtone is received.
- Total bugs clean for server's stability.

**New in v. 3.1.1**
- Decoders in mobile version:
A new button is added in the mobile view and all decoders can be called from a mobile phone.
- Frequency digit selector:
Now you can select any digit in frequency and change only this, without affecting the other digits. With right click, you can insert a full desired frequency.
- AVAST warnings:
No more fake AVAST warnings about the security of the Website. DX Cluster window is now called from backend.
- Buffer improvements (thanks to F1NSK - Eric):
A small but very importand change in buffer (frontend/src/audio.js) which minimize audio brakes, withoud adding latency.
- Analog smeter calibration from ./toml file:
In .toml has been added an offset for analog smeter, along with the previous for digital. Now you can adjust the two smeters to show the same. Micro-trimming are also kept in analog smeter files in " function _smeterTick -> const visualGain = ". The smeters are calibrated out of the box now under my conditions using a signal generator, but you maybe want to play with them.
- Map registration:
You can now register your WebSDR in both http://list.novasdr.fun/ (or the same http://list.phantomsdr.fun/) AND https://sdr-list.xyz. Changes in .toml file and src/spectrumeserver.cpp (in backend).

**New in v. 3.1.0**

We offer more **features**:
- **New install.sh** procedure, that simplifies initial setup,
- **recompile.sh** script for fast and simply recostruction of either backend and/or frontend,
- Futuristic Design, Admin Panel added
- **CATsync** with the application [CATsync Tool for WebSDRs](https://catsyncsdr.wordpress.com/)
- Admin Panel, password protected, for remote controlling the server without SSH access.


