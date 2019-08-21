# Snapper manual configurations: Creating and Restoring

Official and unofficial `snapper` documentations only mention the creation and modification of manual configurations:
```bash
# snapper -c config_name create-config /path/to/subvolume
# snapper -c config_name set-config "ALLOW_USERS=www_admin" SYNC_ACL="yes"
```

But no one seems to care, after the creation of snapshots, how do we revert to a previous snapshot?

The `snapper rollback` command only works with root configuration. The `snapper undochange` only works with read-only snapshots taken by snapper. To rollback, we have to manually move snapshots around.

First take a snapshot of the snapshot you want to rollback to:
```bash
# btrfs subvolume snapshot /path/to/subvolume/.snapshots/128/snapshot /path/to/new_sub
```

Then we move the `.snapshots` subvolume to the new snapshot:
```bash
# mv /path/to/subvolume/.snapshots /path/to/new_sub/
```

Now we are cleared to delete the old subvolume:
```bash
# btrfs subvolume delete /path/to/subvolume
```

Last, rename the new snapshot to its original name:
```bash
# mv /path/to/new_sub /path/to/subvolume
```
