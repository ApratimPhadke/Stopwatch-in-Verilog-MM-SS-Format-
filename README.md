# Stopwatch in Verilog (MM\:SS Format)

## 📖 Overview

This project implements a **digital stopwatch** in Verilog that displays time in **minutes and seconds (MM\:SS)**. The design uses a **clock divider**, **seconds counter**, and **minutes counter** to achieve accurate time tracking. It is fully synthesizable and verified using a testbench.

---

## 🧩 Design Approach

The stopwatch logic is broken down into **modular blocks**:

1. **Clock Divider**: Converts the fast FPGA clock (e.g., 50 MHz) into a 1 Hz signal.
2. **Seconds Counter**: Counts from 0 to 59.
3. **Minutes Counter**: Counts from 0 to 59, increments when seconds roll over.

### Block Diagram
<img width="1536" height="1024" alt="ChatGPT Image Aug 19, 2025, 03_50_11 PM" src="https://github.com/user-attachments/assets/fb0d63a5-1567-4265-97cc-6bf976873ac1" />


```
System Clock ---> Clock Divider ---> 1Hz Pulse ---> Seconds Counter ---> Minutes Counter
```

---

## ⚡ Stopwatch Verilog Code

```verilog
`timescale 1ns/1ps
`default_nettype none

module stopwatch #(
    parameter CLK_FREQ = 50_000_000  // system clock frequency
)(
    input  wire clk,
    input  wire rst,
    output reg  [5:0] sec,
    output reg  [5:0] min
);

    // Clock divider
    localparam integer DIVIDER = CLK_FREQ - 1;
    reg [$clog2(CLK_FREQ)-1:0] count;
    reg one_hz;

    always @(posedge clk) begin
        if (rst) begin
            count <= 0;
            one_hz <= 0;
        end else if (count == DIVIDER) begin
            count <= 0;
            one_hz <= 1;
        end else begin
            count <= count + 1;
            one_hz <= 0;
        end
    end

    // Stopwatch counter
    always @(posedge clk) begin
        if (rst) begin
            sec <= 0;
            min <= 0;
        end else if (one_hz) begin
            if (sec == 59) begin
                sec <= 0;
                if (min == 59)
                    min <= 0;
                else
                    min <= min + 1;
            end else begin
                sec <= sec + 1;
            end
        end
    end

endmodule
```

---

## 🧪 Testbench Code

The testbench simulates the stopwatch with a **scaled-down clock divider** (`CLK_FREQ = 10`) to quickly observe results.

```verilog
`timescale 1ns/1ps
`default_nettype none

module tb_stopwatch;

    reg clk;
    reg rst;
    wire [5:0] sec;
    wire [5:0] min;

    // Instantiate DUT with small divider
    stopwatch #(.CLK_FREQ(10)) dut (
        .clk(clk),
        .rst(rst),
        .sec(sec),
        .min(min)
    );

    // Clock generation: 100 MHz (10 ns period)
    always #5 clk = ~clk;

    initial begin
        clk = 0;
        rst = 1;
        #20;
        rst = 0;

        #500;
        $finish;
    end

    initial begin
        $monitor("Time=%t | MIN=%0d SEC=%0d", $time, min, sec);
    end

endmodule
```

---

## 📊 Simulation Results

### Waveform Output

The stopwatch was simulated in Vivado. Below is the waveform showing seconds incrementing:

<img width="1854" height="1168" alt="Screenshot from 2025-08-19 15-44-42" src="https://github.com/user-attachments/assets/79840712-8ef4-46a4-813e-828169b358bb" />


### Synthesized RTL Schematic (Full)

<img width="1387" height="894" alt="Screenshot from 2025-08-19 15-38-06" src="https://github.com/user-attachments/assets/dd36bba0-169f-4c75-b6e6-dd3e028288c8" />


### Synthesized RTL Schematic (Simplified)

<img width="1387" height="894" alt="Screenshot from 2025-08-19 15-40-53" src="https://github.com/user-attachments/assets/4978203d-78ac-487a-8548-a8ef53459cfc" />


---

## ✅ Key Learnings

* **Think in hierarchy**: Break design into clock divider, sec counter, min counter.
* **Synchronous design**: Use clock edges properly.
* **Simulation first**: Scale down divider for fast testbench runs.
* **Reusable code**: Parameterize clock frequency for portability.

---

## 🔮 Next Steps

* Add **start/stop** functionality with button debounce.
* Display `MM:SS` on a **7-segment display** or **UART output**.
* Extend to **HH\:MM\:SS** for longer duration timing.

---

## 👨‍💻 Author

**Apratim Phadke**
Electronics & Telecommunication Engineer
Verilog Projects Collection
