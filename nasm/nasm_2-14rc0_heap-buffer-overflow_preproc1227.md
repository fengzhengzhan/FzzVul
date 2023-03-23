# Heap buffer overflow in new_Token() at preproc.c:1227

Heap buffer overflow in nasm at function new_Token in preproc.c:1227.

## Environment
Ubuntu 18.04, 64 bit
nasm 2.14rc0

## Steps to reproduce
1. download file
```
git clone git://repo.or.cz/nasm.git nasm
cd nasm
git checkout 7a81ead
```
2. compile libming with ASAN
```
export FORCE_UNSAFE_CONFIGURE=1
export LLVM_COMPILER=clang
./autogen.sh
CC=wllvm CXX=wllvm++ CFLAGS="-g -O0 -fcommon -Wno-error" ./configure --prefix=`pwd`/obj-bc --disable-shared
make
make install

cd obj-bc/bin/
extract-bc nasm
clang -fsanitize=address nasm.bc -o nasm_asan
```
3. command for reproducing the error
```
./nasm_asan -f bin poc -o /dev/null
```
Download poc: [nasm_2-14rc0_heap-buffer-overflow_preproc1227.zip](https://github.com/fengzhengzhan/FzzVul/blob/main/nasm/nasm_2-14rc0_heap-buffer-overflow_preproc1227.zip)

## ASAN report
```
> ./nasm_asan -f bin nasm_2-14rc0_heap-buffer-overflow_preproc1227 -o /dev/null
nasm_2-14rc0_heap-buffer-overflow_preproc1227:1: warning: unterminated string
nasm_2-14rc0_heap-buffer-overflow_preproc1227:1: error: label or instruction expected at start of line
nasm_2-14rc0_heap-buffer-overflow_preproc1227:2: error: label or instruction expected at start of line
nasm_2-14rc0_heap-buffer-overflow_preproc1227:3: error: attempt to define a local label before any non-local labels
nasm_2-14rc0_heap-buffer-overflow_preproc1227:3: error: parser: instruction expected
nasm_2-14rc0_heap-buffer-overflow_preproc1227:4: error: unterminated %[ construct
=================================================================
==15450==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x615000001180 at pc 0x00000043ab45 bp 0x7ffd21170bf0 sp 0x7ffd211703a0
READ of size 1 at 0x615000001180 thread T0
    #0 0x43ab44 in __interceptor_memcpy.part.46 /root/LLVM/llvm/projects/compiler-rt/lib/asan/../sanitizer_common/sanitizer_common_interceptors.inc:810
    #1 0x508ae5 in new_Token /root/compiler1804/reportCVEs/nasm/asm/preproc.c:1227:9
    #2 0x509cea in tokenize /root/compiler1804/reportCVEs/nasm/asm/preproc.c:1148:25
    #3 0x50807c in pp_getline /root/compiler1804/reportCVEs/nasm/asm/preproc.c:5124:25
    #4 0x4f53bb in assemble_file /root/compiler1804/reportCVEs/nasm/asm/nasm.c:1233:24
    #5 0x4f438a in main /root/compiler1804/reportCVEs/nasm/asm/nasm.c:453:9
    #6 0x7feda8abf082 in __libc_start_main /build/glibc-SzIz7B/glibc-2.31/csu/../csu/libc-start.c:308:16
    #7 0x41bb79 in _start (/home/fzz/Desktop/STFGFuzz/dataset/compiler1804/reportCVEs/nasm/obj-bc/bin/nasm_asan+0x41bb79)

0x615000001180 is located 0 bytes to the right of 512-byte region [0x615000000f80,0x615000001180)
allocated by thread T0 here:
    #0 0x4ae190 in malloc /root/LLVM/llvm/projects/compiler-rt/lib/asan/asan_malloc_linux.cpp:145
    #1 0x4f7914 in nasm_malloc /root/compiler1804/reportCVEs/nasm/nasmlib/malloc.c:47:15
    #2 0x50a832 in read_line /root/compiler1804/reportCVEs/nasm/asm/preproc.c:823:18
    #3 0x508057 in pp_getline /root/compiler1804/reportCVEs/nasm/asm/preproc.c:5121:20
    #4 0x4f53bb in assemble_file /root/compiler1804/reportCVEs/nasm/asm/nasm.c:1233:24
    #5 0x4f438a in main /root/compiler1804/reportCVEs/nasm/asm/nasm.c:453:9
    #6 0x7feda8abf082 in __libc_start_main /build/glibc-SzIz7B/glibc-2.31/csu/../csu/libc-start.c:308:16

SUMMARY: AddressSanitizer: heap-buffer-overflow /root/LLVM/llvm/projects/compiler-rt/lib/asan/../sanitizer_common/sanitizer_common_interceptors.inc:810 in __interceptor_memcpy.part.46
Shadow bytes around the buggy address:
  0x0c2a7fff81e0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c2a7fff81f0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c2a7fff8200: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c2a7fff8210: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c2a7fff8220: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
=>0x0c2a7fff8230:[fa]fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c2a7fff8240: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c2a7fff8250: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c2a7fff8260: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c2a7fff8270: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 fa
  0x0c2a7fff8280: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
Shadow byte legend (one shadow byte represents 8 application bytes):
  Addressable:           00
  Partially addressable: 01 02 03 04 05 06 07 
  Heap left redzone:       fa
  Freed heap region:       fd
  Stack left redzone:      f1
  Stack mid redzone:       f2
  Stack right redzone:     f3
  Stack after return:      f5
  Stack use after scope:   f8
  Global redzone:          f9
  Global init order:       f6
  Poisoned by user:        f7
  Container overflow:      fc
  Array cookie:            ac
  Intra object redzone:    bb
  ASan internal:           fe
  Left alloca redzone:     ca
  Right alloca redzone:    cb
  Shadow gap:              cc
==15450==ABORTING
```
