# Vending-machine-project-using-verilog-lang
In this i try to show how vending machine works and i try to test my knowledge in verilog by writing the code..
verilog code
`timescale 1ns / 1ps

module vending_machine (
    input clk,
    input reset,
    input cancel,
    input [1:0] coin,   // 2'b01 = 5 Rupees, 2'b10 = 10 Rupees
    input [1:0] sel,    // 2'b00 = Prod A (5/-), 2'b01 = Prod B (10/-), 2'b10 = Prod C (15/-)
    output reg PrA,
    output reg PrB,
    output reg PrC,
    output reg charge
);
    
         
    parameter S0  = 3'b000;
    parameter S5  = 3'b001;
    parameter S10 = 3'b010;
    parameter S15 = 3'b011;
    parameter S20 = 3'b100;

    reg [2:0] current_state, next_state;

    always @(posedge clk or posedge reset) begin
        if (reset) current_state <= S0;
        else       current_state <= next_state;
    end

 
    always @(*) begin
        case (current_state)
            S0:  if (coin == 2'b01) next_state = S5;
                 else if (coin == 2'b10) next_state = S10;
                 else next_state = S0;

            S5:  if (coin == 2'b01) next_state = S10;
                 else if (coin == 2'b10) next_state = S15;
                 else if (cancel) next_state = S0;
                 else next_state = S5;

            S10: if (coin == 2'b01) next_state = S15;
                 else if (coin == 2'b10) next_state = S20;
                 else if (cancel) next_state = S0;
                 else next_state = S10;

            S15: if (coin == 2'b01) next_state = S20;
                 else if (cancel) next_state = S0;
                 else next_state = S15;

            S20: if (cancel) next_state = S0;
                 else next_state = S20;

            default: next_state = S0;
        endcase
    end


    always @(posedge clk or posedge reset) begin
        if (reset) begin
            PrA <= 0; PrB <= 0; PrC <= 0; charge <= 0;
        end else begin
            PrA <= 0; PrB <= 0; PrC <= 0; charge <= 0;

            case (current_state)
                S5:  if (sel == 2'b00) PrA <= 1;
                S10: if (sel == 2'b00) begin PrA <= 1; charge <= 1; end
                     else if (sel == 2'b01) PrB <= 1;
                S15: if (sel == 2'b01) begin PrB <= 1; charge <= 1; end
                S20: if (sel == 2'b00) begin PrA <= 1; charge <= 1; end
                     else if (sel == 2'b01) begin PrB <= 1; charge <= 1; end
                     else if (sel == 2'b10) PrC <= 1;
            endcase

            if (cancel) charge <= 1;
        end
    end

endmodule





test bench 


`timescale 1ns / 1ps

module vending_machine_tb;
    reg clk;          // our ip
    reg reset;
    reg cancel;
    reg [1:0] coin;
    reg [1:0] sel;

    wire PrA;      // o/p
    wire PrB;
    wire PrC;
    wire charge;

    vending_machine uut (
        .clk(clk), .reset(reset), .cancel(cancel),
        .coin(coin), .sel(sel),
        .PrA(PrA), .PrB(PrB), .PrC(PrC), .charge(charge)
    );

    always #5 clk = ~clk;

    initial begin
        $dumpfile("vending_machine_tb.vcd");
        $dumpvars(0, vending_machine_tb);    
    end

    initial begin
        clk = 0; reset = 1; cancel = 0; coin = 2'b00; sel = 2'b00;
        #20 reset = 0;

                                              
        #10 coin = 2'b01; //in test case 1 we insert 5 rs and buy prA 
        #10 sel = 2'b00;  // sel prA
        #10 coin = 2'b00; // stop insert
        #20;

                                                               
        #10 coin = 2'b10;  // TC2 insert 10 rs
        #10 sel = 2'b01;   // sel prB
        #20;

                                                              
        #10 coin = 2'b01;  //tc3 insert 5 rs 
        #10 coin = 2'b01;  // insert 5 rs again
        #10 sel = 2'b00;   // sel prA
        #20;

                                                                 
        #10 coin = 2'b10; //tc 4 insert 10 rs
        #10 coin = 2'b10; // insert 10 ra again
        #10 sel = 2'b10;// sel prC
        #20;

                                                                             
        #10 coin = 2'b10; // Insert 10 rs and cancel it 
        #10 cancel = 1; 
        #10 cancel = 0;
        #20;

        #50 $finish;
    end

                                                                                          
    initial begin
        $monitor("Time=%0d, Coin=%b, Sel=%b, Cancel=%b, PrA=%b, PrB=%b, PrC=%b, Charge=%b", 
                 $time, coin, sel, cancel, PrA, PrB, PrC, charge);
    end

endmodule
