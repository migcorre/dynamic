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

# One Example:
We create a basic conter in verilog.

```verilog
  module count(clk, rst, val);
    input  clk, rst;
    output [3:0] val;

    reg [3:0] val, nval;

    always @(posedge clk or negedge rst)
      if(rst == 0) val = 0;
      else val = nval;

    always @(val)
      nval = val + 1;
  endmodule
```
save it as counter.v

We try to synthetize this code in yosys:
we invoque yosys tool writing "yosys". One inside yosys enviroment
```
> read_verilog -sv counter.v
> hierarchy -top count
> proc
> opt
> techmap
> opt
> write_netlist counter.sv
> exit
```

We endup writing a netlist calls counter.sv 

We need to test it, for that We generate a testbech:

```verilog
module testbench;
  reg clk, rst;
  wire [3:0] val;

  // Instancia del módulo count
  count uut (
    .clk(clk),
    .rst(rst),
    .val(val)
  );

  // Generador de reloj
  always #5 clk = ~clk; // Toggle cada 5 unidades de tiempo

  initial begin
    // Inicialización
    $dumpfile("counter.vcd");
    $dumpvars(0, testbench);
    clk = 0;
    rst = 0;

    // Reset del sistema
    #10 rst = 1;  // Release reset after 10 units of time
    #100 rst = 0; // Apply reset to see the module reset its value
    #10 rst = 1;  // Release reset

    // Simulación durante 200 unidades de tiempo
    #200 $finish;
  end

  initial begin
    // Monitor para observar los cambios
    $monitor("At time %t, val = %d", $time, val);
  end
endmodule

```
Call it tb.v

We need generate the vcd for testing, as you can see in the testbench que generate counter.vcd file where we dump all the test benchs's toggling information.

We "run" the testbench with Icarus Verilog running the following:
```
> iverilog counter.sv tb.v
```

This generate one outputs file call a.out which is a file that need to be run to generate the counter.vcd file. this a.out is a file that use vvp library to be compiled. \
```
> ./a.out
```
Now we are able to see the waveform in GTKWave.
```
> gtkwave counter.vcd
```

This pop up a window as following:

![image](https://github.com/user-attachments/assets/4448775c-8196-48b6-a119-c9122e22c187)

We need add the signals waveforms, this is add clock, rst and count signals. we add in the following way:

![image](https://github.com/user-attachments/assets/5ecb889e-f006-4b5b-a9d6-0bba5aca47c5)

Once added you neet fit the waveforms clicking in the following icon:

![image](https://github.com/user-attachments/assets/c6baa0aa-f7be-4489-9c75-a377f5b0dd76)

And that is ower first rtl to simulation trial!!


