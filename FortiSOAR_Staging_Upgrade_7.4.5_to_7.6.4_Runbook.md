# FortiSOAR Staging Upgrade Runbook

## Upgrade Path: 7.4.5 → 7.5.0 (RHEL 9.3 OS Upgrade) → 7.6.4

**Environment:** Single Node (No HA, No Multi-Tenancy, No Docker, No
Offline Repo)\
**Instance:** https://172.16.209.128/login/\
**Document Generated:** 2026-02-13

------------------------------------------------------------------------

# IMPORTANT NOTES

-   7.4.5 → 7.5.0 uses the **upgrade script (.bin method)**.
-   7.5.0 and later uses the **Upgrade Framework (csadm upgrade)**.
-   7.5.0 includes **OS migration to RHEL 9.3**.
-   VM Snapshot is MANDATORY before starting.

------------------------------------------------------------------------

# PHASE 1 --- Pre-Upgrade Preparation (MANDATORY)

## 1. Stop All Schedules

GUI: Automation → Schedules → Stop All Active

Wait until no playbooks are running.

------------------------------------------------------------------------

## 2. Take VM Snapshot

Take full snapshot from hypervisor. Do NOT proceed without snapshot.

------------------------------------------------------------------------

## 3. Export Built-in Connectors

GUI: Settings → Export Wizard → Select Built-in Connectors

------------------------------------------------------------------------

## 4. Verify Disk Space

``` bash
df -h
```

Ensure sufficient free space in: - / - /boot - /var - /opt - /var/log -
/var/tmp

------------------------------------------------------------------------

## 5. Verify No Duplicate fstab Entries

``` bash
cat /etc/fstab
```

------------------------------------------------------------------------

## 6. Unmount External Mounts (If Any)

``` bash
umount <mount_point>
```

Comment in `/etc/fstab` if required.

------------------------------------------------------------------------

## 7. Install tmux

``` bash
yum install -y tmux
tmux
```

------------------------------------------------------------------------

# PHASE 2 --- Upgrade 7.4.5 → 7.5.0 (Major OS Upgrade)

## 1. Download Upgrade Script

``` bash
cd /
wget https://repo.fortisoar.fortinet.com/7.5.0/upgrade-fortisoar-7.5.0.bin
chmod +x upgrade-fortisoar-7.5.0.bin
```

------------------------------------------------------------------------

## 2. Execute Upgrade

``` bash
sh upgrade-fortisoar-7.5.0.bin
```

Expected Time: 30--60 minutes\
This step includes OS migration to RHEL 9.3.

------------------------------------------------------------------------

## 3. Reboot After Upgrade

``` bash
reboot
```

------------------------------------------------------------------------

## 4. Verify Upgrade

``` bash
csadm version
cat /etc/redhat-release
```

Expected: - FortiSOAR 7.5.0 - RHEL 9.3

Logout and login again in GUI.

------------------------------------------------------------------------

# PHASE 3 --- Upgrade 7.5.0 → 7.6.4 (Upgrade Framework)

## 1. Start tmux

``` bash
tmux
```

------------------------------------------------------------------------

## 2. Run Readiness Check

``` bash
csadm upgrade check-readiness --version 7.6.4
```

Report location: /opt/fsr-elevate/elevate/outputs/

Fix any failures before proceeding.

------------------------------------------------------------------------

## 3. (Recommended) Pre-Download Packages

``` bash
csadm upgrade execute --target-version 7.6.4 --download-packages
```

------------------------------------------------------------------------

## 4. Execute Upgrade

``` bash
csadm upgrade execute --target-version 7.6.4
```

Log file:
/var/log/cyops/upgrade-fortisoar-7.6.4-`<timestamp>`{=html}.log

Monitor:

``` bash
tail -f /var/log/cyops/upgrade-fortisoar-7.6.4-*.log
```

Expected Time: 15--30 minutes

------------------------------------------------------------------------

## 5. Verify Final Version

``` bash
csadm version
```

Login to GUI and confirm functionality.

------------------------------------------------------------------------

# POST-UPGRADE TASKS

## Restart Services (Recommended)

``` bash
csadm services --restart
```

------------------------------------------------------------------------

## Verify Connectors

GUI: Automation → Connectors

If system connectors not upgraded:

``` bash
yum reinstall -y cyops-connector-cyops_utilities                 cyops-connector-imap                 cyops-connector-cyops-schedule-report
```

------------------------------------------------------------------------

## Re-enable Schedules

Automation → Schedules → Enable Required Ones

------------------------------------------------------------------------

# ROLLBACK PLAN

If upgrade fails:

1.  Power off VM
2.  Revert to Snapshot
3.  Boot system
4.  Validate system state

------------------------------------------------------------------------

# ESTIMATED DOWNTIME

  Phase           Estimated Downtime
  --------------- --------------------
  7.4.5 → 7.5.0   30--60 mins
  7.5.0 → 7.6.4   15--30 mins

------------------------------------------------------------------------

# END OF DOCUMENT
