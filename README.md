# Linux Administration Lab

## Objective
Practice core Linux system administration skills on a dedicated Ubuntu 
Server VM: user/group management, permissions, SSH, firewall (UFW), 
a real service (Nginx), logging, and task automation via cron and a 
custom backup script.

## Technologies
- Ubuntu Server 26.04 LTS
- OpenSSH
- UFW (Uncomplicated Firewall)
- Nginx
- Bash scripting, cron

## What was done

### Users, Groups, Permissions
- Created multiple user accounts, granted sudo access via group membership
- Demonstrated default access restrictions (a new user has no sudo by 
  default; home directories are not accessible to other users by default)
- Set up proper file sharing between users using a dedicated group 
  (`shared-team`) and a shared location outside any single user's home 
  directory (`/srv/shared-team`), rather than relying on loosened home 
  directory permissions

### SSH
- Installed and configured OpenSSH server
- Hardened configuration by disabling root login (`PermitRootLogin no`)
- Verified remote access from host machine via VirtualBox NAT port 
  forwarding

### Firewall (UFW)
- Enabled UFW with default deny-incoming policy
- Explicitly allowed SSH before enabling the firewall (avoiding a 
  lockout), then allowed Nginx/HTTP

### Nginx
- Installed and verified a running web service
- Exposed it through UFW and VirtualBox port forwarding, confirmed 
  access from the host browser

### Logs
- Reviewed Nginx access/error logs and systemd journal entries
- Used `journalctl -u ssh` to directly observe failed root login 
  attempts versus a successful standard user login, confirming the 
  SSH hardening was effective

### Automation (cron + backup script)
- Wrote `backup.sh`: creates a timestamped, compressed backup of a 
  target directory, with error handling and logging
- Scheduled it via cron to run daily

## Problems encountered
- Package installation initially failed with 404 errors from Ubuntu 
  mirrors due to a stale local package index; resolved with `apt update` 
  before retrying the install
- A port forward for HTTP testing (8080) conflicted with another local 
  service on the host, causing the browser to load an unrelated login 
  page; resolved by using a different host port (8888)
- The backup script's `tar` command referenced `$HOME` directly instead 
  of consistently deriving paths from the `SOURCE_DIR` variable, causing 
  it to look in the wrong user's home directory even after `SOURCE_DIR` 
  was corrected — fixed using `dirname`/`basename` on `SOURCE_DIR` 
  consistently
- The backup source directory was initially nested inside another 
  user's home directory, which is inaccessible by default regardless of 
  group permissions on the file itself, due to restrictive home 
  directory permissions — resolved by moving the backup target to a 
  dedicated shared location
- A user added to a new group did not gain that group's access until 
  the session was refreshed (`newgrp`), since group membership is 
  evaluated at login/session start, not dynamically
