# RISC-V Toolchain Setup — Full Documentation

This document contains the full installation and set up process done for the RISC-V Toolchain



## Environment

**OS : Ubuntu 24.04.2 LTS (native)**



## Installation Steps

### Step 1 - Install Base Developer Tools
```bash
sudo apt-get update
sudo apt-get install -y git vim autoconf automake autotools-dev curl \
  libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex \
  texinfo gperf libtool patchutils bc zlib1g-dev libexpat1-dev gtkwave

```

This command installs a set of development tools (compilers, linkers, autotools) and libraries essential for building and working with software. Here's the breakdown of each packages:

| Package   | Purpose                 |
|:----------|:------------------------|
| git       | Version control system  |
| vim       | A text editor     |
| autoconf, automake, autotools-dev | Tools for generating configure scripts and Makefiles  |
| curl      |  For transferring data with URLs | 
| libmpc-dev, libmpfr-dev, libgmp-dev | Math libraries needed by GCC and other compilers for handling arithmetic |
| gawk | Text processing tool used in build scripts |
| build-essential | Meta-package that installs *gcc*, *g++*, *make*, and other essential compilation tools. |
| bison | Parser generator used by compilers |
| flex | Lexical Analyzer generator |
| texinfo  | Documentation system used by the GNU project to produce online and printed manuals.|
| gperf | A perfect hash function generator |
| libtool | Manages creation of portable shared libraries across platforms |
| patchutils | A collection of small programs that help in applying and managing patches (changes) to source code.|
| bc | A precision calculator language |
| zlib1g-dev |A compression library |
| libexpat1-dev | XML parser library|
| gtkwave | A waveform viewer|

----




### Step 2 - Create a clean workspace and capture your home path
The aim is to organise all the files. Here, all RISC-V toolchain components are contained in ~/riscv_toolchain folder, making it easy to remove, update, or
archive.

```bash
cd
pwd=$PWD
mkdir -p riscv_toolchain
cd riscv_toolchain
```

$PWD - A built-in environment variable in the shell that always stores your ***current working directory path***

```pwd=$PWD``` - Stores your current working directory (full path) into a shell variable called pwd

---






### Step 3 - Get a prebuilt RISC‑V GCC toolchain
Downloading and unpacking a prebuilt RISC-V compiler instead of building it from source. GCC compiler targeting the RISC-V 64-bit architecture (riscv64) and bare-metal programs (unknown-elf).
```
wget "https://static.dev.sifive.com/dev-tools/riscv64-unknown-elf-gcc-
8.3.0-2019.08.0-x86_64-linux-ubuntu14.tar.gz"
tar -xvzf riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-
ubuntu14.tar.gz
```
---





### Step 4 — Add toolchain to your PATH
Allows riscv64-unknown-elf-gcc and related binaries to be available without typingfull paths, both now and after you reopen the terminal.

Current shell
```
export PATH=$pwd/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-x86_64-linux-ubuntu14/bin:$PATH

```
Persistent
```
echo 'export PATH=$HOME/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-
2019.08.0-x86_64-linux-ubuntu14/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```
---





### Step 5 — Install Device Tree Compiler
A **device tree** is a data structure describing the hardware layout of a system. A ***Device Tree Compiler*** tool that **compiles** device tree source files (.dts) into device tree blobs (.dtb) that can be read by an operating system or simulator.
```
sudo apt-get install -y device-tree-compile
```
---




### Step 6 — Build and install Spike (RISC‑V ISA simulator)
Spike is the reference ISA simulator. It can be almost considered as a *virtual* CPU where it takes a RISC-V program and executes it instruction by instruction in software.
```
cd $pwd/riscv_toolchain
git clone https://github.com/riscv/riscv-isa-sim.git
cd riscv-isa-sim
mkdir -p build && cd build
../configure --prefix=$pwd/riscv_toolchain/riscv64-unknown-elf-gcc-
8.3.0-2019.08.0-x86_64-linux-ubuntu14
make -j$(nproc)
sudo make install
```
---




### Step 7 — Build and install the RISC‑V Proxy Kernel (riscv-pk)
pk provides a minimal runtime so you can run newlib ELF binaries under Spike with 'spike pk ./your_prog'. It bridges your compiled program to the simulator.
```
cd $pwd/riscv_toolchain
git clone https://github.com/riscv/riscv-pk.git
cd riscv-pk
git checkout v1.0.0 
mkdir -p build && cd build
../configure --prefix=$pwd/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14 --host=riscv64-unknown-elf
make -j$(nproc)
sudo make install
```
---




### Step 8 — Ensure the cross bin directory is in PATH
Some installs place pk and related utilities into a nested riscv64-unknown-elf/bin. Adding this ensures pk is found by 'which pk'.

Command (current shell):
```
export PATH=$pwd/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14/riscv64-unknown-elf/bin:$PATH
```
Command (persistent):
```
echo 'export PATH=$HOME/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-
2019.08.0-x86_64-linux-ubuntu14/riscv64-unknown-elf/bin:$PATH' >>
~/.bashrc
source ~/.bashrc
```
---




### Step 9 — Install Icarus Verilog
For digital design education, you’ll often verify RTL with Icarus Verilog and view waveforms with GTKWaves. 
```
cd $pwd/riscv_toolchain
git clone https://github.com/steveicarus/iverilog.git
cd iverilog
git checkout --track -b v10-branch origin/v10-branch
git pull
chmod +x autoconf.sh
./autoconf.sh
./configure
make -j$(nproc)
sudo make install
```
---




### Step 10 — Quick sanity checks
Confirms the toolchain and simulator are visible and runnable from your shell.
```
which riscv64-unknown-elf-gcc
riscv64-unknown-elf-gcc -v
which spike
which pk
```
---

## Uniqueness Test
**1) Code** - [source file](unique_test.c)

**2) Compile Command** - [command file](compile_command.txt)
```
riscv64-unknown-elf-gcc -O2 -Wall -march=rv64imac -mabi=lp64 \
  -DUSERNAME=\"$(id -un)\" -DHOSTNAME=\"$(hostname -s)\" \
  unique_test.c -o unique_test

```

**3) Run on Spike with the proxy kernel**
```
spike pk ./unique_test
```

**4) Program output** - [output file](output.txt)
