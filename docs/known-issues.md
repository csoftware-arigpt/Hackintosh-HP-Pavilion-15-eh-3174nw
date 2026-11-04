# Known issues

Current state of this hackintosh build. Updated Nov 2026.

## 1. Sleep / wake — ❌ broken

Sleep does not work: the machine does not go to sleep / wake up reliably.

This is the hardest area on **AMD Ryzen (Renoir/Barcelo) hackintoshes** — Apple's S3 suspend path is heavily macOS-SMBIOS dependent, and Renoir platform support is incomplete in the open-source stack.

Things that usually help to try (not yet confirmed working here):

- `pmset -g` — check which assertion generators block sleep: `pmset assertions`
- Disable in Settings → Energy: "Wake for network access", "Power Nap"
- Remove USB devices that hold wakeassertions
- Check `log show --predicate 'sender == "powerd"' --last 10m` for wake reason
- Inspect `_PRW` objects of USB/XHC/GLAN in DSDT; add corrected values in a new SSDT
- Try `sudo pmset -a hibernatemode 0` (avoid deep standby)

## 2. CPU: stuck at base clock (no Turbo Boost) — ⚠️ partial

CPU works, but it only runs at the base **2.0 GHz** — the 4.5 GHz boost is never engaged.

Possible angles to investigate:

- SSDT-PLUG-ALT: this build uses an HP-specific (`8BC7`) variant; make sure its `PerfTune` / `MPC[…]`-style objects actually match this DSDT
- ForgedInvariant — already loaded (1.5.0), but boost on Ryzen needs correct `_PPC`/frequency reporting; check with `log show --last 1m --predicate 'process contains "kernel"' | grep -i amd`
- Try kernel flag `-disablecpufix`? — no, the AMD kernel currently has no "turbo switch"; real fix is in AMDRyzenCPUPowerManagement (ChefKiss fork)
- On some Barcelo platforms boost is toggled by `Booter → quirks.js`-type code with `ProvideCurrentCpuInfo`

## 3. Graphics — ⚠️ partial

Metal 3 reported OK, but general behavior is glitchy.

Known/typical NootedRed limitations on Renoir/Barcelo:

- Boot-arg `-NRedDPDelay` is already used here for DP/external display handshake
- Some monitors/DP timings may need additional NootedRed boot-args (see NootedRed README)
- Cosmetic: 512 MB VRAM is reported instead of real carve-out
- Occasional issues with hardware video decode rates are common

## 4. Bluetooth — ❌ broken

RTL8852BE's Bluetooth part (USB-connected) is not initialized by
`RealtekBluetoothFirmware.kext` + `BlueToolFixup.kext` on Sequoia.

Realistic options:

1. Replace the Wi-Fi card with an Intel AX200/AX210 (needs `itlwm`/`OpenIntelWireless-Bluetooth`) — the standard real fix for Realtek cards
2. Use a supported USB BT dongle (Broadcom BCM20702A1 is easiest)
3. Keep watching the `RealtekBluetoothFirmware` GitHub for Sequoia fixes

## 5. Wi-Fi — works, legacy stack

Wi-Fi works only because this build injects the old `IO80211FamilyLegacy` +
`IOSkywalkFamily` (SIP is partially relaxed via `csr-active-config = 0x0B0B`).
Keep these three in perfect sync: `AirPort_RTW89` + `IO80211FamilyLegacy` + `IOSkywalkFamily`.
