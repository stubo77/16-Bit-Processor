\# High‑Level Processor Architecture



This document describes the structure and signal flow of the high‑level processor. The system is composed of three main components:



\- \*\*Brainless Processor\*\*  

\- \*\*Address Generator\*\*  

\- \*\*Controller\*\*



The processor also receives three \*\*global inputs\*\*:



\- \*\*clock\*\*  

\- \*\*reset\*\*  

\- \*\*in\*\* 



All components receive \*\*clock\*\* and \*\*reset\*\*, while \*\*in\*\* is exclusive to the brainless processor.



\---



\## 1. Brainless Processor



\### \*\*Inputs\*\*



| Input Signal | Bits | Source | Description |

|--------------| ---- | --------| ----------- |

| in | 16 | Global input | Data input from the user |

| clock | 1 | Global input | Clock signal from clock module or controlled directly by user |

| clear | 1 | Global reset | Clears the entire processor to defaults |

| address\_in | 16 | Address generator | Controls which address in RAM we intend to manipulate |

| arith | 1 | Controller | If high, sets the \*\*ALU\*\* to arithmetic mode |

| invert | 1 | Controller | If high, sets the \*\*ALU\*\* to inversion mode |

| pass | 1 | Controller | If high, sets the \*\*ALU\*\* to passthrough mode |

| load\_accum | 1 | Controller | If high, loads the \*\*ALU\*\* output the \*\*Accumulator\*\* |

| load\_bus | 1 | Controller | If high, chooses the \*\*Accumulator\*\* value to send to the \*\*data bus\*\*, otherwise chooses \*\*data\_in\*\*/\*\*RAM\*\* (see read control) |

| write | 1 | Controller | If high, sets the \*\*RAM\*\* to write mode |

| read | 1 | Controller | If high, chooses \*\*RAM\*\* value to send to \*\*data bus\*\*, otherwise chooses \*\*data\_in\* |



\### \*\*Output\*\*



\- \*\*data\_out\*\* — Drives the \*\*main data bus\*\* which supplies both the \*\*address generator\*\* and the \*\*controller\*\*



\### \*\*Function\*\*



The \*\*Brainless Processor\*\* is the very core of the full \*\*microprocessor\*\*. If one knows what they're doing, it is possible to manually control the functionality of the \*\*brainless processor\*\* without the help of the \*\*controller\*\* and \*\*address generator\*\*. The \*\*brainless processor\*\* is composed of five main components:



\- \*\*Arithmetic Logic Unit (ALU)\*\*

\- \*\*Accumulator\*\* (16 bit register)

\- \*\*programRam\*\*

\- two sixteen bit multiplexers for signal flow



\#### a. Arithmetic Logic Unit



The \*\*Arithmetic Logic Unit (ALU)\*\* is responsible for performing all arithmetic and logical operations within the processor. It is composed of two combinational sub‑units: \*\*not‑neg\*\* and \*\*and‑add\*\*. These units operate sequentially, with the output of not‑neg feeding into and‑add, collectively producing the final \*\*ALU\*\* result.



\##### \*\*not‑neg:\*\*



The not‑neg unit accepts three inputs:



\- in — the 16‑bit operand

\- invert — control signal selecting logical inversion

\- neg — control signal selecting arithmetic negation



Based on these control signals, the unit performs one of three operations:



\- Pass‑through of the input operand

\- One’s complement of the operand

\- Two’s complement of the operand



The functional behavior is summarized below:

| invert | neg | Function |

| ------ | --- | -------- |

| 0 | 0 | Signal passes through |

| 0 | 1 | Signal passes through |

| 1 | 0 | 1's complement |

| 1 | 1 | 2's complement |



\##### \*\*and-add:\*\*



The and‑add unit accepts five inputs:



\- in\_a — primary 16‑bit operand

\- in\_b — secondary 16‑bit operand

\- c\_in — carry‑in bit

\- add — control signal selecting addition

\- pass — control signal selecting pass‑through



This unit performs one of the following operations:



\- Bitwise AND of in\_a and in\_b

\- Addition of in\_a and in\_b

\- Pass‑through of in\_a (ignoring in\_b)



The functional behavior is summarized below:

| pass | add | Function |

| ---- | --- | -------- |

| 0 | 0 | AND |

| 0 | 1 | ADD |

| 1 | 0 | in\_a passes through |

| 1 | 1 | in\_a passes through |



\#### b. Accumulator



The \*\*accumulator\*\* is a dedicated 16‑bit register used to store intermediate values for \*\*ALU\*\* operations. Because the processor’s \*\*main data bus\*\* can carry only one 16‑bit value at a time, the \*\*accumulator\*\* provides a second operand path for arithmetic operations requiring two inputs (e.g., addition, subtraction). The \*\*accumulator’s\*\* input is exclusively sourced from the \*\*ALU\*\* output. Consequently, loading a value from the \*\*data bus\*\* into the \*\*accumulator\*\* requires routing the bus value through the \*\*ALU’s\*\* pass‑through path.



\#### c. Program RAM



The \*\*Program RAM\*\* (hereafter \*\*RAM\*\*) serves as the primary working memory for the processor. It contains four input signals and one output signal:



\- \*\*addr\*\* — 16‑bit address specifying the memory location to access

\- \*\*data\_in\*\* — 16‑bit input value supplied by the \*\*main data bus\*\*

\- \*\*write\*\* — control signal enabling memory write operations

\- \*\*clk\*\* — clock signal used to latch writes

\- \*\*data\_out\*\* — 16‑bit output containing the value stored at \*\*addr\*\*



Memory reads occur at the rising edge of \*\*clk\*\*: \*\*data\_out\*\* always reflects the contents of the last accessed addressed location. Memory writes also occur on the rising edge of \*\*clk\*\* when \*\*write\*\* is asserted, at which point \*\*data\_in\*\* is stored at the selected address.



\#### c. Multiplexers



Two multiplexers coordinate the routing of signals onto the processor’s \*\*main data bus\*\*.



1\. RAM/Data Input Selector  

This multiplexer selects between:



\- the \*\*RAM\*\* output (\*\*data\_out\*\*) when read is asserted

\- the external \*\*data\_in\*\* signal when read is deasserted



Its output feeds the second multiplexer.



2\. Accumulator/Data Bus Selector  

This multiplexer selects between:



\- the \*\*accumulator\*\* value, when \*\*load\_bus\*\* is asserted

\- the output of the previous multiplexer, when \*\*load\_bus\*\* is deasserted



Together, these multiplexers determine which internal component drives the data bus at any given time, ensuring correct operand flow throughout the processor.



\---



\## 2. Controller



\### \*\*Inputs\*\*



| Input Signal | Bits | Source | Description |

|--------------| ---- | --------| ----------- |

| clock | 1 | Global input | Clock signal from clock module or controlled directly by user |

| clear | 1 | Global reset | Clears the entire processor to defaults |

| data\_in | 16 | Main data bus | Input to the \*\*instruction register\*\*

| load\_ir | 1 | Feedback from control bit 0 | If high, allows input to the \*\*instruction register\*\*



\### \*\*Output\*\*



\- \*\*control\*\* — A 10‑bit microcontrol word mapped as such:



| Bit | Destination | Function |

|-----|-------------|----------|

| 9 | Address Generator | use\_pc |

| 8 | Address Generator | load\_mar |

| 7 | Brainless Processor | arith |

| 6 | Brainless Processor | invert |

| 5 | Brainless Processor | pass |

| 4 | Brainless Processor | load\_accum |

| 3 | Brainless Processor | load\_bus |

| 2 | Brainless Processor | read |

| 1 | Brainless Processor | write |

| 0 | Controller (feedback) | load\_ir |



\### \*\*Function\*\*



The \*\*Controller\*\* is holds the main instructions for the processor. As discussed before, it is \*possible\* to control the \*\*brainless processor\*\* without the help of the \*\*controller\*\* and \*\*address generator\*\*, but it is not advised. The \*\*controller\*\* holds a predetermined set of instructions which the user can easily switch between. The \*\*controller\*\* is very simple containing only two registers and a \*\*microcode ROM\*\*. The \*\*ROM\*\* contains the predetermined instruction control signals, and when an address is given to the \*\*ROM\*\* it outputs a control signal which controls inputs of various components. The \*\*controller\*\* uses two registers to form the \*\*ROM\*\* address: the \*\*instruction register (IR)\*\* a 16‑bit register that encodes one of 65,536 possible instructions and the \*\*step register (SR)\*\* a 2‑bit register that encodes the current micro‑step of the instruction, allowing up to 4 micro‑operations per instruction. These two registers are concatenated to form an 18‑bit microcode address:



ROM Address = IR\[15:0] ∥ SR\[1:0]



This produces a total address space of: 262,144 microinstruction slots. Although the \*\*ROM\*\* contains 262,144 addressable microinstructions, they are conceptually grouped into 65,536 instruction blocks, each block containing 4 sequential micro‑steps. Thus, the architecture defines 65,536 distinct instructions, each implemented as a sequence of up to four microinstructions. 





\---



\## 3. Address Generator



\### \*\*Inputs\*\*



| Input Signal | Bits | Source | Description |

|--------------| ---- | --------| ----------- |

| clock | 1 | Global input | Clock signal from clock module or controlled directly by user |

| clear | 1 | Global reset | Clears the entire processor to defaults |

| data\_in | 16 | Main data bus | Input to the \*\*Memory Address Register (MAR)\*\* |

| use\_pc | 1 | Controller (control bit 9) | If high, propagates \*\*Project Counter\*\* to \*Address Bus\*\*, otherwise propagates \*\*MAR\*\* to \*\*Address Bus\*\* |

| load\_mar | 1 | Controller (control bit 8) | Loads \*\*data\_in\*\* to \*\*MAR\*\* |



\### \*\*Output\*\*



\- \*\*addr\_out\*\* — Drives the \*\*address bus\*\* into the brainless processor



\### \*\*Function\*\*



The \*\*address generator\*\* is responsible for producing the 16‑bit memory address used by the processor when accessing \*\*Program RAM\*\*. It consists of three primary components:



\- \*\*Program Counter (PC)\* — a 16‑bit register that increments automatically on each \*\*clk\*\* cycle when enabled.

\- \*\*Memory Address Register (MAR)\*\* — a 16‑bit register that can be explicitly loaded from the \*\*data bus\*\*.

\- \*\*Address Select Multiplexer\*\* — a 16‑bit multiplexer that selects either the \*\*PC\*\* or the \*\*MAR\*\* as the active memory address source.



Together, these components allow the processor to access memory both sequentially and non‑sequentially. When the \*\*PC\*\* is selected, the memory address automatically increments after each instruction fetch or operand read. This enables linear program execution without requiring explicit address manipulation. When the \*\*MAR\*\* is selected, the processor may read from or write to any memory location. This is used for data access, indirect addressing, and control‑flow instructions that modify the program’s execution path.



