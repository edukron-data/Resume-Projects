# Day 1: Linux Fundamentals — Interview Questions and Answers

## Interview questions

### Q1. Explain Linux architecture.
**Answer:**

Hardware provides CPU, memory, storage, and networking. The kernel manages  
these resources and exposes system calls. User-space programs, libraries,  
services, and shells use those interfaces. A shell interprets commands; it is  
not the kernel. Distributions package these components with management tools.

For example, running a command starts a user-space process that requests  
kernel services to read files or open network connections. During  
troubleshooting, I identify whether the problem belongs to the application, operating  
system, or underlying hardware before selecting diagnostic commands.

### Q2. What are the important Linux directories?
**Answer:**

`/etc` contains system configuration, `/var` changing data such as logs, `/home` user  
files, `/tmp` temporary files, `/usr` installed user-space programs and libraries,  
and `/boot` boot components. `/proc` and `/sys` expose kernel information. `/dev`  
contains device interfaces. Some distributions merge `/bin` and `/sbin` into `/usr`.

In administration, I check configuration under `/etc`, service logs under  
`/var/log`, and process details under `/proc`. Understanding these  
locations reduces investigation time. I also distinguish persistent files from  
temporary or virtual files before editing, copying, or backing them up.

### Q3. Which basic commands do you use daily?
**Answer:**

`pwd`, `ls -la`, and `cd` navigate files. `cp`, `mv`, `mkdir`, and  
`rm` manipulate paths. `less`, `head`, and `tail` inspect content.  
`stat` shows metadata and `file` identifies file types. Quote paths  
containing spaces and inspect targets before recursive changes.

For example, I use `tail -f` to watch a log during a deployment  
and `less` to search older entries. Before moving or removing  
application files, I confirm the current directory, ownership,  
and exact target so the command affects the intended release.

### Q4. Explain file and directory permissions.
**Answer:**

Read, write, and execute permissions apply to owner, group, and others. For  
a directory, read lists names, execute permits traversal, and write permits  
changing directory entries when traversal is also allowed. Removing a file  
is primarily controlled by its parent directory, not the file's write bit.

For example, a user may read a file but still fail to access  
it because a parent directory lacks execute permission. I inspect  
every directory in the path and check ACLs or security  
policies when ordinary permission bits do not explain the denial.

### Q5. What do chmod 640 and chmod 750 mean?
**Answer:**

Permission values are read 4, write 2, and execute 1. `chmod 640 config`  
gives the owner read/write, the group read, and others nothing. `chmod  
750 script` gives the owner all three permissions, the group read/execute,  
and others nothing. Symbolic syntax includes `chmod u+x script`.

For a private application configuration, `640` can allow an administrator  
to update it while the application group only reads it. For a deployment  
script, `750` can allow an authorized group to run it. I select permissions  
from actual access requirements rather than copying defaults blindly.

### Q6. How do chmod, chown, and chgrp differ?
**Answer:**

`chmod` changes permission bits, `chown` changes ownership, and `chgrp`  
changes the owning group. For example, `sudo chown appuser:appgroup  
/srv/app/config` assigns an owner and group. Recursive ownership changes  
require care because they may overwrite intentional ownership boundaries.

A file can have correct permission bits but still belong to the wrong  
application account. I inspect both ownership and mode before changing  
anything. After a correction, I test the operation using the actual service  
identity because a successful root command can hide the original problem.

### Q7. How do you manage users and groups?
**Answer:**

Use `id`, `getent passwd`, and `getent group` to inspect identities. `useradd  
-m username` creates a user on many distributions; `usermod -aG group username`  
appends a supplementary group. Without `-a`, `-G` can replace existing  
supplementary groups. New group membership normally requires a fresh login session.

Groups simplify shared access because permissions can be assigned to a team  
rather than repeated for individual accounts. For service accounts, I avoid  
interactive login unless required. I verify group membership inside the session  
performing the deployment, especially after an administrator changes membership.

### Q8. What is sudo, and how is it different from su?
**Answer:**

`sudo` executes an authorized command as another user and can provide an  
audit trail. `su` switches identity, commonly starting a shell. Configure sudo  
privileges through `visudo`, apply least privilege, and avoid granting  
unrestricted root access when only a particular operational command is needed.

For example, an operator may need permission to restart one service without  
permission to modify every system file. A narrowly defined sudo rule supports  
that requirement. I check command paths, arguments, and audit logs carefully  
because a seemingly limited command may still permit privilege escalation.

### Q9. How does SSH key authentication work?
**Answer:**

The client proves possession of a private key corresponding to an authorized  
public key on the server. The private key stays with the client. Server host  
keys establish server identity. Use protected private keys, verify host  
fingerprints, and investigate host-key changes instead of bypassing verification.

I generate a dedicated key pair for the intended purpose and install only  
its public key on the server. The private key is protected and never shared  
in a repository. Troubleshooting includes checking authorized-key paths,  
ownership, server policy, and whether the client selected the expected key.

### Q10. How do you copy files securely between servers?
**Answer:**

`scp ./app.tar.gz deploy@server:/srv/releases/` copies a file over  
SSH. Use `scp -r` for directories. `scp` uses uppercase `-P` for  
a custom SSH port, while `ssh` uses lowercase `-p`. `rsync` over SSH  
is useful for incremental transfers when installed on both ends.

For deployments, I transfer to an approved staging location, verify ownership  
and integrity, then activate the release through the deployment process. Large  
repeated transfers may benefit from rsync. I also confirm available disk space  
and destination permissions before copying to avoid incomplete release content.

### Q11. How does package management differ across distributions?
**Answer:**

Debian and Ubuntu commonly use APT, for example `sudo apt update` followed by `sudo  
apt install nginx`. RHEL-family systems commonly use DNF, for example `sudo dnf  
install nginx`. Updating repository metadata is different from upgrading installed  
packages. Trusted repositories and controlled version changes improve reproducibility.

In production, I confirm the operating-system version and repository policy  
before installing or upgrading packages. A package update can restart services  
or change dependencies. I review the expected change, follow the maintenance  
process, and verify service health and the installed package version afterward.

### Q12. Explain umask, symbolic links, and hard links.
**Answer:**

`umask` removes default permission bits when files or directories are created;  
applications can request stricter modes. A symbolic link stores a target  
path and can cross filesystems. A hard link references the same inode,  
normally cannot cross filesystems, and generally cannot target directories.

For example, removing one hard-link name does not remove data while another  
link remains, whereas removing a symbolic link leaves its target  
intact. During deployments and backups, I inspect links deliberately because  
following them can copy or modify files outside the expected directory.

## Scenario-based questions

### S1. A deployment user receives Permission denied when running a script. What do you check?
**Answer:**

Check `id`, `ls -l script.sh`, and `namei -l /full/path/script.sh` to inspect the user, file permissions,  
and parent traversal permissions. Confirm the interpreter exists and inspect line endings. Check  
mount options for `noexec` and SELinux/AppArmor denials where applicable. Correct the specific owner,  
group, permission, or policy issue; validate as the deployment user instead of applying `chmod 777`.

I also compare how the script is launched: direct execution needs a usable  
interpreter and execution permission, while invoking Bash reads the  
script as input. After fixing the cause, I rerun the exact deployment  
command and confirm both its exit status and expected application result.

### S2. SSH access suddenly fails. How do you troubleshoot?
**Answer:**

Separate timeout, connection refused, host-key mismatch, and authentication failure. Use `ssh -vvv  
user@host` and test the correct IP and port. Through approved console access, check the SSH  
service, listening socket, firewall rules, authentication logs, disk space, and authorized-key  
ownership. Restore the specific fault and confirm a new session works before closing recovery access.

The error category guides the investigation: a timeout suggests a network path  
problem, while authentication failure points toward identity or policy. I  
compare recent network and access changes with the failure time. Once restored,  
I check server logs to ensure the fix did not introduce unintended access.

### S3. A team must share a deployment directory without giving everyone root. What would you configure?
**Answer:**

Create a deployment group, add authorized users, assign the directory to that group, and set  
appropriate group write/traverse permissions. A setgid directory such as mode `2770` makes new entries  
inherit its group. Use a suitable umask or default ACL for inherited write access. Verify behavior  
with files created by two different users and grant only narrowly scoped sudo commands if needed.

I test directory creation, file editing, and deletion separately because those  
actions depend on different permissions. Shared write access should be limited  
to the deployment team. If services also read these files, I confirm the inherited  
permissions remain compatible with the service account after each release.

### S4. Installing a package fails with Unable to locate package. What is your approach?
**Answer:**

Identify the distribution and release, verify the exact package name, and refresh repository  
metadata. Check repository configuration, supported release status, DNS, proxy settings, and  
repository reachability. Confirm whether the package belongs to an additional approved repository.  
Install from the trusted source and verify the executable or service version afterward.

If metadata refresh fails, I investigate that failure before repeatedly attempting  
installation. An incorrect release repository or unavailable mirror can make valid  
package names appear missing. I avoid downloading arbitrary installers to bypass the  
problem and record any approved repository addition for repeatable server provisioning.
