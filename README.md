Chickadee OS
============

Быстрй старт с помощью команд: `make run` или `make run-PROGRAM`

Определенные виды запуска
------------

`make NCPU=N run` если хотите запустить ОС с `N` виртуальными процессорами (по умолчанию их 2]). Для того чтобы выйти из ОС, закройте окно QEMU или нажмите `q`.

`make run-console` если хотите запустить ОС в окне консоли.

`make SAN=1 run` to run with sanitizers enabled.

Normally Chickadee’s debug log is written to `log.txt`. `make LOG=stdio run`
will redirect the debug log to the standard output, and `make
LOG=file:FILENAME run` will redirect it to `FILENAME`.

Run `make D=1 run` to ask QEMU to print verbose information about interrupts
and CPU resets to the standard error. This setting will also cause QEMU to
quit after encountering a [triple fault][] (normally it will reboot).

`make run-PROGRAM` runs `p-PROGRAM.cc` as the first non-init process. The
default is `alloc`.

`make HALT=1 run-PROGRAM` should make QEMU exit once all processes are done.

Решение проблем при запуске
---------------

There are several ways to kill a recalcitrant QEMU (for instance, if your
OS has become unresponsive).

* If QEMU is running in its own graphical window, then close the window. This
  will kill the embedded OS.

* If QEMU is running in a terminal window (in Docker, for instance), then
  press `Alt-2`. This will bring up the QEMU Monitor, which looks like this:

    ```
    compat_monitor0 console
    QEMU 4.2.0 monitor - type 'help' for more information
    (qemu)
    ```

    Type `quit` and hit Return to kill the embedded OS and return to your
    shell. If this leaves the terminal looking funny, enter the `reset` shell
    command to restore it.

    If `Alt-2` does not work, you may need to configure your terminal to
    properly send the Alt key. For instance, on Mac OS X’s Terminal, go to
    Terminal > Preferences > Keyboard and select “Use Option as Meta key”. You
    can also configure a special keyboard shortcut that sends the `Escape 2`
    sequence.

Run `make run-gdb` to start up the OS with support for GDB debugging. This
will start the OS, but not GDB. You must run `gdb -x build/weensyos.gdb` to
connect to the running emulator; when GDB connects, it will stop the OS and
wait for instructions.

If you experience runtime errors involving `obj/libqemu-nograb.so.1`, put
`QEMU_PRELOAD_LIBRARY=` in `config.mk`. This disables a shim we use that
prevents QEMU from grabbing the mouse.

Исходные файлы
------------

### Общие файлы

| Файл            | Описание                               |
| --------------- | -------------------------------------- |
| `types.h`       | Определение типов                      |
| `lib.hh/cc`     | Библиотека С                           |
| `x86-64.h`      | x86-64 определение аппаратных средств  |
| `elf.h`         | ELF64 структура                        |

### Boot loader

| Файл            | Описание                     |
| --------------- | ---------------------------- |
| `bootentry.S`   | Boot loader entry point      |
| `boot.cc`       | Boot loader main code        |
| `boot.ld`       | Boot loader linker script    |

### Kernel core

| Файл                | Описание                             |
| ------------------- | ------------------------------------ |
| `kernel.hh`         | Kernel declarations                  |
| `k-exception.S`     | Kernel entry points                  |
| `k-init.cc`         | Kernel initialization                |
| `k-lock.hh`         | Kernel spinlock                      |
| `k-vmiter.hh/cc`    | Page table iterators                 |
| `k-cpu.cc`          | Kernel `cpustate` type               |
| `k-proc.cc`         | Kernel `proc` type                   |
| `kernel.cc`         | Kernel exception handlers            |
| `k-memviewer.cc`    | Kernel memory viewer                 |
| `kernel.ld`         | Kernel linker script                 |

### Kernel libraries

| Файл                | Описание                             |
| ------------------- | ------------------------------------ |
| `k-memrange.hh`     | Memory range type tracker            |
| `k-hardware.cc`     | General hardware access              |
| `k-devices.hh/cc`   | Keyboard, console, memory files      |
| `k-apic.hh/cc`      | Interrupt controller hardware        |
| `k-pci.hh`          | PCI bus hardware                     |
| `k-mpspec.cc`       | Boot-time configuration              |
| `k-sanitizers.cc`   | Sanitizer support                    |

### Processes

| Файл              | Описание                                         |
| ----------------- | ------------------------------------------------ |
| `u-lib.cc/hh`     | Process library and system call implementations  |
| `p-allocator.cc`  | Allocator process                                |
| `process.ld`      | Process binary linker script                     |

### File system

| Файл                  | Описание                                         |
| --------------------- | ------------------------------------------------ |
| `chickadeefs.hh`      | Defines chkfs (ChickadeeFS) layout               |
| `journalreplayer.cc`  | Logic for replaying chkfs journals               |

Build files
-----------

The main output of the build process is a disk image,
`chickadeeos.img`. QEMU “boots” off this disk image, but the image
could conceivably boot on real hardware! The build process also
produces other files that can be useful to examine.

| Файл                       | Описание                             |
| -------------------------- | ------------------------------------ |
| `obj/kernel.asm`           | Kernel assembly (with addresses)     |
| `obj/kernel.sym`           | Kernel defined symbols               |
| `obj/p-PROCESS.asm`, `sym` | Same for process binaries            |

[CS 161]: https://read.seas.harvard.edu/cs161/2021/
[triple fault]: https://en.wikipedia.org/wiki/Triple_fault
