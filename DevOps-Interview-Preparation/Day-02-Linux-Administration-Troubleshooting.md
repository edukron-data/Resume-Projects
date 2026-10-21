# Day 2: Linux Administration and Troubleshooting — Interview Questions and Answers

## Interview questions

### Q1. How do you inspect and stop processes?
**Answer:**

Use `ps aux`, `pgrep -af name`, and `top` to inspect processes. Send  
`SIGTERM` with `kill PID` for graceful shutdown; use `SIGKILL` only when  
necessary because it prevents cleanup. Prefer the service manager for  
supervised applications, since it may otherwise restart the process.

Before stopping a process, I identify its owner, parent, command line,  
and role in the application. A graceful signal gives it time to finish  
requests and release resources. After stopping it, I verify whether a  
supervisor recreated it and whether dependent services remain healthy.

### Q2. What is the difference between systemctl start and enable?
**Answer:**

`systemctl start nginx` starts a service now. `systemctl enable nginx` configures  
startup through the unit's installation rules, usually at boot; it does  
not necessarily start it immediately. `enable --now` does both. Use `status`,  
`is-active`, and `is-enabled` to inspect separate runtime and boot states.

For a newly installed service, I verify both that it is running  
now and that it will start after a reboot. A masked unit is different  
from a disabled unit and may refuse activation. I inspect unit  
dependencies when startup behavior does not match expectations.

### Q3. How do you inspect logs with journalctl?
**Answer:**

`journalctl -u nginx --since '30 minutes ago'` filters by service and  
time. `journalctl -b` selects the current boot, `-f` follows new messages,  
and `-p err` selects error severity and higher. Availability of  
older boots depends on journal persistence and retention configuration.

I narrow logs by service and incident window to reduce noise, then correlate  
application messages with system events. Permission and retention settings may limit  
visibility. When investigating a restart, I inspect the messages immediately  
before termination instead of looking only at successful startup entries afterward.

### Q4. How do cron jobs work?
**Answer:**

User crontabs have five time fields followed by a command: minute, hour,  
day of month, month, and day of week. `0 2 * * * /opt/jobs/backup.sh`  
runs daily at 02:00 in the scheduler's configured timezone. Use absolute  
paths, explicit environment settings, logging, and overlap protection.

For example, a nightly backup should record its start, end, and exit  
status and should not overlap the previous run. I verify the execution identity  
and timezone explicitly. When stronger dependency handling or missed-run  
behavior is needed, I consider a suitably configured systemd timer.

### Q5. How do you investigate disk usage?
**Answer:**

`df -h` reports filesystem space and `df -i` inode usage. `du -xhd1 /var`  
finds large directories without crossing filesystems. `lsblk -f` identifies  
block devices and filesystems. Deleted files still held open by processes can  
explain why `df` usage exceeds visible file totals; inspect with `lsof +L1`.

Space exhaustion and inode exhaustion need different remedies, so I check both  
before cleanup. I identify which filesystem actually contains the affected application  
path. After freeing capacity, I confirm the responsible process can write  
again and investigate whether logs, temporary files, or growth caused the incident.

### Q6. How do you interpret memory usage?
**Answer:**

`free -h` separates used memory, buffers/cache, and available memory. Low  
free memory alone is not evidence of exhaustion because Linux uses memory  
for caching. Check available memory, swap activity with `vmstat 1`, process  
RSS, cgroup limits, and kernel OOM messages to determine actual pressure.

For a suspected memory leak, I compare process memory over time with request  
volume and deployment changes. Swap activity and OOM events provide stronger  
evidence than one snapshot. I distinguish host memory pressure from a  
container limit because a container can fail while the host still has capacity.

### Q7. What does load average tell you?
**Answer:**

Load average measures runnable tasks and tasks in uninterruptible sleep  
over one, five, and fifteen minutes. Compare it with CPU capacity and investigate  
I/O waits. High load does not always mean CPU saturation; blocked  
storage operations can raise it even when CPU utilization is relatively low.

For example, a load average of eight has different implications on a  
two-core server and a thirty-two-core server. I inspect the trend and task  
states together. CPU utilization, I/O wait, and application latency help  
explain whether users are affected and which resource is constrained.

### Q8. Which networking commands are useful for troubleshooting?
**Answer:**

`ip addr` checks interfaces, `ip route` routing, `ss -lntp` listening  
TCP sockets, and `dig` or `getent hosts` name resolution. `curl -v` tests  
application connectivity. `ping` tests ICMP reachability where allowed,  
but failure does not prove the destination application is unavailable.

I work from name resolution to routing, listening ports, and application responses  
rather than running commands without a hypothesis. For HTTPS failures, I inspect  
certificate and protocol errors too. A reachable TCP port only proves connection  
establishment, not successful authentication or correct application behavior.

### Q9. How do you analyze application logs effectively?
**Answer:**

Establish the incident window and timezone, identify affected requests, and  
correlate application, reverse-proxy, system, and dependency logs using timestamps  
or request IDs. Use `grep`, `less`, and structured log queries to identify the  
first meaningful failure rather than focusing only on repeated downstream errors.

I preserve timestamps and correlation identifiers when collecting evidence so separate  
services can be compared accurately. Structured fields are preferable to searching  
arbitrary text. After finding a likely cause, I verify that its timing and affected  
requests explain the customer impact rather than assuming every error is relevant.

### Q10. How do partitions, filesystems, and mounts differ?
**Answer:**

A partition divides a block device. A filesystem organizes data on a device  
or logical volume. A mount attaches a filesystem to the directory tree.  
Persistent mounts commonly use `/etc/fstab`; UUIDs avoid reliance on device-name  
ordering. Resizing requires filesystem-specific steps and capacity checks.

Before modifying disks, I map the application path to its filesystem and  
underlying device or logical volume. Backups and a recovery plan are necessary  
for risky changes. After resizing or changing mounts, I verify available  
capacity, filesystem health, persistent configuration, and application access.

### Q11. How do you investigate CPU and I/O performance?
**Answer:**

Start with `uptime`, `top`, and `vmstat 1`; use `pidstat 1` or `iostat  
-xz 1` if installed. Determine whether CPU is busy, tasks wait for storage,  
or one process dominates. Capture representative intervals and compare  
against normal behavior before changing capacity or tuning parameters.

I collect several intervals because a single sample may capture a temporary  
spike or miss the problem entirely. Per-process measurements connect system  
pressure to a workload. Any tuning change should have a clear hypothesis,  
a measurable success criterion, and a reversal plan if performance worsens.

### Q12. What happens after changing a systemd unit file?
**Answer:**

Run `systemctl daemon-reload` so systemd rereads unit definitions, then restart  
the affected service when needed. A service's `reload` operation asks that service  
to reload its own configuration when supported. These operations are different;  
validate service configuration before restarting and check health afterward.

For example, changing `ExecStart` requires systemd to reload its unit definitions,  
while changing an NGINX configuration requires NGINX to reload or restart.  
I validate both kinds of configuration before acting. Finally, I inspect  
status and logs and test an actual request to confirm the intended behavior.

## Scenario-based questions

### S1. A server reports 100% disk usage and the application stops. What do you do?
**Answer:**

Confirm the affected filesystem with `df -h` and check inodes with `df -i`. Inspect large paths and  
deleted open files. Preserve incident evidence and remove or rotate only identified disposable data  
under the retention policy. If a deleted log remains open, arrange a controlled reopen or restart.  
Verify free space, application writes, and health; then correct retention or capacity thresholds.

I avoid deleting unknown application data simply because it is large. If  
immediate space is required, I choose an approved reversible mitigation where  
possible and coordinate with the data owner. Long-term prevention includes retention  
limits, growth monitoring, and alerts early enough to allow intervention.

### S2. A service fails immediately after a configuration change. How do you recover?
**Answer:**

Inspect `systemctl status service` and its journal around the deployment. Run the application's  
configuration validator, such as `nginx -t` for NGINX. Check changed paths, permissions,  
dependencies, and occupied ports. Restore the last valid configuration if needed, restart  
through systemd, and verify both local and external requests before declaring recovery.

If restart attempts are rate-limited, I first fix the underlying configuration  
and then address the failed-state handling as appropriate. Repeated blind  
restarts create noise and can extend impact. I compare the restored configuration  
with source control so the next deployment does not reintroduce the same fault.

### S3. A cron job works manually but fails overnight. Why?
**Answer:**

Reproduce execution as the cron user with a minimal environment. Check PATH, working directory,  
interpreter, executable permissions, credentials, timezone, and output redirection. Interactive shell profiles  
are not automatically available. Use absolute paths and explicit settings, capture stderr and exit  
status, and add locking if overlap caused the failure. Confirm the next scheduled execution succeeds.

I also check whether the scheduled run uses a different home directory,  
credential cache, or filesystem mount than the manual test. The job should report  
failure clearly to its scheduler or monitoring system. A successful manual  
rerun alone does not demonstrate that the scheduled environment is repaired.

### S4. Load is high but CPU utilization is low. What is likely happening?
**Answer:**

Investigate blocked tasks and storage or network filesystem latency rather than assuming more  
CPUs will help. Check `vmstat`, disk latency, process states, kernel logs, and NFS health if  
used. Identify the blocked dependency, reduce harmful concurrency or restore that dependency,  
and verify load and request latency return to baseline without introducing data corruption.

For example, a stalled network filesystem can leave many tasks waiting without  
consuming much CPU. I check the underlying dependency before restarting the application  
because restarting may create more blocked work. Recovery is confirmed when  
blocked operations complete and normal throughput returns without data inconsistency.
