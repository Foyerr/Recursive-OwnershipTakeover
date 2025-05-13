# Ensure-DeepNTFSPerms.ps1

This PowerShell script recursively enforces NTFS ownership and FullControl permissions for a specified group, as deep as access allows. It takes ownership of files and directories where applicable and appends a log of ownership changes to a CSV file.

## Features

- Recursively traverses a directory tree
- Takes ownership of files/directories not already owned by the specified group
- Applies FullControl permissions (inherited or explicit)
- Tracks ownership changes and logs them to a CSV (optional)
- Periodic live progress display without spamming the console

## Parameters

| Parameter         | Type    | Description |
|------------------|---------|-------------|
| `-RootPath`       | String  | Root directory to begin processing. Required. |
| `-Group`          | String  | Group to assign as owner and grant FullControl. Defaults to `BUILTIN\Administrators` if not specified. |
| `-UpdateIntervalMs` | Int | Milliseconds between progress updates. Default: `1000` |
| `-logPath`        | String  | Optional file path to write CSV log of ownership changes. |

## Example Usage

```powershell
.\Ensure-DeepNTFSPerms.ps1 -RootPath "C:\Data" -Group "BUILTIN\Administrators" -logPath "C:\Logs\OwnershipChanges.csv"
