# Live Resize of A `btrfs` Partition

*First edition 2019-11-02.*
*Updated 2020-01-16.*

## Preps

The following packages are needed to provide necessary functions:

* `gdisk`: adjusting GPT.
* `parted`: includes `partprobe` which can inform the kernel of partition table change.
* `btrfs-progs`: resizes the file system.

## Steps

GPT stores a backup header at the end of the disk. If you just enlarged the VHD, the backup header would be somewhere in the middle, which would limit the available space GPT can address. Fix the GPT by rebuilding the backup header:

```bash
# gdisk
/dev/sda
x
e
```

Now that we have a good GPT, the partition can now be enlarged by deleting and recreating it.

```bash
m
d
n
```

Verify your changes and save the GPT:

```bash
p
w
```

After saving the new GPT, use `partprobe` to inform the kernel of the change:

```bash
# partprobe /dev/sda
```

Finally, use the `btrfs` tool to resize the file system:

```bash
# btrfs filesystem resize max /
```

## References

* man page: `btrfs-filesystem(8)`
* [partitioning - Live resize of a GPT partition on Linux - Super User](https://superuser.com/questions/660309/live-resize-of-a-gpt-partition-on-linux)