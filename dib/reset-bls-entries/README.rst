reset-bls-entries
=================

Clears the machine-id in the image and renames `/boot/loader/entries`
filenames to match the machine-id generated at first boot.

`kernel-install` names BLS entries `<machine-id>-<kernel>.conf`. The finalise
hook truncates `/etc/machine-id` (empty, not "uninitialized", so first boot
semantics are not triggered), and `reset-bls-entries.service` renames the
entries on every boot. The rename is idempotent and also recovers systems
whose machine-id is regenerated after deployment.

TuneD's `92-tuned.install` hook only patches entries matching
`<machine-id>-*.conf` and silently exits 0 when none match.
