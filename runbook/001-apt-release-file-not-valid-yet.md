# apt fails: release file "not valid yet"

Date: 2026-11-17
Area: Linux, time sync

## Symptom
`apt update` fails with: Release file ... is not valid yet (invalid for another 7h 55min). Packages do not install.

## First commands
- `date`: the VM clock was about 8 hours behind real time.
- `chronyc tracking`: showed a large offset from the time servers.

## Cause
The VM clock stopped while the host Mac slept. Ubuntu signs its package lists with a valid-from time, so a clock in the past makes valid signatures look like they are from the future. Expired or not-yet-valid TLS certificates fail the same way.

## Fix
    sudo chronyc makestep

## How to prevent it
Let chrony step the clock whenever it drifts by more than one second:

    echo "makestep 1 -1" | sudo tee /etc/chrony/conf.d/makestep.conf
    sudo systemctl restart chrony

On real nodes, alert on clock offset. Kubernetes certificates, etcd and log timelines all depend on correct time.
