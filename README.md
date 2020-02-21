# zld
## A faster version of Apple's linker

### Introduction

For large projects, the linking phase ([explanation](https://stackoverflow.com/questions/6264249/how-does-the-compilation-linking-process-work)) can significantly increase incremental build times. This project is a fork of the Apple linker, `ld`. It is a drop-in replacement that can substantially speed things up.

### Performance

In general, `zld` is at least 50% faster for link times < 1 second, and twice as fast for link times > 1 second. Feel free to file an issue if you find otherwise (make sure to run it twice in a row to ensure that [caches](#caches) have been generated).

### Stability

`zld` is forked from the most recently open-sourced version of `ld`. It produces byte-for-byte equivalent binaries (besides a few optimizations which you can disable with `-Wl,-zld-reproducible`). It also passes `ld`'s own suite of unit tests.  Although it's not ideal to mix compiler and linker versions (the forked `ld` is the one used in Xcode 10.2), this has been tested to work on Swift 5 projects, and there's no apparent reason why it shouldn't. The linker, after all, is fairly language-agnostic. `zld` will be updated with more recent versions of the linker as Apple open-sources them.

### Installation

`zld` can be installed with Homebrew:

```
brew install zld
```

### Usage

If using Xcode, add `-fuse-ld=zld` to "Other Linker Flags" in the build settings (debug configuration):

<img src="img/usage.png" width="75%">

### Caching

By default, `zld` stores some metadata in `/tmp/zld-...` to speed things up. This is the first step towards making `zld` a truly incremental linker. Currently, the only things that are stored are object file and library names. You can disable all caching with `-Wl,-zld-no_cache`.

### Why is it faster?

Apple's approach is a very reasonable one, using C++ with STL data structures. However, there are a number of ways in which `zld` has sped things up, for instance:

- Using [Swiss Tables](https://abseil.io/blog/20180927-swisstables) instead of STL for hash maps and sets
- Parallelizing in various places (the parsing of libraries, writing the output file, sorting, etc.)
- Optimizations around the hashing of strings (caching the hashes, using a better hash function, etc.)

### Other things to speed up linking

Whether you use this project or not, there are a number of things that can speed linking up (again, this is only for debug builds):

- The linker flag `-Wl,-no_uuid`, which disables UUID creation
- Turning off dead stripping (referred to as "Dead Code Stripping" in Xcode build settings)
- If you're not using `zld`, using `-Wl,-force_load` for libraries can sometimes speed things up
- Linking with dynamic libraries instead of static ones

### Contributing

The biggest way to contribute to `zld` is to file issues! If you encountered any problems, feel free to file an issue, and you can expect a prompt response. PRs, as well as issues to discuss potential new directions, are also welcome.

Special thanks to @dmaclach's [ld64](https://github.com/dmaclach/ld64), which helped with building `ld`.
