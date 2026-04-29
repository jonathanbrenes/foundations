# Module 2 - OS Fundamentals II

## Module Overview

- **Training day:** Day 1 (Part A, 2h) and Day 2 (Part B, 1h)
- **Planned duration:** 3 hours
- **Time breakdown:** ~1.5h reading and concept primer | ~1h lab execution (Labs 06-07) | ~30m knowledge check and completion criteria
- **Required VM(s):** SLES 16, RHEL 10
- **Prior module required:** Module 1 - OS Fundamentals I

## Learning Objectives

- Create and validate hard and soft links.
- Manage Linux services and review service logs with system tools.
- Modify and edit files, and work with compression and extraction utilities.
- Use performance and diagnostic tools such as `vmstat`, `iostat`, `iotop`, and `sar`.
- Work with cron, date/time, and network time synchronization basics.
- Collect support data with `sosreport` or `supportconfig` awareness.

## Key Topics

- Hard and soft links: creation, comparison, and real-world use cases.
- Linux service management with `systemctl` and `journalctl`.
- File modification and editing, file navigation, and file operations.
- Compression and extraction basics.
- `vmstat`, `iostat`, `iotop`, and `sar` overview.
- `sosreport` and `supportconfig` awareness.
- Cron jobs, date and time, and network time protocol basics.

## Command Highlights

**Links:**
- `ln`, `ln -s`, `ls -i`, `stat`, `find -type l`, `find -links`

**Service Management:**
- `systemctl status`, `systemctl start`, `systemctl stop`, `systemctl restart`
- `systemctl enable`, `systemctl disable`, `systemctl list-units`, `systemctl list-unit-files`
- `journalctl -u`

**Performance & Diagnostics:**
- `vmstat`, `iostat`, `iotop`, `sar`

**Support Data:**
- `sosreport`, `supportconfig`

**Scheduling & Time:**
- `crontab`, `date`, `timedatectl`, `chronyc`

## Labs

### Lab 06 — Hard and Soft Links
- **Duration:** ~20-30 minutes
- **VM:** SLES 16 (FoundationsLab06.json)
- **Objectives:**
  - Create hard links and understand shared inode behavior
  - Create symbolic links (soft links) and verify independent inode behavior
  - Compare link behavior when original files are deleted
  - Understand hard link limitations: directories and filesystem boundaries
  - Create cross-filesystem symbolic links
  - Compare hard and soft links side by side in a practical exercise
- **Key Commands:** `ln`, `ln -s`, `ls -i`, `stat`, `find -type l`

### Lab 07 — Managing Services in Linux
- **Duration:** ~30 minutes
- **VM:** RHEL 10 (FoundationsLab07.json)
- **Objectives:**
  - List and inspect systemd units using `systemctl`
  - Filter units by type and state (service, active, running)
  - Investigate failed services and review logs with `journalctl`
  - Manage service lifecycle: start, stop, restart, status
  - Control boot behavior: enable and disable services
- **Key Commands:** `systemctl`, `systemctl list-units`, `systemctl list-unit-files`, `systemctl status`, `journalctl -u`
