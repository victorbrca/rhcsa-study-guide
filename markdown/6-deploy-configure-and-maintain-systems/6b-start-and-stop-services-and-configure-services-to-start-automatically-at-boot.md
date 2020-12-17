# 6.b Start and stop services and configure services to start automatically at boot

**Systemctl options to Know**
- status - Show terse runtime status information about one or more units
- list-unit-files - List unit files installed on the system
- list-units - List units that systemd currently has in memory
- enable - Enable one or more units or unit instances
- Remember that it creates a symlink (/etc/systemd/system/[target]/)
- disable - Disables one or more units.
- start - Start (activate) one or more units specified on the command line
- stop - Stop (deactivate) one or more units specified on the command line
- restart - Stop and then start one or more units specified on the command line
- mask - Mask one or more units, as specified on the command line
- unmask - Unmask one or more unit files, as specified on the command line
- reload - Asks all units listed on the command line to reload their configuration
- daemon-reload - Reload the systemd manager configuration
- is-enabled - Checks whether any of the specified unit files are enabled
- is-failed - Check whether any of the specified units are in a "failed" state
- cat - Show backing files of one or more units
- list-dependencies - Shows units required and wanted by the specified unit

## Mask vs Disable

### Disable

Disabling the service deletes the symlink, so the unit file itself is not affected, but the service is not loaded at the next boot, when systemd reads `/etc/systemd/system`.
However, a disabled service can be loaded, and will be started if a service that depends on it is started; `enable` and `disable` only configure auto-start behaviour for units, and the state is easily overridden.

### Mask

A masked service is one whose unit file is a symlink to /dev/null. This makes it "impossible" to load the service, even if it is required by another, enabled service.
When you mask a service, a symlink is created from /etc/systemd/system to /dev/null, leaving the original unit file elsewhere untouched. When you unmask a service the symlink is deleted.

---
[⬅️ Back](6-deploy-configure-and-maintain-systems.md)
