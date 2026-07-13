# Brunch & ChromeOS System Updater

A native, automated utility pipeline designed to track, download, and seamlessly deploy recovery framework updates for ChromeOS devices running on x86_64 architectures using the Brunch framework.

This tool protects against corruption via cryptographic hashing and incorporates process-level mutual exclusion to guarantee system safety during upgrades.

---

## 🚀 Features

* **Smart Downloads:** Automatically calculates local `SHA-1` payload hashes to skip downloading large gigabyte-scale binaries if they are already present locally.
* **Safety Lock:** Uses kernel-level file locking (`fcntl`) to ensure that **only one instance** of the updater can run at any given time, preventing parallel corrupted image writes.
* **Modular Upgrades:** Provides targeted flags to update *only* the Brunch framework or *only* the ChromeOS underlying image depending on your maintenance window.
* **Automated Housekeeping:** Includes an integrated purge switch to quickly scrub large temporary archives and binary recoveries to reclaim multi-gigabyte disk space.
* **Self Updateing** Includes an integrated switch to natively pull down the latest version of this script via git.

---

## ⚙️ Installation

To set up the updater as a permanent system command, execute the following block to clone the repository to its dedicated workspace and create a global symlink:

```bash
# 1. Create the dedicated workspace and clone the repository
git clone https://github.com/binyaminyblatt/brunch-updater-chromeos.git /usr/local/brcr_updater

# 2. Make the script executable
chmod +x /usr/local/brcr_updater/brcru

# 3. Expose it globally by symlinking to /usr/local/bin/brcru
sudo ln -s /usr/local/brcr_updater/brcru /usr/local/bin/brcru
```

---

## 📦 Prerequisites & Architecture

This script is engineered strictly for modern x86_64 architectures.

> [!CAUTION]
> **Legacy System Block:** Execution will automatically abort if the script identifies an `i386` or `i686` 32-bit hardware kernel architecture to prevent unbootable partitions.

The update pipeline internally calls the system binary tool:

```bash
sudo chromeos-update
```

Ensure your host user has appropriate administrative rights (`sudo`) configured.

---

## 🛠️ Command-Line Interface Syntax

```text
usage: Brunch & ChromeOS Updater [-h] [-u] [-i] [--dry-run] [-c] [--update] [-o | -b]

options:
  -h, --help        show this help message and exit
  -u, --unstable    Fetch the latest unstable brunch release instead of stable
  -i, --info        Only display the latest available versions without downloading
  -c, --cleanup     Delete downloaded recovery zip/bin files and Brunch archives to free space
  --update          Perform a git pull to fetch the latest version of this updater tool
  --dry-run         Perform a full version check and display results without downloading or updating

Operational Modes (Mutually Exclusive):
  -o, --os-only     Only download and update ChromeOS
  -b, --brunch-only Only download and update the Brunch framework
```

---

## 💡 Practical Examples

### 1. Perform a Safe Version Inquiry (Dry Run)

Inspect what your current hardware board reports and contrast it with the newest builds available on upstream servers without altering files:

```bash
brcru --info

```

### 2. Standard Monolithic Upgrade

Download and upgrade both the Brunch base layer and matching ChromeOS target milestone simultaneously:

```bash
brcru
```

### 3. Upgrade the Brunch Layer Individually

Perfect for applying rapid stability or kernel patches without redownloading the main ChromeOS system file:

```bash
brcru --brunch-only
```

### 4. Reclaim Disk Storage Space

Once the upgrade is finalized and your machine successfully boots, run the storage cleaner to delete cached setup files (`.zip`, `.bin`, and `.tar.gz` configurations):

```bash
brcru --cleanup

```

### 5. Update this Tool

Seamlessly fetch the latest version of the tool code repository using upstream tracking:

```bash
brcru --update

```

---

## 🔒 Process Exclusivity & Integrity Details

The single-instance protection module operates on standard POSIX advisory locking structures.

```text
[User Runs Script] ──► [Checks /usr/local/brcr_updater/brcr_updater.lock]
                             │
                             ├─► [Lock Free] ──► Secures Lock & Executes Pipeline
                             │
                             └─► [Lock Busy] ──► Aborts with Error (Exit Code 9)

```

If a crash or structural `KeyboardInterrupt` occurs mid-flight, the kernel automatically clears the active file descriptor tracking handle. You do **not** need to manually delete stale lockfiles from your file system.