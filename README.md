# flok-fuzzing-stuff


## Set up a ramdisk
This spares your poor drive from getting destroyed
```bash
mkdir -p /mnt/ramdisk
sudo mount -t ramfs -o size=1024MB ramfs /mnt/ramdisk
sudo chown $(whoami):$(whoami) /mnt/ramdisk
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
Note that cmplog does not appear to be implemented for anything other than `i386` and `x86_64` at this point, so if you need good coverage information on an unsupported architecture you will have to implement that on your own.
```bash
AFL_TMPDIR=/mnt/ramdisk afl-fuzz -i /mnt/ramdisk/input -o /mnt/ramdisk/output -Q -c 0 -- ./fuzz_qemu
```

### Running binaries in the wrong system environment
Binaries are normally compiled to run on a single version of linux with a set version of glibc. 
This is almost always true for ctf challenges. If you still want to run it on your system and you don't have the same os version/environment that a ctf challenge was made for, you can bypass that to some degree by copying the requisite binary and libraries for a challenge locally, changing the binary's interpreter with `patchelf --set-interpreter "<ld-interpreter>" "<binary>"`, and 
setting the environment variable `QEMU_SET_ENV="LD_LIBRARY_PATH=$(pwd)"`, which will work for the underlying qemu instance that afl uses
```
AFL_TMPDIR=/mnt/ramdisk QEMU_SET_ENV="LD_LIBRARY_PATH=$(pwd)" afl-fuzz -Q -c 0 -i /mnt/ramdisk/input -o /mnt/ramdisk/output -- ./ctf_chal
```

## Unicorn mode
Example using the sample test python harness. *NOTE:* There is a good chance that the `unicornafl` python package will yell at you if you try to run it without `afl-fuzz`
```bash
AFL_TMPDIR=/mnt/ramdisk afl-fuzz -U -m none -i /mnt/ramdisk/input -o /mnt/ramdisk/output -- python3 simple_test_harness.py ./simple_target.bin
```
