# **Architecture & Microcode Reference Manual**

---

## **1. Overview**
This document defines the instruction set architecture (ISA) for the microprocessor. Each instruction executes in 4 instruction steps, where:

- **Steps 0–1** are the *universal fetch cycle*  
- **Steps 2–3** implement the instruction’s behavior  
- After Step 1, the **Step Register (SR)** wraps to 0 for the next instruction

The microcode ROM is addressed by:

```
[ IR(9:0) | SR(1:0) ]
```

The ROM outputs a 10‑bit control word driving the datapath.

---

## **2. Control Signals**
| Bit | Name        | Function |
|----:|-------------|----------|
| 9   | `use_pc`    | Select PC as address bus source (1) or MAR (0) |
| 8   | `load_mar`  | Latch data bus into MAR |
| 7   | `arith`     | ALU add/negate enable |
| 6   | `invert`    | ALU operand invert |
| 5   | `pass`      | ALU passthrough mode |
| 4   | `load_accum`| Latch ALU output into accumulator |
| 3   | `load_bus`  | Drive accumulator onto data bus |
| 2   | `read`      | Read from RAM into data bus |
| 1   | `write`     | Write data bus to RAM |
| 0   | `load_ir`   | Latch data bus into IR |

---

## **3. Universal Fetch Cycle**
Every instruction begins with the same two micro‑steps.

### **Step 0 — Fetch instruction word**
```
use_pc=1, load_mar=0, arith=0, invert=0, pass=1, load_accum=0, load_bus=0, read=1, write=0, load_ir=0
```
- PC drives address bus  
- RAM word appears on data bus  

### **Step 1 — Load IR and increment PC**
```
load_ir=1 (all other signals 0)
```
- IR ← data bus  
- PC ← PC + 1  
- SR wraps to 0 for next instruction  

---

## **4. Instruction Set**

Each instruction defines **Step 2** and **Step 3** only.  
Steps 0–1 are always the fetch cycle.

---

### **No Operation**
**Opcode:** `NOP`  

**Description:** Does nothing for two micro‑steps.

**Step 2 & 3:**  
```
(all signals 0)
```

---

### **Load from RAM**
**Opcode:** `LOAD address`  

**Step 2 — Capture address operand**
```
use_pc=0, load_mar=1, arith=0, invert=0, pass=1, load_accum=0, load_bus=0, read=0, write=0, load_ir=0
```

**Step 3 — Read RAM → ACC**
```
use_pc=0, load_mar=0, arith=0, invert=0, pass=1, load_accum=1, load_bus=0, read=1, write=0, load_ir=0
```

---

### **Store accumulator to RAM**
**Opcode:** `STORE address`  

**Step 2 — Capture address operand**
```
use_pc=0, load_mar=1, arith=0, invert=0, pass=0, load_accum=0, load_bus=0, read=0, write=0, load_ir=0
```

**Step 3 — Write ACC → RAM**
```
use_pc=0, load_mar=0, arith=0, invert=0, pass=0, load_accum=0, load_bus=1, read=0, write=1, load_ir=0
```

---

### **Load immediate literal**
**Opcode:** `LOADI`  

**Step 2 — Load next word directly**
```
use_pc=0, load_mar=0, arith=0, invert=0, pass=1, load_accum=1, load_bus=0, read=0, write=0, load_ir=0
```

**Step 3:**  
```
(all signals 0)
```

---

### **Add RAM value to accumulator**
**Opcode:** `ADD address`  

**Step 2 — Capture address**
```
use_pc=0, load_mar=1, arith=0, invert=0, pass=0, load_accum=0, load_bus=0, read=0, write=0, load_ir=0
```

**Step 3 — ACC ← ACC + RAM**
```
use_pc=0, load_mar=0, arith=1, invert=0, pass=0, load_accum=1, load_bus=0, read=1, write=0, load_ir=0
```

---

### **Subtract RAM value**
**Opcode:** `SUB address`  

**Step 2 — Capture address**
```
use_pc=0, load_mar=1, arith=0, invert=0, pass=0, load_accum=0, load_bus=0, read=0, write=0, load_ir=0
```

**Step 3 — ACC ← ACC − RAM**
```
use_pc=0, load_mar=0, arith=1, invert=1, pass=0, load_accum=1, load_bus=0, read=1, write=0, load_ir=0
```

---

### **Bitwise AND**
**Opcode:** `AND address`  

**Step 2 — Capture address**
```
use_pc=0, load_mar=1, arith=0, invert=0, pass=0, load_accum=0, load_bus=0, read=0, write=0, load_ir=0
```

**Step 3 — ACC ← ACC AND RAM**
```
use_pc=0, load_mar=0, arith=0, invert=0, pass=0, load_accum=1, load_bus=0, read=1, write=0, load_ir=0
```

---

### **One’s complement (NOT)**
**Opcode:** `NOT`  

**Step 2 — Invert accumulator**
```
use_pc=0, load_mar=0, arith=0, invert=1, pass=1, load_accum=1, load_bus=1, read=0, write=0, load_ir=0
```

**Step 3:**
```
(all signals 0)
```

---

### **Two’s complement**
**Opcode:** `NEG`  

**Step 2 — Invert**
```
use_pc=0, load_mar=0, arith=1, invert=1, pass=0, load_accum=0, load_bus=1, read=0, write=0, load_ir=0
```

**Step 3 — Write back**
```
use_pc=0, load_mar=0, arith=0, invert=0, pass=0, load_accum=1, load_bus=0, read=0, write=0, load_ir=0
```

---

### **Alias of LOAD/STORE**
**Opcode:** `MOV src,dst`  

- `MOV ACC → RAM` = `STORE`
- `MOV RAM → ACC` = `LOAD`

No new microcode required.

---

### **Unconditional jump**
**Opcode:** `JMP addr`  

**Step 2 — Capture target address**
```
use_pc=0, load_mar=1, arith=0, invert=0, pass=0, load_accum=0, load_bus=0, read=0, write=0, load_ir=0
```

**Step 3 — Redirect PC on next fetch**
```
(all signals 0)
```

---

### **Conditional branches** 
**Opcode:** `JZ / JNZ`  

Architecture currently lacks zero flags. WIP

---

### **Read external input**
**Opcode:** `IN`  

**Step 2 — Load data_in**
```
use_pc=0, load_mar=0, arith=0, invert=0, pass=1, load_accum=1, load_bus=0, read=0, write=0, load_ir=0
```

**Step 3:**  
```
(all signals 0)
```

---

### **Drive accumulator to external bus**
**Opcode:** `OUT`  

**Step 2 — Drive ACC onto bus**
```
use_pc=0, load_mar=0, arith=0, invert=0, pass=1, load_accum=0, load_bus=1, read=0, write=0, load_ir=0
```

**Step 3:**  
```
(all signals 0)
```
