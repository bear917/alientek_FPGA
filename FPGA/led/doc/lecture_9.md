1. FPGA 開發流程
1. FPGA 工程文件夾管理
1. 點亮 LED
1. 總結

## 開發流程
+ 需求分析
+ 系統設計
    + 功能劃分 MCU 還是 FPGA
    + 模組分割
+ 硬體選擇
    + 成本、算力
+ 繪製系統框圖
    + 規劃各個模組，交付各自團隊完成
+ 繪製波形圖
+ 編寫 RTL 程式碼
+ 軟體模擬
    + 初步驗證：語法、邏輯是否有錯？對比手繪波形圖
+ 創建專案
+ 分析與綜合
    + RTL 對應至電路圖，檢查是否仍有錯誤 (被compiler化簡，省略掉)
+ 約束輸入
    + 引腳約束、時序約束
+ 設計實現
    + 佈局佈線：代碼與實際元件對應
+ 上板驗證與調試
    + 生成比特流，上板驗證
    + 用示波器、邏輯分析儀觀測

## FPGA 工程文件夾管理

[專案名稱]
+ doc
+ rtl
+ sim
+ prj

## 點亮 LED 設計
### 需求分析
    點亮LED，使用按鍵控制 LED 亮滅。
    沒有按下，LED 滅
    按下，LED 亮
### 硬體選型
    根據 Davinci FPGA 原理圖：
    沒有按，輸出 3.3 Volt。按下，輸出 0 Volt。
    輸入 3.3，LED 亮。輸入 0 Volt，LED 暗。

## 思維導圖
+ LED 點亮模組
    + input: button
    + 取反

        **~**: bit wise 按位取反

        **!**: logical 邏輯取反

        + single bit
            ```
            a = 1'b0;
            ~a; // 1'b1
            !a; // 1'b1
        + multiple bits
            ```
            a = 4'b0011;
            ~a; // 4'b1100;
            !a; // 4'b0000; value a is larger than 0, so a is true. complement of a is false
            b = 4'b0000;
            !b; // 4'b0001; 
        
    + output: LED

# Modelsim 示範
+ 創建專案
+ 加入檔案
+ Compile all
+ Run
+ Zoom
+ Add cursor
# Vivado 示範
+ 加入檔案
+ 選擇 FPGA 晶片
+ Open Elaborated Design
    + Schematic 畫面，邏輯閘/元件表示，與晶片硬體無關
    + 端口約束
        + XDC 文件使用
+ After generate bitstream
    + Open Synthesized Design
        + Schematic 畫面，與 FPGA 實際硬體有關，例如 LUT、BUF
# 固化文件
將固化文件透過 SPI 放入 flash，使得 FPGA 開機時可以讀取

+ BIN: 由 Vivado 編譯產生，存儲在特定目錄下的二進制文件
+ MCS: MCS 文件包含 BIN 文件，此外，每行開始有地址訊息，最後一 byte 為 CRC 校驗訊息。文件大小較大
    + 先有 .bin 再轉乘 .mcs，
+ 需要在 .xdc 加入命令，加速 FPGA 開機後讀取 flash 效率
