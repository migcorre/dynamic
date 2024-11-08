# dynamic

## *Yosys installation:* 
Yosys is a RTL synthesis with extensive Verilog 2005 support \

2 ways: \
number 1: yosys inside OSS CAD suite. has many soft inluding yosys.
this way is almost straightforward"
 Download oss-cad-suite from https://github.com/YosysHQ/oss-cad-suite-build/releases/tag/2024-11-07 This containt yosys. \
 Extract \
 yosys will be installed in oss-cad-suite/bin folder. you can run here running > ./yosys or adding the path to $PATH variable to the bash interface. \
```
export PATH="<extracted_location>/oss-cad-suite/bin:$PATH" 
```
you can add this line in bashrc file located in your HOME, in this way you can invoque yosys from anyway typing yosys

references: \
https://github.com/YosysHQ/oss-cad-suite-build?tab=readme-ov-file

number 2: installing ONLY yosys:
```
git clone https://github.com/YosysHQ/yosys.git
$ sudo apt-get install build-essential clang lld bison flex \
	libreadline-dev gawk tcl-dev libffi-dev git \
	graphviz xdot pkg-config python3 libboost-system-dev \
	libboost-python-dev libboost-filesystem-dev zlib1g-dev
$ make config-clang
$ make config-gcc
```
you will need to initialize all git submodule before next steps. this step avoid abc installation problems.
what is abc? is one syntesis optimizer --> https://people.eecs.berkeley.edu/~alanmi/abc/
```
git submodule -init
```
references: \
https://github.com/YosysHQ/yosys
https://github.com/YosysHQ/yosys/issues/4403

## *iverilog installation*
Icarus Verilog (iverilog) is intended to compile ALL of the Verilog HDL, as described in the IEEE-1364 standard \

Clone git repository https://github.com/steveicarus/iverilog \
NOTE: do the following in superuser mode (sudo) \
```
apt install -y autoconf gperf make gcc g++ bison flex 
sh autoconf.sh
./configure
make
make check
make install
```
add the folder to the $PATH variable, as we did it in Yosys.

references: \
https://github.com/steveicarus/iverilog

## *GTKWave installation*
GTKWave is a fully featured GTK+ based wave viewer for Unix and Win32 which reads FST, and GHW files as well as standard Verilog VCD/EVCD files and allows their viewing. \
NOTE: do the following in superuser mode (sudo)
```
apt install build-essential meson gperf flex desktop-file-utils libgtk-4-dev \
            libbz2-dev libjudy-dev libgirepository1.0-dev
git clone "https://github.com/gtkwave/gtkwave.git"
cd gtkwave
meson setup build && cd build && meson install
```
NOTE:Is not necessary addit to the $PATH variable

references: \
https://github.com/gtkwave/gtkwave


