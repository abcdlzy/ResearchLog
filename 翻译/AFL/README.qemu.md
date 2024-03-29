High-performance binary-only instrumentation for afl-fuzz
=========================================================

  (See ../docs/README for the general instruction manual.)

1) Introduction
---------------

The code in this directory allows you to build a standalone feature that
leverages the QEMU "user emulation" mode and allows callers to obtain
instrumentation output for black-box, closed-source binaries. This mechanism
can be then used by afl-fuzz to stress-test targets that couldn't be built
with afl-gcc.

The usual performance cost is 2-5x, which is considerably better than
seen so far in experiments with tools such as DynamoRIO and PIN.

The idea and much of the implementation comes from Andrew Griffiths.

2) How to use
-------------

The feature is implemented with a fairly simple patch to QEMU 2.10.0. The
simplest way to build it is to run ./build_qemu_support.sh. The script will
download, configure, and compile the QEMU binary for you.

QEMU is a big project, so this will take a while, and you may have to
resolve a couple of dependencies (most notably, you will definitely need
libtool and glib2-devel).

Once the binaries are compiled, you can leverage the QEMU tool by calling
afl-fuzz and all the related utilities with -Q in the command line.

Note that QEMU requires a generous memory limit to run; somewhere around
200 MB is a good starting point, but considerably more may be needed for
more complex programs. The default -m limit will be automatically bumped up
to 200 MB when specifying -Q to afl-fuzz; be careful when overriding this.

In principle, if you set CPU_TARGET before calling ./build_qemu_support.sh,
you should get a build capable of running non-native binaries (say, you
can try CPU_TARGET=arm). This is also necessary for running 32-bit binaries
on a 64-bit system (CPU_TARGET=i386).

Note: if you want the QEMU helper to be installed on your system for all
users, you need to build it before issuing 'make install' in the parent
directory.

3) Notes on linking
-------------------

The feature is supported only on Linux. Supporting BSD may amount to porting
the changes made to linux-user/elfload.c and applying them to
bsd-user/elfload.c, but I have not looked into this yet.

The instrumentation follows only the .text section of the first ELF binary
encountered in the linking process. It does not trace shared libraries. In
practice, this means two things:

  - Any libraries you want to analyze *must* be linked statically into the
    executed ELF file (this will usually be the case for closed-source
    apps).

  - Standard C libraries and other stuff that is wasteful to instrument
    should be linked dynamically - otherwise, AFL will have no way to avoid
    peeking into them.

Setting AFL_INST_LIBS=1 can be used to circumvent the .text detection logic
and instrument every basic block encountered.

4) Benchmarking
---------------

If you want to compare the performance of the QEMU instrumentation with that of
afl-gcc compiled code against the same target, you need to build the
non-instrumented binary with the same optimization flags that are normally
injected by afl-gcc, and make sure that the bits to be tested are statically
linked into the binary. A common way to do this would be:

$ CFLAGS="-O3 -funroll-loops" ./configure --disable-shared
$ make clean all

Comparative measurements of execution speed or instrumentation coverage will be
fairly meaningless if the optimization levels or instrumentation scopes don't
match.

5) Gotchas, feedback, bugs
--------------------------

If you need to fix up checksums or do other cleanup on mutated test cases, see
experimental/post_library/ for a viable solution.

Do not mix QEMU mode with ASAN, MSAN, or the likes; QEMU doesn't appreciate
the "shadow VM" trick employed by the sanitizers and will probably just
run out of memory.

Compared to fully-fledged virtualization, the user emulation mode is *NOT* a
security boundary. The binaries can freely interact with the host OS. If you
somehow need to fuzz an untrusted binary, put everything in a sandbox first.

QEMU does not necessarily support all CPU or hardware features that your
target program may be utilizing. In particular, it does not appear to have
full support for AVX2 / FMA3. Using binaries for older CPUs, or recompiling them
with -march=core2, can help.

Beyond that, this is an early-stage mechanism, so fields reports are welcome.
You can send them to <afl-users@googlegroups.com>.

6) Alternatives: static rewriting
---------------------------------

Statically rewriting binaries just once, instead of attempting to translate
them at run time, can be a faster alternative. That said, static rewriting is
fraught with peril, because it depends on being able to properly and fully model
program control flow without actually executing each and every code path.

If you want to experiment with this mode of operation, there is a module
contributed by Aleksandar Nikolich:

  https://github.com/vrtadmin/moflow/tree/master/afl-dyninst
  https://groups.google.com/forum/#!topic/afl-users/HlSQdbOTlpg

At this point, the author reports the possibility of hiccups with stripped
binaries. That said, if we can get it to be comparably reliable to QEMU, we may
decide to switch to this mode, but I had no time to play with it yet.