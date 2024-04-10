Atomic Disk Write
====

Implementations
----

* [SQLite](https://www.sqlite.org/atomiccommit.html)
* [PostgreSQL](https://wiki.postgresql.org/wiki/Full_page_writes)

Readings
----

[stack overflow: Are disk sector writes atomic?](https://stackoverflow.com/a/61832882/1491175)

[Arch Linux: Advanced Format](https://wiki.archlinux.org/title/Advanced_Format)

[AWS: Torn write prevention](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/storage-twp.html)

[Linux Kernel Mail List: [PATCH v6 00/10] block atomic writes](https://lkml.org/lkml/2024/3/26/746)

Considerations
----

1. **Isolation**: A write operation to one block must not interfere with other blocks.
2. **Alignment**: The block boundaries need to be kept by the filesystem so that they are still visible as offsets in a file.
3. **Native Atomicity** (optional): If a block cannot be written atomically natively, double-write and checksum is needed.

### Isolation

If we assume that the writing operations for different sectors are isolated, an application needs to be able to access the sector size of underlying device of a file.

`stat()` can be used to read the block size of underlying filesystem. The value is `st_blksize`. Ideally, the block size of filesystem is equal or larger than the sector size of underlying device. However, it's possible that the block size of filesystem is misconfigured when it's formatted, which would also break the internal journaling mechanism of the filesystem.

`lsblk -td` can be used to check the sector size of a block device.

Sometimes, the sector size can be changed. For a SCSI disk, it can be done with

    hdparm --set-sector-size 4096 --please-destroy-my-drive /dev/sdX

For a NVMe disk, it can be done with

    nvme format --lbaf=1 /dev/nvmeXnX

#### Concerns of Cache Page Size

When I/O is done with `mmap`, the minimum unit of a writing operation cannot be smaller than a page. If a page is larger than a sector, the isolation among sectors is still broken implicitly.

### Alignment

XFS supports setting block size to a value from 512 to 65536. The default is 4096.

EXT4 filesystem supports `bigalloc` feature to ensure the alignment of extents of files.

### Native Atomicity

NVM disks seem all supporting certain level of native atomicity, which is exposed with `AWUPF` field.