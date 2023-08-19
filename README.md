# flok-fuzzing-stuff


## Set up a ramdisk
This spares your poor drive from getting destroyed
```bash
mkdir -p /mnt/ramdisk
sudo mount -t ramfs -o size=1024MB ramfs /mnt/ramdisk
chown $(whoami):$(whoami) /mnt/ramdisk
mkdir -p /mnt/ramdisk/{input,output}
```

## Set up the core pattern and set cpu settings
```bash
echo core >/proc/sys/kernel/core_pattern
cd /sys/devices/system/cpu && (echo performance | tee cpu*/cpufreq/scaling_governor) && cd -
```

# Instrumenting target
- Reread `fuzzing_in_depth.md`, but in general, try to build with `afl-clang-lto`, then `afl-clang-fast`, then `afl-gcc` and use the first available.
- Don't instrument full shared libraries if you can help it. Go out of your way to avoid it. Make a new target and statically link it to your main (shared libraries that won't be instrumented are fine).

# Source-code target
In summary, build the binary multiple times.
- Once with `AFL_LLVM_CMPLOG=1` so this binary can be passed to `-c` to measure what coverage is achieved with each input
- Once with `AFL_USE_ASAN=1` to make sure a crash occurs as soon as possible
- Optionally, some others that probably should only be used on their own
    - `AFL_USE_MSAN=1` for finding reads to uninitialized memory
    - `AFL_USE_UBSAN=1` for detecting sketchy operations
    - `AFL_USE_CFISAN=1` for helping to detect C++ type confusion
    - `AFL_USE_TSAN=1` for race conditions

## Persistent mode
If you are fuzzing and have access to source code, you should probably be using persistent mode to speed things up. `instrumentation/README.persistent_mode.md` has the information on how to set this up very quickly.

# Binary-only target

## With qemu
```bash
AFL_TMPDIR=/mnt/ramdisk afl-fuzz -i /mnt/ramdisk/input -o /mnt/ramdisk/output -Q -c 0 -- ./fuzz_qemu
```
