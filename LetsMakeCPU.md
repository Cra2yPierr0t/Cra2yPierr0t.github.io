---
layout: default
title: 無から始める自作CPU
image: "https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/letsmakecpu.png"
---

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/letsmakecpu.png)

# 無から始める自作CPU(作成中)
自作CPUはコンピュータサイエンスの全てを理解する第一歩！

この記事はコンピュータサイエンスやプログラミングの知識が無くても、CPUを自作出来る所までお前を持っていく事を目的としています。やっていきましょう。

## 基礎知識

ここではCPUを作るのに必要な知識を説明します。いくつかは知っているかもしれません、もしくはどれも初めて見る単語かもしれません。ここで全て覚えろとは言いません。とりあえずざっと読んで必要になったら読み返してください。

### CPU

CPU、我々が作る対象です。CPUはとは一体なんでしょうか？概要すら知らないのに作ろうとするのは流石に無謀と言えます。ちょっとだけ先に知っておきましょう。

プログラミングという単語は皆さん人生のどこかで聞いたことがあるでしょう。最近の若人は中学高校の授業で実際にプログラミングをした経験があるかもしれませんし、実務で散々やっている方もいるかもしれませんし、報道番組で「時代はプログラミングだ！」とか与太話を聞いた事があるかもしれません。プログラミング、プログラムを書いて画面に何かが出たり音が出たりモーターが動いたりするアレですね。あなたがこの記事を読んでいるSafariやChromeもプログラムですし、YoutubeもTwitterもInstagramもTiktokもプログラムです。ああ素晴らしきかなプログラム。プログラムが無ければお前は生きてはいけません。

```c
#include <stdio.h>
int main(){
    printf("Hello World!\n");
    return 0;
}
```
↑ 素晴らしいプログラム

ですが考えてみてください、そのプログラム、一体どこで動いているのでしょうか？説明できますか？ その答えがCPUです。

CPUとはCentral Processing Unit、中央演算装置の略で、石みたいなガラスみたいな物質で出来ています。見た目は大体こんな感じです。

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/cpu_image.jpeg)

この世に存在するプログラムは全てCPUの上で動いています。このCPUが無ければプログラムは動かせませんし、いくらプログラムを書いても意味がありません。我々の生活基盤はこの石に支えられているという訳ですね、ありがとうCPU。消えないでCPU。本記事ではそんなCPUを作ります。

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/world_of_cpu.png)

### ディジタル回路

### FPGA

* FPGA : ディジタル回路を内部に生成できる半導体
* HDL : ディジタル回路(RTL)を記述する言語の種類
* Quartus : Intel FPGAの開発環境 Vivadoの敵
* Vivado : Xilinx FPGAの開発環境 Quartusの敵

## Verilog HDL入門

この章ではVerilog HDLについて解説します。Verilog HDLとはハードウェア記述言語(Hardware Description Language)の一種で、ディジタル回路の設計・検証に用いられます。早い話**ハードウェア用のプログラミング言語**みたいなものです。その他のHDLとしてSystemVerilog, VHDL等が存在しています。

本書では、開発環境の快適さと対応ツールの多さからVerilog HDLを採用しています。

### 開発の流れ

ここでは本書での開発の流れを簡潔に説明します。以下が主な開発の流れです。

図にする
- シミュレーション：Verilog file -> シミュレータ -> 波形ビューワ
- 実機検証：Verilog file -> EDA -> FPGA or LSI

開発はシミュレーションと実機検証に分かれています。基本的に書いたコードの動作確認は上のシミュレーションで行い、現実世界での動作の確認は下の実機検証の流れで行います。

シミュレーションにおいて、利用するソフトウェアはシミュレータと波形ビューワの２つです。自分で書いたVerilogファイルをシミュレータに渡し、そしてシミュレータが出力した波形ファイルを波形ビューワで開き、動作を確認します。

実機検証では、利用するソフトウェアはEDA(Electronic Design Automation)のみです。書いたVerilogファイルをEDAに渡し、様々な設定を行った後ビルドし、出力ファイルをFPGAに与えて回路を生成します。/* 良くない表現 */

本書の大半の作業はシミュレーションで行います。実機検証に関しては環境構築も作業も非常に煩雑であるため、本書の後半で解説します。

### 開発環境を作ろう

初めに開発環境を構築していきましょう。大半の初心者は環境構築で躓く事が経験的に知られていますので、出来る限り丁寧に解説します。

#### テキストエディタのインストール

visual studio codeをインストールせえ

#### Verilog HDLシミュレータのインストール

icarus verilogとgtkwaveをインストールをせえ

### 開発環境に慣れよう

開発環境の構築が完了したら、次は実際に開発環境を触って開発の流れを体感しましょう。

#### Verilogファイルとテストベンチ

とりあえず以下のコードを`day01.v`として保存しましょう。
何を書いているのか分からなくて不安でしょうが、今は写経していただくだけで結構です。

```verilog
module adder8(
     input wire       clk,
     input wire [7:0] in0,
     input wire [7:0] in1,
     output reg [7:0] out
);
    always @(posedge clk) begin
        out <= in0 + in1;
    end
endmodule
```

次に以下のコードを`day01_tb.v`として保存して下さい。
```verilog
module addr8_tb;
    reg             clk = 1'b0;
    reg    [7:0]    in0 = 8'h00;
    reg    [7:0]    in1 = 8'h00;
    wire   [7:0]    out;
    
    initial begin
        $dumpfile("wave.vcd");
        $dumpvars(0, DUT);
    end
    
    always #1 begin
        clk <= ~clk;
    end
    
    adder8 DUT(
        .clk    (clk    ),
        .in0    (in0    ),
        .in1    (in1    ),
        .out    (out    )
    );
    
    initial begin
        #2
        in0 = 8'h05;
        in1 = 8'h04;
        #2
        in0 = 8'hff;
        in1 = 8'h01;
        #2
        $finish;
    end
endmodule
```

さて、今しがた`day01.v`と`day01_tb.v`というファイルを作りました。`day01.v`はVerilog HDLで書いたディジタル回路です。この他に`day01_tb.v`というファイルも書きましたね、Verilogをシミュレータ上で動かす為にはディジタル回路を記述したVerilogファイルの他に、**テストベンチ**と呼ばれるVerilogファイルが必要です。

テストベンチとは文字通り回路のテストを行うためのVerilogファイルです。例えば、あなたが何か回路を作成したとして、その回路が意図した通りに動作するか確かめたい場合、回路に入力を与え、その出力を調べる必要がありますね？テストベンチはそういった回路へ入力を与えたり、出力を確認したり、内部信号を検査したり、回路内の波形ファイルを出力したりするのに使います。

具体的なテストベンチの書き方は後々解説しますので、今はとりあえず手元でシミュレーションを動かしてみましょう。

#### シミュレーションを実行

以下のコマンドを実行してください。

```bash
$ iverilog day01_tb.v day01.v
$ vvp a.out
```

実行が完了したらディレクトリ内に`wave.vcd`というファイルが生成されている筈です。これは波形ファイルです。波形ファイルは以下のコマンドで開きます。

```bash
$ gtkwave wave.vcd
```

GTKwaveの使い方は気合で慣れてください。信号を選択してAppendボタンを押すと多分見れますので適当に触ってみてください。

以上の通り、Verilog HDLでディジタル回路とテストベンチを記述し、シミュレーションを実行。波形を見てデバッグ、そして再びシミュレーションを実行という流れになります。この流れは今後何度も繰り返します。今全てを覚える必要はありませんので大丈夫です、そのうち手癖で開発を回せるようになります。

### Verilogの文法を学ぼう(基礎１)

では本格的にVerilog HDLを学んでいきましょう。最初は必須の構文です。

#### module

Verilog HDLではシステム全体をモジュールという単位で分割して構成します。

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/module.png)

モジュールのイメージ

モジュール名は`module`の後ろに記述し、回路の内容は`module` ~ `endmodule`の間に記述します。

```verilog
module test_module();
 // ここに回路を記述
endmodule
```

#### 入出力ポート

回路には入出力が必要ですね、回路の入出力ポートは`input`, `output`で作成します。入力信号名は`input`の後ろに宣言し、出力信号名は`output`の後ろに宣言します。IOの個数や順番に決まりはありませんが、最後の宣言にはカンマが必要ないことに注意しましょう。

```verilog
module test_module (
    input    test_input1, // 入力信号線の定義
    input    test_input2,
    output   test_output1 // 出力信号線の定義
);
 // 回路
endmodule
```
その他に入出力両方に使える`inout`が存在しますが必須ではありませんので必要になった時に各自で調べてください。

#### wire型

次は`wire`です。人生のどこかで見た回路の上には配線がありましたね、あの配線は信号を伝達する信号線です。`wire`ではその信号線を生成します。

`wire`の使い方は以下の通りです。

```verilog
wire [31:0] 信号線名;
```

`wire`の後ろにある`[31:0]`は信号線の幅です。この場合は32本の幅を持った信号線になります。`[15:0]`にすれば16本の幅を持った信号線になりますし、`[3:0]`にすると4本の幅を持った信号線になります。

また、以下のように信号線の幅の記述を省略すると1本の幅を持った信号線になります。

```verilog
wire 信号線名;
```

#### 演算子

Verilog HDLには様々な演算子が存在します。これらの演算子は複数ビットを纏めて演算が可能です。以下によく使う演算子を載せておくきますので適宜参照してください。

#### assign文

`wire`で作られた信号線にはキーワード`assign`を用いて入力できます。`assign`を用いて書かれた回路は、入力が変化すると即座に出力が変化する**組み合わせ回路**となります。
以下のコードでは`test_output0`に`test_input`の値をそのまま入力しています。

```verilog
module test_module (
    input [9:0] test_input,
    output wire [9:0] test_output0,
    output wire [4:0] test_output1
);
    assign test_output0 = test_input;
    // 入力の上位5ビットを出力に直結している
    assign test_output1 = test_input[9:5]; 
endmodule
```
`test_output1`に入力している`test_input`の直後に書いてある`[9:5]`は**スライス**という操作であり、`test_input`の5ビットから9ビットの計5ビットを抜き出して、`test_output1`に出力しています。スライスはよく使う操作ですので覚えておきましょう。

#### 数値リテラル

Verilog HDLで数値を記述する際は、`ビット幅'進数 数値`という形式で記述します。`進数`は`b`で2進数、`d`で10進数、`h`で16進数です。`ビット幅`を指定しない場合は処理系によりけりです(Quartusでは32ビット)。
```verilog
    assign test_wire1 = 5'b10101; //2進5ビット
    assign test_wire2 = 8'd43;    //10進8ビット 
    assign test_wire3 = 16'hdead; //16進16ビット
```

### Verilogの文法を学ぼう(練習１)

覚えてばかりもつまらないでしょうし、ここで少し手を動かしてみましょう。

#### 練習問題１

以下の回路と同等のモジュール`problem1`を作成してください。入出力は画像の通りです。

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/problem1.png)

#### 模範解答１

```verilog
module problem1(
  input       [7:0] i_p0,
  input       [7:0] i_p1,
  input       [7:0] i_p2,
  output wire [7:0] o_p
);

    wire [7:0] w_p;

    assign w_p = i_p0 & i_p1;
    assign o_p = w_p | i_p2;

endmodule
```

### Verilogの文法を学ぼう(基礎２)

#### 定数
Verilogには定数として`define`と`parameter`が存在するが、`define`は定数の利用時に定数名の前に｀が必要でキモいので`parameter`を使うことを推奨する。
```verilog
module test_module(
    // 省略
);
    parameter TEISU1 = 123;
    parameter TEISU2 = 32'h5555_5555;
    parameter TEISU3 = 32;
    
    wire [TEISU3-1:0] w1;
    assign w1 = TEISU2;
endmodule
```

#### reg型
#### always文

過去及び現在の値から出力を決定する回路である**順序回路**を書く際には`always`文を用いる。型が`reg`の変数にはこの`always`文内でのみ入力が可能である。`@(posedge clk)`は信号`clk`が0から1に立ち上がった時のみ`always`文内の記述を実行する、という意味である。`negedge`の場合は1から0に立ち下がった場合という意味になる。
後述するが、`reg`変数に入力する際は`<=`と`=`で意味が変わってくるので注意。
接続するときは<=というイメージ
(多分out(t)=out(t-1)のように、代入先と元に時間差がある時には<=するのかな？)
```verilog
module test_module (
    input  wire clk,    // 入出力の信号線も種類を定義できる
    input  wire [4:0] test_input,
    output reg  [4:0] test_output
);
    
    always @(posedge clk) begin // 順序回路の定義
        test_output <= test_output + test_input;
    end
endmodule
```

#### if文

if文は順序回路及び後述するfunction文で記述可能であり、`begin`~`end`で処理を囲う。
```verilog
module test_module(
    // 省略
);
    // 省略

    always @(posedge clk) begin
        if (test_wire) begin
            test_output <= 1;
        end else begin
            test_output <= 0;
        end
    end
endmodule
```


#### function文

assignで書けない複雑な組み合わせ回路の場合、function文を利用する。`function`~`endfunction`で回路の内容を囲み、`function 返り値のビット幅 回路名(input 入力信号)`と記述する。functionの返り値はfunction名に入力する形で記述する。
```verilog
    assign test_wire = test_function(test_input);

    function [4:0] test_function(input [2:0] func_in);
    begin
        case(func_in)
            3'b000 : test_function = 5'b00001;
            3'b001 : test_function = 5'b10101;
            default : test_function = 5'b11111;
        endcase
    end
    endfunction
```

#### case文

case文は、always文とfunction文内で記述可能であり、`case`~`endcase`で囲んで利用する。`case`の後に条件に利用する信号名を記述し、コロン`:`の後ろにマッチした場合の処理を記述する。どの条件にも当てはまらない場合の処理は`default`の後ろに記述する。`default`を記述していない場合、処理系に依るが警告かエラーが出る。
```verilog
always @(posedge clk) begin
    case(test_input)
        5'b0000 : test_out <= 5'b10101;
        5'b0001 : test_out <= 5'b00110;
        5'b0010 : test_out <= 5'b11001;
        default : test_out <= 5'b11111;
    endcase
end
```

#### 三項演算子

三項演算子はいつでも使える(?)。
代入先 = (条件) ? (条件が真の時に代入したい値) :(条件が偽の時に代入したい値)
ネストも可能。

#### モジュール呼び出し

作成したモジュールは`モジュール名 インスタンス名`で生成する。またIOは`.IO名(信号線|変数)`で接続する。
※インスタンスは他言語でいうオブジェクトだと理解すればよい(これ説明いらない感じ？)(気になる人もいるだろうし残しとこう)
```verilog

module test_module(
    // 省略
);
    wire [5:0] w1;
    reg  [5:0] r1;
    wire [5:0] w2;
    
    small_module sm(
        .in1    (w1    ),
        .in2    (r1    ),
        .out1   (w2    )
    );
    
    small_module sm_danger(w1, r1, w2); //こうも書ける(危険)
endmodule
```

#### generate文
#### 体操：UARTで送信回路を作る

### FPGAでVerilogを動かす

### シミュレーション方法
シミュレーション用の関数と構文を説明した後、サンプルコードを載せる。

#### システムタスク
シミュレーション用の関数(システムタスク)の解説
* `$dumpfile` : 波形ファイルの名前設定
* `$dumpvars` : 内部信号を波形に出力するモジュールを設定
* `$display`  : 標準出力に信号を出力する
* `$finish` : シミュレーションを終了する(必須)

#### シミュレーション用構文
シミュレーションに使うverilogの構文の解説
* クロック生成
```verilog
always #1 begin
    clk <= !clk;
end
```
1秒毎にclkの値を反転させる。これをクロック信号として扱う
* 時間経過で入力を変化
```verilog
initial begin
    #2
    data_i <= 8'h05;
    #2
    data_i <= 8'h11;
end
```
`#2`で2秒待機した後`data_i`への入力を変化させている。この場合`clk`の反転は1秒毎なので`#2`で1クロック待機となる。

#### テストベンチの例
適当なモジュール例(インクリメントするだけのモジュール)
```verilog
module test (
    input   clk,
    input   [7:0] data_i,
    output  [7:0] data_o
);

    reg [7:0] data;

    always @(posedge clk) begin
        data <= data_i + 8'h1;
    end
    assign data_o = data;
endmodule
```
テストベンチの例
```verilog
module test_sim();
    reg             clk = 0;
    reg     [7:0]   data_i = 8'h00;
    wire    [7:0]   data_o;

    initial begin
        $dumpfile("wave.vcd");  // 波形ファイル
        $dumpvars(0, test);     // testインスタンスの全信号を出力
    end

    test test(
        .clk        (clk    ), 
        .data_i     (data_i     ),
        .data_o     (data_o     )
    );

    always #1 begin    // クロックの生成 10秒毎に反転
        clk <= !clk;
    end

    initial begin
        data_i  <= 8'h11;
        #2     // 20秒 (1 クロック)待機
        data_i  <= 8'h20;
        #2
        data_i  <= 8'h30;
        $display("data_o : %d\n", data_o); // 標準出力にも出力可能
        #10
        $finish;    // 終了
    end
endmodule
```

#### ツール解説
* icarus verilog : オープンソースのVerilogシミュレータ(Linux, Windows, Mac?)
    * iverilog : シミュレータ用のVerilogコンパイラ
    * vvp : シミュレーション用バーチャルマシン(実行環境)
* gtkwave : オープンソースの波形ビューワー

使い方
```bash
iverilog -o 出力ファイル名 -s トップモジュール名 Verilogファイル名
vvp 出力ファイル名
gtkwave 波形ファイル名
```
実行例
```bash
iverilog -s test_sim test_sim.v test.v 
vvp a.out
gtkwave wave.vcd &
```

## Quartus入門
### 論理合成
### ピンアサイン
### 書き込み

## プログラミング入門
CPUを作るために読んでるのに、なんでプログラミングを勉強しなければならないんだと思われるかもしれません。CPUとはプログラムを動かすために存在しています。ここで一度プログラミングを学び、これから作るものがどの様に使われるのか、確固たるイメージを持っておきましょう。

### プログラムが動く流れ
### ちいさなプログラムを書こう

## CPU自作入門

さあ始めましょう

### コンピュータアーキテクチャ概論

ここではCPUの内部構造、つまりアーキテクチャについて解説していきます。

以下の図は非常に単純化したCPUのアーキテクチャです。一つずつ見ていきましょう。

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/computer_abst.png)

#### Instr Mem

まずは一番左にある部品、**Instr Mem**です。これは命令メモリ(Instruction Memory)と呼び、中に命令が入っています。命令メモリ内の各命令にはアドレスが振られており、命令メモリにアドレスを入力すると命令が出力されます。

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/instr_mem.png)

#### PC

次はCPUの中を見ていきましょう。このCPUの中にある**PC**はプログラムカウンタ(Program Counter)と呼び、命令メモリにアドレスを供給します。通常、プログラムは上から下に実行されるのでプログラムカウンタが出す値は毎クロック増えていきます。

プログラムカウンタ・命令メモリ・CPUの関係をまとめると、プログラムカウンタがアドレスを命令メモリに入力し、命令メモリが命令をCPUへを出力するという流れになります。このデータの流れを**フェッチ(Fetch)**と呼びます。覚えておきましょう。

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/fetch.png)

#### Decoder

### ディジタルビルディングブロックを作る

## 自作CPUでプログラムを動かそう

## CPUを実用的にしよう
### MMIO
### 割り込み

## CPUを高速にしよう
### パイプライン化
### キャッシュ
### より高度なアーキテクチャ

## この先へ
