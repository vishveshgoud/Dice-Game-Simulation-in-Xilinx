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
