# Vending-machine-project-using-verilog-lang
In this i try to show how vending machine works and i try to test my knowledge in verilog by writing the code..
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
