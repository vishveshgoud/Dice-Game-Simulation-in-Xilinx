# Dice-Game-Simulation-in-Xilinx
 The whole setup consists of various sub blocks those are as folows:
# Loadable 2 to 12 counter
  This block is built from two sub-blocks 
  1) Negative edge triggered t flipflop
  2) Negative edge triggered t flipflop with preset
 module tff_neg(


    input clk,
    
    
    input reset,
    
    
    input t,
    
    
    output reg q
    
    
    );
    
    
    always@(negedge clk,posedge reset)
    
    
    begin
    
        
        if(reset==1'b1)
        
            
            q<=0;
        
        
        else 
        
        
        begin
        
            
            if(t==1'b0)
            
                
                q<=q;
            
            
            else
            
                
                q<=~q;
        
        
        end 
    
    
    end


endmodule

-------------------------------------------------------------------------------------------------------
module tff_negp(
  
   input clk,
   
   input reset,
   
   input t,
   
   output reg q
   
   );
   
   always@(negedge clk,posedge reset)
   
   begin
   
       if(reset==1'b1)
       
           q<=1;
       
       else 
       
       begin
       
           if(t==1'b0)
           
               q<=q;
           
           else
               
               q<=~q;
       
       end 
   
   end

endmodule

-------------------------------------------------------------------------------------------
module loadable2to12_counter(
   
    input clk,
    
    input load,
    
    input cntenable,
    
    output [3:0] Q
    
    );
    
    wire reset,chk,clk1;
    
    assign clk1= cntenable & clk;
    
    tff_neg ff3(Q[2],reset,1'b1,Q[3]);
    
    tff_neg ff2(Q[1],reset,1'b1,Q[2]);
    
    tff_negp ff1(Q[0],reset,1'b1,Q[1]);
    
    tff_neg ff0(clk1,reset,1'b1,Q[0]);
    
    assign chk= Q[3] & Q[2] & ~Q[1] & ~Q[0];
    
    assign reset= chk | load;

endmodule

----------------------------------------------------------------
# Attempt counter

module attempt_counter(
   
    input clk,
    
    input enofa,
    
    input start,
    
    output [6:0] Q
    
    );
    
    wire reset,chk,clk1;
    
    assign clk1 = enofa ;
    
    tff_neg ff6(Q[5],reset,1'b1,Q[6]);
    
    tff_neg ff5(Q[4],reset,1'b1,Q[5]);
    
    tff_neg ff4(Q[3],reset,1'b1,Q[4]);
    
    tff_neg ff3(Q[2],reset,1'b1,Q[3]);
    
    tff_neg ff2(Q[1],reset,1'b1,Q[2]);
    
    tff_neg ff1(Q[0],reset,1'b1,Q[1]);
    
    tff_neg ff0(clk1,reset,1'b1,Q[0]);
    
    assign chk= Q[6] & Q[5] & ~Q[4] & ~Q[3] & Q[2] & ~Q[1] & ~Q[0] ;
    
    assign reset=start | chk;

endmodule
----------------------------------------------------------------
# Dice value comparator

module dicecomparator(
    
    input [3:0] dv,
    
    output dg9,
    
    output dl4
    
    );
    
    assign dg9 = (dv[3] & dv[2]) | (dv[3] & dv[0]) | (dv[3] & dv[1]);
    
    assign dl4 = (~dv[3] & ~dv[2]) | (~dv[3] & ~dv[0] & ~dv[1]);

endmodule
-----------------------------------------------------------
# score comparator

module score_comparator(
    
    input [6:0] sc,
    
    output scg99,
    
    output scl10
    
    );
    
    assign scl10=~(sc[4] | sc[5] | sc[6] | sc[3]) | ((~sc[6]&~sc[5]&~sc[4]&sc[3]&~sc[2]&~sc[1]&~sc[0]) | (~sc[6]&~sc[5]&~sc[4]&sc[3]&~sc[2]&~sc[1]&sc[0]));  
    
    assign scg99= (sc[6] & sc[5]) & ~((sc[6]&sc[5]&~sc[4]&~sc[3]&~sc[2]&~sc[1]&~sc[0]) | (sc[6]&sc[5]&~sc[4]&~sc[3]&~sc[2]&~sc[1]&sc[0]) | (sc[6]&sc[5]&~sc[4]&~sc[3]&~sc[2]&sc[1]&~sc[0]) | (sc[6]&sc[5]&~sc[4]&~sc[3]&~sc[2]&sc[1]&sc[0])) ;

endmodule
-------------------------------------------------
# dicevalue selector

This block is built from 2 cross 1 multiplexers

module mux2x1(
    
    input i0,
    
    input i1,
    
    input s,
    
    output y
    
    );
    
    assign y= (~s & i0) | (s & i1); 

endmodule
------------------------------------------------------------
module diceselect(
    
    input [3:0] dv,
    
    input s,
    
    output [3:0] out
    
    );
    
    wire [3:0] ten;
    
    assign ten=4'b1010;
    
    mux2x1 m3(dv[3],ten[3],s,out[3]);
    
    mux2x1 m2(dv[2],ten[2],s,out[2]);
    
    mux2x1 m1(dv[1],ten[1],s,out[1]);
    
    mux2x1 m0(dv[0],ten[0],s,out[0]);

endmodule
---------------------------------------------
# score selector

module scoreselect(
    
    input [6:0] asc,
    
    input pr,
    
    output [6:0] sc
    
    );
    
    wire [6:0] nine;
    
    assign nine=7'b1100011;
    
        mux2x1 m6(asc[6],nine[6],pr,sc[6]);
        
        mux2x1 m5(asc[5],nine[5],pr,sc[5]);
        
        mux2x1 m4(asc[4],nine[4],pr,sc[4]);
        
        mux2x1 m3(asc[3],nine[3],pr,sc[3]);
        
        mux2x1 m2(asc[2],nine[2],pr,sc[2]);
        
        mux2x1 m1(asc[1],nine[1],pr,sc[1]);
        
        mux2x1 m0(asc[0],nine[0],pr,sc[0]);
           
endmodule
---------------------------------------------
# Adder cum Substractor

This block is built from the sub-blocks half adder ->fulladder->Ripple carry adder

module halfadder(
   
    input a,
    
    input b,
    
    output sum,
    
    output carry
    
    );
    
    assign sum= a^ b;
    
    assign carry= a & b;

endmodule
--------------------------------
module fulladder(
    
    input a,
    
    input b,
    
    input cin,
    
    output sum,
    
    output cout
    
    );
    
    wire w1,w2,w3;
    
    halfadder ha1(a,b,w1,w2);
    
    halfadder ha2(cin,w1,sum,w3);
    
    assign cout= w2 | w3;

endmodule
------------------------------

module rca7bit(
    
    input [6:0] A,
    
    input [6:0] B,
    
    input cin,
    
    output [6:0] sum,
    
    output cout
    
    );
    
    wire w1,w2,w3,w4,w5,w6;
    
    fulladder fa1(A[0],B[0],cin,sum[0],w1);
    
    fulladder fa2(A[1],B[1],w1,sum[1],w2);
    
    fulladder fa3(A[2],B[2],w2,sum[2],w3);
    
    fulladder fa4(A[3],B[3],w3,sum[3],w4);
    
    fulladder fa5(A[4],B[4],w4,sum[4],w5);
    
    fulladder fa6(A[5],B[5],w5,sum[5],w6);
    
    fulladder fa7(A[6],B[6],w6,sum[6],cout);

endmodule
---------------------------------------------------
module addsub(
   
    input [6:0] sc,
    
    input [3:0] dv,
    
    input ads,
    
    output [6:0] upsc
    
    );
    
    wire [6:0] dvv,toad,sum;
    
    wire cout,cin,k;
    
    assign cin=ads;
    
    assign dvv[6:4]={3{ads}};
    
    assign dvv[3]= dv[3] ^ ads;
    
    assign dvv[2]= dv[2] ^ ads;
    
    assign dvv[1]= dv[1] ^ ads;
    
    assign dvv[0]= dv[0] ^ ads;
    
    rca7bit a1(sc,dvv,cin,sum,cout);
    
    assign k = ads & ~cout;
    
    assign upsc[6]= k ^ sum[6];
    
    assign upsc[5]= k ^ sum[5];
    
    assign upsc[4]= k ^ sum[4];
    
    assign upsc[3]= k ^ sum[3];
    
    assign upsc[2]= k ^ sum[2];
    
    assign upsc[1]= k ^ sum[1];
    
    assign upsc[0]=  k ^ sum[0];

endmodule
----------------------------------------------
# Score register

This block is built from D flipflops

module dff(
    
    input clk,
    
    input reset,
    
    input d,
    
    output reg q
    
    );
    
    always@(posedge clk,posedge reset)
    
    begin
    
        if(reset==1'b1)
        
            q<=0;
        
        else
        
            q<=d;
    
    end

endmodule
--------------------------------------------------
module scoreregister(
    
    input [6:0] sc,
    
    input load,
    
    input clk,
    
    input clr,
    
    input start,
    
    output [6:0] scd
    
    );
    
    wire clk1,clr1;
    
    assign clr1= clr | start;
    
    assign clk1= load;
    
    dff ff6(clk1,clr1,sc[6],scd[6]);
    
    dff ff5(clk1,clr1,sc[5],scd[5]);
    
    dff ff4(clk1,clr1,sc[4],scd[4]);
    
    dff ff3(clk1,clr1,sc[3],scd[3]);
    
    dff ff2(clk1,clr1,sc[2],scd[2]);
    
    dff ff1(clk1,clr1,sc[1],scd[1]);
    
    dff ff0(clk1,clr1,sc[0],scd[0]);

endmodule
------------------------------------------
# Dice game Controller

This block is built from th sub-block Rom content table

module romcont(
    
    input [8:0] a,
    
    output [10:0] c
    
    );
    
    assign c[10] = (~a[8] & a[7]&a[6])|(~a[8] & a[7]&~a[6] &~a[3] & a[2] & a[1]) | (~a[8] & a[7]&~a[6] &~a[3] & a[2] & ~a[1]) | (~a[8] & a[7]&~a[6] &~a[3] & ~a[2]) | (a[8] & ~a[7]&~a[6] &~a[3] & a[4]) |  (a[8] & ~a[7]&a[6] &~a[3] );
    
    assign c[9]=(~a[8] & a[7]&~a[6] &a[3]) | (~a[8] & ~a[7]&a[6] &a[4]) ;
    
    assign c[8]=(~a[8] & ~a[7]&~a[6] &a[5]) | (~a[8] & ~a[7] & a[6] &~a[4]) | (~a[8] & a[7]&~a[6] &a[3]) | (~a[8] & a[7]&~a[6] &~a[3] & a[2] & ~a[1]) | (a[8] & ~a[7] &~a[6] & ~a[4]);
    
    assign c[7]=(~a[8] & ~a[7]&~a[6] &a[5]);
    
    assign c[6]=(~a[8] & ~a[7]&a[6] &~a[4]);
    
    assign c[5]= (~a[8] & a[7]&~a[6] &~a[3] & a[2] &~a[0]) | (a[8] & ~a[7]&a[6]);
    
    assign c[4]=(~a[8] & a[7]&~a[6] &a[3] & a[0]);
    
    assign c[3]=(~a[8] & a[7]&~a[6]  ) | (~a[8] & a[7]&a[6]) | (a[8] & ~a[7]&a[6]);
    
    assign c[2]=(~a[8] & a[7]&~a[6] &~a[3] & a[2] & a[1]);
    
    assign c[1]=(~a[8] & a[7]&~a[6] &a[3] & ~a[0]) |  (~a[8] & a[7]&~a[6] &~a[3] & a[2] & ~a[1]) | (~a[8] & a[7]&a[6]) | (a[8] & ~a[7]&a[6]);
    
    assign c[0]=(~a[8] & a[7]&~a[6]  );
    
endmodule
---------------------------------------------------------------
module controller(
    
    input start,
    
    input td,
    
    input dg9,
    
    input dl4,
    
    input scl10,
    
    input scg99,
    
    input clk,
    
    output  load2,
    
    output cnten,
    
    output  ads,
    
    output  pr,
    
    output  load,
    
    output  clr,
    
    output  s,
    
    output  enofa
    
    );
    
    wire a,b,c,ap,bp,cp;
    
    dff ff2(clk,start,ap,a);
    
    dff ff1(clk,start,bp,b);
    
    dff ff0(clk,start,cp,c);
    
    romcont r1({a,b,c,start,td,dg9,dl4,scl10,scg99},{ap,bp,cp,load2,cnten,ads,pr,load,clr,s,enofa});
    
    initial 
    
        $monitor("a:",a,"b:",b,"c:",c,"atart:",start,"td:",td,"dg:",dg9,"dl:",dl4,"scl10:",scl10,"scg99:",scg99,"ap:",ap,"bp:",bp,"cp:",cp,"load2:",load2,"cnten:",cnten,"ads:",ads,"pr:",pr,"load:",load,"clr:",clr,"s:",s,"enofa:",enofa);

endmodule
----------------------------------------------------------------------
# The dice game top module fits all these sub blocks


module thedicegame(
    
    input start,
    
    input td,
    
    input clk,
    
    input clr,
    
    output [3:0] dice,
    
    output [6:0] attempts,
    
    output [6:0] tscore
    
    );
    
    wire cntenable,enofa,dg9,dl4,load,scg99,scl10,pr,s,ads,load2;
    
    wire [6:0] asc,sc;
    
    wire [3:0] addin;
    
    loadable2to12_counter c1(clk,load2,cntenable,dice);
    
    attempt_counter c2(clk,enofa,start,attempts);
    
    dicecomparator d1(dice,dg9,dl4);
    
    score_comparator s1(tscore,scg99,scl10);
    
    scoreselect m1(asc,pr,sc);
    
    diceselect m2(dice,s,addin);
    
    addsub a4(tscore,addin,ads,asc);
    
    scoreregister sr(sc,load,clk,clr,start,tscore);
    
    controller c9(start,td,dg9,dl4,scl10,scg99,clk,load2,cntenable,ads,pr,load,clr,s,enofa);
    
    initial
    
        $monitor(tscore,"  ",addin,"  ",ads,"  ",asc,"  ",sc);

endmodule
-----------------------------------------------------------------
# Testbench

module game(

    );
    
    reg clk,clr,start,td;
    
    wire [3:0] dice;
    
    wire [6:0] attempts,tscore;
    
    thedicegame uut(start,td,clk,clr,dice,attempts,tscore);
    
    initial begin
    
        clk=1'b1;
        
        forever #5 clk=~clk;
    
    end
    
    initial begin
    
        clr=1'b1;
        
        start=1'b1;
        
        #10;
        
        clr=1'b0;
        
        start=1'b0;
        
        td=0;
        
        #30;
        
        td=1;
        
        #10;
        
        td=0;
        
        #50;
        
        td=1;
        
        #10;
        
        td=0;
        
        #50;
        
        td=1;
        
        #10;
        
        td=0;
        
        #50;
        
        td=1;
        
        #10;
        
        td=0;
        
        #40;
        
        td=1;
        
        #10;
        
        td=0;
        
        end

endmodule
----------------------------
