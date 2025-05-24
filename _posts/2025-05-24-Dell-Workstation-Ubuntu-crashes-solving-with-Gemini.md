---
layout: post
title: Fixing a Gnome Shell Crash (SIGSEGV in libgallium) on Ubuntu
date: 2025-05-24 10:00:00 +0100
categories: [Ubuntu, Troubleshooting, Gnome, Graphics]
---

## Solving a Gnome Shell Crash: From `deb-src` to NVIDIA Drivers

Recently, I encountered persistent `gnome-shell` crashes on my Ubuntu system (Dell Precision Tower 5810), leading to frustrating desktop freezes and restarts. The `apport-retrace` utility provided a crucial stack trace that pointed towards issues within the graphics stack. This post details the steps taken to diagnose and resolve the problem.

I found out about apport-retrace and was trying to redirect output to a log.txt file, but getting deb-src errors. After some investigation, I realized that the issue stemmed from missing `deb-src` entries in my sources list, which are necessary for downloading source packages.

### Initial Problem: `deb-src` URIs Missing

When attempting to retrace the crash with `apport-retrace`, the first hurdle was an error: "E: You must put some 'deb-src' URIs in your sources.list". This meant the system couldn't download the source code necessary for `apport-retrace` to function fully.

**Key Learning:** On modern Ubuntu versions (like 22.04+), `sources.list` has been modularized. Instead of editing `/etc/apt/sources.list` directly for source repositories, you'll find source definitions in files within `/etc/apt/sources.list.d/` (e.g., `ubuntu.sources`).

**Solution:**
1.  List files in the sources directory: `ls /etc/apt/sources.list.d/`
2.  Edit the primary source file (e.g., `sudo nano /etc/apt/sources.list.d/ubuntu.sources`).
3.  For each `Types: deb` entry, add `deb-src` to the line, like so:
    ```
    Types: deb deb-src
    URIs: [http://gb.archive.ubuntu.com/ubuntu/](http://gb.archive.ubuntu.com/ubuntu/)
    Suites: noble noble-updates noble-backports
    Components: main restricted universe multiverse
    Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
    ```
4.  Save the file (`Ctrl+X`, then `Y`, `Enter`).
5.  Update package lists: `sudo apt update`

### Diagnosing the Crash: `libgallium` identified

With `deb-src` enabled, `apport-retrace` ran successfully. The resulting `log.txt` showed a `SIGSEGV` (Segmentation Fault) deep within `libgallium-24.2.8-1ubuntu1~24.04.1.so`. Gallium3D is an open-source graphics driver abstraction layer used by Mesa, which handles OpenGL. This immediately suggested a graphics driver-related issue.

**Key Learning:** A `SIGSEGV` crash at low-level library calls like `pthread_kill` is often a symptom, not the root cause. Examining the full stack trace (frames `#6` and higher) helps identify the problematic component, which in this case was clearly `libgallium`.

### The Fix: Switching to Proprietary NVIDIA Drivers

My system has an NVIDIA Quadro K2200, and I was using the open-source `nouveau` driver. The crash in `libgallium` strongly pointed to a driver instability with `nouveau`.

**Solution:**
1.  Open "Software & Updates" (search in Activities).
2.  Navigate to the "Additional Drivers" tab.
3.  Select the **recommended proprietary NVIDIA driver**. For my Quadro K2200 on Ubuntu 24.04, this was **`Using NVIDIA driver metapackage from nvidia-driver-570 (proprietary, tested)`**.
4.  Click "Apply Changes" and allow the installation to complete.
5.  **Reboot your system.**

After the reboot, `gnome-shell` was stable. I will observe the system for a few days to ensure the issue is fully resolved. The NVIDIA driver installation was smooth, and I noticed improved graphics performance immediately.
Verifying with `nvidia-smi` confirmed the proprietary driver was active.

This experience highlights the importance of using appropriate graphics drivers, especially for NVIDIA hardware, to ensure system stability on Linux. Google Gemini was used to support the fault - finding process.