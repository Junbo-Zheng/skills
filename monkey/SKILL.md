---
name: monkey
description: Guide for configuring Monkey testing on devices. Use when helping beginners set up Monkey testing or when troubleshooting issues that require specific debug configurations.
---

# Monkey Testing Configuration Guide

This skill provides guidance for configuring Monkey testing on devices and selecting appropriate debug configurations for troubleshooting.

## What is Monkey Testing?

Monkey testing is an automated testing method that simulates random user operations (taps, swipes, etc.) to test system stability and robustness. On devices, Monkey testing is used for:

- Stress testing system stability
- Discovering memory leaks, crashes, and other issues
- Verifying application behavior during long-running operations

## Essential Monkey Testing Configurations

### Basic Configuration (Always Enable)

| Configuration | Enable/Disable | Purpose |
|--------------|---------------|---------|
| `CONFIG_MIWEAR_APPS_MONKEYTEST` | Enable | Enable Monkey testing functionality |
| `MIWEAR_APPS_MONKEYTEST_SCREEN_ALWAYSON` | Enable | Support screen on/off during Monkey testing |

### Debug Configurations (Enable as Needed)

| Configuration | Enable/Disable | Purpose | Use Case |
|--------------|---------------|---------|----------|
| `CONFIG_WATCHDOG` | Disable | Disable OS watchdog | When system hangs, use BES I2C tool to check PC location |
| `CONFIG_MM_BACKTRACE=8`<br>`CONFIG_MM_BACKTRACE_DEFAULT=y` | Enable | Record malloc call pid and backtrace | View memory ownership, dump which pid allocated each memory block |
| `CONFIG_COREDUMP`<br>`CONFIG_BOARD_COREDUMP_BLKDEV`<br>`CONFIG_BOARD_COREDUMP_DEVPATH="/dev/coredump"`<br>`CONFIG_SYSTEM_COREDUMP`<br>`CONFIG_BOARD_COREDUMP_COMPRESSION`<br>`CONFIG_BOARD_COREDUMP_FULL` | Enable | GDB debugging requires coredump | Analyze crash issues |
| `CONFIG_MM_FILL_ALLOCATIONS` | Enable | Fill memory with 0xaa before allocation, 0x55 after free | Detect memory out-of-bounds access |
| `CONFIG_MM_KASAN` | Enable | Debug memory corruption issues | Bin size may increase 1.5x, slower execution. **Note**: Consider per-app KASan instead of global to reduce size impact |
| `CONFIG_FORTIFY_SOURCE` | Enable | Add bounds checking to libc functions | Enhance memory safety |
| `CONFIG_TTY_FORCE_PANIC` | Enable | Force crash when system hangs with no response | Dump现场信息 |
| `CONFIG_STACK_CANARIES` | Disable | Add bounds checking to stack variables | Enable during early project phase, disable later |
| `CONFIG_ARCH_DEADLOCKDUMP` | Disable | Auto-analyze deadlock on crash | Enable when debugging deadlock issues |
| `CONFIG_MIWEAR_LPA`<br>`CONFIG_MIWEAR_LPA_AUTH` | Disable | eSIM devices can disable | Save size, not needed for Monkey testing |
| `CONFIG_QUICKAPP` | Disable | QuickApp framework | Disable to save size when not needed |
| `CONFIG_MIWEAR_LPA` | Disable | LPA (Local Profile Assistant) | Disable to save size when not needed |
| `CONFIG_BES_GPU_DUMP_TIMEOUT` | Enable | GPU dump timeout | Capture crash logs for GPU timeout issues |

## Vela Debug Configurations

### Runtime Memory Checking

| Feature Name | CONFIG | Purpose | Impact | Recommended |
|-------------|--------|---------|--------|-------------|
| _FORTIFY_SOURCE | `CONFIG_FORTIFY_SOURCE` | Add bounds checking to libc functions | Minimal impact on size and speed | Always enable |
| Stack overflow | `CONFIG_STACK_COLORATION` | Check stack usage by filling with magic | Almost no impact | Always enable |
| Heap dump | `CONFIG_MM_BACKTRACE` | Record malloc call pid and backtrace | Slower malloc, increased memory usage | Use `CONFIG_MM_BACKTRACE=0` |
| Leak detection | `CONFIG_MM_BACKTRACE`<br>`CONFIG_MM_BACKTRACE_DEFAULT` | Memory leak analysis based on heap dump | Same as heap dump | Enable when needed |
| Heap check | `CONFIG_DEBUG_MM` | Check heap integrity | Slight impact on malloc/free speed | Always enable |
| ASan | `CONFIG_SIM_ASAN` | Instrument memory, more checks than kasan | Increased size, slower speed | Always enable (sim only) |
| KASan | `CONFIG_MM_KASAN`<br>`CONFIG_MM_KASAN_ALL` | Debug memory corruption | Bin size may increase 1.5x, slower | Not for regular use, debug only. **Tip**: Use per-app KASan instead of global to reduce size impact |
| Stack canaries | `CONFIG_STACK_CANARIES` | Add bounds checking to stack variables | Increased size, minimal speed impact | Enable early, disable later |
| UBSan | `CONFIG_MM_UBSAN` | Detect undefined behavior in code | Increased size, slower speed | Not for regular use, debug only |
| Runtime stack size analysis | `CONFIG_SCHED_STACK_RECORD`<br>`CONFIG_ARCH_INSTRUMENT_ALL` | Record max depth and call stack per task | Minimal size impact, slower | Not for regular use, debug only |

### Crash Scene Restoration

| Feature Name | CONFIG | Purpose | Impact | Recommended |
|-------------|--------|---------|--------|-------------|
| Deadlock | `CONFIG_ARCH_DEADLOCKDUMP` | Auto-analyze deadlock on crash | Almost no impact | Always enable |
| Coredump | `CONFIG_BOARD_COREDUMP_BLKDEV` | Store coredump in block device | Almost no impact | Always enable if storage available |
| gdbserver.py | `CONFIG_ARCH_STACKDUMP` | Restore crash scene and start gdbserver | Almost no impact | Recommended |

### Online Debugging

| Feature Name | CONFIG | Purpose | Impact | Recommended |
|-------------|--------|---------|--------|-------------|
| GDB Stub | `CONFIG_LIB_GDBSTUB`<br>`CONFIG_SYSTEM_GDBSTUB`<br>`CONFIG_SERIAL_GDBSTUB`<br>`CONFIG_BOARD_MEMORY_RANGE` | Analyze crash via serial using GDB | Almost no impact | Based on serial port resources |
| Force panic | `CONFIG_TTY_FORCE_PANIC` | Force crash when system hangs | Almost no impact | Always enable, disable serial nsh before release |

### Performance Analysis

| Feature Name | CONFIG | Purpose | Impact | Recommended |
|-------------|--------|---------|--------|-------------|
| cpuload | `CONFIG_SCHED_CPULOAD_SYSCLK`<br>`CONFIG_SCHED_CRITMONITOR`<br>`CONFIG_SCHED_CPULOAD_CRITMONITOR` | Track CPU usage | Slightly slower execution | Always enable |
| Interrupt frequency | `CONFIG_SCHED_IRQMONITOR` | Track interrupt frequency | Slower interrupts, slower system | Enable when debugging interrupt issues |
| Execution time | `CONFIG_SCHED_CRITMONITOR_MAXTIME_*` | Track various execution times | Slower execution | Enable when debugging performance |
| Trace analysis | Multiple CONFIGs | Instrument code for analysis | Slower system | Enable when debugging code performance |
| gprof | `CONFIG_SYSTEM_GPROF`<br>`CONFIG_SCHED_GPROF`<br>`CONFIG_SCHED_GPROF_ALL`<br>`CONFIG_SIM_GPROF` | Collect function call and timing info | Extra memory usage | Enable when analyzing performance |
| showinfo | `LOW_RESOURCE_TEST` | Periodically print system info to logs | Almost no impact | Enable in test builds, disable in release |

### File Transfer

| Feature Name | CONFIG | Purpose | Impact | Recommended |
|-------------|--------|---------|--------|-------------|
| Ymodem | `CONFIG_SYSTEM_YMODEM` | Support Ymodem serial file transfer | Almost no impact | Always enable |
| OFLoader | `CONFIG_SYSTEM_OFLOADER` | Flash any device via jlink | Requires project adaptation | Based on project support |
| Zmodem | `CONFIG_SYSTEM_ZMODEM` | Support Zmodem serial file transfer | Almost no impact | Not recommended, use Ymodem |
| Semihosting | `CONFIG_FS_HOSTFS`<br>`CONFIG_ARM_SEMIHOSTING_HOSTFS` | Access host files via Jlink | Almost no impact | Enable if Jlink/trace32 available |
| Fastboot | `CONFIG_SYSTEM_FASTBOOTD` | Support fastboot commands | Almost no impact | Enable for usb/net bootloaders |
| adb file transfer | Multiple CONFIGs | Support adb commands | Almost no impact | Enable for usb/net devices |
| curl | `CONFIG_TOOLS_CURL` | Support curl commands | Adds curl library size | Enable for networked devices |
| scp | Multiple CONFIGs | Support scp commands | Adds ssh library, larger size | Enable if needed |

### File Debug and CLI Tools

| Feature Name | CONFIG | Purpose | Impact | Recommended |
|-------------|--------|---------|--------|-------------|
| hexdump | `CONFIG_NSH_LIBRARY`<br>`# CONFIG_NSH_DISABLE_HEXDUMP is not set` | Hex dump files | Slightly larger build size | Enable if size allows |
| md5 | `CONFIG_NETUTILS_CODECS`<br>`CONFIG_CODECS_HASH_MD5` | Calculate file md5 | Slightly larger build size | Enable if size allows |
| fdinfo | `CONFIG_FS_PROCFS=y`<br>`CONFIG_NSH_DISABLE_FDINFO=n` | View open fd info for current process | Slightly larger build size | Enable when needed |
| mount | `CONFIG_DISABLE_MOUNTPOINT=n`<br>`CONFIG_FS_<filesystem>=y` | Mount filesystems | - | - |
| losetup | `CONFIG_DISABLE_MOUNTPOINT=n`<br>`CONFIG_NSH_DISABLE_LOSETUP=n`<br>`CONFIG_DEV_LOOP=y` | Map file as block device | - | - |
| lomtd | `CONFIG_DISABLE_MOUNTPOINT=n`<br>`CONFIG_NSH_DISABLE_LOMTD=n`<br>`CONFIG_MTD_LOOP=y` | Map file as mtd device | - | - |

### Testing

| Feature Name | CONFIG | Purpose | Impact | Recommended |
|-------------|--------|---------|--------|-------------|
| fstest | `CONFIG_TESTING_FSTEST` | Test filesystem stability | - | Always enable |
| opus_ramtest | `CONFIG_TESTING_OPUS_RAMTEST` | Test memory stability | 200k array, larger size | Not for regular use, test only |
| memstress | `CONFIG_TESTING_MEMORY_STRESS` | Memory stress testing | Increased system memory load | Always enable for memory issues |
| memtester | `CONFIG_UTILS_MEMTESTER` | Test memory stability | - | Not for regular use, test only |
| blktest | `CONFIG_TESTING_DRIVER_TEST` | Test block devices | - | Not for regular use, test only |
| Monkey | `CONFIG_TESTING_MONKEY=y` | Simulate real device operations for stress testing | - | - |

### Benchmark

| Feature Name | CONFIG | Purpose | Impact | Recommended |
|-------------|--------|---------|--------|-------------|
| CacheSpeed | `CONFIG_BENCHMARK_CACHESPEED` | Get cache interface performance | None | Always enable |
| ramspeed | `CONFIG_BENCHMARK_RAMSPEED`<br>`CONFIG_LIBC_FLOATINGPOINT` | Evaluate memory performance | None | Always enable |
| Dhrystone | `CONFIG_BENCHMARK_DHRYSTONE` | Test CPU integer and logic operations | None | Not for regular use, test only |
| whetstone | `CONFIG_BENCHMARK_WHETSTONE` | Evaluate CPU floating point performance | None | Not for regular use, test only |
| CoreMark | `CONFIG_BENCHMARK_COREMARK` | Measure CPU performance | None | Not for regular use, test only |
| Coremark-Pro | `CONFIG_LIBC_FLOATINGPOINT`<br>`BENCHMARK_COREMARK_PRO` | Test entire processor performance | None | Not for regular use, test only |
| dd | `CONFIG_NSH_DISABLE_DD=n`<br>`CONFIG_NSH_CMDOPT_DD_STATS=y` | Read, convert, and output data | None | Always enable |
| RTOS-Benchmark | `CONFIG_SIG_SIGSTOP_ACTION`<br>`CONFIG_RTOS_BENCHMARK` | Evaluate OS performance | None | Not for regular use, test only |
| tinymembench | `CONFIG_BENCHMARKS_TINYMEMBENCH` | Memory bandwidth and latency benchmark | None | Not for regular use, test only |
| cyclictest | `CONFIG_BENCHMARKS_RTTESTS` | Measure clock latency and jitter | None | Not for regular use, test only |
| test-tlb | `CONFIG_BENCHMARKS_TESTTLB` | Measure memory latency under different access patterns | None | Not for regular use, test only |
| TacleBench | `CONFIG_BENCHMARKS_TACLEBENCH` | Theoretical real-time analysis | None | Not for regular use, test only |
| Lmbench | `CONFIG_BENCHMARK_LMBENCH` | Bandwidth and latency testing | None | Not for regular use, test only |
| operf | `CONFIG_BENCHMARK_OSPERF`<br>`CONFIG_PIPES` | Analyze system basic operation performance | None | Not for regular use, test only |

## Troubleshooting Configuration Recommendations

### Memory Issues

For memory-related problems (leaks, corruption, etc.), enable:

- `CONFIG_MM_BACKTRACE=8` + `CONFIG_MM_BACKTRACE_DEFAULT=y` - Record memory allocation info
- `CONFIG_MM_FILL_ALLOCATIONS` - Detect memory out-of-bounds
- `CONFIG_MM_KASAN` - Debug memory corruption
- `CONFIG_DEBUG_MM` - Check heap integrity
- `CONFIG_TESTING_MEMORY_STRESS` - Memory stress testing

### Crash Issues

For system crashes, enable:

- `CONFIG_COREDUMP` + `CONFIG_BOARD_COREDUMP_BLKDEV` - Generate coredump
- `CONFIG_ARCH_STACKDUMP` - Restore crash scene
- `CONFIG_TTY_FORCE_PANIC` - Force crash dump scene
- `CONFIG_ARCH_DEADLOCKDUMP` - Analyze deadlock

### Performance Issues

For performance problems, enable:

- `CONFIG_SCHED_CPULOAD_SYSCLK` - Track CPU usage
- `CONFIG_SCHED_IRQMONITOR` - Track interrupt frequency
- `CONFIG_SCHED_CRITMONITOR_MAXTIME_*` - Track execution times
- `CONFIG_BENCHMARK_CACHESPEED` - Cache performance testing
- `CONFIG_BENCHMARK_RAMSPEED` - Memory performance testing

### Hang Issues

For system hangs, enable:

- `CONFIG_WATCHDOG` - Disable watchdog to check PC location
- `CONFIG_TTY_FORCE_PANIC` - Force crash dump scene
- `CONFIG_ARCH_DEADLOCKDUMP` - Analyze deadlock

## Important Notes

1. **Flash Size Limitations**: Debug configurations consume additional flash size. Choose configurations based on actual needs.
2. **Performance Impact**: Some configurations (e.g., KASan, ASan) significantly affect system speed. Enable only during debugging.
3. **Product Release**: Disable debug-related configurations (e.g., serial nsh) before product release.
4. **eSIM Devices**: Disable `CONFIG_MIWEAR_LPA` and `CONFIG_MIWEAR_LPA_AUTH` to save size.
5. **KASan Size Optimization**: Enabling global KASan (`CONFIG_MM_KASAN_ALL`) can cause significant size increases (up to 1.5x). Consider:
   - Using per-app KASan instead of global KASan to reduce size impact
   - Disabling unused configurations like `CONFIG_QUICKAPP` and `CONFIG_MIWEAR_LPA` to free up space for KASan
   - Only enable KASan for the specific app being debugged rather than system-wide
