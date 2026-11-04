# Serial generation guide

`config.plist` in this repo is shipped **without** serial numbers. Generate yours and fill them in. There are 4 values in `PlatformInfo → Generic`:

| Key | Where it comes from |
|---|---|
| `SystemSerialNumber` | macserial / GenSMBIOS |
| `MLB` (Board Serial) | macserial / GenSMBIOS |
| `SystemUUID` | `SMBIOS UUID` — macserial or `uuidgen` |
| `ROM` | Your real network adapter MAC (6 bytes), base64-encoded |

## Option A — GenSMBIOS (GUI, easiest)

1. Download the latest release: <https://github.com/corpnewt/GenSMBIOS>
2. Run it, pick **Generate SMBIOS**
3. When asked for a model, choose **MacBookPro16,2** (this is what the EFI is configured for)
4. Copy the generated `Serial`, `Assembly Serial (MLB)` and `Apple UUID`: they map to `SystemSerialNumber`, `MLB`, `SystemUUID` in `config.plist`

## Option B — macserial (CLI)

Get it from `OpenCorePkg` release → `Utilities/macserial`:

```bash
# Generate 5 serials + MLB (board serial)
./macserial -m MacBookPro16,2 -g -n 5
```

Output columns are `serial | MLB`. Pick any line (they are all random and safe).

## ROM — the MAC address part (important for iMessage!)

`ROM` must be your **real network adapter MAC address** (or any random stable 6 bytes). Serialize as base64:

```bash
# Get your en0 (usually Wi-Fi) MAC
ifconfig en0 | awk '/ether/ {print $2}' | tr -d ':'

# Convert hex to base64 (replace 28:d0:43:53:0f:30 with your MAC value minus colons)
python3 -c 'import base64; print(base64.b64encode(bytes.fromhex("000000000006")).decode())'
# Example output: AAnBQ1PDDjA=
```

Paste the base64 string into `PlatformInfo → Generic → ROM` (`<data>`).

## Putting it together

Edit `config.plist`, `PlatformInfo → Generic`:

```xml
<key>MLB</key>            <string>YOUR_MLB</string>
<key>ROM</key>            <data>YOUR_BASE64_MAC</data>
<key>SystemProductName</key><string>MacBookPro16,2</string>
<key>SystemSerialNumber</key><string>YOUR_SERIAL</string>
<key>SystemUUID</key>     <string>YOUR_UUID</string>
```

Then **reset NVRAM** on next boot (otherwise iMessage/FaceTime won't let you in).

## Sanity check

Generated serial should ideally show as **"not activated" / not found** on Apple Check Coverage — that's a good sign it doesn't collide with a real Mac's serial. Never use a serial from another real Apple device.

## Do not

- Do **not** commit real serials/ROM/MAC to a public repo.
- Do **not** reuse serials between two booted installs (iCloud/iMessage bans).
