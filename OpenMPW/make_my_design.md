---
layout: default
title: 2.デザインを自作する
---
### 2.デザインを自作する

8-bit CPUをCaravelに載せた時のサンプルを置いておきます。適宜参照するとよろし。<br>
[https://github.com/cpu-dev/caravel_jacaranda-8/tree/submission-mpw-3/verilog/rtl](https://github.com/cpu-dev/caravel_jacaranda-8/tree/submission-mpw-3/verilog/rtl)

#### Caravelの概要
caravelのUser Project Areaに自分のデザインを突っ込む訳ですが、その前にUser Project Areaについて多少知っておいた方が良いのでここで説明する。

以下がCaravelパッドフレームの概略図

![caravelの概要](https://i.imgur.com/CQLoSjf.png)

User Project Areaについて抑えておくべき事項は3つ
* Wishboneバスが繋がっている(32bit)
* Logic Analyzer Signalsが繋がっている(128bit)
* GPIOが生えている(38本)

Wishbone、Logic Analyzer Signalsに関してはCPU側からMMIOを通してアクセスが可能であり、Wishboneのアドレスは`0x30000000`から`0x80000000-1`の間で自由に設定できる。

以下では、Wishbone、Logic Analyzer Signals、GPIOそれぞれの扱い方を説明する。

#### Wishboneを扱う
という訳でWishboneの扱い方を付け焼き刃程度に解説する。既にWishboneについて知ってる方はこの部分を読む必要は無い。

Caravelでは10種類のWishboneに関係した線が存在している。



| 信号名   | 説明 |
| ----- | --- |
|wb_clk_i| マスターから入力されるクロック。この立ち上がりのタイミングで値が読まれる。                               |
|wb_rst_i| リセット信号。wishboneインターフェイスのステートマシンをリセットさせる。IPコアをリセットする必要は無い。 |
|wbs_stb_i| ストローブ入力。スレーブが選択されている事を示す。 |
|wbs_cyc_i| サイクル入力。有効なバスサイクルが進行している事を示す。|
|wbs_we_i| ライトイネーブル入力。現在のバスサイクルがReadかWriteか示す。1でWrite、0でRead。|
|wbs_sel_i[3:0]| セレクタ入力。入力データ(wbs_dat_i[31:0])の内、有効なデータがどれか示す。8ビット粒度。|
|wbs_dat_i[31:0]|データ入力|
|wbs_adr_i[31:0]|アドレス入力 |
|wbs_ack_o|アクノリッジ出力。1にすると正常なバスサイクルの終了を示す。|
|wbs_dat_o[31:0]|データ出力|

とまあ沢山信号線があって制御がダルいように思うかもしれないが、割とシンプルな回路で扱える。

まず信号を整理する。[^25] 
```verilog
// WB MI A
assign valid = wbs_cyc_i & wbs_stb_i; 
assign wbs_ack_o = ready;
assign we = wbs_we_i;
assign wbs_dat_o = rdata;
assign wdata = wbs_dat_i;
assign addr = wbs_adr_i;
// wb_sel_iはお好みで

assign reset = wb_rst_i;
assign clk   = wb_clk_i;
```

そしてシンプルなデータ読み出し、書き込みが出来るロジックを以下に書いた。
```verilog
always @(posedge clk) begin
    if(reset) begin
        // ここにリセットの処理
        ready <= 1'b0;
    end else begin
        if(ready) begin
            ready <= 1'b0;
        end
        // Read
        if (valid && !ready && !we) begin
            case(addr)
                // こんな感じでRead処理
                ADDR1: rdata <= nanka_data1;
                ADDR2: rdata <= nanka_data2;
                // ・・・
            endcase
            ready <= 1'b1;
            
        // Write
        end else if (valid && !ready && we) begin
            case(addr)
                // こんな感じでWrite処理
                ADDR8: nanka_reg1 <= wdata;
                ADDR9: nanka_reg2 <= wdata;
                // ・・・
            endcase
            ready <= 1'b1;
        end
    end
end
```
あとはこれをたたき台にして、ここのアドレスに書き込んだらスタート〜とかこのアドレスのレジスタがHighになったら処理完了〜とか適当な回路を作れば大体なんとかなる。

#### Logic Analyzer Signalsを扱う
ついでにLogic Analyzer Signalsの扱い方も解説する。

Caravelでは3種類のLogic Analyzer Signalsに関連した信号線が存在している。

| 信号名 | 説明 |
| ------ | ---- |
| la_data_in[127:0] | 入力データ、128ビット。 |
| la_data_out[127:0] |出力データ、128ビット。 |
| la_oenb[127:0]|出力イネーブルバー。0で出力、1で入力。これはSoC側がMMIOで1ビットずつ設定するので読むことしか出来ない。  |

まあ流れで扱える
```verilog
assign data1 = la_data[31:0];
```
とか
```verilog
always @(posedge clk) begin
    la_data_out[63:32] <= data2;
end
```
とか。*_outへの入力をassignかalwaysのどちらでやるかは多分適当でよい。

#### GPIOを扱う
GPIOの扱い方を解説する。

Caravelでは3種類のGPIOに関連した信号線が存在している。

| 信号名 | 説明 |
| ------ | ---- |
| io_in[37:0] | GPIO入力 |
| io_out[37:0]|GPIO出力 |
| io_oenb[37:0] |GPIO出力イネーブルバー。0で出力、1で入力。 |


io_oenbの設定はファームウェア側でどうとでもなるので基本的に全部0でいい。

使い方
```verilog
assign test_data = io_in[34:32];
assign io_out[31:0] = data_reg;
```

ピンによっては別の機能と同居している場合があるので、ここ[^26]のmprj_ioを参照した方が良い。

#### **割り込み**
次に割り込みについて説明する。CaravelではUser Project Areaから割り込み信号を出す事が可能である。

割り込みは、信号線`user_irq[2:0]`の内どのビットを立てても引き起こす事が(多分)可能である。 割り込みベクタはアドレス0である。正確な情報はこの動画[^27]以外に存在しない。

またサンプルコード[^28]を一度目を通すことを強くおすすめする。

以上を組み合わせてキミだけのデザインを自作しよう！！
