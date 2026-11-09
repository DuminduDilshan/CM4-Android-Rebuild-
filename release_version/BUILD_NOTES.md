# Waveshare CM4-NANO-B – Android 14 Firmware Build Notes

## Base Firmware
- **Source:** Radxa CM4 (Rockchip RK3576) Android 14 rkr6 Release (20250903)
- **SoC:** Rockchip RK3576

## Changes Applied
### Device Tree: `rk-kernel-waveshare-cm4-nano-b.dts/.dtb`

1. **`usb@23000000` (DWC3 Controller 0 — connected to USB-A Host port on CM4-NANO-B)**
   - `dr_mode` changed from `"otg"` → `"host"`
   - `phys` removed USB3 PHY ref: `<0x6E 0x6F>` → `<0x6E>`
   - `phy-names` changed: `"usb2-phy", "usb3-phy"` → `"usb2-phy"`
   - Removed `usb-role-switch;` (was waiting for FUSB302 that doesn't exist on Waveshare board)
   - Removed `port { ... }` role-switch endpoint linking

2. **`usb2-phy@0 / otg-port` (USB 2.0 PHY 0 OTG port node)**
   - Removed `rockchip,typec-vbus-det;` (Type-C GRF-based VBUS detect, not applicable)
   - Added `phy-supply = <0x75>;` (5V host supply regulator)

3. **`fusb302@22` (Type-C PD controller, NOT present on Waveshare CM4-NANO-B)**
   - `status` changed from `"okay"` → `"disabled"`

4. **`__symbols__` table (Stale DTBO symbol entries — Bug fix)**
   - Removed: `dwc3_0_role_switch` (pointed to deleted `/usb@23000000/port/endpoint@0`)
   - Removed: `usbc0` (pointed to now-disabled `fusb302@22`)
   - Removed: `usbc0_role_sw` (pointed to `fusb302@22` port node)
   - Removed: `usbc0_orien_sw` (pointed to `fusb302@22` orientation endpoint)

## Image Verification Results (All Passed)
| Check | Result |
|---|---|
| DTB parse + round-trip byte match | OK |
| `usb@23000000` dr_mode = host | OK |
| `usb-role-switch` removed | OK |
| phys = USB2-only | OK |
| role-switch remote-endpoint removed | OK |
| `usb2-phy@0` phy-supply present | OK |
| `typec-vbus-det` removed | OK |
| `fusb302@22` disabled | OK |
| `usb@23400000` unchanged (host) | OK |
| stale `dwc3_0_role_switch` symbol removed | OK |
| SHA-1 id in boot header verified | OK |
| DTB magic (`d00dfeed`) in boot header DTB field | OK |
| DTB magic in RSCE container entry | OK |
| RSCE container 10 entries verified | OK |
| Full GPT partition table intact | OK |
| **Final errors / warnings** | **0 / 0** |

## Hardware Verification (Waveshare CM4-NANO-B Schematic)
- **U3 (FSUSB42UMX):** Routes USB_P/USB_N to USB-A host connector when BOOT switch is OFF.
- **U2 (STMPS2171STR):** USB 5V power switch permanently enabled via R8 (10kΩ pull-up).
- **No FUSB302 / No USB Type-C PD controller present on this carrier board.**

## Output Files
| File | Size | Purpose |
|---|---|---|
| `boot_waveshare_cm4_nano_b.img` | 64 MB | Boot partition only — flash at LBA 0xC800 |
| `Radxa-cm4-android14-waveshare-cm4-nano-b.img` | 4.33 GB | Full GPT disk image |
