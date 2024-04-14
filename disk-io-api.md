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

Cache pollution can be a more significant concern than copying. `RWF_UNCACHED` is proposed to resolve this issue. But this patch set hasn't been accepted. See [LWN: Buffered I/O without page-cache thrashing](https://lwn.net/Articles/806980/).

## Memory-Mapped I/O

Operations can be done in a zero-copy manner.

Cache pollution can occur if `madvice()` is not used properly. The default behaviour, which performs read-ahead, may also cause issues. See [a performance benchmark](https://smalldatum.blogspot.com/2022/05/using-mmap-with-rocksdb.html) conducted by Mark Callaghan, the author of RocksDB.

The invocation to `madvice()` can be a considerable overhead by itself.

A write to a page can cause an implicit read to fill it.

A page fault can block a thread. `mincore()` can be used to determine the presence of pages. But it still involves some overhead.

Each time, only one page gets page faults. The use of `madvice()` is necessary to submit multiple concurrent read requests.

Although Linux does not currently support huge pages as mapped pages of files, a 4KiB page can still be larger than the minimum I/O unit of the underlying hardware.

The performance of the Translate Lookaside Buffer can significantly impact the efficiency of memory-mapped I/O. When dealing with a large mapped file, the likelihood of encountering a cache miss with the TLB increases.

## POSIX AIO

Too many syscalls to setup one I/O request and get its result.

The current implementation in Linux is provided in user space by glibc.

## io_uring

The availability seems limited for now.

`O_DIRECT` is still available.

`madvice()` can also be submitted with `io_uring`.