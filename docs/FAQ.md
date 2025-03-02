# FAQ

## Table of Contents
* [About GEF's file or directory](#about-gefs-file-or-directory)
* [About the host environment](#about-the-host-environment)
* [About the guest (debugged) environment](#about-the-guest-debugged-environment)
* [About GEF settings](#about-gef-settings)
* [About commands](#about-commands)
* [About internal mechanism](#about-internal-mechanism)
* [About python interface](#about-python-interface)
* [About development schedule](#about-development-schedule)
* [About reporting, etc.](#about-reporting-etc)
* [Other memo (Japanese)](#other-memo-japanese)


# About GEF's file or directory

## Where is `gef.py`?
GEF (`gef.py`) is placed in `/root/.gdbinit-gef.py` by default. GEF is one file.

## How to change the location of GEF?
Move `/root/.gdbinit-gef.py` and edit `/root/.gdbinit`.

If you want to use GEF as a user other than root, add `source /path/to/.gdbinit-gef.py` to that user's `$HOME/.gdbinit`.

## What is `~/.gef.rc`?
This is the GEF config file. Not present by default.

Executing the `gef save` command saves the current settings to disk (`~/.gef.rc`).
The next time GEF starts, it will be automatically loaded and the settings will be reflected.
This includes the current values of items configurable with `gef config` and alias settings for commands.

## What is `/tmp/gef`?
This is the directory where GEF temporarily stores files.

Since it is used for caching, there is no problem in deleting it.
It will be created automatically the next time GEF starts.

## What is `install-minimal.sh`?
This is an installer for running GEF in limited environments where required packages cannot be installed for some reason.

* To use all feature (=gef's command), use `install.sh`.
* If you do not need some features (used in a limited environment), use `install-minimal.sh`. It should work at least except some commands.


# About the host environment

## Does GEF work properly on OS other than Ubuntu?
Yes, it probably works fine for regular Linux.

I have used it on debian. Also it seems to be working fine on WSL2 (ubuntu) so far.
Some users are running it on Arch Linux.
However, I have not confirmed that all commands work correctly.

## Will this GEF work as a plugin for `hugsy/gef`?
No, it doesn't work. It replaces `hugsy/gef`.

The compatibility with `hugsy/gef` has already been lost. Of course, `hugsy/gef-extras` too.
Think of it as a completely different product.

## GDB will not load GEF.
This is probably because gdb does not support cooperation with python3.

Consider building gdb from latest tarball or git.

* from latest tarball
    * Download latest tarball from https://ftp.gnu.org/gnu/gdb/
    ```
    tar xf gdb-14.2.tar.xz && cd gdb-14.2
    ./configure --enable-targets=all --with-python=/usr/bin/python3
    make && make install
    ```
* from git
    ```
    apt install -y libdebuginfod-dev libreadline-dev
    git clone --depth 1 https://github.com/bminor/binutils-gdb && cd binutils-gdb
    ./configure --disable-{binutils,ld,gold,gas,sim,gprof,gprofng} --enable-targets=all --with-python=/usr/bin/python3 --with-debuginfod --with-system-{zlib,readline}
    make && make install
    ```

## If I use `install-minimal.sh`, which commands will no longer be available?
Following are the breakdown. It may not be comprehensive.

If you install with `install-minimal.sh`, you will not be able to use these commands unless you install the required packages and tools.

|GEF command/feature|required apt package|required python3 package|required other tools|
|:---|:---|:---|:---|
|(`gef`)|`gdb` or `gdb-multiarch`|-|-|
|`got`|`binutils` (`objdump`, `readelf`)|-|-|
|`rp --kernel`|`binutils` (`nm`)|-|-|
|`qemu-device-info`|`binutils` (`nm`)|-|-|
|`add-symbol-temporary`|`binutils` (`objcopy`)|-|-|
|`ksymaddr-remote-apply`|`binutils` (`objcopy`)|-|-|
|`diffo git-diff`|`git`|-|-|
|`vmlinux-to-elf-apply`|`python3-pip`, `git`|`vmlinux-to-elf`|-|
|`uefi-ovmf-info`|`python3-pip`|`crccheck`|-|
|`hash-memory -v`|`python3-pip`|`crccheck`|-|
|`unicorn-emulate`|`python3-pip`|`unicorn`, `capstone`|-|
|`capstone-disassemble`|`python3-pip`|`capstone`|-|
|`dasm`|`python3-pip`|`capstone`|-|
|`asm-list`|`python3-pip`|`capstone`|-|
|`i8086` mode|`python3-pip`|`capstone`|-|
|`ropper`|`python3-pip`|`ropper`|-|
|`mprotect`|`python3-pip`|`keystone-engine`|-|
|`asm`|`python3-pip`|`keystone-engine`|-|
|(Progress Indicator)|`python3-pip`|`tqdm`|-|
|`onegadget`|`ruby-dev`|-|`one_gadget`|
|`seccomp-tools`|`ruby-dev`|-|`seccomp-tools`|
|`ktask -S`|`ruby-dev`|-|`seccomp-tools`|
|`rp`|-|-|`rp++`|

## How can I install GEF offline?
Please refer to [`install.sh`](https://github.com/bata24/gef/blob/dev/install.sh) or [`install-minimal.sh`](https://github.com/bata24/gef/blob/dev/install-minimal.sh), and set it up manually.

## When debugging with gdb, how can I display the source code of preinstalled libraries and commands?
Although it is limited to Ubuntu 22.10 or later, it is recommended to use `debuginfod`.

* Enable `debuginfod` (ubuntu 22.10~)
    * `export DEBUGINFOD_URLS="https://debuginfod.ubuntu.com"`
    * `echo "set debuginfod enabled on" >> ~/.gdbinit`

However, for some reason `debuginfod` does not display the `glibc` source code, so you need to obtain and place the source code separately.
I don't really understand the reason for this.

* Get `glibc` source
    * `sed -i -e "s/^# deb-src/deb-src/g" /etc/apt/sources.list`
        * This is invalid on ubuntu 24.04. See [here](https://askubuntu.com/questions/1512042/).
    * `cd /usr/lib/debug && apt update && apt source libc6`
    * `echo "directory /usr/lib/debug/glibc-2.38" >> ~/.gdbinit`
        * Need to fix version for your environment.

* Also add `glibc` symbols
    * `apt install libc6-dbg`
    * `echo "set debug-file-directory /usr/lib/debug" >> ~/.gdbinit`.


# About the guest (debugged) environment

## What Linux kernel versions does GEF support as guests in qemu-system?
I have confirmed that most commands work on versions 3.x ~ 6.10.x.

However, I have not verified every kernel version.
For example, certain symbols in some versions may not be supported by heuristic symbol detection.
Also, the structure may differ depending on the build config and the compiler that built the kernel.
So there may be environments where GEF does not work.
If you have any trouble, please report it on the issue page.

## Is there a way to get a pre-built kernel of each version?
I use [https://kernel.ubuntu.com/](https://kernel.ubuntu.com/mainline).

Download `linux-image-unsigned-*_amd64.deb` your preferred, and extract `/boot/vmlinuz-*`.
No filesystem image is provided. Please use the one created with `buildroot` or provided in past CTF challenges.

## Will each GEF command be more accurate if I have `vmlinux` with debug symbols?
No, whether `vmlinux` includes debug information has no effect on GEF behavior.

GEF always uses its own resolved address with `ksymaddr-remote`.
It also performs its own heuristic structure member detection in each command.

## Does GEF support i386 16-bit mode (real mode)?
Yes, GEF supports real mode experimentally.

Use `qemu-system-i386`, and do NOT use `qemu-system-x86_64`.
Explicitly specify the i8086 architecture before connecting: `gdb -ex 'set architecture i8086' -ex 'target remote localhost:1234'`.

GEF will switch to and from 32-bit mode automatically.

## Does GEF support to debug Android?
I have never tried it, so I don't know.

I think it will work for userland debugging.
However, Android does not use `glibc`, so the heap structure is different.
Therefore, I think at least `heap` related commands will not work.

Regarding kernel debugging, I haven't been able to confirm how much the structure is different.

## Does GEF support TEE OS other than OP-TEE?
No, GEF does not support it.

If there is publicly available test image, I consider developing to support that OS.

## Is it possible to debug userland with GEF when using qemu-system?
Partially yes.

I think it can be used when you want to track before and after a system call.
However, of course, I do not recommend continually debugging userland with qemu-system.
This is because many commands are restricted for various reasons.
Consider setting up `gdbserver` in the guest and connecting from the outside.

Note: If KPTI is enabled, many kernel-related commands cannot be used.
The reason is that most memory access to kernel space is unavailable if KPTI is enabled.

## How do I break in userland when using qemu-system?
Use a hardware breakpoint.

When you are stopped inside the kernel, is it in the intended process context? If so, just use `break *ADDRESS` as usual.
But if you're stuck in the kernel context of a different user process than you expected, or in a kernel thread like `swapper/0`,
the virtual address of the process you wanted isn't mapped.
For this reason, software breakpoints that embed `0xcc` in virtual memory cannot be used in some situations.
However, hardware breakpoints can be used without any problems.


# About GEF settings

## I prefer the AT&T style.
Please specify each time using the `set disassembly-flavor att` command.

Or, since the `set disassembly-flavor intel` command is executed in the main function of GEF, it may be a good idea to comment it out.
However, since GEF does not take AT&T syntax parsing into consideration, so some commands may do not work fine.
If you find a case where it doesn't work, please report it on the issue page.

## I don't like the color scheme.
Customize it using the `theme` command, then `gef save`. The config is saved to `~/.gef.rc`.

There is another option is disable colors. Try `gef config gef.disable_color True`.

## I don't want to add `-n` to every command to disable pager.
Try `gef config gef.always_no_pager True` then `gef save`.


# About commands

## What command should I start with when debugging the kernel?
Try `pagewalk` , `ks-apply` and `kchecksec`.
After that, try `slub-dump`, `ktask` and `ksysctl` as well.

Other commands are less important, so check them with `gef help` if necessary.

## Is GEF possible to re-display the results of a command (for using less-pager)?
Basically you can't.

Please save as appropriate with `|$cat > /tmp/foo.txt` while the less-pager is running.
Or try `gef config gef.keep_pager_result True` then `gef save`. From next time onwards, temporary files will no longer be deleted.

## Is GEF possible to pass the result of a command to a shell command?
Yes, you can use built-in `pipe` command.

For example, `pipe elf-info -n |grep .data` or `|pdisas |grep call`.

## `ktask` (or other kernel related commands) does not work.
The kernel you are debugging may have been built with `CONFIG_RANDSTRUCT=y`.

In this case, except for a few commands, they will not work correctly.
Currently, at least following commands do not work.
* `ktask`
* `kmod`
* `kbdev`
* `kcdev`
* `kops`
* `kpipe`
* `ksysctl`
* `kmalloc-tracer`
* `kmalloc-allocated-by`
* `kfiles`
* `kregs`
* `ksighands`
* `kpcidev`
* `knamespaces`
* `kipcs`
* `kfilesystems`

## `vmmap` command does not recognize option.
Try `pagewalk` command.

When connected to qemu-system or vmware's gdb stub, the `vmmap` command is just redirected to the `pagewalk` command.
All options are ignored at this time. If you want to use some options, please use the `pagewalk` command instead of `vmmap` command.

## `vmlinux-to-elf-apply` command causes an error of creating ELF.
Please update `vmlinux-to-elf` to the latest version.

If the problem persists, try using the `ks-apply` command. The logic is different a little, so it might work.
If it still doesn't work, please report it on the issue page.

## `got` command does not display PLT address well.
This problem is probably caused by an outdated version of `binutils`.

The `got` command uses `objdump` internally to obtain the PLT address.
However, with certain combinations of `binutils` and `glibc` versions, `objdump` does not display the PLT address.

The currently known combinations are as follows.
* `binutils 2.38` (Ubuntu 22.04 default) + `glibc 2.37 or later`

This problem occurs when you try to use newer `glibc` in an Ubuntu 22.04 environment using `patchelf` etc.
The workaround is to build and install new `binutils` from source code.

## Can I switch to a mode that references physical memory?
Yes. It is possible if you are using qemu-system. You can switch with `pi enable_phys()` and `pi disable_phys()`.

GEF uses this function internally to switch.
If the mode remains switched due to an interruption during command execution, etc., you will need to fix it manually.

## `magic` command has few valid results.
This is because libc symbols are not loaded.

Unlike kernel symbols, userland symbols do not undergo heuristic detection (with some special exceptions).
Therefore, missing symbols may not be detected by the `magic` command.

If you're referring to system-wide `glibc`, you can resolve it with these steps:
1. Install the symbols with `apt install libc6-dbg`.
2. Add `set debug-file-directory /usr/lib/debug` in `~/.gdbinit`.

## The command to get the source (e.g. `ptr-mangle --source`) doesn't work.
Please do not use tilde (`~`) in the path to specify `.gdbinit-gef.py` in `.gdbinit`.

Depending on the environment, python `inspect` module may not interpret tildes.
I encountered this behavior in python 3.9.2 on debian 11.
This is because `source ~/.gdbinit-gef.py` was written in `/root/.gdbinit`.
I modified it to `source /root/.gdbinit-gef.py`, then it worked.


# About internal mechanism

## How does GEF implement kernel analysis related commands without symbols?
Internally, it consists of several steps.

1. Enumerate memory map information from the page table structure.
2. Detect `.rodata` area of kernel from memory map information.
3. Scan `.rodata` to identify the kernel version.
4. Parse the structure of `kallsyms` in `.rodata` and get all "symbol and address" pairs.
5. If global variable symbols are available at this point, use it. (= `CONFIG_KALLSYMS_ALL=y`).
    * If not, GEF disassembles the function which uses specified global variable.
    * By parsing the result, GEF obtains the address of the required global variable.
    * This is implemented at `KernelAddressHeuristicFinder` class and `KernelAddressHeuristicFinderUtil` class.
6. Detect the offset of the member of the structure, if necessary.
    * To identify it heuristically, GEF uses the fact such as whether a value in memory is an address or whether a structure in memory has a specific structure.
    * At this time, GEF takes into account the presence or absence of members and changes in their order due to differences in kernel versions.
7. Parse and display the value in memory using all the information detected so far.

As you can see, it doesn't work well if structure members are arranged randomly (`CONFIG_RANDSTRUCT=y`).
Also, depending on the assembly output by the compiler, it may not be possible to parse it correctly.


# About python interface

## Can I access each GEF command or alias instance from `python-interactive`?
Yes, you can access it from `GCI` or `GAI`.

For example, `pi GCI["vmmap"]` or `pi GAI["us"]`.
`GCI` means Gef Command Instances.
`GAI` means Gef Alias Instances.

## The class name `KernelAddressHeuristicFinder` is too long.
You can access it using `KF`.

For example, `pi KF.get_slab_caches()` instead of `pi KernelAddressHeuristicFinder.get_slab_caches()`.

## Can I revert the output of `python-interactive` back to decimal from hex?
Yes, you can get it back by executing `pi hexoff()`.

## How can I get the instruction object?
You can get instruction object by `pi get_insn(addr=None)`.

There are also similar functions. Here are the list.
* `get_insn(addr=None)`
* `get_insn_next(addr=None)`
* `get_insn_prev(addr=None)`

## Are there any other globally accessible functions that are useful?
* Memory access
    * `write_memory(addr, data)`, `read_memory(addr, length)`
    * `is_valid_addr(addr)`
    * `read_int_from_memory(addr)`
    * `read_cstring_from_memory(addr, max_length=None, ascii_only=True)`
    * `read_physmem(paddr, size)`, `write_physmem(paddr, data)`
* Register access
    * `get_register(regname, use_mbed_exec=False, use_monitor=False)`
* Other
    * `String.str2bytes(x)`, `String.bytes2str(x)`
    * `slicer(data, n)`, `slice_unpack(data, n)`
    * `p8`, `p16`, `p32`, `p64`
    * `u8`, `u16`, `u32`, `u64`, `u128`

If you want the complete list, run `gef pyobj-list`.

## I want to add a command, how do I get started?
Copy and paste the `TemplateCommand` class and edit it as you like.

Following are some notes.
* Class name
    * Rename the newly added command class to any name you like.
    * Make sure to end it with `...Command`.
* Inheritance
    * Make sure you inherit the `GenericCommand` class.
    * This is the condition for registering the command.
* Important attributes
    * `_cmdline_`: to use to invoke the command.
    * `_aliases_`: to make command alias.
    * `_category_`, `_syntax_`, `_example_` and `_note_`: used by `gef help`.
    * `_repeat_`: to enable command repetition.
* `__init__()`
    * This method is executed only once, when GEF starts.
    * There is usually no need to override this method.
    * Delete it if you don't need to do anything special.
* `do_invoke()`
    * It is important to override this method.
    * When a command is executed, it starts from this method.
* Arguments to command
    * They should be controlled with the `argparse` module.
    * They are handled by the `parse_args` decorator of the `do_invoke()` method.
* Command execution conditions
    * Add decorators to the `do_invoke()` method as needed.
    * You can check the list of decorators that can be added with `gef pyobj-list`.
* Other
    * Use the `gef_print()` function instead of the `print()` function whenever possible.
    * The function named `complete()` is reserved.


# About development schedule

## Are there any plans to support kernels for other architectures?
There are no plans.

## Are there any plans to support more architectures with qemu-user?
Yes. However, it is becoming difficult to find new support targets.

This is because three things are required:
1. toolchain
    * `linux-headers`, `binutils`, `gcc`, `glibc` (or `uClibc`) are needed.
    * Prebuilt tarball is prefer.
2. qemu-user
    * It needs implementation of gdb-stub.
3. gdb
    * It needs python3-support.


# About reporting, etc.

## After I upgraded GEF, it stopped working.
The format of the configuration file may have changed. Try renaming `/root/.gef.rc`.

## I found a bug.
Please feel free to report it on the issue page. I will respond as soon as possible.

## Can you please add this feature? / I don't like a certain feature, so please fix it.
I will consider it, so please report it on the issue page.

But this is a personal development, so I have the final decision. I appreciate your understanding.

## What information should I provide when reporting a issue?
You will need a screenshot or a copy of the terminal output when the problem occurred.
In addition, I am glad if there are the results of the `version` command and `arch-info` command.

Additionally, if the issue is related to kernel debugging, please provide a set of environments (`run.sh`, `bzImage`, `rootfs`, etc.) or where to get them.

## Is it okay to fork and modify?
Yes. However, please follow the license.


# Other memo (Japanese)
* Why I decided to make this
    * [gefを改造した話](https://hackmd.io/@bata24/rJVtBJsrP)
* The story behind each command, etc.
    * [bata24/gefの機能紹介とか](https://hackmd.io/@bata24/SycIO4qPi)
