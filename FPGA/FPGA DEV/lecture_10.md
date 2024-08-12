+ 早期數位IC邏輯功能是固定不變的。
+ 後來發明PLD; Programmable Logic Device，用戶可自行修改內部連接的 IC。PLD 內部的電路結構可以透過寫入編程數據來設置，還可以擦除重寫.
+ PLD的邏輯功能可由使用者編程來設定。
+ CPLD; Complex Programmable Logic Device
    + 基於「乘積項」的與或 (and-or) 邏輯陣列
+FPGA; Field Programmable Gate Array
    + 基於「查找表」的 CLB 陣列

FPGA 可以透過編程來修改其邏輯功能的數位IC (跟PLD解釋相同)，硬體連接可以改變。

單晶片系統，編程不改變其內部連接結構，只根據要求實現的功能去編寫程式(指令)

# 數位系統設計流程
+ 邏輯設計(前端)
    + HDL 用於描述數位電路結構與功能，可以在不同層次對數位電路的結構、功能、行為進行描述。

+ 電路實現(後端)
    + HDL 所描述的電路，可以通過綜合工具，將其轉換為門級電路網表，然後將其與某種工藝的基本元件逐一對應起來，再通過佈局工具轉換為電路佈線結構。

+ 系統驗證

---
Verilog 1983年發明，1995 成為 IEEE 標準。

---
FGPA 設計中，已經不再採用 schematic 輸入法，現在電路規模太大，以 Verilog 設計方法為主流。

# Verilgo 和 C 的區別
Verilog 是硬體描述語言，編譯下載到 FPGA 後，會生成電路。所以 Verilog 是並行處理。(平行處理)

C 語言是軟體程式語言，編譯下載到單晶片系統後，是存儲器的一組指令。單晶片系統需經過「取指」、「譯碼」、「執行」，過成為串行處理。(循序處理)

兩者差別，FPGA 由於全並行處理，處理速度快。單晶片/CPU 無法取代。

# 基本語法
## 邏輯值
+ 0 代表 GND
+ 1 代表 VCC
+ X 未知，可為：VCC 或 GND
+ Z 高阻態，外部沒有激勵訊號，浮接狀態，可為任意值
## 數字進制格式
+ binary，4'b0101，表示 4 位數二進位數字 0101
+ decimal，4'd2，表示 4 位數十進位數字 2
+ hexadecimal，4'ha，表示 4 位數十六進位數字 a
+ 範例：16'b1001_1010_1010_1001 = 16'b9aa9
## 標示符 identifier
+ 定義模組名、端口名、訊號名
+ 字母、數字、$、_
+ 首字應為字母或 _
+ 大小寫視為不同
### 建議規則
+ 內部訊號全小寫
+ 簡潔、清晰、易懂
    + 有意義、有效。例：sum、cpu_addr
    + 用 _ 分詞，例：cpu_addr
    + 採用前綴或後綴，例：時鐘訊號，clk_50、clk_cpu
## 資料類型
+ reg (真正在電路存在)
+ net (真正在電路存在)
+ parameter
### reg
+ 抽象的資料存儲單元，通過賦值語句可以改變 reg 的值。
+ reg 預設為 X
+ 只能在 always 與 initial 內部賦值
always 無時鐘訊號 (組合邏輯)，reg 為硬體連線。
always 有時鐘訊號 (時序邏輯)，reg 為觸發器。
### net
+ 結構實體之間的物理連線
+ 不能儲存值，其值由驅動元件決定
+ 可驅動 wire 的有：gate、assign
+ 無驅動的 wire，其值為 Z
+ 有 wire、tri
### parameter
+ 為常數
+ 常用於：狀態機的狀態、資料寬度、延遲大小
+ 可提高可讀性、可維護性
+ 模組調用時，可用參數傳遞改變模組內已定義的參數
### 運算符
+ arithmetic priority! (*)
+ \+ - *
+ / quotient (浪費資源)
+ % mod (浪費資源)
+ \>, <, >=, <=
+ !=
+ condition / logical: !, &&, ||
+ ?:
    + result = (a>=b) ? a : b;
+ bit_wise: ~, &, |, ^
    + When bit_width is not equal, smaller one is extended to align larger one (insert 0 to MSB)
+ \<<, \>>
    + insert 0 to removed bit
    + 4'b1001<<2 = 6'b100100
    + 4'b1001>>1 = 4'b0100
+ \{a_bit, b_bit}
    + c = {a, b[3;0]}
### comment
+ //
+ /* */
### 模組的結構
一個 module 由兩的部分組成，一部分描述端口，另一部份描述功能。

```v
module block(a,b,c,d);
input   a, b;
output  c, d;

assign c = a | b;
assign d = a & b;

endmodule
```
+ 端口定義 \+ IO 說明
+ 內部訊號聲明 (上面案例沒有)
+ 功能定義

```v
module flow_led (
    input sys_clk,
    input sys_rst_n,

    output reg [3:0] led
);

// reg define
reg [23:0] counter;

// count 0.2 seconds
always @(posedge sys_clk or negedge sys_rst_n) begin
    if (!sys_rst_n)
        counter <= 24'd0;
    else if (counter < 24'd100_0000)
        counter <= counter + 1'b1;
    else
        counter <= 24'd0;
end

// shift out signal
always @(posedge sys_clc or negedge sys_rst_n) begin
    if (!sys_rst_n)
        led <= 4'b0001;
    else if (counter == 24'd1000_0000)
        led [3:0] <= {led[2:0], led[3]};
    else
        led <= led;
end

endmodule
```
### 功能定義方法
1. assign 語句，描述組合邏輯
1. always 語句，描述組合/時序邏輯
1. 例化實例元件， and #2 u1(q, a, b);
+ 上述邏輯功能為並行
+ always 內部，if 為循序執行
+ 多個 always，全為並行執行

### 模組調用
```v
module seg_led_static_top (
    input           sys_clk,
    input           sys_rst_n,
    output [5:0]    sel,
    output [7:0]    seg_led
);

// parameter define
parameter TIME_SHOW = 25'D25000_000;

// wire define
wire add_flag;

// generate a clock signal every 0.5 second
time_count #(
    .MAX_NUM (TIME_SHOW)
) u_time_count (
    .clk    (sys_clk    ),
    .rst_n  (sys_rst_n  ),
    .flag   (add_flag   )
);

seg_led_static u_seg_led_static(
    .clk        (sys_clk    ),
    .rst_n      (sys_rst_n  ),
    .add_flag   (add_flag   ),
    .sel        (sel        ),
    .seg_led    (seg_led    )
);

endmodule

module time_count(
    input clk,
    input rst_n,
    output reg flag
);

parameter MAX_NUM = 50000_000;

// parameter define
parameter MAX_NUM = 50000_000;

// reg define
reg [24:0] cnt;

endmodule
```
+ module input type: wire / reg
+ module output type: wire
+ passing signal 位寬需一致

# 結構語句
## initial
initial 只執行一次，常用於 testbench，產生仿真訊號/激勵訊號，或是對存儲器賦予初值。
## always
不斷重複活動。須與時間控制結合才有作用。

```v
// assign value
initial begin
    sys_clk <= 1'b0;
    sys_rst_n <= 1'b0;
    touch_key <= 1'b0;
    #20
    sys_rst_n <= 1'b1;
    #10
    touch_key <= 1'b1;
    #30
    touch_key <= 1'b0;
    #110
    touch_key <= 1'b1;
    #30
    touch_key <= 1'b0;
end

// generate 50 MHz clock signal
always #10 sys_clk <= ~sys_clk;
```
## always@()
+ @sensitivity list 內部用 or 連接
+ 可以是 edge 或 level
    + level 觸發，通常為組合邏輯
    ```v
    always @(a or b or c)
    ```
    + \* 對於所有輸入訊號敏感
    ```v
    always @(*) // @*
    ```
# 賦值語句
+ blocking, b = a;
+ non-blocking, b <= a;
## blocking
共一步
+ 計算 RHS 值，並立刻更新 LHS。

```v
always @(posedge clk or negedge rst_n) begin
    if (!rst_n) begin
        a = 1;
        b = 2;
        c = 3;
    end
    else begin
        a = 0;
        b = a;
        c = b; // a=0, b=0, c=0
    end
end
```
### non-blocking
共二步
1. 計算全部的 RHS
1. 上面結束後，更新 LHS
```v
always @(posedge clk or negedge rst_n) begin
    if (!rst_n) begin
        a <= 1;
        b <= 2;
        c <= 3;
    end
    else begin
        a <= 0;
        b <= a;
        c <= b; // a=0, b=1, c=2
    end
end
```
+ 僅能用於 always 與 initial 內部，因此只有 reg 才能使用。
# 條件語句
## if_else
```v
if (a > b)
    out = data_1;

if (a > b)
    out = data_1;
else
    out = data_2;

if (condition1)
    statement1;
else if (condition2)
    statement2;
else if (condition3)
    statement3;
else
    statement4;
```
+ 簡寫
    ```v
    if (a)  => if (a==1)
    if (!a) => if (a!=1)
    ```
+ 條件運算為 0, X, Z 皆視為 false
+ 條件運算為 1 視為 true
+ 使用 begin, end 涵蓋多個命令
+ 允許 nested if (消耗資源)
## case
```v
always @(posedge clk or negedge rst_n) begin
    if (!rst_n)
        seg_led <= 8'b0;
    else begin
        case (num)
            4'h0:
            4'h1:
            ...
            default:
        endcase
    end
end
```
# 狀態機
+ 密碼鎖，等於序列檢測器。可用狀態機實現。
+ 狀態機適合實現順序邏輯，Verilog 預設平行處理，不好順序執行。
+ finite state machine;有限狀態機：有限個狀態之間按照一定規律轉換的時序電路

## Mealy 狀態機
產生狀態的組合邏輯 F -> 狀態暫存器 -> 產生輸出的組合邏輯 G。
G 接受「輸入訊號」與「目前狀態」決定功能輸出

## Moore 狀態機
G 接受「目前狀態」決定功能輸出

# 狀態機設計
1. 狀態空間定義
1. 狀態跳轉
1. 判斷下一狀態
1. 各狀態的功能

## 狀態空間定義
```v
// define state space
parameter SLEEP     = 2'b`00;
parameter STUDY     = 2'b`01;
parameter EAT       = 2'b`10;
parameter AMUSE     = 2'b`11;

// intrnal variable
reg [1:0] current_state;
reg [1:0] next_state;
```

```v
// define state space
// one hot encoded
parameter SLEEP     = 2'b`1000;
parameter STUDY     = 2'b`0100;
parameter EAT       = 2'b`0010;
parameter AMUSE     = 2'b`0001;

// intrnal variable
reg [3:0] current_state;
reg [3:0] next_state;
```

## 狀態跳轉 (時序邏輯)
```v
always @(posedge clk or negedge rst_n) begin
    if (!rst_n)
        current_state <= SLEEP;
    else
        current_state <= next_state;
end
```

## 下一狀態判斷
```v
always @(current_state or input_signals) begin
    case(current_state)
        SLEEP: begin
            if (clock_alarm)
                next_state = STUDY;
            else
                next_state = SLEEP;
        end

        STUDY: begin
            if (lunch_time)
                next_state = EAT;
            else
                next_state = STUDY;
        end

        EAT: ...
        AMUSE: ...
        default: ...
    endcase
    
end
```
