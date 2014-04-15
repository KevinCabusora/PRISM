PRISM
=====

#### Lab 4, Kevin Cabusora, Dr. Neebel, ECE281, 15 April 2014

# DESIGN

## Discussion of ALU Modifications

![alt text][Schematic.PNG]

[Schematic.PNG]:  https://github.com/KevinCabusora/PRISM/blob/master/Schematic.PNG?raw=true "Schematic.PNG"

The goal with this shell was to construct a multiplexer which chose between the eight ALU operations:  AND, NEG, NOT, ROR, OR, IN, ADD, and LD.

The first step was to see which functions were being defined in this shell, and how to relate that in terms of using the VHDL syntax.  There were four signals in the ALU:  OpSel, Accumulator and Data being the inputs, and Result being the output.  In order to define the operations in terms of VHDL code, I would use if, then, and elsif statements to define each function.  For example, for the AND function, if OpSel="000" then I would declare Result as <= Data AND Accumulator.

[ALU code](ALU_shell.vhd)

![alt text][ALU_signal_declarations.PNG]

[ALU_signal_declarations.PNG]:  https://github.com/KevinCabusora/PRISM/blob/master/ALU_signal_declarations.PNG?raw=true "ALU_signal_declarations.PNG"

For AND, I AND-ed Data and Accumulator.  

For NEG, my goal was to take the 2's complement of the value in the accumulator.

For NOT, it was a simply NOT-ing the Accumulator.

For ROR, the goal was to rotate the bits of the accumulator to the right, and to do this, I took the unsigned std logic vector value, then a ROR function of 1 of the accumulator.

For OR, a simple OR function was implemented with Data and the Accumulator.

IN took in the Data, and therefore the result was Data.  The same went for LD.

ADD took the unisgned std logic vector values of the Data and Accumulator and added them together that way.

## ALU Test and Debug

[ALU testbench](ALU_testbench.vhd)

![alt text][ALU_testbench.PNG]

[ALU_testbench.PNG]:  https://github.com/KevinCabusora/PRISM/blob/master/ALU_testbench.PNG?raw=true "ALU_testbench.PNG"

I checked my syntax with the shell, and then used the ALU_testbench.vhd file.  The ADD output looked strange on the testbench, so I went back to check what was wrong with the original shell.  Apparently I did not account for the fact that doing Result <= Data + Accumulator was not enough.  Therefore, I made them unsigned in std logic vector.  My testbench then worked beautifully afterwards.

## Discussion of Datapath Modifications

[Datapath](Datapath_shell.vhd)

Probably the most difficult section in the lab for me was to understand what was going on in the Datapath shell.  The first step was to make a declaration for my ALU as a component.

Next, in the Internal Signals section in lines 86-87, the signals MARHi, MARLo, Accumulator, ALU_Result and PC were given their length.  I noticed that all the signals besides PC were 4-bit, with PC being 8 bit.  I then realized that this corresponded with the PRISM diagram in Lab 4 and in the PRISM manual.  This PRISM diagram would prove very useful in coding up the datapath.

The Program Counter was already previously created, and this along with the diagram was very useful in understanding what was happening.  When the variable Reset_L='0', then PC <= "00000000" which essentially would initialize the Program Counter.  In the next lines, if there was a rising edge, and PCLd and JmpSel both equalled '1', then bits 7 down to 4 of the PC would be defined as MARHi, and 3 down to 0 were defined as MARLo.  Basically, 1 in the multiplexer above the PC would take in the inputs of MARHi and MARLo.  We defined 7 downto 4 for MARHi just because we could; it was arbitrary.

Next, I built the Instruction Register, defined as IR in the shell.  Again, if Reset_L='0', then IR <= "0000", initializing the IR.  Similar to the prior signal, a rising edge would activate the IR, but only if MARHiLd='1' as well.  The result would be that IR<=Data.

The Memory Access Register was also run on the same principle, in that both a rising edge and the respective load functions must be met, in order for the Data to be stored in the MARHi or MARLo.

Next, I made the Address Selector, which basically was the multiplexer that picked between MARHi and MARLo.  In line 155, the signals were AddrSel, MARHi, MARLo, and PC, and as shown from before, the PC took in MARHi as 7 downto 4 and MARLo as 3 downto 0, so the Address Bus would take in this data in a similar way.  This process would happen if AddrSel='1', and else, the Addr would basically contain the PC.

Next, I connected the ALU using the arithmator value.  I defined OpSel as OpSel, Data as Data and so on, except that I connected the ALU code with the datapath by stating that Result => ALU_Result.

I then implemented the Accumulator.  It used the same syntax as the IR and MAR codes, in that Reset_L='0' would initialize the accumulator, then a rising edge and a load function would get the accumulator to perform a result.  In this case, it was Accumulator => ALU_Result.

Next, I made a tri-state buffer which placed the Accumulator data on the Data Bus when enabled, and then went to High Z for the rest of the time ('Z').  To do this, I used the Accumulator when the EnAccBuffer (Enable Accumulator Buffer) would = '1', and else it would = Z.

I then defined the AlessZero and AeqZero functions, which were all self-explanatory.  AeqZero would be enabled and defined as '1' whenver Accumulator would = "0000".  If not, it would be defined as '0'.  AlessZero would be defined as Accumulator(3).

## Datapath Test and Debug

![alt text][Datapath_testbench.png]

[Datapath_testbench.png]:  https://github.com/KevinCabusora/PRISM/blob/master/Datapath_testbench.PNG?raw=true "Datapath_testbench.png"

[Testbench](Datapath_testbench.vhd)

I used the attached testbench to test my design.  However, there appeared to be syntax errors in my code.  It only happened to be mere confusion between using apostrophes and quotes (ex. "0000" as opposed to '0000').  After that, I verified that my code was correct.

## Discussion of Testbench Operation

The testbench represented the cycle of functions of the PRISM, and it represented the schematic and process of the Datapath and ALU portions and their outputs.

# REVERSE ENGINEERING

## Simulation Analysis

### 50-100 ns

![alt text][50-100ns.PNG]

[50-100ns.PNG]:  https://github.com/KevinCabusora/PRISM/blob/master/50-100ns.PNG?raw=true "50-100ns.PNG"

Below is the excerpt of my simulation.

First, the Controller gets the instruction 3 from Data, which translates to the 
the ROR function (Rotate)

It is then stored into the  memory.

In the next cycle the command is stored in the Information Register.

The next cycle then rotates the bits in the Accumulator once.

The next instruction is 4 which corresponds with OUT, which then outputs it to Output Port 3.

### 225 ns Jump

At 225 ns is a rising edge.  At 226 ns, data reads 7, which is the JMP function.

Aeqzero has a value of 0 but Alesszero has a value of 1.  Therefore the value in the accumulator is negative.

# PRISM

Here is the PRISM program implemented into PRISM.

![alt text][PRISM Implementation.PNG]

[PRISM Implementation.PNG]:  https://github.com/KevinCabusora/PRISM/blob/master/PRISM%20Implementation.PNG?raw=true "PRISM Implementation.PNG"

# Documentation

I received help from C3C Erik Thompson in understanding why my ADD function would not work, and he suggested to make them unsigned.  









