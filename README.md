# BUILDING A RISC-V CORE - MYTH

This repository contains all the information regarding the 5-day RISC-V based CPU Core Design MYTH (Microprocessor for You in Thirty Hours) Workshop, offered by for VLSI System Design (VSD) and Redwood EDA. In a short span of 5-days, the basic RISC-V ISA was studied & a simple RISC-V core with base instruction set was implemented. The RISC-V CPU Core has been designed with the help of Transaction Level Verilog(TL-Verilog) in addition with the Makerchip IDE Platform. Find below the accompanying details.

# Contents of The workshop

Check the individual day folders for the documentation, source codes and assignments of the respective days of the workshop.


<details>
<summary>DAY 1: Introduction to RISCV ISA and GNU Compiler Toolchain</summary>
<br>

## Introduction to Risc-v Basic Keywords
- **Instruction Set Architecture(ISA)**
  - An Instruction Set Architecture (ISA) refers to the set of instructions that a computer's central processing unit (CPU) can understand and execute. It defines the interface between software and hardware, specifying the operations that a CPU can perform, the data types it can manipulate, and the memory addressing modes it supports.

- **Risc-V ISA**
  - Risc-V ISA is an open-source ISA that has simpler and fixed length instructions that allows us to create custom processors for specific needs without being tied to proprietary architectures
 
- **Tools Used for the flow**
  - As we are aware of the flow, we will be using Risc-v ISA ALP and the RTL used will be picorv32a (We will be using rv64i during initial stages)

# Goal : Any High level Program that is written should be able to get executed in our CHIP

### List of well-known extensions present in Risc-V ISA

``` rv32i``` ``` rv64i``` ```rv32imc``` ```rv64imc``` ```rv32imafdc``` ```rv64imafdc``` ```rv32imcb``` ```rv64imcb``` ```rv32imc_sv32``` ```rv64gcv```

### Extensions and their Applications

- **I (Integer)** :The I set includes the base integer instruction set for RISC-V. It provides fundamental integer arithmetic and logical operations, data movement, and control flow instructions.
  - ADD, SUB, AND, OR, XOR, ADDI, SLTI, JAL, BEQ, LW

- **M (Multiply and Divide)** : The M set adds integer multiplication and division instructions to the base integer set. These instructions are particularly useful for arithmetic-heavy computations.
  - MUL, MULH, DIV, REM
  
- **A (Atomic)** : The A set introduces atomic memory access instructions. These instructions enable multiple operations on memory locations to be performed atomically, ensuring that other processors or threads cannot observe intermediate states.
  - LR (Load-Reserved), SC (Store-Conditional), AMO (Atomic Memory Operation)
  
- **F (Single-Precision Floating-Point)**: The F set adds single-precision floating-point instructions. These instructions enable arithmetic operations on 32-bit floating-point numbers.
  - FADD.S, FSUB.S, FMUL.S, FDIV.S, FCVT.W.S, FCVT.S.W

- **D (Double-Precision Floating-Point)** : The D set includes double-precision floating-point instructions. These instructions allow arithmetic operations on 64-bit floating-point numbers.
  - FADD.D, FSUB.D, FMUL.D, FDIV.D, FCVT.W.D, FCVT.D.W

- **C (Compressed)** : The C set introduces a compressed instruction format that reduces the size of code. Compressed instructions maintain the same functionality as their non-compressed counterparts but use shorter encodings.
  - C.ADDI4SPN, C.LWSP, C.ADDI, C.SW, C.JALR, C.BEQZ

- **G (Atomic and Lock-Free Operations)** : The G set, also known as the "GAS Set," is an alternative to the A set. It focuses on providing atomic and lock-free instructions to simplify hardware implementation.
  - LRV (Load-Reserved Variant), SCV (Store-Conditional Variant), AMO (Atomic Memory Operation Variants)

- **V (Vector)** :The V set adds vector instructions to the ISA, enabling Single Instruction, Multiple Data (SIMD) operations. These instructions allow efficient parallel processing of data elements in vectors.
  - VADD, VMUL, VFMADD, VLW, VSW

- **S (Supervisor)** : The S set, often used in privileged modes, includes instructions for managing and interacting with the supervisor-level operations of the system, such as handling exceptions and interrupts.
  - ECALL, EBREAK, SRET, MRET, WFI

- **B (Bit Manipulation)** : The B set introduces instructions for bit manipulation operations, allowing efficient manipulation of individual bits in registers and memory.
  - ANDI, ORI, XORI, SLLI, SRLI, SRAI

## 1. Create a simple C program That calculates sum from 1 to N -> sum_1_to_N.c

_____Compile it using C compiler_____
```
gcc sum1ton.c -o 1ton.o
./1ton.o
```
-o allows you to name your output file

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/d3fa6df2-8b79-4f0a-9972-88d97193d2df)


_____compile using riscv compiler and view the output_____
```
riscv64-unknown-elf-gcc -O1 -mabi=lp64 -march=lp64 -o sum1ton.o sum1ton.c
spike pk sum1ton.o
```

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/3f56d9e1-c6a3-4008-ae08-50702eb112a3)


- ```-O<number>``` : level of optimisation required
- ```-mabi``` : specifies the ABI (Application Binary Interface) to be used during code generation according to the requirements
- ```-march``` : specifies target architecture

_______We can check the different options available for all these fields using the commands_______ 
go to the directory where riscv64-unkonwn-elf is present
- -O1 : ``` riscv64-unkonwn-elf --help=optimizer```
- -mabi : ```riscv64-unknown-elf-gcc --target-help```
- -march : ```riscv64-unknown-elf-gcc --target-help```

_____To view the disassembled ALP code_____
```
riscv64-unknown-elf-objdump -d 1_to_N.o
```

_____To debug the ALP generated by the compiler_____
```
spike -d pk 1_to_N.o
```
![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/c251cf5e-998b-4b8b-8306-db552ee3e446)


- press ENTER : shows the first line and successive ENTER shows successive lines
- reg 0 a2 : checks content of register a2 0th core
- q : quit the debug process

##### Difference between the ALP commands when used different optimizers
- use the command ```riscv64-unknown-elf-objdump -d sum1ton.o | less```
- use ``` /instance``` to search for an instance 
- press ENTER
- press ```n``` to search next occurance
- press ```N``` to search for previous occurance. 
- use ```esc :q``` to quit

_____Contents of main when used -O1 optimizer_____
![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/7a1ff564-bd03-4c13-ab6b-0811e3b6de07)


## Integer number Representation (n-bit)
- Range of Unsigned numbers : [0, (2^n)-1 ]
* Range of signed numbes : Positive : [0 , 2^(n-1)-1]
                         Negative : [-1 to 2^(n-1)]

## 2. create a C program that shows the maximum and minimum values of 64bit unsigend and signed numbers

```
signed.c
```
![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/15030a1a-d4e1-4029-a502-3b53107790d1)




</details>
<details>
<summary>DAY 2 : Introduction to ABI and Basic Verification Flow </summary>
<br>

## BASICS :

Instructions that act on signed or unsigned integers are called Base Integer Instructions
There are 47 Base Integer Instructions present in RISC-V ISA

### Types of Instruction based on encoding format

1. **R-Type (Register-Type):**
   - These instructions operate on registers and have a fixed format for their operands.
   - Examples: ADD, SUB, AND, OR, XOR, SLL, SRL, SRA, SLT, SLTU

2. **I-Type (Immediate-Type):**
   - These instructions have an immediate operand and one register operand.
   - Examples: ADDI, SLTI, SLTIU, XORI, ORI, ANDI, SLLI, SRLI, SRAI, LB, LH, LW, LBU, LHU, JALR

3. **S-Type (Store-Type):**
   - These instructions are used for storing values from registers to memory.
   - Examples: SB, SH, SW

4. **B-Type (Branch-Type):**
   - These instructions perform conditional branching based on comparisons.
   - Examples: BEQ, BNE, BLT, BGE, BLTU, BGEU

5. **U-Type (Upper Immediate-Type):**
   - These instructions have a larger immediate field for encoding larger constants.
   - Examples: LUI, AUIPC

6. **J-Type (Jump-Type):**
   - These instructions are used for unconditional jumps and function calls.
   - Examples: JAL

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/f9515e48-105d-45d1-85f6-786ec7dbfaa6)


**[number]** represents number of bits occupied by that field

1. **Opcode [7] :** The opcode is a field within a machine language instruction that indicates the operation to be performed by the instruction. It defines the type of operation, such as arithmetic, logic, memory access, or control flow. Opcodes are used by the CPU to determine how to execute the instruction.

2. **rd (Destination Register) [5]:** The "rd" field represents the destination register in an assembly language instruction. It indicates the register where the result of the operation will be stored. After executing the instruction, the computed value will be placed in this register.

3. **rs1 (Source Register 1) [5]:** The "rs1" field represents the first source register in an assembly language instruction. It indicates the register that holds the value used in the operation. For instructions that involve two operands, "rs1" typically corresponds to the first operand.

4. **rs2 (Source Register 2) [5]:** The "rs2" field represents the second source register in an assembly language instruction. It indicates the register that holds the value used in the operation. For instructions that involve three operands, "rs2" typically corresponds to the second operand.

5. **func7 and func3 (Function Fields)[7] [3]:** These fields further refine the operation specified by the opcode. The "func7" field is used to distinguish different variations of instructions within the same opcode category. The "func3" field is used to specify a more specific operation within the opcode category. Together, these fields allow for a finer level of instruction differentiation.

6. **imm (Immediate Value):** The "imm" field represents an immediate value that is part of the instruction. Immediate values are constants that are embedded within the instruction itself. They can be used for various purposes, such as specifying offsets, constants, or small data values directly within the instruction.


#### ABI : Application Binary Interface

The instructions generated by compiler using a target ISA can be accessed by OS and User directly
- The parts of ISA accessible to User : User ISA
- The parts of ISA accessible to OS : system ISA
The access is done using Sysytem calls with the help of ABI

==> If we want to access hardware resources of processor, it has to be done via registers using ABI(names)

### ABI Names : 
- ABI names for registers serve as a standardized way to designate the purpose and usage of specific registers within a software ecosystem. These names play a critical role in maintaining compatibility, optimizing code generation, and facilitating communication between different software components.

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/56834df8-1006-455b-8861-559a495cc1cd)


#### Data can be stored in register by 2 methods
1. Directly store in registers
2. Store into registers from memory

To store 64 bits of data from mem to reg, we use 8*8bit stores ie., m[0],m[1]......m[7]. 

- ___RISC-V uses Little Endian format to store the data ie., Least significant Byte is stored in m[0]___

## Simulate a C program using ABI function call (using registers) and execute 

The required program files are under day 2 folder

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/8bb24df0-b408-4fc3-84e6-c8b4bd317222)


![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/2c2537e3-b292-4440-bfaa-fa9bbfc9d846)


Here we can observe that at 5th line, inorder to comute the result ,its going to the "load"  function

### Further we will see how to run a C program on on RISC-V CPU

- Input : C Program loaded into memory to RISC-V CPU in Hex format

CPU processes the contents of the memory and provides with output using iverilog 

- Risc-V CPU : ```Picorv32.v```
- Testbench for verification : ```testbench.v```
- Tool : ```iverilog```
- script : ```rv32im.sh``` : has the commands to get the c-program, ALP, converts into hex format, loads into memory of riscv cpu, passes it iverilog and provides the output



<details>
<summary>DAY 3 : Digital Logic With TL Verilog in Makerchip IDE</summary>
<br>

# Example 

### Inverter
- Learn -> Examples -> Makerchip Default Template

#### A) Inverter in TLV using command

- under TLV Section type ```$out = ! $in1```
- Now compile ``` press E -> compile```

#### B) Xor gate using operators

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/81e276fb-8089-4d24-99da-0b06faa3a766)

#### C) Vectors

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/82534d5e-9a2f-44ca-b7a4-ae4006cb3529)


#### D) Mux (with and without vectors)

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/b1faddd2-e9dd-412b-9472-370a0336cba9)


#### E) Simple Claculator

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/70b8149a-9eba-4825-a637-5634218559d8)


### Sequential Logic

- Sequential logic is sequenced by a clock signal
- A D-flip-flop transitions next state to current state on a rising clock edge
- Reset signal helps the circuit come to a known state

#### F) Fibonacci series

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/3f33eab4-1d7d-439a-b433-0c48ac33d6c7)


#### G) Up-Counter

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/f6690c3c-ae4e-40d2-b5f1-cae6d37defdd)



#### H) Sequential Calculator 
  Input val1 is the previous output of calculator

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/d7b9a358-b3e4-404a-b405-384083e8d71f)


#### I) A simple pipeline through Pythagorean example

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/a787a5e1-42ff-4457-8233-8bd05c5c4ceb)


#### J) Pipeline Implementation example

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/6cd9684c-1962-4b2e-a2bf-98dbeb28b61f)



## Validity 
- Easier debug
- cleaner design
- Better error checking
- Automated clock gating

#### K) 2 cycle calculator with validity

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/54c23e86-e5a1-46cf-919a-fe6820bdf459)



#### L) Distance Calculator

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/8da13890-a5e7-44ae-aebb-11eb75083279)


#### M) Calulator_memory

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/d675c9a5-1bea-4791-a83a-1e4a254e3584)



</details>

<details>
<summary>DAY 4 : Basic RISC-V CPU Micro Architecture</summary>
<br>

# RISC-V Implementation Blueprint

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/72319701-3467-4be9-8b55-283b1083fb80)


Link for the starter code [starter code](https://myth.makerchip.com/sandbox?code_url=https:%2F%2Fraw.githubusercontent.com%2Fstevehoover%2FRISC-V_MYTH_Workshop%2Fmaster%2Frisc-v_shell.tlv#) 



## 1. Program Counter

- Program Counter is a register that contains the address of the next instruction to be executed. It is a pointer into the instruction memory, for the instruction that we are going to execute next. Since the memory is byte addressable and each instruction length is 32 bits, the Program Counter adder adds 4 bytes to the address to point to the next address.

- For the initial state, before fetching the first ever instruction, there is a presence of a reset signal that will reset the PC value to 0.

- For branch instructions, we will have immediate instructions, for which we have to add an offset value to the PC. So for branch instructions, NextPC = Incremented PC + Offset value.

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/131ac65f-fe10-47cf-a269-564e15206942)


## 2. Instruction Fetch

- Here the instruction memory is added to the program. In the Instruction Fetch logic, the instructions are fetched from the instruction memory amd passed to the Decode logic for computation. 
- The instruction memory read address pointer is computed from the program counter and it outputs a 32 bit instruction. (instr[31:0]) .
- In our case, the Makerchip shell provides us an instantiation to the instruction memory, which contains a test program to compute the sum of numbers from 1 to 9.

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/85df14e2-48ce-42a6-9493-2e88cf01f1fd)


## 3.Instruction Decode

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/0c899320-7bcb-4b0e-9a3f-c66f2a052af5)


## 4. Instruction Decode with validity

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/7ee06827-9290-4f4f-9e2f-b0c11c6055e1)

## 5. Individual Instruction decode

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/1b8af205-2548-4b0a-92d9-8939741d821a)

## 6. Register file read

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/c3e8010b-d423-408a-85e8-273af717fc48)


## 7. ALU

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/f85450c2-5100-4cd2-ad2a-be2a91f5144b)

## 8. Register File Write

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/63cf3823-034e-4a90-9b40-fe0562e3a030)


## 9. Branch Instructions

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/9797b6b6-6c75-41f4-84be-c745864d535f)


## 10. Testbench to check functionality

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/5d606302-52d9-433b-9756-dd95797d2967)


</details>

<details>
<summary>DAY 5 : Complete Pipelined RISC-V Micro Architecture </summary>
<br>

- Pipelining helps improve the operating frequency by breaking down the micro-arch ito substages that consume lesser time.But this process intriduces some hazards and dependencies such as
  - Data Hazards
  - Structural Hazards
  - Control Hazards
  - Name Dependence
  - Anti Dependence
  - Output Dependence

 ## 3-cycle RISC-V

 - A simple pipeline approach where we divide the arch into 3 stages ie., PC, Decode to ALU, Reg write.
 - This requires a valid signal that is generated every 3 clock cylces

### Generation of 3 cycle valid signal

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/56b60e95-17a1-4d8d-a70b-cf4aa5306ebb)


- There might be some invalid cycles (invalid operation when valid is on) being encountered in this proccess. SO we have to take care of them.

### 3-cycle RISC-V To take care of invalid signals

- Avoid writing into register file for invalid operations
- Avoid redirecting PC for nvalid instructions(branch)

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/d1afe8fc-1155-4240-b007-d81a534990f3)


### Introduce 5 stage pipeline 

- 5 stage pipeline : PC, Decode, Reg Rd, ALU, Reg write

## Solutions to Pipeline Hazards

1. Register file bypass

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/57397d0d-6edb-4cec-afd7-eb2da13bb7f3)


2. Correct the branch target path

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/fac3a20e-2ad9-4995-9ff7-1f7fe8a7d50b)


3. Complete Instruction Decode

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/52dbd380-6ee2-4cdc-bc94-8924715a0289)


4. Complete ALU

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/787d9f9c-2c70-4250-9922-fc109e0af60b)


## Completing RISC-V CPU with final touch of Load/Store Instructions

1. Redirecting Loads

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/05eb5137-e266-4277-986d-8ed8de08d9d0)


2. Load Data from Memory to register file

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/508cebd1-925e-4c53-b646-0ea7a86e98ed)


3. Instantiate Data memory to CPU

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/60e3c8fc-be37-455b-9ed8-86fb12d1f4d0)


4. Add loads and stores to test the program

```
m4_asm(SW r0, r10, 100)
m4_asm(LW r15, r0, 100)
```
5. Lab for jump instructions

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/38b9df7f-c15a-4810-803c-add233ea0c43)


</details>

# Code Comparision

The code required for the RISC-V Core written in TL-Verilog and System Verilog can be compared by selecting the "Show Verilog" on the makerchip platform under the "E" tab. Upon visualization, a significant code reduction can be seen in the comparision chart.

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/d4e4b7e6-79e2-416b-b190-ae628c8519f7)



# Final RISC-V core block Diagram

![image](https://github.com/Sbj3333/skanda_riscv/assets/95922889/4d9d1ba2-b9b1-4ba8-8435-ec0f1e87e48e)


