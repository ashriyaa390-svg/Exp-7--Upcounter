# Exp-7--Upcounter
# Design and Verification of a 4-bit Synchronous Up-Counter
## Aim
To design and verify a 4-bit Synchronous Up-Counter using Verilog HDL, and to observe its behaviour using a testbench and VCD waveform.

## Theory / Synopsis
A synchronous up-counter is a sequential circuit whose output increments by one on every active edge of a common clock, all flip-flops being clocked together (hence 'synchronous', as opposed to a ripple counter where each stage clocks the next). Here a 4-bit counter is realized, so the count cycles through 0000 to 1111 (0 to 15 in decimal) and then wraps around (overflows) back to 0000.

An enable input (en) is included so that the counter only increments when en = 1 and simply holds its value when en = 0. An active-high asynchronous reset clears the count to zero irrespective of the clock.

## Functional Description

On every rising edge of clk:

if (rst) count <= 4'b0000;

else if (en) count <= count + 1;   // increments, wraps 1111 -> 0000

else count <= count;                // holds present value

## Design / RTL Program
```
// gedit shift_reg.v
 
module shift_reg (
    input  wire       clk,   // clock
    input  wire       rst,   // asynchronous active-high reset
    input  wire       sin,   // serial input
    output wire       sout,  // serial output (MSB, about to shift out)
    output reg  [3:0] Q      // parallel output
);
    always @(posedge clk or posedge rst) begin
        if (rst)
            Q <= 4'b0000;
        else
            Q <= {Q[2:0], sin};   // shift left, sin enters at LSB
    end
 
    assign sout = Q[3];
endmodule
```


## Testbench Program
```
// gedit tb6.v
 
module tb6;
    reg  clk, rst, sin;
    wire sout;
    wire [3:0] Q;
 
    // Instantiate the shift register
    shift_reg uut (
        .clk(clk),
        .rst(rst),
        .sin(sin),
        .sout(sout),
        .Q(Q)
    );
 
    // ---- Clock generation: 10 time-unit period ----
    initial clk = 0;
    always #5 clk = ~clk;
 
    initial begin
        // ---- VCD dump setup ----
        $dumpfile("shift_reg.vcd");   // name of the VCD file to be generated
        $dumpvars(0, tb6);              // dump all signals in this testbench hierarchy
 
        $monitor("Time=%0t rst=%b clk=%b sin=%b | Q=%b sout=%b",
                  $time, rst, clk, sin, Q, sout);
 
        // ---- Apply asynchronous reset ----
        rst = 1; sin = 0;
        #7  rst = 0;
 
        // ---- Serially shift in the bit pattern 1 0 1 1 ----
        @(negedge clk); sin = 1;
        @(negedge clk); sin = 0;
        @(negedge clk); sin = 1;
        @(negedge clk); sin = 1;
 
        // ---- Hold sin = 0 and observe the pattern shift out ----
        @(negedge clk); sin = 0;
        @(negedge clk);
        @(negedge clk);
 
        // ---- Apply reset again mid-simulation ----
        @(negedge clk); rst = 1;
        @(negedge clk); rst = 0; sin = 1;
 
        #20 $finish;
    end
endmodule
```



## Operation / Truth Table

State transition (en = 1, rst = 0 unless noted)

<img width="406" height="205" alt="image" src="https://github.com/user-attachments/assets/f67141a6-9926-4baa-b4fc-af002e8f4824" />

## Simulation Procedure

STEP 1 – Open Terminal
Open a terminal in the experiment folder. 

STEP 2 – Load Synopsys Environment
source /synopsys/start.sh

STEP 3 – Compile Using VCS
vcs up_counter.v tb.v -full64
If compilation is successful, VCS generates the simulation executable:
simv

STEP 4 – Run Simulation
./simv
The terminal displays the reset, clock, enable and the resulting count value (in binary and decimal) at every change (via $monitor). A VCD waveform file is also generated:
up_counter.vcd

STEP 5 – Open DVE
dve -full64
Other option:
dve -full64 &
A DVE environment will open.

## DVE Waveform Verification

In DVE:

●	Open the testbench hierarchy.

●	Locate the signals: clk, rst, en, count[3:0].

●	Add the signals to the waveform window.

●	Run / inspect the waveform.

●	Verify that count increments by 1 on every rising edge of clk while en = 1 and rst = 0.

●	Verify that count holds its present value whenever en = 0.

●	Confirm that count wraps from 1111 back to 0000 after the 16th enabled clock edge (overflow).

●	Confirm that count resets to 0000 immediately whenever rst = 1, independent of clk.

The waveform should agree with the state transition table.

## Expected Result

The 4-bit Synchronous Up-Counter was realized using Verilog HDL, with the characteristic behaviour:

count(next) = count + 1   (when en = 1), wrapping from 1111 to 0000

The design was compiled and simulated using Synopsys VCS, and the functionality was verified using DVE waveform analysis, matching the expected state transition table.

## Output
<img width="1920" height="1080" alt="Screenshot (18)" src="https://github.com/user-attachments/assets/df169222-f375-4e6f-850b-0faa1bdddaf0" />



## Viva-Voce Questions

●	What is the difference between a synchronous counter and a ripple (asynchronous) counter?

●	What is the modulus of this counter, and how is it related to the number of bits?

●	What happens to the count value on overflow, and why is this called 'wrap-around'?

●	What is the purpose of the enable (en) input, and what would happen if it were removed?

●	How would you modify this design to create a down-counter?

●	How would you modify this design to create an up/down counter with a direction control input?

●	What is the difference between a synchronous and an asynchronous reset, and which one is used here?

●	Why is a VCD file generated, and what does $dumpvars(0, tb) do?

●	How would you extend this design to an 8-bit or a mod-10 (decade) counter?

## Result

Thus, the 4-bit Synchronous Up-Counter was designed, implemented using Verilog HDL, successfully simulated using Synopsys VCS, and verified using Synopsys DVE.

