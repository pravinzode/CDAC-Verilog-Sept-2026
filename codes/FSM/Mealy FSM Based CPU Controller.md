# Mealy FSM Based ALU Controller

## 1. Objective

Design and implement an **ALU Controller using a Mealy Finite State Machine (FSM)**.

The controller accepts a `start` signal and a 2-bit `opcode`. Based on the current state and opcode, it generates control signals for:

* Loading operand A
* Loading operand B
* Addition
* Subtraction
* AND operation
* OR operation
* Completion indication

The important characteristic of this design is:

> **The ALU operation output depends on both the present state and the input opcode.**

Therefore, the ALU operation control is implemented using **Mealy FSM logic**.

---

# 2. What is a Mealy FSM?

In a Mealy FSM:

```text
Next State = f(Present State, Input)

Output = f(Present State, Input)
```

For comparison:

### Moore FSM

```text
Output = f(Present State)
```

### Mealy FSM

```text
Output = f(Present State, Input)
```

In this ALU controller, the operation control signals depend on:

```text
Present State + Opcode
```

For example:

```text
EXECUTE + 00 → ADD
EXECUTE + 01 → SUB
EXECUTE + 10 → AND
EXECUTE + 11 → OR
```

Thus, the ALU operation outputs are **Mealy outputs**.

---

# 3. ALU Operations

The controller supports four ALU operations.

| Opcode  | Operation |
| ------- | --------- |
| `2'b00` | ADD       |
| `2'b01` | SUB       |
| `2'b10` | AND       |
| `2'b11` | OR        |

The controller does not perform the actual arithmetic or logic operation.

Instead, it generates **control signals** for the ALU datapath.

---

# 4. Inputs and Outputs

## Inputs

| Signal   | Width | Description             |
| -------- | ----: | ----------------------- |
| `clk`    |     1 | Clock                   |
| `reset`  |     1 | Active-high reset       |
| `start`  |     1 | Starts an ALU operation |
| `opcode` |     2 | Selects ALU operation   |

## Outputs

| Signal   | Width | Description         |
| -------- | ----: | ------------------- |
| `load_A` |     1 | Load operand A      |
| `load_B` |     1 | Load operand B      |
| `add`    |     1 | Perform addition    |
| `sub`    |     1 | Perform subtraction |
| `and_op` |     1 | Perform AND         |
| `or_op`  |     1 | Perform OR          |
| `done`   |     1 | Operation completed |

---

# 5. Overall Architecture

The controller can be considered part of a larger ALU system:

```text
                    +------------------+
 start ------------>|                  |
 opcode[1:0] ------>|  ALU Controller |
 clk -------------->|    Mealy FSM    |
 reset ------------>|                  |
                    +--------+---------+
                             |
             +---------------+----------------+
             |       Control Signals          |
             |                                |
        load_A  load_B  add  sub  AND  OR done
             |                                |
             v                                v
      +---------------------------------------------+
      |                 ALU Datapath                |
      |                                             |
      |        Register A    Register B             |
      |             |             |                 |
      |             +------+------+                 |
      |                    |                        |
      |                   ALU                       |
      +---------------------------------------------+
```

The FSM controls the datapath.

---

# 6. FSM States

We use five states.

| State     | Description                       |
| --------- | --------------------------------- |
| `IDLE`    | Wait for `start`                  |
| `LOAD_A`  | Load operand A                    |
| `LOAD_B`  | Load operand B                    |
| `EXECUTE` | Select ALU operation using opcode |
| `DONE`    | Indicate completion               |

State sequence:

```text
IDLE
  |
  | start = 1
  v
LOAD_A
  |
  v
LOAD_B
  |
  v
EXECUTE
  |
  | opcode determines output
  v
DONE
  |
  v
IDLE
```

---

# 7. Why EXECUTE is a Mealy State

This is the most important part of this design.

When the FSM reaches `EXECUTE`, the opcode determines which control signal is generated.

```text
                 opcode
                    |
                    v
              +-----------+
              |  EXECUTE  |
              +-----------+
               |   |   |   |
              00  01  10  11
               |   |   |   |
              ADD SUB AND  OR
```

Therefore:

```text
add    = f(EXECUTE, opcode)
sub    = f(EXECUTE, opcode)
and_op = f(EXECUTE, opcode)
or_op  = f(EXECUTE, opcode)
```

This is the key difference from the Moore implementation.

---

# 8. State Diagram

A simplified state diagram is:

```text
                    start=1
             +--------------------+
             |                    |
             v                    |
        +---------+               |
        |  IDLE   |<--------------+
        +----+----+
             |
             | start=1
             v
        +---------+
        | LOAD_A  |
        +----+----+
             |
             v
        +---------+
        | LOAD_B  |
        +----+----+
             |
             v
        +----------------+
        |    EXECUTE     |
        +----------------+
          |    |    |    |
       00/ADD  |    |    |
          | 01/SUB  |    |
          |    | 10/AND   |
          |    |    | 11/OR
          +----+----+----+
                   |
                   v
              +---------+
              |  DONE   |
              +----+----+
                   |
                   v
                 IDLE
```

The transitions from `EXECUTE` are labeled as:

```text
Input / Output
```

which is the standard Mealy FSM representation.

---

# 9. Mealy Transition / Output Table

The complete FSM transition table is:

| Present State | Input Condition | Next State | Mealy Output |
| ------------- | --------------- | ---------- | ------------ |
| IDLE          | `start=0`       | IDLE       | None         |
| IDLE          | `start=1`       | LOAD_A     | None         |
| LOAD_A        | X               | LOAD_B     | `load_A=1`   |
| LOAD_B        | X               | EXECUTE    | `load_B=1`   |
| EXECUTE       | `opcode=00`     | DONE       | `add=1`      |
| EXECUTE       | `opcode=01`     | DONE       | `sub=1`      |
| EXECUTE       | `opcode=10`     | DONE       | `and_op=1`   |
| EXECUTE       | `opcode=11`     | DONE       | `or_op=1`    |
| DONE          | X               | IDLE       | `done=1`     |

Here:

```text
X = Don't care
```

---

# 10. Detailed Mealy Transition Table

The important Mealy portion is:

| Present State | Opcode | Next State | `add` | `sub` | `and_op` | `or_op` |
| ------------- | ------ | ---------- | ----: | ----: | -------: | ------: |
| EXECUTE       | `00`   | DONE       |     1 |     0 |        0 |       0 |
| EXECUTE       | `01`   | DONE       |     0 |     1 |        0 |       0 |
| EXECUTE       | `10`   | DONE       |     0 |     0 |        1 |       0 |
| EXECUTE       | `11`   | DONE       |     0 |     0 |        0 |       1 |

Notice that the output changes according to the **input opcode while the FSM is in EXECUTE**.

This is a Mealy characteristic.

---

# 11. Output Table

The complete control-output behavior is:

| State          | `load_A` | `load_B` | `add` | `sub` | `and_op` | `or_op` | `done` |
| -------------- | -------: | -------: | ----: | ----: | -------: | ------: | -----: |
| IDLE           |        0 |        0 |     0 |     0 |        0 |       0 |      0 |
| LOAD_A         |        1 |        0 |     0 |     0 |        0 |       0 |      0 |
| LOAD_B         |        0 |        1 |     0 |     0 |        0 |       0 |      0 |
| EXECUTE + `00` |        0 |        0 |     1 |     0 |        0 |       0 |      0 |
| EXECUTE + `01` |        0 |        0 |     0 |     1 |        0 |       0 |      0 |
| EXECUTE + `10` |        0 |        0 |     0 |     0 |        1 |       0 |      0 |
| EXECUTE + `11` |        0 |        0 |     0 |     0 |        0 |       1 |      0 |
| DONE           |        0 |        0 |     0 |     0 |        0 |       0 |      1 |

---

# 12. State Encoding

Five states require at least three flip-flops.

We use:

| State   | Encoding |
| ------- | -------- |
| IDLE    | `3'b000` |
| LOAD_A  | `3'b001` |
| LOAD_B  | `3'b010` |
| EXECUTE | `3'b011` |
| DONE    | `3'b100` |

Therefore:

```text
Number of states = 5

Required flip-flops = ceil(log2(5))

                  = 3
```

---

# 13. State Transition Logic

The next-state behavior is:

```text
IDLE:

    start = 0 → IDLE
    start = 1 → LOAD_A


LOAD_A:

    → LOAD_B


LOAD_B:

    → EXECUTE


EXECUTE:

    opcode = 00 → DONE
    opcode = 01 → DONE
    opcode = 10 → DONE
    opcode = 11 → DONE


DONE:

    → IDLE
```

Notice that all four opcode values take the FSM from `EXECUTE` to `DONE`.

The opcode does not need to change the next state because it changes the **operation control output**.

---

# 14. Mealy Output Logic

The ALU operation outputs are generated using:

```text
Current State + Opcode
```

Conceptually:

```text
if state == EXECUTE:

    if opcode == 00
        add = 1

    if opcode == 01
        sub = 1

    if opcode == 10
        and_op = 1

    if opcode == 11
        or_op = 1
```

Therefore:

```text
add    = 1 when state = EXECUTE and opcode = 00

sub    = 1 when state = EXECUTE and opcode = 01

and_op = 1 when state = EXECUTE and opcode = 10

or_op  = 1 when state = EXECUTE and opcode = 11
```

---

# 15. Boolean Equations

The Mealy output equations can be written as:

```text
add = EXECUTE · ~opcode[1] · ~opcode[0]

sub = EXECUTE · ~opcode[1] ·  opcode[0]

and_op = EXECUTE · opcode[1] · ~opcode[0]

or_op = EXECUTE · opcode[1] · opcode[0]
```

Thus, the ALU operation outputs depend on both:

```text
Present State
```

and

```text
Opcode
```

---

# 16. Complete Verilog RTL

The following Verilog code implements the Mealy FSM.

```verilog
`timescale 1ns/1ps

module alu_controller_mealy (

    input  wire       clk,
    input  wire       reset,
    input  wire       start,
    input  wire [1:0] opcode,

    output reg        load_A,
    output reg        load_B,
    output reg        add,
    output reg        sub,
    output reg        and_op,
    output reg        or_op,
    output reg        done
);

    //====================================================
    // State Encoding
    //====================================================

    parameter IDLE    = 3'b000;
    parameter LOAD_A  = 3'b001;
    parameter LOAD_B  = 3'b010;
    parameter EXECUTE = 3'b011;
    parameter DONE    = 3'b100;

    //====================================================
    // Opcode Encoding
    //====================================================

    parameter OP_ADD = 2'b00;
    parameter OP_SUB = 2'b01;
    parameter OP_AND = 2'b10;
    parameter OP_OR  = 2'b11;

    //====================================================
    // State Registers
    //====================================================

    reg [2:0] state;
    reg [2:0] next_state;

    //====================================================
    // State Register
    //====================================================

    always @(posedge clk or posedge reset)
    begin

        if (reset)
            state <= IDLE;

        else
            state <= next_state;

    end

    //====================================================
    // Next-State Logic
    //====================================================

    always @(*)
    begin

        // Default state
        next_state = IDLE;

        case (state)

            IDLE:
            begin
                if (start)
                    next_state = LOAD_A;
                else
                    next_state = IDLE;
            end

            LOAD_A:
            begin
                next_state = LOAD_B;
            end

            LOAD_B:
            begin
                next_state = EXECUTE;
            end

            EXECUTE:
            begin
                case (opcode)

                    OP_ADD:
                        next_state = DONE;

                    OP_SUB:
                        next_state = DONE;

                    OP_AND:
                        next_state = DONE;

                    OP_OR:
                        next_state = DONE;

                    default:
                        next_state = DONE;

                endcase
            end

            DONE:
            begin
                next_state = IDLE;
            end

            default:
            begin
                next_state = IDLE;
            end

        endcase

    end

    //====================================================
    // Mealy Output Logic
    //====================================================

    always @(*)
    begin

        // Default outputs
        load_A = 1'b0;
        load_B = 1'b0;

        add    = 1'b0;
        sub    = 1'b0;
        and_op = 1'b0;
        or_op  = 1'b0;

        done   = 1'b0;

        case (state)

            //============================================
            // LOAD A
            //============================================

            LOAD_A:
            begin
                load_A = 1'b1;
            end

            //============================================
            // LOAD B
            //============================================

            LOAD_B:
            begin
                load_B = 1'b1;
            end

            //============================================
            // EXECUTE
            // Mealy output depends on opcode
            //============================================

            EXECUTE:
            begin

                case (opcode)

                    OP_ADD:
                        add = 1'b1;

                    OP_SUB:
                        sub = 1'b1;

                    OP_AND:
                        and_op = 1'b1;

                    OP_OR:
                        or_op = 1'b1;

                    default:
                    begin
                        add    = 1'b0;
                        sub    = 1'b0;
                        and_op = 1'b0;
                        or_op  = 1'b0;
                    end

                endcase

            end

            //============================================
            // DONE
            //============================================

            DONE:
            begin
                done = 1'b1;
            end

            default:
            begin
                // All outputs remain zero
            end

        endcase

    end

endmodule
```

---

# 17. Understanding the RTL

The design contains three important blocks.

## Block 1: State Register

```verilog
always @(posedge clk or posedge reset)
```

This block stores the present state.

```text
             +----------------+
             | State Register |
             +----------------+
                     |
                     v
                Present State
```

---

## Block 2: Next-State Logic

```verilog
always @(*)
```

This block determines:

```text
Next State = f(Present State, start, opcode)
```

For example:

```verilog
IDLE:
    if (start)
        next_state = LOAD_A;
```

and:

```verilog
LOAD_B:
    next_state = EXECUTE;
```

---

## Block 3: Mealy Output Logic

The output block contains:

```verilog
case (state)

    EXECUTE:
    begin

        case (opcode)

            OP_ADD:
                add = 1'b1;

            OP_SUB:
                sub = 1'b1;

            OP_AND:
                and_op = 1'b1;

            OP_OR:
                or_op = 1'b1;

        endcase

    end

endcase
```

This is the most important portion of the design.

The output is a function of:

```text
state + opcode
```

Therefore, it is a Mealy implementation.

---

# 18. Complete Testbench

The following testbench tests all four ALU operations.

```verilog
`timescale 1ns/1ps

module tb_alu_controller_mealy;

    //====================================================
    // Testbench Signals
    //====================================================

    reg       clk;
    reg       reset;
    reg       start;
    reg [1:0] opcode;

    wire load_A;
    wire load_B;
    wire add;
    wire sub;
    wire and_op;
    wire or_op;
    wire done;

    //====================================================
    // Instantiate DUT
    //====================================================

    alu_controller_mealy DUT (

        .clk     (clk),
        .reset   (reset),
        .start   (start),
        .opcode  (opcode),

        .load_A  (load_A),
        .load_B  (load_B),

        .add     (add),
        .sub     (sub),
        .and_op  (and_op),
        .or_op   (or_op),

        .done    (done)

    );

    //====================================================
    // Clock Generation
    //====================================================

    always #5 clk = ~clk;

    //====================================================
    // Task to Test an ALU Operation
    //====================================================

    task test_operation;

        input [1:0] op;

        begin

            opcode = op;

            // Start operation
            start = 1'b1;

            #10;

            start = 1'b0;

            // Wait for FSM operation
            #40;

        end

    endtask

    //====================================================
    // Test Sequence
    //====================================================

    initial
    begin

        // Initialize signals
        clk    = 1'b0;
        reset  = 1'b1;
        start  = 1'b0;
        opcode = 2'b00;

        // Reset
        #10;

        reset = 1'b0;

        //================================================
        // ADD
        //================================================

        $display("------------------------------------");
        $display("Testing ADD");
        $display("------------------------------------");

        test_operation(2'b00);

        //================================================
        // SUB
        //================================================

        $display("------------------------------------");
        $display("Testing SUB");
        $display("------------------------------------");

        test_operation(2'b01);

        //================================================
        // AND
        //================================================

        $display("------------------------------------");
        $display("Testing AND");
        $display("------------------------------------");

        test_operation(2'b10);

        //================================================
        // OR
        //================================================

        $display("------------------------------------");
        $display("Testing OR");
        $display("------------------------------------");

        test_operation(2'b11);

        //================================================
        // Finish Simulation
        //================================================

        #20;

        $finish;

    end

    //====================================================
    // Monitor
    //====================================================

    initial
    begin

        $monitor(
            "Time=%0t | State=%b | Start=%b | Opcode=%b | LA=%b LB=%b ADD=%b SUB=%b AND=%b OR=%b DONE=%b",
            $time,
            DUT.state,
            start,
            opcode,
            load_A,
            load_B,
            add,
            sub,
            and_op,
            or_op,
            done
        );

    end

endmodule
```

---

# 19. Expected State Sequence

For every ALU operation, the FSM follows:

```text
IDLE
  ↓
LOAD_A
  ↓
LOAD_B
  ↓
EXECUTE
  ↓
DONE
  ↓
IDLE
```

The difference is what happens in `EXECUTE`.

---

# 20. ADD Operation

For:

```text
opcode = 00
```

the sequence is:

```text
IDLE
   |
   | start=1
   v
LOAD_A
   |
   v
LOAD_B
   |
   v
EXECUTE
   |
   | opcode=00 / add=1
   v
DONE
   |
   v
IDLE
```

During `EXECUTE`:

```text
add = 1
```

All other ALU operation outputs are zero.

---

# 21. SUB Operation

For:

```text
opcode = 01
```

the sequence is:

```text
IDLE
   ↓
LOAD_A
   ↓
LOAD_B
   ↓
EXECUTE
   |
   | opcode=01 / sub=1
   v
DONE
   ↓
IDLE
```

During `EXECUTE`:

```text
sub = 1
```

---

# 22. AND Operation

For:

```text
opcode = 10
```

the sequence is:

```text
IDLE
   ↓
LOAD_A
   ↓
LOAD_B
   ↓
EXECUTE
   |
   | opcode=10 / and_op=1
   v
DONE
   ↓
IDLE
```

During `EXECUTE`:

```text
and_op = 1
```

---

# 23. OR Operation

For:

```text
opcode = 11
```

the sequence is:

```text
IDLE
   ↓
LOAD_A
   ↓
LOAD_B
   ↓
EXECUTE
   |
   | opcode=11 / or_op=1
   v
DONE
   ↓
IDLE
```

During `EXECUTE`:

```text
or_op = 1
```

---

# 24. Example Timing Sequence

Assume:

```text
Clock period = 10 ns
```

and:

```text
opcode = 00
```

for ADD.

The FSM operation is approximately:

| Clock | State   | Output     |
| ----: | ------- | ---------- |
|     0 | IDLE    | All 0      |
|     1 | LOAD_A  | `load_A=1` |
|     2 | LOAD_B  | `load_B=1` |
|     3 | EXECUTE | `add=1`    |
|     4 | DONE    | `done=1`   |
|     5 | IDLE    | All 0      |

For subtraction:

```text
EXECUTE + opcode=01
```

produces:

```text
sub = 1
```

For AND:

```text
EXECUTE + opcode=10
```

produces:

```text
and_op = 1
```

For OR:

```text
EXECUTE + opcode=11
```

produces:

```text
or_op = 1
```

---

# 25. Why This is a Mealy FSM

The critical equations are:

```text
add    = f(state, opcode)

sub    = f(state, opcode)

and_op = f(state, opcode)

or_op  = f(state, opcode)
```

For example:

```text
add = 1
```

only when:

```text
state = EXECUTE
```

and:

```text
opcode = 00
```

Therefore:

```text
Output = f(Present State, Input)
```

---

# 26. Moore vs Mealy Comparison

| Feature                    | Moore ALU Controller  | Mealy ALU Controller    |
| -------------------------- | --------------------- | ----------------------- |
| Output depends on          | State                 | State + Input           |
| Operation selection        | State-based           | Opcode-based            |
| Number of operation states | More                  | Fewer                   |
| Output changes             | With state transition | Can change with input   |
| Design                     | Simpler timing        | More responsive         |
| Output states              | ADD, SUB, AND, OR     | One EXECUTE state       |
| Main equation              | `Output=f(State)`     | `Output=f(State,Input)` |

A Moore implementation could require:

```text
ADD
SUB
AND
OR
```

as separate states.

The Mealy implementation can use:

```text
EXECUTE
```

and let the opcode select the operation.

---

# 27. Moore Implementation

A Moore implementation would look conceptually like:

```text
LOAD_B
   ↓
ADD
   ↓
SUB
   ↓
AND
   ↓
OR
   ↓
DONE
```

Each operation has a separate state.

For example:

```text
ADD state → add=1
SUB state → sub=1
AND state → and_op=1
OR state  → or_op=1
```

The output depends only on the state.

---

# 28. Mealy Implementation

The Mealy implementation instead uses:

```text
LOAD_B
   ↓
EXECUTE
```

and inside EXECUTE:

```text
opcode=00 → add=1
opcode=01 → sub=1
opcode=10 → and_op=1
opcode=11 → or_op=1
```

Therefore, one `EXECUTE` state replaces four operation states.

---

# 29. Important Design Observation

The following is **not** a proper Mealy operation selection:

```verilog
case (state)

    ADD:
        add = 1;

    SUB:
        sub = 1;

    AND_OP:
        and_op = 1;

    OR_OP:
        or_op = 1;

endcase
```

because the output depends only on:

```text
state
```

This is Moore-style output logic.

For the Mealy design, use:

```verilog
case (state)

    EXECUTE:
    begin

        case (opcode)

            OP_ADD:
                add = 1;

            OP_SUB:
                sub = 1;

            OP_AND:
                and_op = 1;

            OP_OR:
                or_op = 1;

        endcase

    end

endcase
```

Now:

```text
Output = f(State, Opcode)
```

which is Mealy.

---

# 30. Synthesis Considerations

The RTL is synthesizable Verilog.

The FSM will synthesize into:

```text
             +----------------+
             | State Flip-Flops|
             +--------+-------+
                      |
                      v
             +----------------+
             | Next-State     |
             | Combinational  |
             | Logic          |
             +----------------+
                      |
                      v
                  D inputs
```

The Mealy output logic will synthesize into combinational logic driven by:

```text
State
+
Opcode
```

Conceptually:

```text
             State
               |
               +--------+
               |        |
               v        v
           +----------------+
Opcode --->| Mealy Output   |----> add
           | Logic          |----> sub
           |                |----> and_op
           +----------------+----> or_op
```

---


```

**This is the defining feature of the Mealy FSM-based ALU controller.**

