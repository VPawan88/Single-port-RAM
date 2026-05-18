```systemverilog
module RAM_64B #(
    parameter width = 8,
    parameter addr_width = 6
)(
    input  logic                   clk,
    input  logic                   en,
    input  logic                   we,
    input  logic [width-1:0]       data_in,
    input  logic [addr_width-1:0]  address,
    output logic [width-1:0]       data_out
);

    // Memory declaration : 64 locations for addr_width = 6
    logic [width-1:0] mem[(2**addr_width)-1:0];

    // Synchronous Read and Write
    always_ff @(posedge clk) begin
        if (!en) begin
            data_out <= '0;
        end
        else begin
            if (we) begin
                mem[address] <= data_in;
                data_out    <= '0;
            end
            else begin
                data_out <= mem[address];
            end
        end
    end

endmodule
```
