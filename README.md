# Voidstrap Resources

Data and optional components that Voidstrap downloads at runtime. Nothing here runs on its own, nothing here is required for Voidstrap to launch Roblox, and every file is either plain text or an official, code signed library from Microsoft or NVIDIA.

Voidstrap used to fetch these from its own website. The website is shut down, so they live here instead.

## What is in this repository

| Path | What it is |
|---|---|
| `Translations.json` | Community translations for the Voidstrap interface, 33 languages |
| `ServerLocations.json` | Roblox datacenter IP ranges mapped to city, region, country and coordinates, used by the Voidstrap Matchmaker to pick the closest server |
| `bin/DirectML.dll` | Microsoft DirectML, optional, only for RiShade AI depth |
| `bin/NvOFFRUC.dll` | NVIDIA Optical Flow frame rate up conversion, optional, only for NVIDIA frame interpolation |
| `bin/cudart64_110.dll` | NVIDIA CUDA runtime, optional, required by `NvOFFRUC.dll` |

any other DLL, I completely forgotten about now and moved on from and isn't inside voidstraps code any longer

Voidstrap downloads the two JSON files into its data folder and refreshes them at most once every six hours using conditional requests, so an unchanged file costs no bandwidth.

## The DLLs, in plain terms

These three files are **not written by Voidstrap**. They are the standard redistributable libraries published by Microsoft and NVIDIA, the same ones shipped inside games, video editors and machine learning tools. They are included here only so that two optional features do not require you to install a large SDK by hand.

### DirectML.dll, 18.5 MB

Microsoft DirectML is part of DirectX. It is the API that runs small machine learning models on your GPU through Direct3D 12, the same way a game runs a shader.

Voidstrap uses it for **RiShade AI depth**: RiShade reads the rendered frame and runs a tiny depth estimation model so effects like ambient occlusion and depth of field know what is near and what is far. DirectML is what makes that model run on the GPU instead of stalling the CPU.

* Publisher: Microsoft Corporation
* Version: DirectML Library 1.15.4
* Origin: the `Microsoft.AI.DirectML` redistributable package
* If it is missing: Voidstrap looks for a system copy of DirectML, and if there is none, RiShade simply runs without AI depth.

### NvOFFRUC.dll, 783 KB

NVIDIA Optical Flow is a dedicated hardware block on NVIDIA GPUs that measures how pixels move between two frames. FRUC stands for frame rate up conversion: given frame 1 and frame 2, it generates the in between frame.

Voidstrap uses it for **frame interpolation**, which raises perceived smoothness by inserting generated frames. This is the same NVIDIA technology used by their own optical flow samples and by several third party frame generation tools.

* Publisher: NVIDIA Corporation
* Origin: the NVIDIA Optical Flow SDK redistributable
* Requires: an NVIDIA GPU, and `cudart64_110.dll`
* If it is missing: Voidstrap falls back to its own built in frame generator, which works on any GPU.

### cudart64_110.dll, 466 KB

The NVIDIA CUDA runtime library, version 11.2. It is a dependency, nothing more: `NvOFFRUC.dll` is built against CUDA 11 and will not load without it. It does no work by itself.

* Publisher: NVIDIA Corporation
* Version: NVIDIA CUDA Runtime 11.2.28
* Origin: the CUDA Toolkit redistributable

## Why this is safe, and how to check for yourself

1. **All three DLLs are code signed and the signatures are valid.** Right click any of them, open Properties, then Digital Signatures. You will see `Microsoft Corporation` for DirectML and `NVIDIA Corporation` for the other two. A file cannot be modified without breaking that signature.
2. **Voidstrap verifies every download before it is used.** The exact byte size and SHA-256 hash of each file are hard coded in the app. If a download does not match, byte for byte, it is deleted and the feature falls back to its non accelerated path. Swapping a file in this repository for a different one would not make Voidstrap load it, it would make Voidstrap refuse it.
3. **Nothing is downloaded until you turn the feature on.** A default install never fetches these. RiShade AI depth fetches DirectML, NVIDIA frame interpolation fetches the two NVIDIA files, and that is the whole trigger list.
4. **You can verify the hashes yourself.** In PowerShell:

```powershell
Get-FileHash .\bin\DirectML.dll -Algorithm SHA256
```

| File | Bytes | SHA-256 |
|---|---|---|
| `bin/DirectML.dll` | 18,527,776 | `9C9E6D822561C6C41B90E6994B3E8857CF1D66DBFB1E0C4C799C7C89B4E92DA1` |
| `bin/NvOFFRUC.dll` | 783,416 | `5A0B6701D30709E25E7E5B92CA46B18AAB1459160CECD4F629872369D85C8B0A` |
| `bin/cudart64_110.dll` | 466,488 | `EDC35E7D0FA3F257BBEDFA7888911080C5696ACDD40B6187B6DD0173F20759AD` |

5. **You can skip the download entirely.** Leave the optional features off, or drop the files in yourself: `DirectML.dll` goes in `Extensions\RiShade` inside your Voidstrap folder, the two NVIDIA files go next to `Voidstrap.exe`. If the files are already present and match, Voidstrap never reaches out at all.

## Where they end up on disk

| File | Destination |
|---|---|
| `Translations.json` | Voidstrap data folder |
| `ServerLocations.json` | Voidstrap data folder |
| `DirectML.dll` | `<Voidstrap>\Extensions\RiShade` |
| `NvOFFRUC.dll` | next to `Voidstrap.exe` |
| `cudart64_110.dll` | next to `Voidstrap.exe` |

## Licensing

The DLLs are redistributed under their vendors' own redistribution terms: DirectML under the Microsoft DirectML license, and the NVIDIA libraries under the NVIDIA Optical Flow SDK and CUDA Toolkit redistribution terms. They remain the property of Microsoft and NVIDIA respectively. The JSON data is contributed by Voidstrap users.

## Layout

Voidstrap builds its download URLs from a single base URL plus these exact paths, so the names and the `bin` folder need to stay as they are:

```
Translations.json
ServerLocations.json
bin/DirectML.dll
bin/NvOFFRUC.dll
bin/cudart64_110.dll
```
