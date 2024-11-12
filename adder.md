```verilog 
module LED_ctrl( 
    input [15:0] sw, 
    input clk200P, 
	 input clk200N,
    input reset, 
    output reg led_do, 
    output led_pen, 
    output led_clk, 
    output led_clr 
    ); 
  wire Clk_100M; 
  reg [15:0] sw_d1; 
  parameter piso_shift = 16; 
  reg [piso_shift-2:0] sw_shift; 
  reg [16:0] counter = 17'h0; 
  wire [15:0] shift_load; 

  SWORD_LED_CLK ClkGen100M 
  (// Clock in ports 
  .CLK_IN1_P(clk200P), // IN 
  .CLK_IN1_N(clk200N),
  // Clock out ports 
  .CLK_OUT1(Clk_100M), // OUT 
  // Status and control signals 
  .RESET(RESET),// IN 
  .LOCKED(LOCKED)); 

  always@(posedge Clk_100M) 
      if(!reset) sw_d1 <= 16'h0; 
      else sw_d1 <= sw;
  assign shift_load = sw^sw_d1; 
  always @(posedge Clk_100M) 
       if (shift_load) begin 
           sw_shift <= sw[piso_shift-2:0]; 
           led_do <= ~sw[15]; 
           counter <= 17'h1ffff; 
       end 
       else begin 
           sw_shift <= {sw_shift[piso_shift-3:0], 1'b0}; 
           led_do <= ~sw_shift[14]; 
           counter <= {1'b0, counter[16:1]}; 
       end 
  assign led_clk = Clk_100M & counter[0]; 
  assign led_clr = reset; 
  assign led_pen = 1'b1; 
endmodule

```

```verilog
    module top( 
        input [3:0] a, 
        input [3:0] b, 
        input clk200P, 
		 input clk200N,
        input RSTN, 
        output LEDCLK, 
        output LEDDT, 
        output LEDCLR
        ); 
		  
       wire [3:0] s; 
       wire co; 
       wire [4:0] sum; 
       assign sum = {co, s}; 

       adder_4bits  U1 ( .a(a), .b(b), .ci(1'b0), .s(s), .co(co) ); 
       LED_ctrl  U2 ( 
         .sw({11'b0, sum}), 
			.clk200P(clk200P),
         .clk200N(clk200N), 
         .reset(RSTN), 
         .led_do(LEDDT), 
      // .led_pen(led_pen), 
         .led_clk(LEDCLK), 
         .led_clr(LEDCLR) 
         ); 
      endmodule
```
```verilog
NET "clk200P"	LOC="AC18" | IOSTANDARD = LVDS;
NET "clk200N"	LOC="AD18" | IOSTANDARD = LVDS;


NET "RSTN"		LOC = W13 | IOSTANDARD = LVCMOS18 ;

NET "a[0]"   LOC = AA10   | IOSTANDARD = LVCMOS15 ; 
NET "a[1]"   LOC = AB10   | IOSTANDARD = LVCMOS15 ; 
NET "a[2]"   LOC = AA13   | IOSTANDARD = LVCMOS15 ; 
NET "a[3]"   LOC = AA12   | IOSTANDARD = LVCMOS15 ; 
NET "b[0]"   LOC = Y13    | IOSTANDARD = LVCMOS15 ; 
NET "b[1]"   LOC = Y12    | IOSTANDARD = LVCMOS15 ; 
NET "b[2]"   LOC = AD11   | IOSTANDARD = LVCMOS15 ; 
NET "b[3]"   LOC = AD10   | IOSTANDARD = LVCMOS15 ; 

NET "LEDCLK"			LOC = N26   | IOSTANDARD = LVCMOS33 ;
NET "LEDCLR"			LOC = N24   | IOSTANDARD = LVCMOS33 ;
NET "LEDDT"			LOC = M26   | IOSTANDARD = LVCMOS33 ;

```
