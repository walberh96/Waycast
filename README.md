# Waycast

Waycast is a cross-platform TUI “script package manager” that organizes, validates, installs dependencies for, and runs scripts as shareable **script packages**. Each package includes a generated `manifest.json` that describes how to run the script, what it depends on, and how it should be validated.

---

## Features

* **Cross-platform:** Windows, Linux, macOS
* **TUI-first workflow:** browse, search, and run scripts without leaving the terminal
* **Wizard-based package creation:** add scripts through prompts (no manual manifest writing required)
* **Package validation before install:** runner checks + optional test run to ensure the package works
* **Dependency installation:** declare dependencies so Waycast can install them before running
* **Runner support for scripting languages only:** bash/sh, PowerShell, Python (extensible later)
* **Logs and results:** view stdout/stderr and exit code inside the TUI

---

## What is a Script Package?

A **script package** is a folder containing:

* `manifest.json` (required, generated/managed by Waycast)
* one or more script files (required)
* optional extra files (README, templates, assets)

Example structure:

```
packages/
  system-cleanup/
    manifest.json
    cleanup.sh
  reset-network/
    manifest.json
    reset.ps1
  backup-notes/
    manifest.json
    backup.py
    requirements.txt
```

Packages can be platform-specific or cross-platform. The Waycast app is cross-platform, but a package should declare its supported platforms.

---

## Core Concepts

### Package ID

* Every package has a unique `id`
* The package folder name is the `id`
* IDs should be folder-safe (e.g. `system-cleanup`, `reset-network`)

### Runners

Waycast runs scripts using a declared **runner** (scripting languages only), such as:

* **bash/sh** (`bash`, `sh`)
* **PowerShell** (`pwsh`)
* **Python** (`python`, `python3`)

> Note: A runner must exist on the system. For example, bash scripts can run on Windows only if a bash environment is installed (Git Bash / MSYS2 / WSL, etc.). PowerShell scripts can run on Linux/macOS only if `pwsh` is installed.

---

## Adding a Package (Wizard Workflow)

Waycast packages are created through an interactive TUI wizard.

Inside the TUI, choose **Add Package**, then:

1. Enter package metadata (id, name, description, tags, supported platforms)
2. Choose a runner (bash / PowerShell / Python)
3. Import an existing script file **or** generate a script from a template
4. Declare dependencies (commands, OS packages, runtime deps)
5. Choose a validation method (example: run `--help`)
6. Waycast verifies:

   * the entrypoint exists
   * the runner is available
   * dependencies are interpretable/checkable
7. Waycast optionally test-runs the validation command
8. If validation passes, the package is saved and becomes available in the script list

This keeps your library clean: installed packages are expected to be runnable.

---

## Installing Dependencies

Waycast supports declared dependencies so scripts can be reproducibly run on new machines.

Before running a script, Waycast can:

1. Check what dependencies are missing
2. Offer to install them (with confirmation)
3. Verify installation (via required commands / runtime checks)
4. Run the script

Dependencies can include:

* **Required commands** (e.g. `git`, `jq`, `ffmpeg`)
* **OS packages** (installed through available providers)
* **Runtime deps**

  * Python: pip packages (often via a per-package virtual environment)
  * PowerShell: modules (often installed in CurrentUser scope)

---

## `manifest.json`

Each package has a `manifest.json` that describes how Waycast should manage it.

A typical manifest contains:

* metadata: id, name, version, description, tags, platforms
* runner: type + program
* entrypoint: script path inside the package
* deps: commands, OS packages, runtime deps
* validation: how to test the script
* run: args/env/cwd/timeouts
* safety: confirmation and network policy

Example:

```json
{
  "schema_version": 1,

  "id": "system-cleanup",
  "name": "System Cleanup",
  "version": "1.0.0",
  "description": "Cleans caches and temp files safely.",
  "tags": ["maintenance", "disk"],
  "platforms": ["linux", "macos"],

  "runner": {
    "type": "bash",
    "program": "bash"
  },

  "entrypoint": "cleanup.sh",

  "deps": {
    "commands": ["rm", "du"],
    "os_packages": [
      {
        "name": "jq",
        "providers": {
          "apt": "jq",
          "dnf": "jq",
          "pacman": "jq",
          "brew": "jq",
          "winget": null,
          "choco": null,
          "scoop": null
        }
      }
    ],
    "python": null,
    "powershell": null
  },

  "install": {
    "auto_install": true,
    "requires_admin": false
  },

  "validation": {
    "enabled": true,
    "args": ["--help"],
    "timeout_seconds": 10
  },

  "run": {
    "default_args": [],
    "cwd": ".",
    "env": {},
    "timeout_seconds": 300,
    "interactive": false
  },

  "safety": {
    "confirm_before_run": true,
    "allow_network": false
  }
}
```

---

## Security Notes

* Waycast runs scripts as real commands on your machine.
* Waycast does **not** sandbox scripts.
* Only add and run packages you trust.
* Always review scripts before running them, especially if they install dependencies or require admin privileges.

---

## Roadmap Ideas

* Script notes editor + per-package docs viewer
* Import/export bundles (zip) for sharing packages
* Git-backed package sources (optional)
* Profiles (different “script libraries” per machine/use case)
* Better dependency providers per OS (winget/choco/scoop, apt/dnf/pacman, brew)

---

## License

Choose a license (MIT/Apache-2.0 recommended) and add it here.
