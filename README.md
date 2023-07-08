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
- Don't instrument full shared libraries. Make a new target and statically link it to your main (shared libraries that won't be instrumented are fine)

# Source-code target
In summary, build the binary multiple times.
- Once with `AFL_LLVM_CMPLOG=1` so this binary can be passed to `-c`
- Once with

# Binary-only target
```bash
AFL_TMPDIR=/mnt/ramdisk afl-fuzz -i /mnt/ramdisk/input -o /mnt/ramdisk/output -Q -c 0 -- ./fuzz_qemu
```
