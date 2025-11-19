# RAILWAY COACH AUTOMATION 
## Aim
To design and implement a smart, energy-efficient system for railway coaches that automatically manages **lighting** and **ventilation** based on passenger presence and **ambient light conditions**.

## Apparatus Required
* Computer with **Vivado Design Suite** installed

## Objective
The **main objectives** of this project are:
1. Detect passenger presence using a **sensor** input.
2. Automatically **switch ON** lights and fans when a passenger is detected.
3. To implement **automatic brightness control** using an **8-bit ambient light sensor**.
4. Activate a **delay-based timer** when the coach becomes empty.
5. **Switch OFF** lights and fans only after the timer expires.
6. Display the system status as **Active** or **Idle**.
7. Simulate and verify the complete design using **Vivado**.

## Block Diagram
<img width="826" height="321" alt="image" src="https://github.com/user-attachments/assets/c8bd3f2a-fdbd-450b-88f1-f5c4804ca8b0" />

## FSM State Diagram
<img width="554" height="699" alt="image" src="https://github.com/user-attachments/assets/72565302-b8dd-4412-b36e-8bd3ae3b2c16" />

## Verilog Code
```verilog
module coach_control(
    input clk,
    input reset,
    input passenger_sensor,
    input [7:0] ambient_light,    
    output reg lights,
    output reg fan,
    output reg [7:0] light_level,  
output reg [1:0] status
);

    localparam IDLE   = 2'b00;
    localparam ACTIVE = 2'b01;
    localparam WAIT   = 2'b10;

    reg [1:0] state, next_state;
    reg [31:0] timer;

    parameter TIMEOUT = 5;

    always @(posedge clk or posedge reset) begin
        if (reset)
            state <= IDLE;
        else
            state <= next_state;
    end

    always @(posedge clk or posedge reset) begin
        if (reset)
            timer <= 0;
        else if (state == WAIT)
            timer <= timer + 1;
        else
            timer <= 0;
    end

    always @(*) begin
        case(state)
            IDLE: begin
                if (passenger_sensor)
                    next_state = ACTIVE;
                else
                    next_state = IDLE;
            end

            ACTIVE: begin
                if (!passenger_sensor)
                    next_state = WAIT;
                else
                    next_state = ACTIVE;
            end

            WAIT: begin
                if (passenger_sensor)
                    next_state = ACTIVE;
                else if (timer >= TIMEOUT)
                    next_state = IDLE;
                else
                    next_state = WAIT;
            end

            default: next_state = IDLE;
        endcase
    end

    always @(*) begin
        case(state)

            IDLE: begin
                lights = 0;
                fan = 0;
                status = 2'b00;
                light_level = 8'd0;
            end

            ACTIVE: begin
                lights = 1;
                fan = 1;
                status = 2'b01;

                light_level = 255 - ambient_light;
            end

            WAIT: begin
                lights = 1;
                fan = 1;
                status = 2'b01;

                light_level = 255 - ambient_light;
            end
        endcase

    end
endmodule
```
## Testbench
```verilog
`timescale 1ns/1ps

module coach_control_tb;

    reg clk;
    reg reset;
    reg passenger_sensor;
    reg [7:0] ambient_light;

    wire lights;
    wire fan;
    wire [7:0] light_level;
    wire [1:0] status;

coach_control DUT(
        .clk(clk),
        .reset(reset),
        .passenger_sensor(passenger_sensor),
        .ambient_light(ambient_light),
        .lights(lights),
        .fan(fan),
        .light_level(light_level),
        .status(status)
    );

    always #5 clk = ~clk;

    initial begin
        clk = 0;
        reset = 1;
        passenger_sensor = 0;
        ambient_light = 50;  
        #20 reset = 0;

        #30 passenger_sensor = 1;
        ambient_light = 20;   

        #100 passenger_sensor = 0;

        #200 passenger_sensor = 1;
        ambient_light = 200; 

        #50 passenger_sensor = 0;

        #200 $stop;

    end
endmodule
```
## Simulation Output
<img width="1920" height="1080" alt="Screenshot 2025-11-18 141924" src="https://github.com/user-attachments/assets/1d0933bc-8c04-446a-ad5d-dab9af070ab1" />

## FPGA Board Implementation
1. **passenger_sensor = 1**, after another 200 ns, a **second passenger** arrives.

2. **ambient_light = 200**
set ambient high to simulate **bright environment** so light_level = 255 - 200 = 55 **(dim LEDs)**.

3. **led 1** – Indicates **lights status** (general indicator for lighting).

4. **led 2** – Indicates **fan status**.

5. **next 8 leds** (led 3–led 10) – Represent the **binary value** of the number **55**, i.e. **00110111**.

6. **next 2 leds** (led 11 & led 12) – Indicate **status register bits**:

    - LED 11 → Status Register Bit = 0
      
    - LED 12 → Status Register Bit = 1

<img width="937" height="865" alt="image" src="https://github.com/user-attachments/assets/d24336c1-6af6-4ed9-921e-8610ded2852b" />

## Results
The successful implementation and verification of the design achieved the objectives of intelligent power management, resulting in reduced energy consumption in railway coaches.
