# Disk full: writes and installs fail

Date: 2026-11-17
Area: Linux, storage

## Symptom
Apps cannot save files. `dd` and package installs fail with "No space left on device".

## First commands
- `df -h`: `/` on /dev/sda1 at 100%, 456K free.
- `sudo du -xh / --max-depth=1 2>/dev/null | sort -h | tail`: /var holds 16G of 19G.
- `sudo find / -xdev -size +1G 2>/dev/null`: one 16G file, /var/tmp/.cache-blob.

## Cause
A single large file in /var/tmp filled the root filesystem.

## Fix
    ls -lh /var/tmp/.cache-blob
    sudo lsof /var/tmp/.cache-blob   # confirm nothing has it open
    sudo rm /var/tmp/.cache-blob
    df -h /

If `df` stays full after deleting, a process still holds the file open. Find it with `sudo lsof +L1` and restart that process.

## How to prevent it
Alert when a filesystem passes 80 percent. On Kubernetes nodes, the kubelet marks the node DiskPressure and starts evicting pods, so watch node disk and image garbage collection thresholds.
