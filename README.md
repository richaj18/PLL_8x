
![](https://github.com/richaj18/PLL_8x/blob/main/vsd%20workshop%20logo.jpg)

# PLL 8x Clock Multiplier

2 Day Workshop on PLL Design using Google SkyWater 130nm Technology<br>
The spice simulations were done using the Ngspice Open source EDA.<br>
The lab sessions carried out for Layout using the Ngspice EDA Tool.


<h2> Contents: </h2>

1. [Phase Locked Loop - Theory](https://github.com/richaj18/PLL_8x/blob/main/README.md#-Phase-Locked-Loop-)
2. [Tool Setup](https://github.com/richaj18/PLL_8x/blob/main/README.md#-Tool-Setup-)
3. [Development Flow](https://github.com/richaj18/PLL_8x/blob/main/README.md#-Development-Flow-)
4. [Specifications](https://github.com/richaj18/PLL_8x/blob/main/README.md#-Specifications-)
5. [Pre Layout Simulations](https://github.com/richaj18/PLL_8x/blob/main/README.md#-Pre-Layout-Simulations-)
6. [Layout](https://github.com/richaj18/PLL_8x/blob/main/README.md#-Layout-)
7. [Parasitic Extractions](https://github.com/richaj18/PLL_8x/blob/main/README.md#-Parasitic-Extractions-)
8. [Post layout Simulations](https://github.com/richaj18/PLL_8x/blob/main/README.md#-Post-Layout-Simulations-)
9. [GDS and Tapeout](https://github.com/richaj18/PLL_8x/blob/main/README.md#-GDS-and-Tapeout-)
10. [References](https://github.com/richaj18/PLL_8x/blob/main/README.md#-References-)
11. [Acknowledgements](https://github.com/richaj18/PLL_8x/blob/main/README.md#-Acknowledgements-)
<br>

<h2> Phase Locked Loop - Theory </h2>

The SOC and other ICs requires the clock signal to perform certain specific functions,<br>
and thus we need to generate the clock signal of a particular frequency without any frequency <br>
or phase noise.<br>
This clock signal can be generated by the two methods: <br>
<b> a. Voltage Controlled Oscillator </b> <br>
       It has good flexibility and on-chip implementation but it has noise in phase <br>
<b> b. Quartz Crystal Oscillator </b> <br>
       It has purity in spectrum which means it does not have any unwanted frequency but it is not flexible like VCO <br>

Thus we need to design a PLL having a pure spectral as well as flexibility.<br>

PLL is a control system which mimics the reference signal having the same frequency <br>
and a constant phase difference between the reference signal and output signal generated. <br>

![](https://github.com/richaj18/PLL_8x/blob/main/1.png)

Futher dwelling into each part of this control system 

![](https://github.com/richaj18/PLL_8x/blob/main/PLL%20loop.png)

<h3> Phase Frequency detector(PFD) </h3>

There are two inputs to it - the reference signal from the Quartz crystal and the output signal from the VCO. <br>
It detects the phase difference between the two. <br>

For this we can use the XOR gate but it is not an optimum solution as it is not able to detect the difference at nπ, <br>
also it gets locked into the harmonics of the refernce signal.<br>

Thus, we use a sequential circuit for the same using the D flip flop. <br>
Below is the lagging signal ie when the output signal from the VCO is delayed from the reference signal. <br>

![](https://github.com/richaj18/PLL_8x/blob/main/lagging.PNG)

But if the output signal is leading the reference signal, thus we require another D flip flop <br>
to detect this leading signal caleed as the DOWN signal. <br>

![](https://github.com/richaj18/PLL_8x/blob/main/leading.PNG)

As we can see from the above diagrams, we see the following observations : <br>
1. When the falling edge of the reference signal arrives first, the Output is High, and then falling edge of output 
   signal arrives it gets cleared. <br>
   This makes the UP signal. <br>

2. When the falling edge of the output signal arrives first, the output becomes High, and then falling edge of reference 
   signal arrives it gets cleared. <br>
   This is the DOWN signal. <br>

Thus making the State Machine for this is as follows : <br>

![](https://github.com/richaj18/PLL_8x/blob/main/state%20machine.png)

Since there are three states, we require two flip flops for the same, for one flip flop reference signal is the clock, <br>
for the other flip flop the output signal is the clock. The output of both are connected to the AND gate which goes to the CLR
such that when both are 1 the flip flop is reset. <br>

![](https://github.com/richaj18/PLL_8x/blob/main/d%20flip%20flop.PNG)

But the disadvantage of this is the Dead Zone. When the phase difference between the refernce signal and output signal is <br>
very less, then due to the delay of gates it is not able to detect the difference and might be skipped. <br>

<h3> Charge Pump </h3>

PFD generates the digital signal, but the input to the VCO is an analogue signal, thus Charge Pump converts the <br>
output of the PFD into analogue signal. This can be done through "Current Steering" circuit ie. by charging <br>
and discharging of the capacitor. But why only the capacitor is used and not a resistive load, this is because we <br>
are interested in the avarage time of [UP-DOWN] which is achieved by observing the voltage of capacitor. <br>

![](https://github.com/richaj18/PLL_8x/blob/main/current%20steering.png)

The output voltage may have fluctuations due to the rise and fall times of UP and DOWN, and thus to have a smooth transition
we have a loop filter.

<h3> Loop Filter </h3>

Only one capacitor at the output the Charge pump makes the system unstable because if we see the frequency domain analysis of
the circuit then there will be two poles at in the transfer function which makes it oscillating anf highly unstable.
Thus a RC LPF is added at the output such that output gets stabilize.

![](https://github.com/richaj18/PLL_8x/blob/main/loop%20filter.png) 

The values of R1 and Cx are selected such that the output signal(input to the VCO) is stable, <br>
<b> Cx = C/10 </b>
Also, we want the output signal not to fluctuating, so
<b> Loop BW = fref/10 </b>

<h3> Voltage Controlled Oscillator and Frequency Divider </h3>

Till now we have achieved two things - accurancy and spectral purity. Now for flexibility Current Starved is used in VCO.
VCO is a combination of odd number of inverters and in particular the ring oscillator.<br>
Current sources are used at the top and the bottom with the Vctrl voltage to control the ring oscillator.

![](https://github.com/richaj18/PLL_8x/blob/main/vco%20current%20starved.PNG)

The frequency Divider is as follows:

![](https://github.com/richaj18/PLL_8x/blob/main/frequency%20divider.PNG)
<br>

<h2> Tool Setup </h2>

<h3> NgSpice Tool </h3>
NgSpice directly simulates the circuit(.cir) file given and plots the results according to the specifications mentioned in the
circuit file.
To execute the circuite file, on the CMD window type : 
<b> <directory_name_where_files_are_present> <file_name>.cir </b>

<h3> Magic Tool </h3>
Magic is used for designing the layout file, write the GDS file for fabrication and also to extracrt the parisitics.
To execute the file, on the CMD window type :
<b> <directory> magic -T <technology_file_name_from_PDK> <layout_file> </b>

In this project, we have used sky130A.tech for the 130nm node technology from Google Skywater library.
<br>

<h2> Development Flow </h2>

1. Specifications for the design are initiated
2. Circuit is being made to meet the specifications in pre-layout 
3. Developing the layout 
4. Extraction of the parastics ie capacitive effect due to width, area etc
5. Spice Netlist for the Layout
6. Post Layput Simulation 
7. GDS file and Tapeout
<br>

<h2> Specifications </h2>

1. Corner - TT(typical-typical)
   This shows the outcome of the doping process
   When the doping of the transistor is done, the exact concentartion is varied which makes transistor fast or slow

2. Supply Voltage - 1.8Volts
3. Room Temperature

These three makes <b> PVT(Process Voltage Temperature corner) </b>

4. Input frequency
   fmin = 5MHz
   fmax = 12.5 MHz

5. N = 8 (Multiplier)
6. Jitter(RMS) <~ 20nsec
7. Duty Cycle = 50%
<br>

<h2> Pre-Layout Simulations </h2>

This shows the theoretical or ideal simulations

<h3> Frequency Divider </h3>
The circuit for frequency divider was seen above using the D flip flop, and at the transistor level this can be 
observed as follows : 

![](https://github.com/richaj18/PLL_8x/blob/main/frequency%20divider%20circuit.PNG)

Firstly cloning the transistor modules needed for simulation from the primitive libraries of sky130.lib
The files needed <b>nfet_1v8</b> and <b>pfet_1v8</b> and <b> "tt.pm3.spice" </b>
Copying these files in a seperate folder <b> spice_lib </b>
Generating the <b> sky130.lib </b> by using the commands : 

![](https://github.com/richaj18/PLL_8x/blob/main/sky130_lib%20cmd.PNG)

Now writing for the circuit in fd.cir file

the transistor in sky130 is modelled as sub circuit which is written as follows :

<b> xm1 d g s b sky130_fd_pr_pfet_1v8 length width </b>

<b>x</b> : the sub circuit
<b>m</b> : mosfet

The code for frequency divider is shown below : 

![](https://github.com/richaj18/PLL_8x/blob/main/fd%20code.PNG)

Running the code :

![](https://github.com/richaj18/PLL_8x/blob/main/FD%20terminal%20window%201.PNG)

Simulation result :

<b> Frequency Divider </b>
![](https://github.com/richaj18/PLL_8x/blob/main/FD%20plot.PNG)
 <b> Blue : </b> Input Signal<br>
 <b> Red : </b> Output Signal

Similarly pre layout simulations were carried out for all the circuits of the PLL.

<h3> Charge Pump </h3>

<b> With UP and DOWN both 0 </b>
![](https://github.com/richaj18/PLL_8x/blob/main/CP%20plot%20with%20up%20and%20down%200.PNG)
<br>

<b> With 'UP' Sigmal</b>
![](https://github.com/richaj18/PLL_8x/blob/main/CP%20plot%20with%20up%201.PNG)

<h3> Phase Frequency detector </h3>

<b> With Leading VCO signal</b>
![](https://github.com/richaj18/PLL_8x/blob/main/PFD%20plot%20with%20down%201.PNG)
<b> Blue : </b> Reference Signal<br>
<b> Red : </b> VCO Signal<br>
<b> Orange : </b> UP Signal<br>
<b> Green: </b> DOWN Signal<br>

<h3> Voltage Controlled Oscillator </h3>

![](https://github.com/richaj18/PLL_8x/blob/main/VCO%20plot.PNG)

<h3> Phase Locked Loop </h3>

![](https://github.com/richaj18/PLL_8x/blob/main/Plot%20for%20pll_pre.PNG)<br>
<b> Purple : </b> Charge Pump Output<br>
<b> Pink : </b> VCO Frequency by 2<br>
<b> Green : </b> VCO Frequency by 4<br>
<b> Orange : </b> VCO Frequency by 8<br>
<b> Red : </b> Reference Frequency<br>
<b> Yellow : </b> DOWN Signal<br>
<b> Maroon : </b> UP Signal<br>
<br>

<h2> Layout </h2>

Opening the layout window, command : <br>
<b> <directory_name> magic -T sky130A.tech </b> <br>
In layout, the following color codes are used : <br>
  1.n-diffusion - plain green<br>
  2.p-diffusion - plain orange<br>
  3.polysilicon - red<br>
  4.metal1 layer - purple<br>
  5.locali - blue<br>

To connect two metal layers, we use m2contact. To connect local interconnect(locali) 
and metal, we use via(viali).

<h3> Charge Pump </h3>

![](https://github.com/richaj18/PLL_8x/blob/main/cp%20layout.PNG)<br>
<b> Area : </b> 132.29 sq units<br>

<h3> Frequency detector </h3>

![](https://github.com/richaj18/PLL_8x/blob/main/fd%20layout.PNG)<br>
<b> Area : </b> 29.92 sq units<br>

<h3> MUX </h3>

![](https://github.com/richaj18/PLL_8x/blob/main/mux%20layout.PNG)<br>
<b> Area : </b> 12.12 sq units<br>

<h3> Phase Frequency Detector </h3>

![](https://github.com/richaj18/PLL_8x/blob/main/pfd%20layout.PNG)<br>
<b> Area : </b> 49.09 sq units<br>


<h3> Voltage Controlled Oscillator </h3>

![](https://github.com/richaj18/PLL_8x/blob/main/vco%20layout.PNG)<br>
<b> Area : </b> 57.73 sq units<br>


<h3> Phase Locked Loop </h3>

![](https://github.com/richaj18/PLL_8x/blob/main/pll%20layout.PNG)<br>
<b> Area : </b> 496.03 sq units<br>
<br>

<h2> Parasitic Extraction </h2>

When the layout has been done, spice file is being generated to know the effects of
wire capacitance and resistenace, their lengths, area etc.

TO select the design, press "i". In the tckon window type : <br>
<b> extract all </b> - this extracts all the information in the .ext file<br>
Now we need to convert this .ext into .spice file
<b> ext2spice cthresh <value> rthresh <value> </b><br>
<b> ext2spice </b><br>
<br>

<h2> Post Layout Simulation </h2>



















































































































































<h2> References </h2>

1. GT, LLC - https://zipcpu.com/dsp/2017/12/14/logic-pll.html
2. Electronics Stackexchange - https://electronics.stackexchange.com/questions/301402/phase-frequency-detector-in-pll