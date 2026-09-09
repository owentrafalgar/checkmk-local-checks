# checkmk-local-checks

Backup and single source of truth for all custom Checkmk local check scripts running across
the homelab. Each host clones this repo and symlinks the relevant scripts into place, so
editing a script here (on any host) is the same as editing the live version.

## Inventory

| File                          | Host(s)          | Target path                                          | Notes |
|-------------------------------|------------------|-------------------------------------------------------|-------|
| docker/tld_updater_status     | docker           | /usr/lib/check_mk_agent/local/tld_updater_status       | No interval dir - piggybacks to `tld-updater` host |
| docker/ddclient                | docker           | /usr/lib/check_mk_agent/local/3600/ddclient            | Piggybacks to `ddclient` host |
| docker/immich                  | docker           | /usr/lib/check_mk_agent/local/300/immich               | Needs /etc/check_mk/immich.secrets (not in repo) |
| docker/unifi_protect_backup    | docker           | /usr/lib/check_mk_agent/local/unifi_protect_backup     | Piggybacks to `unifi-protect-backup` host |
| proxmox-pbs/backup_status      | proxmox AND pbs  | /usr/lib/check_mk_agent/local/3600/backup_status       | Identical script on both hosts |
| rpi/check_throttled            | pihole (RPi)     | /usr/lib/check_mk_agent/local/60/check_throttled       | Reads `vcgencmd get_throttled` |
| rpi/check_fan                  | pihole (RPi)     | /usr/lib/check_mk_agent/local/60/check_fan             | Reads PWM fan via hwmon |

## Setup on a host (one-time)

    git clone https://github.com/owentrafalgar/checkmk-local-checks.git /opt/checkmk-local-checks

Then symlink the relevant scripts (see per-host commands below).

## Making changes

Edit the file directly under /opt/checkmk-local-checks/... (this repo), then:

    cd /opt/checkmk-local-checks
    git add -A
    git commit -m "describe the change"
    git push

On other hosts, pull the update when convenient:

    git -C /opt/checkmk-local-checks pull

No restart needed - scripts are read fresh by the Checkmk agent on every poll (or every cache
interval for scripts in numbered folders).

## Why some scripts have no interval directory

Scripts that emit piggyback markers (`<<<<hostname>>>>`) must live directly under `local/`
(no numbered interval subfolder). The numbered-folder caching mechanism does not preserve
piggyback headers correctly - only the plain check line gets cached, breaking the piggyback
routing. Regular (non-piggyback) checks are fine in interval folders for caching/performance.

## Immich API key

`docker/immich` reads its API key from `/etc/check_mk/immich.secrets`, which is NOT part of
this repo (see `.gitignore`). To set it up on the docker host:

    mkdir -p /etc/check_mk
    cp docker/immich.secrets.example /etc/check_mk/immich.secrets
    # then edit /etc/check_mk/immich.secrets and insert the real API key
