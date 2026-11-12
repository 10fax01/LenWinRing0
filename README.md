# LenWinRing0

LenWinRing0 is a WinRing0 Clone signed by Lenovo in 2024, this was on my private git and ive decided to post it here, companies like Lenovo shouldn't be re-using Vulnerable driver templates from the early 2000s like WinRing0 all the way in 2024. (like what are we doing lmao)

## Driver Info

| Field | Value |
|-------|-------|
| Name | LenWinRing0.sys |
| SHA-256 | 61aa8d59f557ef708d6e7b86c9be879cd6841c1a871a6c215967f09b535c2239 |
| Arch | x64 |
| Class | Hardware Access / WinRing0 Clone |
| Family | WinRing0 / Lenovo |
| Signer | Lenovo |
| Signed Date | 2024-10-16 13:04:48 |

## Our testing

During this discovery i did a few things, 
1. does EAC flag it? no, it eac allowed the driver to stay loaded.
2. is it on the MS blocklist or Lol drivers? at the current moment no.
3. Is flagged via VirusTotal Code in-sights? Yes it was marked as BYOVD.

## Primitives

| Primitive | IOCTL | Mechanism | Function |
|-----------|-------|-----------|----------|
| Arbitrary Physical Memory Read | `-1673502460` | `MmMapIoSpace` on caller PA, restricted to legacy BIOS region `[0xC0000, 0xFFFFF]`, byte/word/dword | `sub_1400013A0` |
| MSR Read | `0x9C402084` | `__readmsr(caller_msr)` no whitelist (out buffer ≥ 8) | `sub_140001484` |
| MSR Write | `0x9C402088` / `-1673502396` | `__writemsr(caller_msr, caller_value)` — no whitelist | `sub_1400015C4` |
| IO Port Read | `-1673502516` / `-1673502512` / `-1673502508` | `__inbyte` / `__inword` / `__indword` | `sub_140001350` |
| IO Port Write | `-1673486120` / `-1673486116` / `-1673486112` | `__outbyte` / `__outword` / `__outdword` (validated: input length = 8) | `sub_14000157C` |
| IO Port R/W | Above | Combined | `sub_140001350`, `sub_14000157C` |
| PCI Config Access | `-1673502396` (r), `-1673486008` (w) | `HalGetBusDataByOffset` / `HalSetBusDataByOffset` | `sub_14000160C`, `sub_140001678` |

## Notes & Extras

### Driver / Device

- Driver: `LenWinRing0.sys`
- Device: `\Device\LenWinRing0_1_0_0`
- Symlink: `\DosDevices\LenWinRing0_1_0_0`
- WDF driver (KMDF)
- Dispatch: `IRP_MJ_CREATE`, `IRP_MJ_CLOSE`, `IRP_MJ_DEVICE_CONTROL` → `sub_1400010E0`

### Notable IOCTLs

- `0x9C402000` - returns `0x1000000` (`16908293` is the v1/v2 value; here the driver returns `0x1000000` = 16777216)
- `0x9C402004` - reference count
- `0x9C40208C` — `__readpmc`
- `0x9C402090` - `__halt()`
- `-1673486072` - length-checked no-op (returns `STATUS_INVALID_PARAMETER` if input < 8)
- Note: I/O port write validates `a3 == 8` in `sub_14000157C` (the `a3` parameter is input length)

### Not Observed

Same as other WinRing0 clones: no physical memory write, no arbitrary kernel/virtual R/W, no CR/GDT/IDT/MSR-based leaks, no code execution, no process primitives.
