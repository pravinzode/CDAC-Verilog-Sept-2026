
# Moore FSM Based ALU Controller

A simple **Moore Finite State Machine (FSM)** designed to control an ALU through a sequence of operations.

---

## 1. Objective

Design a Moore FSM-based controller that performs the following sequence:

```text
IDLE
  ↓
LOAD_A
  ↓
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
  ↓
IDLE
```

The FSM generates control signals for an ALU and associated registers.

---

## 2. Moore FSM Concept

In a **Moore FSM**, the outputs depend only on the present state.

```text
Output = f(Present State)
```

The next state depends on:

```text
Next State = f(Present State, Input)
```

For this example:

```text
Input:
    start

Outputs:
    load_A
    load_B
    add
    sub
    and_op
    or_op
    done
```

---

## 3. Block Diagram

```text
                    +-------------------+
                    |                   |
              +---->|   Next-State      |
              |     |     Logic         |
              |     +---------+---------+
              |               |
              |               v
          +---+-------------------------+
          |        State Register       |
          |         (3 Flip-Flops)      |
          +---+-------------------------+
              |
              | Present State
              |
       +------+----------------+
       |                       |
       v                       v
+--------------+       +---------------+
| State Decode |       | Output Logic  |
+--------------+       +-------+-------+
                               |
                               v
                    +----------------------+
                    | ALU Control Signals  |
                    +----------------------+
```

---

## 4. Inputs

| Signal  | Width | Description                       |
| ------- | ----: | --------------------------------- |
| `clk`   |     1 | System clock                      |
| `reset` |     1 | Asynchronous reset                |
| `start` |     1 | Starts the ALU operation sequence |

---

## 5. Outputs

| Signal   | Width | Description              |
| -------- | ----: | ------------------------ |
| `load_A` |     1 | Load operand A           |
| `load_B` |     1 | Load operand B           |
| `add`    |     1 | Select ALU addition      |
| `sub`    |     1 | Select ALU subtraction   |
| `and_op` |     1 | Select ALU AND operation |
| `or_op`  |     1 | Select ALU OR operation  |
| `done`   |     1 | Indicates completion     |

---

# 6. FSM States

The controller has eight states.

| State    | Description           |
| -------- | --------------------- |
| `IDLE`   | Wait for `start`      |
| `LOAD_A` | Load operand A        |
| `LOAD_B` | Load operand B        |
| `ADD`    | Perform addition      |
| `SUB`    | Perform subtraction   |
| `AND_OP` | Perform AND operation |
| `OR_OP`  | Perform OR operation  |
| `DONE`   | Indicate completion   |

---

# 7. State Diagram

```text
                         start = 1
                    +------------------+
                    |                  |
                    v                  |
                 +-------+             |
                 | IDLE  |-------------+
                 +---+---+
                     |
                     |
                     v
                 +-------+
                 |LOAD_A |
                 +---+---+
                     |
                     v
                 +-------+
                 |LOAD_B |
                 +---+---+
                     |
                     v
                 +-------+
                 |  ADD  |
                 +---+---+
                     |
                     v
                 +-------+
                 |  SUB  |
                 +---+---+
                     |
                     v
                 +-------+
                 | AND_OP|
                 +---+---+
                     |
                     v
                 +-------+
                 | OR_OP |
                 +---+---+
                     |
                     v
                 +-------+
                 | DONE  |
                 +---+---+
                     |
                     v
                   IDLE
```

### IDLE State Transition

```text
start = 0  → IDLE
start = 1  → LOAD_A
```

All other states automatically proceed to the next state on the next clock edge.

---

# 8. State Transition Table

| Present State | Input `start` | Next State |
| ------------- | ------------: | ---------- |
| `IDLE`        |             0 | `IDLE`     |
| `IDLE`        |             1 | `LOAD_A`   |
| `LOAD_A`      |             X | `LOAD_B`   |
| `LOAD_B`      |             X | `ADD`      |
| `ADD`         |             X | `SUB`      |
| `SUB`         |             X | `AND_OP`   |
| `AND_OP`      |             X | `OR_OP`    |
| `OR_OP`       |             X | `DONE`     |
| `DONE`        |             X | `IDLE`     |

`X` = Don't care.

---

# 9. Moore Output Table

Since this is a Moore FSM, outputs depend **only on the present state**.

| State    | load_A | load_B | add | sub | and_op | or_op | done |
| -------- | -----: | -----: | --: | --: | -----: | ----: | ---: |
| `IDLE`   |      0 |      0 |   0 |   0 |      0 |     0 |    0 |
| `LOAD_A` |      1 |      0 |   0 |   0 |      0 |     0 |    0 |
| `LOAD_B` |      0 |      1 |   0 |   0 |      0 |     0 |    0 |
| `ADD`    |      0 |      0 |   1 |   0 |      0 |     0 |    0 |
| `SUB`    |      0 |      0 |   0 |   1 |      0 |     0 |    0 |
| `AND_OP` |      0 |      0 |   0 |   0 |      1 |     0 |    0 |
| `OR_OP`  |      0 |      0 |   0 |   0 |      0 |     1 |    0 |
| `DONE`   |      0 |      0 |   0 |   0 |      0 |     0 |    1 |

---

# 10. State Encoding

There are 8 states.

Therefore:

```text
2^3 = 8
```

Three flip-flops are required.

| State    | Binary Encoding |
| -------- | --------------- |
| `IDLE`   | `000`           |
| `LOAD_A` | `001`           |
| `LOAD_B` | `010`           |
| `ADD`    | `011`           |
| `SUB`    | `100`           |
| `AND_OP` | `101`           |
| `OR_OP`  | `110`           |
| `DONE`   | `111`           |

---

# 11. FSM Design Equations

### Next-State Logic

```text
Next State = f(Present State, start)
```

### Moore Output Logic

```text
Output = f(Present State)
```

### State Register

```text
if reset:
    State = IDLE
else:
    State = Next State
```

---

# 12. Verilog Implementation

## `alu_controller_moore.v`

```verilog
`timescale 1ns/1ps

module alu_controller_moore (
    input  wire clk,
    input  wire reset,
    input  wire start,

    output reg load_A,
    output reg load_B,
    output reg add,
    output reg sub,
    output reg and_op,
    output reg or_op,
    output reg done
);

    //================================================
    // State Encoding
    //================================================

    parameter IDLE   = 3'b000;
    parameter LOAD_A = 3'b001;
    parameter LOAD_B = 3'b010;
    parameter ADD    = 3'b011;
    parameter SUB    = 3'b100;
    parameter AND_OP = 3'b101;
    parameter OR_OP  = 3'b110;
    parameter DONE   = 3'b111;

    reg [2:0] state;
    reg [2:0] next_state;

    //================================================
    // State Register
    //================================================

    always @(posedge clk or posedge reset)
    begin
        if (reset)
            state <= IDLE;
        else
            state <= next_state;
    end

    //================================================
    // Next-State Logic
    //================================================

    always @(*)
    begin

        case (state)

            IDLE:
                if (start)
                    next_state = LOAD_A;
                else
                    next_state = IDLE;

            LOAD_A:
                next_state = LOAD_B;

            LOAD_B:
                next_state = ADD;

            ADD:
                next_state = SUB;

            SUB:
                next_state = AND_OP;

            AND_OP:
                next_state = OR_OP;

            OR_OP:
                next_state = DONE;

            DONE:
                next_state = IDLE;

            default:
                next_state = IDLE;

        endcase

    end

    //================================================
    // Moore Output Logic
    //================================================

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

            LOAD_A:
                load_A = 1'b1;

            LOAD_B:
                load_B = 1'b1;

            ADD:
                add = 1'b1;

            SUB:
                sub = 1'b1;

            AND_OP:
                and_op = 1'b1;

            OR_OP:
                or_op = 1'b1;

            DONE:
                done = 1'b1;

            default:
            begin
                // Outputs remain 0
            end

        endcase

    end

endmodule
```

---

# 13. Testbench

## `tb_alu_controller_moore.v`

```verilog
`timescale 1ns/1ps

module tb_alu_controller_moore;

    reg clk;
    reg reset;
    reg start;

    wire load_A;
    wire load_B;
    wire add;
    wire sub;
    wire and_op;
    wire or_op;
    wire done;

    //================================================
    // DUT Instantiation
    //================================================

    alu_controller_moore DUT (
        .clk(clk),
        .reset(reset),
        .start(start),

        .load_A(load_A),
        .load_B(load_B),
        .add(add),
        .sub(sub),
        .and_op(and_op),
        .or_op(or_op),
        .done(done)
    );

    //================================================
    // Clock Generation
    //================================================

    always #5 clk = ~clk;

    //================================================
    // Test Sequence
    //================================================

    initial
    begin

        clk   = 1'b0;
        reset = 1'b1;
        start = 1'b0;

        // Apply reset
        #10;
        reset = 1'b0;

        // Start operation
        #10;
        start = 1'b1;

        #10;
        start = 1'b0;

        // Wait for FSM to complete
        #100;

        $finish;

    end

    //================================================
    // Monitor
    //================================================

    initial
    begin

        $monitor(
            "Time=%0t | State=%b | Start=%b | "
            "LA=%b LB=%b ADD=%b SUB=%b AND=%b OR=%b DONE=%b",
            $time,
            DUT.state,
            start,
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

# 14. Expected FSM Sequence

After `start = 1`, the FSM moves through:

```text
        Clock
          |
          v

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
        ADD
         |
         v
        SUB
         |
         v
       AND_OP
         |
         v
       OR_OP
         |
         v
        DONE
         |
         v
       IDLE
```

---

# 15. Expected Output Sequence

| Clock | State    | Output     |
| ----: | -------- | ---------- |
|     0 | `IDLE`   | None       |
|     1 | `LOAD_A` | `load_A=1` |
|     2 | `LOAD_B` | `load_B=1` |
|     3 | `ADD`    | `add=1`    |
|     4 | `SUB`    | `sub=1`    |
|     5 | `AND_OP` | `and_op=1` |
|     6 | `OR_OP`  | `or_op=1`  |
|     7 | `DONE`   | `done=1`   |
|     8 | `IDLE`   | None       |

---

# 16. Expected Waveform

```text
Clock Edge     0      1      2      3      4      5      6      7      8
               |      |      |      |      |      |      |      |      |

State         IDLE   LOAD_A LOAD_B ADD    SUB    AND    OR     DONE   IDLE

start          0       1      0      0      0      0      0      0      0

load_A         0       1      0      0      0      0      0      0      0

load_B         0       0      1      0      0      0      0      0      0

add            0       0      0      1      0      0      0      0      0

sub            0       0      0      0      1      0      0      0      0

and_op         0       0      0      0      0      1      0      0      0

or_op          0       0      0      0      0      0      1      0      0

done           0       0      0      0      0      0      0      1      0
```

---

# 17. Important Moore FSM Observation

Notice that:

```text
LOAD_A → load_A = 1
LOAD_B → load_B = 1
ADD    → add = 1
SUB    → sub = 1
AND    → and_op = 1
OR     → or_op = 1
DONE   → done = 1
```

The output is determined by the **state**.

```

This is the fundamental pattern students should learn before moving to **Mealy FSMs**.
