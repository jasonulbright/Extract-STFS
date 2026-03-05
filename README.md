# Extract-STFS

A PowerShell tool for extracting files from Xbox 360 STFS (Secure Transacted File System) containers.

Supports **LIVE**, **PIRS**, and **CON** package types — the formats used for Xbox 360 DLC, marketplace content, game saves, and profile data.

## Requirements

- PowerShell 5.1+ (included with Windows 10/11)
- No additional dependencies

## Usage

### Extract files from a package

```powershell
.\Extract-STFS.ps1 -Path "path\to\package" -OutputDir "C:\Extracted"
```

### List contents without extracting

```powershell
.\Extract-STFS.ps1 -Path "path\to\package" -ListOnly
```

### Batch extract all packages in a directory

```powershell
Get-ChildItem -Path "C:\DLC" -Recurse -File | ForEach-Object {
    $outDir = Join-Path "C:\Extracted" $_.Name
    .\Extract-STFS.ps1 -Path $_.FullName -OutputDir $outDir
}
```

## Output

The tool displays:

- Package type (LIVE/PIRS/CON)
- Header size and hash table address
- Block separation mode
- Display name (from package metadata)
- File listing with sizes
- Extraction progress

## How It Works

STFS is a filesystem format used inside Xbox 360 content packages. Files are stored in 4KB blocks with SHA1 hash tables inserted at regular intervals for integrity verification.

This tool implements the block addressing algorithm from the [Velocity](https://github.com/hetelek/Velocity) project, handling:

- **L0 hash tables** — inserted every 170 data blocks
- **L1 hash tables** — inserted every 170 L0 groups (28,900 data blocks)
- **L2 hash tables** — for packages exceeding ~4.9M blocks
- **Block separation** — Female (shift=0) and Male (shift=1) hash table layouts
- **Consecutive and chained** block allocation

### Package Types

| Type | Signature | Description |
|------|-----------|-------------|
| LIVE | Microsoft-signed | Xbox Live marketplace content (DLC, demos, arcade games) |
| PIRS | Microsoft-signed | System updates, title updates |
| CON  | Console-signed | Save games, profile data |

## Limitations

- Read-only extraction (no repacking or modification)
- Does not verify SHA1 hashes (extracts regardless of integrity)
- Does not decrypt encrypted packages
- Non-consecutive (chained) block files rely on L0 hash table chain pointers

## References

- [Free60 STFS Documentation](https://free60.org/System-Software/Formats/STFS/)
- [Velocity Project](https://github.com/hetelek/Velocity) — The reference STFS implementation (GPLv3)

## License

MIT License. See [LICENSE](LICENSE) for details.
