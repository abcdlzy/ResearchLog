american fuzzy lop
==================

  Written and maintained by Michal Zalewski <lcamtuf@google.com>

  Copyright 2013, 2014, 2015, 2016 Google Inc. All rights reserved.
  Released under terms and conditions of Apache License, Version 2.0.

  For new versions and additional information, check out:
  http://lcamtuf.coredump.cx/afl/

  To compare notes with other users or get notified about major new features,
  send a mail to <afl-users+subscribe@googlegroups.com>.

  ** See QuickStartGuide.txt if you don't have time to read this file. **

如果没空阅读这个文件，可以查看QuickStartGuide。



1) Challenges of guided fuzzing
-------------------------------

引导式fuzzing的挑战

Fuzzing is one of the most powerful and proven strategies for identifying
security issues in real-world software; it is responsible for the vast
majority of remote code execution and privilege escalation bugs found to date
in security-critical software.

Fuzzing是识别真实软件安全问题的一个强大且行之有效的技术，它可以找到远程代码执行（RCE）和权限提升等相关的软件安全致命的漏洞。



Unfortunately, fuzzing is also relatively shallow; blind, random mutations
make it very unlikely to reach certain code paths in the tested code, leaving
some vulnerabilities firmly outside the reach of this technique.

但是，fuzzing技术由于对代码理解有限、输入随机突变等问题，对于过深的代码路径的漏洞检测存在着巨大的困难。

There have been numerous attempts to solve this problem. One of the early
approaches - pioneered by Tavis Ormandy - is corpus distillation. The method
relies on coverage signals to select a subset of interesting seeds from a
massive, high-quality corpus of candidate files, and then fuzz them by
traditional means. The approach works exceptionally well, but requires such
a corpus to be readily available. In addition, block coverage measurements
provide only a very simplistic understanding of program state, and are less
useful for guiding the fuzzing effort in the long haul.

有很多的人尝试解决这个问题。早期的方法是由Tavis Ormandy提出的语料库提炼法。这个方法依靠覆盖的信号从大规模、高质量的候选文件中选择合适的随机种子，然后再结合传统的方法进行模糊测试。这个方法非常好，但是需要一个好用的语料库。此外，对于块的覆盖率测量，仅仅是一个简单的对程序当前状态的测量，缺少了对后续对fuzzing的引导。

Other, more sophisticated research has focused on techniques such as program
flow analysis ("concolic execution"), symbolic execution, or static analysis.
All these methods are extremely promising in experimental settings, but tend
to suffer from reliability and performance problems in practical uses - and
currently do not offer a viable alternative to "dumb" fuzzing techniques.

同时，其他的一些复杂的技术研究主要在：程序流分析 ("concolic execution")、符号执行或静态分析。这些方法都是不错的，但是通常会遇到可靠性及性能上的一些问题，因此目前没有一个可行的替代dumb模糊测试的技术。



2) The afl-fuzz approach
------------------------

afl-fuzz的一些说明

American Fuzzy Lop is a brute-force fuzzer coupled with an exceedingly simple
but rock-solid instrumentation-guided genetic algorithm. It uses a modified
form of edge coverage to effortlessly pick up subtle, local-scale changes to
program control flow.

AFL是一个暴力的模糊测试器，它使用了极其简单但是稳定的引导式插桩遗传算法。？它使用经过修改的边缘覆盖模式去便捷的覆盖各种规模的程序控制流。？

Simplifying a bit, the overall algorithm can be summed up as:

  1) Load user-supplied initial test cases into the queue,

  1）加载用户提供的初始测试用例进入队列

  2) Take next input file from the queue,

  2）从队列中获取下一个输入的文件

  3) Attempt to trim the test case to the smallest size that doesn't alter
     the measured behavior of the program,

  3）尝试将测试用例裁剪成不改变对程序测量行为的最小的大小。

  4) Repeatedly mutate the file using a balanced and well-researched variety
     of traditional fuzzing strategies,

  4）使用多种平衡且成熟的传统模糊测试策略重复突变文件

  5) If any of the generated mutations resulted in a new state transition
     recorded by the instrumentation, add mutated output as a new entry in the
     queue.

  5）如果任何生成的突变用例到达了之前没进入到的代码覆盖区域，则把这个突变用例加入到队列中，以便后续使用这个路径进一步进行测试。

  6) Go to 2.

  6）跳回第2步

The discovered test cases are also periodically culled to eliminate ones that
have been obsoleted by newer, higher-coverage finds; and undergo several other
instrumentation-driven effort minimization steps.

对已经发现的测试用例，也会定期的去清除已被新的用例替代或低覆盖率的测试用例。？随后经过其他插桩驱动去最小化这些用例。？

As a side result of the fuzzing process, the tool creates a small,
self-contained corpus of interesting test cases. These are extremely useful
for seeding other, labor- or resource-intensive testing regimes - for example,
for stress-testing browsers, office applications, graphics suites, or
closed-source tools.

作为模糊测试程序的附带结果，这个工具会生成一些小型且自包含的有趣测试用例。这些用例对于设定其他劳动或资源密集型的测试系统是非常有用的，这些系统例如：压力测试浏览器、办公室应用程序、图形套件或闭源工具。

The fuzzer is thoroughly tested to deliver out-of-the-box performance far
superior to blind fuzzing or coverage-only tools.

？这个模糊测试器的开箱即用的性能优于其他盲模糊测试和仅覆盖的模糊测试工具。？



3) Instrumenting programs for use with AFL
------------------------------------------

使用AFL对程序进行插桩

When source code is available, instrumentation can be injected by a companion
tool that works as a drop-in replacement for gcc or clang in any standard build
process for third-party code.

当拥有源代码的时候，可以在gcc或者clang对程序进行编译过程中进行中，替换中间过程对程序进行插桩。

The instrumentation has a fairly modest performance impact; in conjunction with
other optimizations implemented by afl-fuzz, most programs can be fuzzed as fast
or even faster than possible with traditional tools.

插桩对性能的影响很小，结合由afl-fuzz实现优化的大多数程序比传统工具进行的模糊测试速度更快。

The correct way to recompile the target program may vary depending on the
specifics of the build process, but a nearly-universal approach would be:

对于需要模糊测试程序的编译，需要通过一些特定设置：

```bash
$ CC=/path/to/afl/afl-gcc ./configure
$ make clean all
```

For C++ programs, you'd would also want to set 

对于C++程序，可能需要设置：

```bash
CXX=/path/to/afl/afl-g++.
```

The clang wrappers (afl-clang and afl-clang++) can be used in the same way;
clang users may also opt to leverage a higher-performance instrumentation mode,
as described in llvm_mode/README.llvm.

clang的包装器（afl-clang和afl-clang++）可以使用相同的方法，clang用户也可以选择其他更高性能的插桩模式，详细见 [llvm_mode/README.llvm](README.llvm.md)。

When testing libraries, you need to find or write a simple program that reads
data from stdin or from a file and passes it to the tested library. In such a
case, it is essential to link this executable against a static version of the
instrumented library, or to make sure that the correct .so file is loaded at
runtime (usually by setting LD_LIBRARY_PATH). The simplest option is a static
build, usually possible via:

当测试静态链接库时，需要写一个简单的程序去处理与被测试的链接库输入相关的数据。在这里需要确保运行时程序可以加载正确的.so文件（这里通常通过LD_LIBRARY_PATH进行设置）。也可通过下面的简单选项来进行静态生成：

```bash
$ CC=/path/to/afl/afl-gcc ./configure --disable-shared
```

Setting AFL_HARDEN=1 when calling 'make' will cause the CC wrapper to
automatically enable code hardening options that make it easier to detect
simple memory bugs. Libdislocator, a helper library included with AFL (see
libdislocator/README.dislocator) can help uncover heap corruption issues, too.

在调用"make"时设置 AFL_HARDEN_1 将导致 CC 包装器自动启用代码强化选项,从而更轻松地检测简单的内存错误。Libdislocator 是 AFL 附带的帮助器库，可以帮助发现堆损坏问题。(请参阅[ libdislocator/README.dislocator ](README.dislocator.md))

PS. ASAN users are advised to review notes_for_asan.txt file for important
caveats.

ASAN用户需要去查看[notes_for_asan.txt](notes_for_asan.md)的重要警告。



4) Instrumenting binary-only apps
---------------------------------

黑盒模糊测试插桩

When source code is *NOT* available, the fuzzer offers experimental support for
fast, on-the-fly instrumentation of black-box binaries. This is accomplished
with a version of QEMU running in the lesser-known "user space emulation" mode.

如果没有源码，模糊测试器也可以进行对黑盒二进制文件的快速、动态的检测。这是通过QEMU的“用户空间仿真”的模式来实现。

QEMU is a project separate from AFL, but you can conveniently build the
feature by doing:

QEMU与AFL均是一个独立的项目，但是你可以通过下面的命令使它们结合起来：

```bash
$ cd qemu_mode
$ ./build_qemu_support.sh
```

For additional instructions and caveats, see qemu_mode/README.qemu.

关于一些用法说明和警告信息,请查阅 [qemu_mode/README.qemu](README.qemu.md) 。

The mode is approximately 2-5x slower than compile-time instrumentation, is
less conductive to parallelization, and may have some other quirks.

以这个模式运行的性能会比编译时插桩的模式慢大约2~5倍，同时对于并行化模糊测试可能会出现一些奇怪的问题。



5) Choosing initial test cases
------------------------------

选择初始测试用例

To operate correctly, the fuzzer requires one or more starting file that
contains a good example of the input data normally expected by the targeted
application. There are two basic rules:

为了保证操作有效，模糊测试器需要一个或多个起始的文件，这些起始文件包含被测程序所期望的输入数据的样本。以下是两个基本原则：

  - Keep the files small. Under 1 kB is ideal, although not strictly necessary.
    For a discussion of why size matters, see perf_tips.txt.

    保证文件相对小，虽然不是强制要求这么小，但是1kB以下这种大小是极好的。相关的讨论请查阅：[perf_tips.txt](perf_tips.md)
    
  - Use multiple test cases only if they are functionally different from
    each other. There is no point in using fifty different vacation photos
    to fuzz an image library.
    
    使用多种测试用例尽可能覆盖不同的代码路径。这里需要说明的是，使用50种不同的度假照片去模糊测试一个图像处理库是没有意义的，这是因为代码路径只覆盖了一种。

You can find many good examples of starting files in the testcases/ subdirectory
that comes with this tool.

可以在testcase文件夹中找到一些例子。

PS. If a large corpus of data is available for screening, you may want to use
the afl-cmin utility to identify a subset of functionally distinct files that
exercise different code paths in the target binary.

此外，如果由大量的数据可供使用筛选，你可以使用afl-cmin实用程序去识别不同代码路径的用例，标记并使用。



6) Fuzzing binaries
-------------------

对二进制程序进行模糊测试

The fuzzing process itself is carried out by the afl-fuzz utility. This program
requires a read-only directory with initial test cases, a separate place to
store its findings, plus a path to the binary to test.

模糊测试由afl-fuzz实用程序执行。这个程序需要带有初始测试用例的只读目录、一个可以存储测试结果的目录及需要测试的二进制文件。

For target binaries that accept input directly from stdin, the usual syntax is:

如果目标程序接受stdin的直接输入，可以使用以下命令：

```bash
$ ./afl-fuzz -i testcase_dir -o findings_dir /path/to/program [...params...]
```

For programs that take input from a file, use '@@' to mark the location in
the target's command line where the input file name should be placed. The
fuzzer will substitute this for you:

如果被测的程序可以读取来源于文件的数据，可以实用@@标记放置到命令行中，这样模糊测试器在测试过程中将会把@@替换成相应的输入文件。

```bash
$ ./afl-fuzz -i testcase_dir -o findings_dir /path/to/program @@
```

You can also use the -f option to have the mutated data written to a specific
file. This is useful if the program expects a particular file extension or so.

你还可以使用 -f 选项将突变的数据写入特定文件。这对某些程序需要特定的文件扩展名特别有用。

Non-instrumented binaries can be fuzzed in the QEMU mode (add -Q in the command
line) or in a traditional, blind-fuzzer mode (specify -n).

没有插桩的程序可以通过QEMU模式进行模糊测试（添加 -Q 参数到命令行即可）或者也可以实用盲目模糊测试模式（通过 -n 参数）。

You can use -t and -m to override the default timeout and memory limit for the
executed process; rare examples of targets that may need these settings touched
include compilers and video decoders.

可以使用 -t 参数去设置超时时间，也可以使用 -m 参数对被模糊测试程序进行内存使用的限制，这通常用于编译器和视频解码器的模糊测试上。

Tips for optimizing fuzzing performance are discussed in perf_tips.txt.

关于优化性能的相关讨论请查阅：[perf_tips.txt.](perf_tips.md)

Note that afl-fuzz starts by performing an array of deterministic fuzzing
steps, which can take several days, but tend to produce neat test cases. If you
want quick & dirty results right away - akin to zzuf and other traditional
fuzzers - add the -d option to the command line.

通常afl-fuzz需要执行一系列优化性能的操作，这些操作可能会耗时几天，但这利于生成较好的测试用例。如果你想对被测程序进行快速的且混乱的测试，可以通过 -d 参数去开启类似于传统模糊测试器的模式。



7) Interpreting output
----------------------

输出信息的说明

See the status_screen.txt file for information on how to interpret the
displayed stats and monitor the health of the process. Be sure to consult this
file especially if any UI elements are highlighted in red.

查看 [status_screen.txt](status_screen.md) 文件了解更多关于统计信息和和程序运行状态信息的说明。如果UI出现高亮的红色提示的时候，请尽量先去查看此文件。

The fuzzing process will continue until you press Ctrl-C. At minimum, you want
to allow the fuzzer to complete one queue cycle, which may take anywhere from a
couple of hours to a week or so.

模糊测试程序会进行到直到你按Ctrl+C为止。一般情况下，模糊测试器完成一个队列周期至少需要几小时或者几周以上。

There are three subdirectories created within the output directory and updated
in real time:

这里有3个文件夹会被创建在输出文件夹中（ -o 的设置），同时这些文件夹的内容会被实时更新：

  - queue/   - test cases for every distinctive execution path, plus all the starting files given by the user. This is the synthesized corpus mentioned in section 2. Before using this corpus for any other purposes, you can shrink it to a smaller size using the afl-cmin tool. The tool will find a smaller subset of files offering equivalent  edge coverage.
            queue/：用户提供的所有初始文件及不同代码路径覆盖的测试用例。这是第2节提到的合成语料库，如果要把这个库另作他用时，可通过afl-cmin去优化边缘覆盖率，且可缩减大小。
          
  - crashes/ - unique test cases that cause the tested program to receive a fatal signal (e.g., SIGSEGV, SIGILL, SIGABRT). The entries are grouped by the received signal.

       crashes/：存放接收到致命信号的测试用例（例如：SIGSEGV、SIGILL、SIGABRT）。这些条目会按接收到的信号类型进行分组。

  - hangs/   - unique test cases that cause the tested program to time out. The default time limit before something is classified as a hang is the larger of 1 second and the value of the -t parameter. The value can be fine-tuned by setting AFL_HANG_TMOUT, but this is rarely necessary.

       hangs/：导致测试程序超时的测试用例。超时为默认时间为1秒或者是参数 -t 设置的数值。可以通过设置 AFL_HANG_TMOUT 来微调该值，但很少需要这样做。

Crashes and hangs are considered "unique" if the associated execution paths involve any state transitions not seen in previously-recorded faults. If a single bug can be reached in multiple ways, there will be some count inflation early in the process, but this should quickly taper off.

对于代码执行路径和之前的记录中没有状态变化的崩溃和挂起的信息将被合并。如果某个bug可以通过多种方式发现，则流程早期可能会出现计数快速上涨，但这通常都会很快的消失。

The file names for crashes and hangs are correlated with parent, non-faulting
queue entries. This should help with debugging.

崩溃和挂起的文件名与父级非故障队列条目文件名相关，这应该有助于调试。

When you can't reproduce a crash found by afl-fuzz, the most likely cause is
that you are not setting the same memory limit as used by the tool. Try:

当无法重现由 afl-fuzz 找到的崩溃时,最可能的原因是您没有设置与该工具相同的内存限制。尝试：

```bash
$ LIMIT_MB=50
$ ( ulimit -Sv $[LIMIT_MB << 10]; /path/to/tested_binary ... )
```

Change LIMIT_MB to match the -m parameter passed to afl-fuzz. On OpenBSD,
also change -Sv to -Sd.

更改 LIMIT_MB 以匹配传递给 afl-fuzz 的 -m 参数。在 OpenBSD 上，也更改 -Sv 和 -Sd。

Any existing output directory can be also used to resume aborted jobs; try:

可以现有的输出目录用于恢复中止的作业，尝试：

```bash
$ ./afl-fuzz -i- -o existing_output_dir [...etc...]
```

If you have gnuplot installed, you can also generate some pretty graphs for any
active fuzzing task using afl-plot. For an example of how this looks like,
see http://lcamtuf.coredump.cx/afl/plot/.

如果你安装了gnuplot，可以使用afl-plot来生成可视化的图。使用例子参考：http://lcamtuf.coredump.cx/afl/plot/.



8) Parallelized fuzzing
-----------------------

并行模糊测试

Every instance of afl-fuzz takes up roughly one core. This means that on
multi-core systems, parallelization is necessary to fully utilize the hardware.
For tips on how to fuzz a common target on multiple cores or multiple networked
machines, please refer to parallel_fuzzing.txt.

每个afl-fuzz实例占用了一个核，如果想充分利用硬件资源，开启并行模糊测试是必不可少的。关于多核和多机器联网进行模糊测试，可以参考：[parallel_fuzzing.txt](parallel_fuzzing.md)

The parallel fuzzing mode also offers a simple way for interfacing AFL to other
fuzzers, to symbolic or concolic execution engines, and so forth; again, see the
last section of parallel_fuzzing.txt for tips.

并行模糊测试模式还提供了一种简单的方法，它用于将 AFL 连接到其他模糊测试器、连接到符号或共发执行引擎等。再次,请参阅 [parallel_fuzzing.txt](parallel_fuzzing.md) 的最后一部分了解相关说明。



9) Fuzzer dictionaries
----------------------



By default, afl-fuzz mutation engine is optimized for compact data formats -
say, images, multimedia, compressed data, regular expression syntax, or shell
scripts. It is somewhat less suited for languages with particularly verbose and
redundant verbiage - notably including HTML, SQL, or JavaScript.

默认情况下，afl-fuzz突变引擎针对紧凑的数据格式进行优化，例如：图像、多媒体、压缩数据、正则表达式语法或者是shell脚本。它不太适合具有特别冗长和冗余性质的语言，例如：HTML、SQL和JavaScript。

To avoid the hassle of building syntax-aware tools, afl-fuzz provides a way to
seed the fuzzing process with an optional dictionary of language keywords,
magic headers, or other special tokens associated with the targeted data type



- and use that to reconstruct the underlying grammar on the go:

  http://lcamtuf.blogspot.com/2015/01/afl-fuzz-making-up-grammar-with.html
  
  

To use this feature, you first need to create a dictionary in one of the two
formats discussed in dictionaries/README.dictionaries; and then point the fuzzer
to it via the -x option in the command line.



(Several common dictionaries are already provided in that subdirectory, too.)



There is no way to provide more structured descriptions of the underlying
syntax, but the fuzzer will likely figure out some of this based on the
instrumentation feedback alone. This actually works in practice, say:

  http://lcamtuf.blogspot.com/2015/04/finding-bugs-in-sqlite-easy-way.html



PS. Even when no explicit dictionary is given, afl-fuzz will try to extract
existing syntax tokens in the input corpus by watching the instrumentation
very closely during deterministic byte flips. This works for some types of
parsers and grammars, but isn't nearly as good as the -x mode.



If a dictionary is really hard to come by, another option is to let AFL run
for a while, and then use the token capture library that comes as a companion
utility with AFL. For that, see libtokencap/README.tokencap.



10) Crash triage
----------------



The coverage-based grouping of crashes usually produces a small data set that
can be quickly triaged manually or with a very simple GDB or Valgrind script.
Every crash is also traceable to its parent non-crashing test case in the
queue, making it easier to diagnose faults.



Having said that, it's important to acknowledge that some fuzzing crashes can be
difficult to quickly evaluate for exploitability without a lot of debugging and
code analysis work. To assist with this task, afl-fuzz supports a very unique
"crash exploration" mode enabled with the -C flag.



In this mode, the fuzzer takes one or more crashing test cases as the input,
and uses its feedback-driven fuzzing strategies to very quickly enumerate all
code paths that can be reached in the program while keeping it in the
crashing state.



Mutations that do not result in a crash are rejected; so are any changes that
do not affect the execution path.



The output is a small corpus of files that can be very rapidly examined to see
what degree of control the attacker has over the faulting address, or whether
it is possible to get past an initial out-of-bounds read - and see what lies
beneath.



Oh, one more thing: for test case minimization, give afl-tmin a try. The tool
can be operated in a very simple way:



```bash
$ ./afl-tmin -i test_case -o minimized_result -- /path/to/program [...]
```

The tool works with crashing and non-crashing test cases alike. In the crash
mode, it will happily accept instrumented and non-instrumented binaries. In the
non-crashing mode, the minimizer relies on standard AFL instrumentation to make
the file simpler without altering the execution path.



The minimizer accepts the -m, -t, -f and @@ syntax in a manner compatible with
afl-fuzz.



Another recent addition to AFL is the afl-analyze tool. It takes an input
file, attempts to sequentially flip bytes, and observes the behavior of the
tested program. It then color-codes the input based on which sections appear to
be critical, and which are not; while not bulletproof, it can often offer quick
insights into complex file formats. More info about its operation can be found
near the end of technical_details.txt.



11) Going beyond crashes
------------------------



Fuzzing is a wonderful and underutilized technique for discovering non-crashing
design and implementation errors, too. Quite a few interesting bugs have been
found by modifying the target programs to call abort() when, say:



  - Two bignum libraries produce different outputs when given the same
    fuzzer-generated input,



  - An image library produces different outputs when asked to decode the same
    input image several times in a row,



  - A serialization / deserialization library fails to produce stable outputs
    when iteratively serializing and deserializing fuzzer-supplied data,



  - A compression library produces an output inconsistent with the input file
    when asked to compress and then decompress a particular blob.



Implementing these or similar sanity checks usually takes very little time;
if you are the maintainer of a particular package, you can make this code
conditional with #ifdef FUZZING_BUILD_MODE_UNSAFE_FOR_PRODUCTION (a flag also
shared with libfuzzer) or #ifdef __AFL_COMPILER (this one is just for AFL).



12) Common-sense risks
----------------------



Please keep in mind that, similarly to many other computationally-intensive
tasks, fuzzing may put strain on your hardware and on the OS. In particular:



  - Your CPU will run hot and will need adequate cooling. In most cases, if
    cooling is insufficient or stops working properly, CPU speeds will be
    automatically throttled. That said, especially when fuzzing on less
    suitable hardware (laptops, smartphones, etc), it's not entirely impossible
    for something to blow up.



  - Targeted programs may end up erratically grabbing gigabytes of memory or
    filling up disk space with junk files. AFL tries to enforce basic memory
    limits, but can't prevent each and every possible mishap. The bottom line
    is that you shouldn't be fuzzing on systems where the prospect of data loss
    is not an acceptable risk.



  - Fuzzing involves billions of reads and writes to the filesystem. On modern
    systems, this will be usually heavily cached, resulting in fairly modest
    "physical" I/O - but there are many factors that may alter this equation.
    It is your responsibility to monitor for potential trouble; with very heavy
    I/O, the lifespan of many HDDs and SSDs may be reduced.

    A good way to monitor disk I/O on Linux is the 'iostat' command:

    
    
    ```bash
    $ iostat -d 3 -x -k [...optional disk ID...]
    ```
    



13) Known limitations & areas for improvement
---------------------------------------------



Here are some of the most important caveats for AFL:



  - AFL detects faults by checking for the first spawned process dying due to
    a signal (SIGSEGV, SIGABRT, etc). Programs that install custom handlers for
    these signals may need to have the relevant code commented out. In the same
    vein, faults in child processed spawned by the fuzzed target may evade
    detection unless you manually add some code to catch that.

    
    
  - As with any other brute-force tool, the fuzzer offers limited coverage if
    encryption, checksums, cryptographic signatures, or compression are used to
    wholly wrap the actual data format to be tested. To work around this, you can comment out the relevant checks (see experimental/libpng_no_checksum/ for inspiration); if this is not possible, you can also write a postprocessor, as explained in experimental/post_library/.

    
    
  - There are some unfortunate trade-offs with ASAN and 64-bit binaries. This
    isn't due to any specific fault of afl-fuzz; see notes_for_asan.txt for
    tips.

    
    
  - There is no direct support for fuzzing network services, background
    daemons, or interactive apps that require UI interaction to work. You may
    need to make simple code changes to make them behave in a more traditional
    way. Preeny may offer a relatively simple option, too - see: https://github.com/zardus/preeny Some useful tips for modifying network-based services can be also found at: https://www.fastly.com/blog/how-to-fuzz-server-american-fuzzy-lop
    

    
  - AFL doesn't output human-readable coverage data. If you want to monitor
    coverage, use afl-cov from Michael Rash: https://github.com/mrash/afl-cov

    
    
  - Occasionally, sentient machines rise against their creators. If this
    happens to you, please consult http://lcamtuf.coredump.cx/prep/.
    
    

Beyond this, see INSTALL for platform-specific tips.



14) Special thanks
------------------

Many of the improvements to afl-fuzz wouldn't be possible without feedback,
bug reports, or patches from:

  Jann Horn                             Hanno Boeck
  Felix Groebert                        Jakub Wilk
  Richard W. M. Jones                   Alexander Cherepanov
  Tom Ritter                            Hovik Manucharyan
  Sebastian Roschke                     Eberhard Mattes
  Padraig Brady                         Ben Laurie
  @dronesec                             Luca Barbato
  Tobias Ospelt                         Thomas Jarosch
  Martin Carpenter                      Mudge Zatko
  Joe Zbiciak                           Ryan Govostes
  Michael Rash                          William Robinet
  Jonathan Gray                         Filipe Cabecinhas
  Nico Weber                            Jodie Cunningham
  Andrew Griffiths                      Parker Thompson
  Jonathan Neuschfer                    Tyler Nighswander
  Ben Nagy                              Samir Aguiar
  Aidan Thornton                        Aleksandar Nikolich
  Sam Hakim                             Laszlo Szekeres
  David A. Wheeler                      Turo Lamminen
  Andreas Stieger                       Richard Godbee
  Louis Dassy                           teor2345
  Alex Moneger                          Dmitry Vyukov
  Keegan McAllister                     Kostya Serebryany
  Richo Healey                          Martijn Bogaard
  rc0r                                  Jonathan Foote
  Christian Holler                      Dominique Pelle
  Jacek Wielemborek                     Leo Barnes
  Jeremy Barnes                         Jeff Trull
  Guillaume Endignoux                   ilovezfs
  Daniel Godas-Lopez                    Franjo Ivancic
  Austin Seipp                          Daniel Komaromy
  Daniel Binderman                      Jonathan Metzman
  Vegard Nossum                         Jan Kneschke
  Kurt Roeckx                           Marcel Bohme
  Van-Thuan Pham                        Abhik Roychoudhury
  Joshua J. Drake                       Toby Hutton
  Rene Freingruber                      Sergey Davidoff
  Sami Liedes                           Craig Young
  Andrzej Jackowski                     Daniel Hodson

Thank you!

15) Contact
-----------

Questions? Concerns? Bug reports? The author can be usually reached at
<lcamtuf@google.com>.

There is also a mailing list for the project; to join, send a mail to
<afl-users+subscribe@googlegroups.com>. Or, if you prefer to browse
archives first, try:

  https://groups.google.com/group/afl-users

PS. If you wish to submit raw code to be incorporated into the project, please
be aware that the copyright on most of AFL is claimed by Google. While you do
retain copyright on your contributions, they do ask people to agree to a simple
CLA first:

  https://cla.developers.google.com/clas

Sorry about the hassle. Of course, no CLA is required for feature requests or
bug reports.