# Design_and_Simulate_the_Memory_Design_using_verilog_HDL
# EXP NO:5. Design-and-Simulate the-Memory-Design-using-Verilog-HDL
# Aim 
To design and simulate a RAM,ROM,FIFO using Verilog HDL, and verify its functionality through a testbench in the Vivado 2023.1 environment. 

# Apparatus Required 
Vivado 2023.1 

# Procedure
1. Launch Vivado 2023.1 Open Vivado and create a new project.
2. Design the Verilog Code Write the Verilog code for the RAM,ROM,FIFO
3. Create the Testbench Write a testbench to simulate the memory behavior. The testbench should apply various and monitor the corresponding output.
4. Create the Verilog Files Create both the design module and the testbench in the Vivado project.
5. Run Simulation Run the behavioral simulation to verify the output.
6. Observe the Waveforms Analyze the output waveforms in the simulation window, and verify that the correct read and write operation.
7. Save and Document Results Capture screenshots of the waveform and save the simulation logs. These will be included in the lab report.

# Code
# RAM
# Verilog code
```
`timescale 1ns/1ps

module ram_16kb (
    input wire clk,
    input wire we,
    input wire [13:0] addr,
    input wire [7:0] din,
    output reg [7:0] dout
);

reg [7:0] mem [0:16383];

always @(posedge clk) begin
    if (we) begin
        mem[addr] <= din;
    end
    else begin
        dout <= mem[addr];
    end
end

endmodule
```
# Test bench
```
`timescale 1ns/1ps

module tb_RAM;

reg clk;
reg we;
reg [13:0] addr;
reg [7:0] data_in;

wire [7:0] data_out;

integer i;

reg [13:0] addr_mem[0:19];
reg [7:0] data_mem[0:19];

ram_16kb uut(
    .clk(clk),
    .we(we),
    .addr(addr),
    .din(data_in),
    .dout(data_out)
);

always #5 clk = ~clk;

initial begin
    clk = 0;
    we = 1;
    addr = 0;
    data_in = 0;

    // Write
    for (i = 0; i < 20; i = i + 1) begin
        @(posedge clk);
        addr_mem[i] = $urandom % 16384;
        data_mem[i] = $urandom;
        
        addr = addr_mem[i];
        data_in = data_mem[i];
        $display("WRITE Address=%0d Data=%h", addr, data_in);
    end

    // Read
    we = 0;
    for (i = 0; i < 20; i = i + 1) begin
        @(posedge clk);
        addr = addr_mem[i];
        
        @(posedge clk);
        #1;
        $display("READ Address=%0d Data=%h", addr, data_out);
    end

    #20;
    $finish;
end

endmodule
```


# Output Waveform
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/e888e24f-3117-49ff-9849-709733a6a298" />


# ROM
# Verilog code for ROM using $random
```
module memory_4KB (
    input clk,
    input we,          // Write Enable
    input re,          // Read Enable
    input [11:0] address, 
    input [7:0] data_in,  
    output reg [7:0] data_out 
);

reg [7:0] memory [0:4095];

always @(posedge clk) begin
    if (we)
        memory[address] <= data_in;
    
    if (re)
        data_out <= memory[address];
end

endmodule
```

# Test bench
```
`timescale 1ns/1ps

module tb_memory_4KB;

    reg clk;
    reg we;
    reg re;
    reg [11:0] address;
    reg [7:0] data_in;
    wire [7:0] data_out;

    // Instantiate the Unit Under Test (UUT)
    memory_4KB uut (
        .clk(clk),
        .we(we),
        .re(re),
        .address(address),
        .data_in(data_in),
        .data_out(data_out)
    );

    // Clock generation: 10ns period
    always #5 clk = ~clk;

    initial begin
        clk = 0;
        we = 0;
        re = 0;
        address = 0;
        data_in = 0;

        #10;

        // Write random data to random addresses
        repeat (10) begin
            @(posedge clk);
            we = 1;
            re = 0;
            address = $random % 4096;
            data_in = $random % 256;
        end

        // Read back from random addresses
        repeat (10) begin
            @(posedge clk);
            we = 0;
            re = 1;
            address = $random % 4096;
        end

        // Disable operations
        @(posedge clk);
        we = 0;
        re = 0;

        #20;
        $finish;
    end

endmodule
```

# Output Waveform
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/2471a2cb-48be-4e82-84a8-a13e6aa66738" />


# FIFO
# FIFO Verilog Code
```
`timescale 1ns/1ps

module FIFO(
    input clk,
    input rst,
    input wr,
    input rd,
    input [7:0] data_in,
    output reg [7:0] data_out,
    output full,
    output empty
);

reg [7:0] mem[0:3];
reg [1:0] wptr, rptr;
reg [2:0] count;

assign full = (count == 3'd4);
assign empty = (count == 3'd0);

always @(posedge clk) begin
    if(rst) begin
        wptr <= 2'd0;
        rptr <= 2'd0;
        count <= 3'd0;
        data_out <= 8'd0;
    end
    else begin
        if(wr && !full) begin
            mem[wptr] <= data_in;
            wptr <= wptr + 1'b1;
        end
        if(rd && !empty) begin
            data_out <= mem[rptr];
            rptr <= rptr + 1'b1;
        end
        case ({(wr && !full), (rd && !empty)})
            2'b10: count <= count + 1'b1;
            2'b01: count <= count - 1'b1;
            default: count <= count;
        endcase
    end
end

endmodule
```
# Testbench Code
```
`timescale 1ns/1ps

module tb_FIFO;

reg clk;
reg rst;
reg wr;
reg rd;
reg [7:0] data_in;

wire [7:0] data_out;
wire full;
wire empty;

FIFO uut(
    .clk(clk),
    .rst(rst),
    .wr(wr),
    .rd(rd),
    .data_in(data_in),
    .data_out(data_out),
    .full(full),
    .empty(empty)
);

always #5 clk = ~clk;

initial begin
    clk = 0;
    rst = 1;
    wr = 0;
    rd = 0;
    data_in = 0;

    #10;
    rst = 0;

    #10;
    wr = 1;
    data_in = 8'd10;

    #10;
    data_in = 8'd20;

    #10;
    data_in = 8'd30;

    #10;
    data_in = 8'd40;

    #10;
    wr = 0;
    rd = 1;

    #10;
    #10;
    #10;
    #10;
    rd = 0;

    #20;
    $finish;
end

endmodule
```

# Output Waveform
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/10352f30-518b-4d6f-8d5b-575dd39f3621" />

# Conclusion
The RAM, ROM, FIFO memory with read and write operations was designed and successfully simulated using Verilog HDL. The testbench verified both the write and read functionalities by simulating the memory operations and observing the output waveforms. The experiment demonstrates how to implement memory operations in Verilog, effectively modeling both the reading and writing processes.
