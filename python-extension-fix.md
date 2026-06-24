# Fixing Python Extension ENOENT Error in Windsurf on WSL Fedora

## Problem

The `ms-python.python` VS Code extension was throwing an ENOENT (File Not Found) spawn error because the `-universal` package flavor doesn't include the platform-specific **Python Environment Tool (pet)** binary required for Linux systems.

The error occurs when the extension tries to spawn the `pet` binary to manage Python environments, but the binary is missing from the universal build.

## Root Cause

When Windsurf installs the Python extension from non-Microsoft registries (like Open VSX), it often gets the `-universal` flavor which lacks the pre-compiled native binaries for specific platforms. The universal build expects the `python-env-tools` directory with the `pet` binary, but it's not bundled.

## Solution Applied

We manually added the missing native binary component from the Linux-x64 build:

### Step 1: Download the Linux-x64 Extension

```bash
wget "https://marketplace.visualstudio.com/_apis/public/gallery/publishers/ms-python/vsextensions/python/2026.4.0/vspackage?targetPlatform=linux-x64" -O /tmp/python-linux-x64.vsix
```

### Step 2: Extract the VSIX Package

```bash
unzip -q /tmp/python-linux-x64.vsix -d /tmp/python-extension-new
```

### Step 3: Copy the Missing Binary Component

```bash
cp -r /tmp/python-extension-new/extension/python-env-tools ~/.devin-server/extensions/ms-python.python-2026.4.0-universal/
```

### Step 4: Set Execution Permissions

```bash
chmod +x ~/.devin-server/extensions/ms-python.python-2026.4.0-universal/python-env-tools/bin/pet
```

### Step 5: Verify the Binary Works

```bash
~/.devin-server/extensions/ms-python.python-2026.4.0-universal/python-env-tools/bin/pet --version
```

Output: `pet 0.1.0`

## Result

The Python extension now has the required native binary and can properly manage Python environments. After reloading Windsurf, the environment picker and related features work without errors.

## Alternative Solutions (Not Used)

1. **Disable Environment Manager**: Add `"python.useEnvironmentsExtension": false` to `.vscode/settings.json` - this would disable the feature entirely
2. **Replace Entire Extension**: Uninstall universal and install only the Linux-x64 version - requires extension management access

The solution we used (manually copying the binary) is the least invasive and preserves all extension functionality.
