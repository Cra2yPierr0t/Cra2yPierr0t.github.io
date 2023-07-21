---
layout: default
title: 無から始める自作CPU
image: "https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/letsmakecpu.png"
---

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/letsmakecpu.png)

# 無から始める自作CPU(作成中)

**CPUは作れる！！！！！！！！ご存知でしたか！！！？？？？？？**

魔法の石、究極のブラックボックス、人類の英知たるCPUは不世出の天才にしか作れないとお思いですか？ **違います**！適切な知識さえあれば割と凡庸な人間でも作ることが出来ます！！！作りましょう！！

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

CPUは**ディジタル回路**というもので構成されています。ディジタル回路、難しそうな単語ですね。安心してください、ディジタル回路は単純なルールの集まりに過ぎません。少しずつ学んでいきましょう。

#### ディジタル回路が扱う値

ディジタル**回路**と呼ぶからには、ディジタル回路は電気的な何かを扱う回路なんだろうと思われてるかもしれません。正解です。ディジタル回路は電圧の高低を扱う回路です。具体的には0Vと5Vや、0Vと1.2Vなどを入力に取りますが、実際に何Vなのかは回路によってまちまちなので入力をHigh, Lowとするか、それすら見づらいし面倒なので電圧の高い入力を1、電圧の低い入力を0とする表記が一般的です。

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/digital.png)

では実際にどのようなディジタル回路が存在するのか見ていきましょう。

#### NOT

一番シンプルなディジタル回路、**NOTゲート**です。

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/not.png)

これは入力を反転します。例えば0が入力されたら1を出力し、1が入力されたら0を出力します。
以下にNOTの真理値表(そういうのがある)を置いておきます。

| `A` | `NOT A` |
| --  | --- |
| `0` | `1` |
| `1` | `0` |

記号としては`~x`とか`!x`とかが使われがちです。

#### OR

次は**ORゲート**です。これは入力の論理和、入力に一つでも1があるなら1を出力し、入力が全て0なら0を出力します。

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/or.png)

真理値表は以下の通りです。

| `A` | `B` | `A AND B` |
| - | - | ------- |
| `0` | `0` | `0` |
| `0` | `1` | `1` |
| `1` | `0` | `1` |
| `1` | `1` | `1` |

記号としては`a | b`が使われがちです。

#### AND

次は**ANDゲート**です。これは入力の論理積、入力の全てが1なら1を出力し、それ以外なら0を出力します。

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/and.png)

真理値表は以下の通りです。

| `A` | `B` | `A AND B` |
| - | - | ------- |
| `0` | `0` | `0` |
| `0` | `1` | `0` |
| `1` | `0` | `0` |
| `1` | `1` | `1` |

記号としては`a & b`が使われがちです。そのまんまですね。

#### NAND

次は**NANDゲート**、これはANDゲートの出力にNOTしたものです、入力の全てが1なら0を出力し、それ以外なら1を出力します。

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/nand.png)

真理値表は以下の通りです。

| `A` | `B` | `A NAND B` |
| - | - | ------- |
| `0` | `0` | `1` |
| `0` | `1` | `1` |
| `1` | `0` | `1` |
| `1` | `1` | `0` |

記号としては`~(a & b)`で表せます。少し難易度が上がりましたかね？

NANDゲートの面白い特徴として、NANDゲートから他の全ての論理回路を構成できるという性質があります(Functional completeness)。実際にNANDゲートからORゲートを作るとこんな感じになります。

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/or_nand.png)

本当にORゲートになってるかわからない？真理値表を書くとこの通り、ORゲートになっている事が分かります。

| `A` | `B` | `A NAND A` | `B NAND B` | `(A NAND A) NAND (B NAND B)` |
| - | - | -------- | -------- | -------------------------- |
| `0` | `0` |       `1`  |        `1` | `0`  |
| `0` | `1` |       `1`  |        `0` |                         `1`  |
| `1` | `0` |       `0`  |        `1` |                         `1`  |
| `1` | `1` |       `0`  |        `0` |                         `1`  |

このNANDゲートから他のディジタル回路を作り、それらを組み合わせCPUを作り、OSをその上で動かして、そのOSの上でテトリスを動かそうぜ！という熱い本が存在します。買うといいです。

[コンピュータシステムの理論と実装](https://amzn.asia/d/bmBqWAs)

#### XOR

論理回路ラスト、**XORゲート**です。これは**排他的論理和**と呼び、入力が異なる場合に1を出力し、同じ場合に0を出力します。

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/xor.png)

真理値表は以下の通りです。

| `A` | `B` | `A XOR B` |
| - | - | ------- |
| `0` | `0` | `0` |
| `0` | `1` | `1` |
| `1` | `0` | `1` |
| `1` | `1` | `0` |

記号としては`a ^ b`が使われがちです。

#### MUX

少し発展したディジタル回路、**MUX(マルチプレクサ)**です。これは入力を選択する回路です。選択用信号Sが0の場合はAからの入力を出力し、Sが1の場合はBからの入力を出力します。

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/mux.png)

真理値表は以下の通りです。

| `A` | `B` | `S` | `Y` |
| - | - | - | - |
| `0` | `0` | `0` | `0` |
| `0` | `0` | `1` | `0` |
| `0` | `1` | `0` | `0` |
| `0` | `1` | `1` | `1` |
| `1` | `0` | `0` | `1` |
| `1` | `0` | `1` | `0` |
| `1` | `1` | `0` | `1` |
| `1` | `1` | `1` | `1` |

このマルチプレクサ、次のようにNOTとANDとORで作ることが出来ます。`Y = (A & S) | (B & ~S)`。

#### HalfAdder

次はディジタル回路で少し面白い回路を紹介します。HalfAdder、**半加算器**です。以下のANDとXORで構成された回路を見てください。

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/halfadder.png)

この回路の真理値表を書いてみましょう。以下の通りです。

| `A` | `B` | `C` | `S` |
| --- | --- | --- | --- |
| `0` | `0` | `0` | `0` |
| `0` | `1` | `0` | `1` |
| `1` | `0` | `0` | `1` |
| `1` | `1` | `1` | `0` |

CがAND回路の出力でSがXOR回路の出力になっています。当然っちゃ当然ですね。ですがよく見てください、このCとS、AとBの二進数の加算になっていますね。
ちなみにSは和を意味するSumの略で、Cは繰り上がりを意味するCarryの略です。

上記の表と下の数式を見比べればよく分かります。

$$
\begin{align}
0_{(2)} + 0_{(2)} &= 00_{(2)} \\
0_{(2)} + 1_{(2)} &= 01_{(2)} \\
1_{(2)} + 0_{(2)} &= 01_{(2)} \\
1_{(2)} + 1_{(2)} &= 10_{(2)}
\end{align}
$$

この様に加算と同じ動作をする回路のことを**加算器**と呼びます。覚えておきましょう。

#### FullAdder

さて、半加算器は二進数の1桁同士の加算を行う回路でした。更に2桁、10桁の加算を行いたい時、この半加算器を並べるだけでは複数桁の加算を行う事は出来ません。
加算をやった事のある皆さんはご存知でしょうが、加算は**繰り上がり**が発生する演算です。よって、この半加算器を繰り上がり入力を受け取るように拡張する必要があります。
その回路を**全加算器**、FullAdderと呼びます。実際に見てみましょう。

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/fulladder.png)

半加算器２つとORで構成されていますね。新しく追加された全加算器の入力のXは繰り上がり入力を意味します。真理値表を見てみましょう。

| `A` | `B` | `X` | `C` | `S` |
| --- | --- | --- | --- | --- |
| `0` | `0` | `0` | `0` | `0` |
| `0` | `0` | `1` | `0` | `1` |
| `0` | `1` | `0` | `0` | `1` |
| `0` | `1` | `1` | `1` | `0` |
| `1` | `0` | `0` | `0` | `1` |
| `1` | `0` | `1` | `1` | `0` |
| `1` | `1` | `0` | `1` | `0` |
| `1` | `1` | `1` | `1` | `1` |

入力のA, B, Xと出力のC, Sを下の二進数の加算の式を見比べてみましょう。一致していますね。

$$
\begin{align}
0_{(2)} + 0_{(2)} + 0_{(2)} &= 00_{(2)} \\
0_{(2)} + 0_{(2)} + 1_{(2)} &= 01_{(2)} \\
0_{(2)} + 1_{(2)} + 0_{(2)} &= 01_{(2)} \\
0_{(2)} + 1_{(2)} + 1_{(2)} &= 10_{(2)} \\
1_{(2)} + 0_{(2)} + 0_{(2)} &= 01_{(2)} \\
1_{(2)} + 0_{(2)} + 1_{(2)} &= 10_{(2)} \\
1_{(2)} + 1_{(2)} + 0_{(2)} &= 10_{(2)} \\
1_{(2)} + 1_{(2)} + 1_{(2)} &= 11_{(2)} 
\end{align}
$$

この全加算器を用いる事で、以下のように複数桁の加算を作ることが可能となります。

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/adder4.png)

二進数の4桁の加算
$$
\begin{align}
0011_{(2)} + 1001_{(2)} &= 1100_{(2)} \\
0010_{(2)} + 0011_{(2)} &= 0101_{(2)}
\end{align}
$$

この例は4桁の加算器ですが、8桁でも32桁でも作ることが出来ます。この世の足し算はこの回路で実行されているんですね。

#### D-FF

さてD-FF、謎の物体が出てきました。高校数学でANDやORをなんか見たことあるなあ、という人もD-FFは高校数学で学びません。これは**D型フリップフロップ**というものです。

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/dff.png)

突然ですが、回路が情報を記憶する為には何が必須だと思いますか？それは**時間**です。時間の概念が無ければ記憶は意味を持ちえません。という訳で回路に時間の概念を導入します。
ディジタル回路における時間の概念、それは**クロック**です。

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/clock.png)

クロックとは一定の周波数で0と1を行ったり来たりする信号を指します。クロックに関係する用語として、0から1になる**立ち上がりエッジ(posedge)**と、1から0になる**立ち下がりエッジ(negedge)**がありますので覚えておきましょう。

クロックを理解した所でこのD-FF、これは信号を記憶する論理素子です。具体的な動作としては、クロックの立ち上がりエッジで入力を出力し、その他のタイミングでは出力を維持します。図にすると分かりやすいです。

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/dff_wave.png)

このフリップフロップ、別名レジスタ(register)とも呼ばれる事があります、覚えておいて損はないです。

#### MUXによるD-FFの改良

最後にこのD-FFをMUXで少し改良して、データを記憶するタイミングを制御できるようにしましょう。

以下では、MUXの`w_en`でD-FFへの入力を、D-FFからの出力か外部からの入力`w_data`かを選択できるようにしています。

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/dff_improve.png)

波形は以下の通りです。`w_en`でD-FFに対する書き込みを制御出来ていますね。

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/dff_improve_wave.png)

以上で我々はディジタル回路の基礎を完全に理解することが出来ました。やったね。次はこのディジタル回路を作り出せる、FPGAの紹介です。

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

## 自作CPU入門

さあ始めましょう

### プログラムのおしごと

### プログラムが動く流れ

CPUはプログラムをどのように実行しているのでしょうか？CPUはプログラムをそのまま実行している訳ではありません。

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/prog_flow.png)

現代では、プログラムは**コンパイラ**というソフトウェアによって**アセンブリ**という中間言語に変換され、そのアセンブリは**アセンブラ**というソフトウェアによって`011010101...`のようなバイナリに変換されます。
CPUとってプログラムはこのバイナリになって初めて実行可能な形式になるというわけです。

### 命令セットアーキテクチャ 

#### RISC-Vとは

### コンピュータアーキテクチャ概論

ここではCPUの内部構造、つまりアーキテクチャについて解説していきます。

以下の図は非常に単純化したCPUのアーキテクチャです。これをベースに学んでいきましょう。

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/computer_abst.png)

#### ディジタルビルディングブロック

ここではCPUの各モジュールについて、その働きを説明していきます。

##### Instr Mem

まずは一番左にあるモジュール、**Instr Mem**です。これは**命令メモリ(Instruction Memory)**と呼び、中に命令が入っています。命令メモリ内の各命令にはアドレスが振られており、命令メモリにアドレスを入力すると命令が出力されます。

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/instr_mem.png)

##### PC

次はCPUの中を見ていきましょう。このCPUの中にある**PC**は**プログラムカウンタ(Program Counter)**と呼び、命令メモリにアドレスを供給します。通常、プログラムは上から下に実行されるのでプログラムカウンタが出す値は毎クロック増えていきます。

プログラムカウンタ・命令メモリ・CPUの関係をまとめると、プログラムカウンタがアドレスを命令メモリに入力し、命令メモリが命令をCPUへを出力するという流れになります。このデータの流れを**フェッチ(Fetch)**と呼びます。覚えておきましょう。

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/fetch.png)

##### Decoder

PCの次は**Decoder**です。これは**デコーダ**と呼び、命令から各種制御信号を生成します。具体的な動作の説明は他の部品の説明をしてから行いますので、とりあえず今は命令に応じて各部品を制御するモジュールだと認識しておいてください。

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/decoder.png)

##### Register File

**Register File**、これは**レジスタファイル**と呼び、CPUが少量のデータを保持するのに使います。

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/regfile.png)

動作としてはアドレスを入力するとデータを出力する**読み出し**と、

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/regfile_read.png)

アドレスとデータを入力するとレジスタファイルに書き込まれる**書き込み**を行います。

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/regfile_write.png)

##### ALU

次はALUです。これは**算術論理演算ユニット(Arithmetic Logic Unit)**の略称で、文字通り論理演算(and, or, xor, シフト, etc..)と算術演算(加算, 減算, etc...)を行うモジュールです。CPUにおいて"計算"はこのALUが行っていると考えていただいてかまいません。ALUは制御信号と２つのデータを受け取り、１つのデータを出力します。

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/alu.png)

##### Data Mem

最後に一番右にあるモジュール**Data Mem**、これは**データメモリ**と呼び。CPUが大量のデータを保存するのに使います。データメモリ対して、CPUはデータの書き込みと読み出しを行います。

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/data_mem.png)

メモリに対してのデータ書き込みと読み出しにはそれぞれ名前があり、メモリへのデータ書き込みを**ストア(Store)**、

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/data_mem_store.png)

メモリからのデータ読み出しを**ロード(Load)**と呼びます。Store命令、Load命令とよく使われる単語ですので覚えておきましょう。

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/data_mem_load.png)

#### データパス

##### Load命令

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/path_load.png)

##### Store命令

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/path_store.png)

##### 算術命令

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/LetsMakeCPU/path_exe.png)

##### ジャンプ命令
##### 分岐命令

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
