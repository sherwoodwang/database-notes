# Kernel API for I/O

1. **Simple I/O**: `preadv()`, `pwritev()`, `preadv2()`, `pwritev2()`
2. **Memory-Mapped I/O**: `mmap()`
3. **POSIX AIO**: `aio_read()`, `aio_write()`
4. `io_uring`

## Simple I/O

Echo operation involves one syscall at least.

Operations are blocking.

### O_DIRECT

For some filesystems, DMA is implemented. Buffers in user space can be used directly as sources or targets of I/O operations.

The blocks to be written are stored in a file and managed by the filesystem anyway. The filesystem is responsible for maintaining consistency between metadata, such as file size, and the content of the file. It also ensures isolation when a block is reassigned from one file to another. Additionally, the filesystem may need to redirect writes, as seen in mechanisms like Copy-on-Write. It's difficult to integrate these requirements with the semantics of `O_DIRECT` correctly. 
Other applications may perform buffered I/O when one application use `O_DIRECT`, which could be racy. Moreover, pages also need to be locked into memory when an `O_DIRECT` operation is ongoing, causing some potential overhead. See Linus Torvalds' [comments](https://lkml.org/lkml/2007/1/11/129).

## Memory-Mapped I/O

Operations can be done in a zero-copy manner.

Cache might be polluted if `madvice()` is not used properly. See [a performance benchmark](https://smalldatum.blogspot.com/2022/05/using-mmap-with-rocksdb.html) conducted by Mark Callaghan, the author of RocksDB.

The invocation to `madvice()` can be a considerable overhead by itself.

A write to a page can cause an implicit read to fill it.

A page fault can block a thread. The application cannot anticipate it and schedule other computational tasks.

Although Linux does not currently support huge pages as mapped pages of files, a 4KiB page can still be larger than the minimum I/O unit of the underlying hardware.

## POSIX AIO

Too many syscalls to setup one I/O request and get its result.

The current implementation in Linux is provided in user space by glibc.

## io_uring

The availability seems limited for now.

`O_DIRECT` is still available.

`madvice()` can also be submitted with `io_uring`.