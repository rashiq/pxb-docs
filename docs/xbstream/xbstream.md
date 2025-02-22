# The xbstream binary

To support simultaneous compression and streaming, a new custom streaming
format called xbstream was introduced to *Percona XtraBackup* in addition to
the TAR format. That was required to overcome some limitations of traditional
archive formats such as tar, cpio and others which did not allow streaming
dynamically generated files, for example dynamically compressed files. Other
advantages of xbstream over traditional streaming/archive format include
ability to stream multiple files concurrently (so it is possible to use
streaming in the xbstream format together with the –parallel option) and more
compact data storage.

This utility has a tar-like interface:

* with the `-x` option it extracts files from the stream read from its
standard input to the current directory unless specified otherwise with the
`-c` option. Support for parallel extraction with the `--parallel`
option has been implemented in *Percona XtraBackup* 2.4.7.

* with the `-c` option it streams files specified on the command line to its
standard output.

* with the `--decrypt=ALGO` option specified xbstream will automatically
decrypt encrypted files when extracting input stream. Supported values for
this option are: `AES128`, `AES192`, and `AES256`. Either
`--encrypt-key` or `--encrypt-key-file` options must be specified to
provide encryption key, but not both. This option has been implemented in
*Percona XtraBackup* 2.4.7.

* with the `--encrypt-threads` option you can specify the number of threads
for parallel data encryption. The default value is `1`. This option has
been implemented in *Percona XtraBackup* 2.4.7.

* the `--encrypt-key` option is used to specify the encryption key that will
be used. It can’t be used with `--encrypt-key-file` option because they
are mutually exclusive. This option has been implemented in *Percona
XtraBackup* 2.4.7.

* the `--encrypt-key-file` option is used to specify the file that contains
the encryption key. It can’t be used with `--encrypt-key` option.
because they are mutually exclusive. This option has been implemented in
*Percona XtraBackup* 2.4.7.

The utility also tries to minimize its impact on the OS page cache by using the
appropriate `posix_fadvise()` calls when available.

When compression is enabled with *xtrabackup* all data is being compressed,
including the transaction log file and meta data files, using the specified
compression algorithm. Percona XtraBackup supports the following compression algorithms:

`quicklz` 

!!! note

    Starting with Percona XtraBackup 8.0.31-24 using qpress/QuickLZ to compress backups is deprecated and may be removed in future versions. We recommend using either `LZ4` or Zstandard (`ZSTD`) compression algorithms.

The resulting files have the qpress archive format. Every `\*.qp` file
produced by *xtrabackup* is essentially a one-file qpress archive and can be extracted and uncompressed by the `qpress file archiver`. This means that there is no need to decompress entire backup to restore a single table as with `tar.gz`. 

To decompress individual files, run *xbstream* with the
`--decompress` option. You may control the number of threads
used for decompressing by passing the `--decompress-threads` option.

Also, files can be decompressed using the **qpress** tool. Qpress supports multi-threaded decompression.

`lz4`

To compress files using the `lz4` compression algorithm, set `--compress` option to `lz4`:

```{.bash data-prompt="$"}
$ xtrabackup --backup --compress=lz4 --target-dir=/data/backup
```

`Zstandard (ZSTD)`

The Zstandard (ZSTD) compression algorithm is a [tech preview](../glossary.md#tech-preview) feature. Before using ZSTD in production, we recommend that you test restoring production from physical backups in your environment, and also use the alternative backup method for redundancy.

[Percona XtraBackup 8.0.30-23](../release-notes/8.0/8.0.30-23.0.md) adds support for the `Zstandard (ZSTD)` compression algorithm. `ZSTD` is a fast lossless compression algorithm that targets real-time compression scenarios and better compression ratios. 
    
To compress files using the `ZSTD` compression algorithm, use the `--compress=zstd` option. The resulting files have the `\*.zst` format. 
    
You can specify `ZSTD` compression level with the [`--compress-zstd-level(=#)`](../xtrabackup_bin/xbk_option_reference.md#compress-zstd-level) option. The defaul value is `1`.

```{.bash data-prompt="$"}
$ xtrabackup --backup --compress-zstd-level=1 --target-dir=/data/backup
```
    
To decompress files, use the `--decompress` option. 

