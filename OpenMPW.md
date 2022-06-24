---
layout: default
title: OpenMPW
---
# OpenMPW

質問、修正案、その他連絡は@Cra2yPierr0tマデ

[https://www.skywatertechnology.com/mpw/open-source-mpw-program/](https://www.skywatertechnology.com/mpw/open-source-mpw-program/)
---

## OpenMPWとOpenLANEとSkywaterPDKの概要
### OpenMPWの概要
* 無料でLSIを焼いてくれるプロジェクト
* Googleがefablessに出資して実現した
* Caravel(RISC-V+各種IO)に自分のデザインを加える形で実装する
* 提出された中から40デザインがランダムに選ばれる

### 必要要件
* Skywater PDKの使用
* オープンなデザイン
* ライセンス諸々
* 再現可能なGDSIIレイアウト
* Caravelの使用
* プリチェックツールでの合格

### まとめ
1. Caravelをforkして改造
2. プリチェックツールで通るならおｋ(おｋでない場合もたまにある)
3. EfablessにGDSIIファイルを送りつける
4. 嬉しい！

### 各種リンク
* [OpenLANE](https://github.com/efabless/openlane) : RTLからGDSII吐くやつ
* [Caravel](https://github.com/efabless/caravel) : パッドフレーム
* [Caravel User Project](https://github.com/efabless/caravel_user_project)
* [プリチェックツール](https://github.com/efabless/open_mpw_precheck)
* [MPW-ONE](https://efabless.com/projects/shuttle_name/MPW-ONE)
* [MPW-TWO](https://efabless.com/projects/shuttle_name/MPW-TWO)
* [MPW-3](https://efabless.com/projects/shuttle_name/MPW-3)

---

## LSIを焼こう！[^21]
**オレオレLSIを焼きたいですよね？ 焼きましょう。** Skywater社がPDKを公開し、OSSなGDSIIコンパイラである**OpenLANE**も生まれました。そしてGoogleの出資によってEfablessが無料でLSIを作らせてくれるプログラム、**Open MPW Shuttle Program**をスタートしました。
今こそオレオレLSIを焼くチャンスです。あなたの作りたいチップをGoogleの金で作りましょう！


---

概要はefablessの動画[^8]を見れば大体わかる。また使うツールのバージョンやCaravelの概要はこのドキュメント[^12]に書いてある。また大抵の問題はSkywaterのSlackに質問を飛ばすと解決するので参加をオススメします。

Analogも対応出来るみたいだけど需要があったら書く

### 必要なもの

* Caravel_user_project  : これのUser Project Areaに自分のデザインを加える
* Skywater PDK  : Process Design Kit、素材の情報みたいななやつ
* OpenLANE : RTLをGDSII[^16]に変換するコンパイラ(Synopsysとか持ってるのなら必要ない)
* 自分のデザイン
* 時間、精神力 : いっぱい

### 1.各種インストール & ビルド

#### 1.Caravelのインストール
基本的に、forkしたCaravel_user_projectのリポジトリ[^11]上で全てを進めていく
```shell
git clone git@github.com:efabless/caravel_user_project.git
cd caravel_user_project
make install
```

#### 2.PDKのビルド
5分くらい掛かる、PDK_ROOTはお好みで設定してよい。
```shell
mkdir ~/pdks
export PDK_ROOT=~/pdks
cd caravel_user_project
make pdk
```

#### 3.OpenLANEのインストール
OpenLNAEのDockerイメージが存在しているのでインストールする、Dockerは事前にインストールしてデーモンを起動しておく。OPENLANE_ROOTはお好みで設定してよい。

```shell
mkdir ~/openlane
export OPENLANE_ROOT=~/openlane
cd caravel_user_project
make openlane
```

以上で必要なツールは揃いました。

### 2.デザインを自作する

8-bit CPUをCaravelに載せた時のサンプルを置いておきます。適宜参照するとよろし。
https://github.com/cpu-dev/caravel_jacaranda-8/tree/submission-mpw-3/verilog/rtl

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
```verilog=
assign test_data = io_in[34:32];
assign io_out[31:0] = data_reg;
```

ピンによっては別の機能と同居している場合があるので、ここ[^26]のmprj_ioを参照した方が良い。

#### **割り込み**
次に割り込みについて説明する。CaravelではUser Project Areaから割り込み信号を出す事が可能である。

割り込みは、信号線`user_irq[2:0]`の内どのビットを立てても引き起こす事が(多分)可能である。 割り込みベクタはアドレス0である。正確な情報はこの動画[^27]以外に存在しない。

またサンプルコード[^28]を一度目を通すことを強くおすすめする。

以上を組み合わせてキミだけのデザインを自作しよう！！

### 3.デザインをCaravelに載せる[^3][^13]

基本的に以下の流れで載せる
1. `caravel_user_project/verilog/rtl/<my_design>`に自分のデザインを入れる
2. `caravel_user_project/openlane/<my_design>/config.tcl`を編集する
3. `caravel_user_project/openlane/user_project_wrapper/config.tcl`を編集する

#### 1.`caravel_user_project/verilog/rtl/<my_design>`に自分のデザインを入れる

まず`/verilog/rtl/`以下に自分のデザインのディレクトリを作成する。そのディレクトリに自分のデザイン(Verilogファイルとか)を入れる。

#### 2. `caravel_user_project/openlane/<my_design>/config.tcl`を編集する
wrapperに自分のデザインを挿入する前に一旦OpenLANEで.gdsと.lefを生成する必要がある。そのために`config.tcl`を、caravelの`openlane/<my_design>/`以下に作る。

以下をたたき台を置いておくので、下の表に従って編集するとよい
```tcl
set script_dir [file dirname [file normalize [info script]]]

set ::env(ROUTING_CORES) 8 #これで配線に使うスレッド数を指定できる

set ::env(DESIGN_NAME) computer

set ::env(DESIGN_IS_CORE) 0
set ::env(FP_PDN_CORE_RING) 0
set ::env(GLB_RT_MAXLAYER) 5

set ::env(VERILOG_FILES) "\
    $::env(CARAVEL_ROOT)/verilog/rtl/defines.v \
    $script_dir/../../verilog/rtl/<my_design>/module1.v \ 
    $script_dir/../../verilog/rtl/<my_design>/module2.v"

set ::env(CLOCK_PORT) wb_clk_i
set ::env(CLOCK_PERIOD) 10

set ::env(FP_SIZING) absolute
set ::env(DIE_AREA) "0 0 2000 2000"

set ::env(FP_PIN_ORDER_CFG) $script_dir/pin_order.cfg

set ::env(VDD_NETS) [list {vccd1}]
set ::env(GND_NETS) [list {vssd1}]
```


| 変数          | 意味                 |
| ------------- | -------------------- |
| ROUTING_CORES | 配線に使うスレッド数 |
| DESIGN_NAME   | 合成するトップデザインの名前 |
| VERILOG_FILES | 合成するVerilogファイル, defines.vは残しておく |
| CLOCK_PORT | トップへのクロック入力ポート, 複数指定できる |
| DIE_AREA | 使うダイの大きさ、最大 0 0 2920 3520 |

以上の設定を終えて
```bash
make <my_design>
```
をuser_project_wrapper以下で実行するとgds/に自分のデザインのGDSIIファイルが生えてるのでKlayoutとかでオレオレLSIのレイアウトが見られる。

#### 3. `caravel_user_project/openlane/user_project_wrapper/config.tcl`を編集する

`user_project_wrapper.v`のみを編集する場合は特に何もしなくてよいが、自作のデザインは複数のVerilogファイルから構成される場合が殆どなので、GDSIIに変換する前にVerilogファイル達をOpenLANEに教えてやる必要がある。そのために`config.tcl`を編集する。[^5] 

まず、`caravel_user_project/openlane/user_project_wrapper/config.tcl`の以下の変数を編集する。

```tcl
set ::env(VERILOG_FILES_BLACKBOX) " \
      $script_dir/../../caravel/verilog/rtl/defines.v \
      $script_dir/../../verilog/rtl/user_proj_example.v"

set ::env(EXTRA_LEFS) " \
   $script_dir/../../lef/user_proj_example.lef"

set ::env(EXTRA_GDS_FILES) " \
   $script_dir/../../gds/user_proj_example.gds"
```

* `VERILOG_FILES_BLACKBOX`には`user_project_example.v`を消して自分のVerilogファイル群を書き込む。
* `defines.v`はそのままでよい。
* `EXTRA_LEFS`と`EXTRA_GDS_FILES`には`user_proj_example.(lef/gds)`を消して、代わりに2で生成したGDSIIとLEFファイルのパスを書き込む。[^20]

最後に`MACRO_PLACEMENT_CFG`の行を削除する。
```cpp
set ::env(MACRO_PLACEMENT_CFG) $script_dir/macro.cfg
```

以上が完了して、
```bash
make user_project_wrapper
```
を実行するとgds/にラッパーのGDSIIファイルが生えてる、これが最終的にテープアウトされる。
### 4.テストベンチ/ファームウェアを書く

オレオレLSIが完成したら、それが正しく動作するか確かめる必要がある。そういうわけでテストベンチ又はファームウェアの書き方を解説する。サンプルは`verilog/dv/`以下に色々入っているので参考にするとよい。

`verilog/dv/`以下で作業を行う。

#### General
ここではCaravelでファームウェア又はテストベンチの実行方法を解説する。

コンパイルとシミュレーション環境に関してはDockerイメージが提供されている。
```shell=
docker pull efabless/dv_setup:latest
```
環境変数の設定
```shell
export PDK_PATH=<pdk-location/sky130A>
export CARAVEL_ROOT=<caravel_root>
export UPRJ_ROOT=<user_project_root>  CARAVEL_ROOTと同じ
```
コンテナをスタート
```shell
docker run -it -v $CARAVEL_ROOT:$CARAVEL_ROOT -v $PDK_PATH:$PDK_PATH -v $UPRJ_ROOT:$UPRJ_ROOT -v $PDK_ROOT:$PDK_ROOT -e CARAVEL_ROOT=$CARAVEL_ROOT -e PDK_PATH=$PDK_PATH -e UPRJ_ROOT=$UPRJ_ROOT -e PDK_ROOT=$PDK_ROOT -u $(id -u $USER):$(id -g $USER) efabless/dv_setup:latest
```
dvディレクトリに移動
```shell
cd $UPRJ_ROOT/verilog/dv/
```
テストが入ったディレクトリでmakeしてシミュレーションを走らせる。
```shell
cd <dv-test>
make
```
`SIM=GL make`でゲートレベル[^24]のシミュレーションも出来る。
```shell
cd <dv-test>
SIM=GL make
```

ここまで解説してもそもそもプログラムとかテストベンチとかMakefileの書き方とか何もわからないと思うので、以降ではそれぞれ解説していく。

#### Makefileの書き方

他のdv以下のディレクトリからコピペでいい。ただ43行目あたりに`PATTERN = 〜`と書いてある行があるので好きな名前に変えるとよい。

#### **テストベンチのたたき台を作る**

テストベンチを書くための大枠をここでは紹介する。基本はそれに書き加えれば大抵のことは出来るはず。またMakefileを改造したりして自作のライブラリとかも利用出来る。

以下に基本的なテストベンチを載せる。これはCaravelにクロックを70000サイクル流して、波形ファイルを出力する。
##### verilog
```verilog
`default_nettype none

`timescale 1 ns / 1 ps

`include "uprj_netlists.v"
`include "caravel_netlists.v"
`include "spiflash.v"

module my_design_tb;
	reg clock;
	reg RSTB;
	reg CSB;
	reg power1, power2;
	reg power3, power4;

	wire gpio;
	wire [37:0] mprj_io;

	// External clock is used by default.  Make this artificially fast for the
	// simulation.  Normally this would be a slow clock and the digital PLL
	// would be the fast clock.

	always #12.5 clock <= (clock === 1'b0);

	initial begin
		clock = 0;
	end

	initial begin
        $dumpfile("my_design_tb.fst");
        $dumpvars(0, my_design_tb);

		// Repeat cycles of 1000 clock edges as needed to complete testbench
		repeat (70) begin
			repeat (1000) @(posedge clock);
			    $display("+1000 cycles");
		end
		$finish;
	end

	initial begin
		RSTB <= 1'b0;
		CSB  <= 1'b1;		// Force CSB high
		#2000;
		RSTB <= 1'b1;	    	// Release reset
		#170000;
		CSB = 1'b0;		// CSB can be released
	end

	initial begin		// Power-up sequence
		power1 <= 1'b0;
		power2 <= 1'b0;
		power3 <= 1'b0;
		power4 <= 1'b0;
		#100;
		power1 <= 1'b1;
		#100;
		power2 <= 1'b1;
		#100;
		power3 <= 1'b1;
		#100;
		power4 <= 1'b1;
	end

	wire flash_csb;
	wire flash_clk;
	wire flash_io0;
	wire flash_io1;

	wire VDD3V3 = power1;
	wire VDD1V8 = power2;
	wire USER_VDD3V3 = power3;
	wire USER_VDD1V8 = power4;
	wire VSS = 1'b0;

	caravel uut (
		.vddio	  (VDD3V3),
		.vssio	  (VSS),
		.vdda	  (VDD3V3),
		.vssa	  (VSS),
		.vccd	  (VDD1V8),
		.vssd	  (VSS),
		.vdda1    (USER_VDD3V3),
		.vdda2    (USER_VDD3V3),
		.vssa1	  (VSS),
		.vssa2	  (VSS),
		.vccd1	  (USER_VDD1V8),
		.vccd2	  (USER_VDD1V8),
		.vssd1	  (VSS),
		.vssd2	  (VSS),
		.clock	  (clock),
		.gpio     (gpio),
        .mprj_io  (mprj_io),
		.flash_csb(flash_csb),
		.flash_clk(flash_clk),
		.flash_io0(flash_io0),
		.flash_io1(flash_io1),
		.resetb	  (RSTB)
	);

	spiflash #(
		.FILENAME("jacaranda_test.hex")
	) spiflash (
		.csb(flash_csb),
		.clk(flash_clk),
		.io0(flash_io0),
		.io1(flash_io1),
		.io2(),			// not used
		.io3()			// not used
	);

endmodule
`default_nettype wire
```
##### **ファームウェア**
以下はIOをコンフィグしたり、0x30000004に0xDEADBEEFを書き込んだりLogic Analyzerの[31:0]に1を入力したり、色々適当な事をやるファームウェアの例。各インターフェースに対する詳しいファームウェアの書き方をこの下から解説する。
```c
#include "verilog/dv/caravel/defs.h"
#include "verilog/dv/caravel/stub.c"

#define BASE_ADDR 0x30000000

static void
write(uint32_t addr, uint32_t val)
{
    *(volatile uint32_t *)addr = val;
}

void
reset()
{
    reg_mprj_io_37 = GPIO_MODE_MGMT_STD_OUTPUT;
    reg_la0_iena = 0;
    reg_la0_oenb = 0;

    reg_la0_data = 0;
    reg_mprj_xfer = 1;
    while (reg_mprj_xfer == 1);
}

void
main()
{
    reg_spimaster_config = 0xa002;

    reset();

    reg_la0_data = 1;

    write(BASE_ADDR+4, 0xDEADBEEF);

    reg_la0_data = 0;

    while(1) {}
}
```

#### User Project Areaから外部へ出ているGPIO

##### Configuration
このGPIOは38本存在するが、それぞれにコンフィグ用のレジスタ(グローバル変数として扱える)`reg_mprj_io_0`~`reg_mprj_io_37`が存在する。それぞれに以下の値を設定出来る。

| 値   | 説明 |
| ---- | ---- |
|  GPIO_MODE_MGMT_STD_INPUT_NOPULL  | SoCへの入力に設定(謎) |
|  GPIO_MODE_MGMT_STD_INPUT_PULLDOWN| SoCへの入力に設定(プルダウン) |
|  GPIO_MODE_MGMT_STD_INPUT_PULLUP  | SoCへの入力に設定(プルアップ) |
|  GPIO_MODE_MGMT_STD_OUTPUT        | SoCからの出力に設定 |
|  GPIO_MODE_MGMT_STD_BIDIRECTIONAL | SoCの入出力に設定 |
|  GPIO_MODE_MGMT_STD_ANALOG        | 謎 |
|  GPIO_MODE_USER_STD_INPUT_NOPULL  | 自作デザイン側への入力に設定(謎) |
|  GPIO_MODE_USER_STD_INPUT_PULLDOWN| 自作デザイン側への入力に設定(プルダウン) |
|  GPIO_MODE_USER_STD_INPUT_PULLUP  | 自作デザイン側への入力に設定(プルアップ) |
|  GPIO_MODE_USER_STD_OUTPUT        | 自作デザイン側からの出力に設定 |
|  GPIO_MODE_USER_STD_BIDIRECTIONAL | 自作デザイン側との入出力に設定 |
|  GPIO_MODE_USER_STD_OUT_MONITORED | 謎 |
|  GPIO_MODE_USER_STD_ANALOG        | 謎 |

こんな感じでコンフィグする。
```c
reg_mprj_io_0 = GPIO_MODE_USER_STD_OUTPUT;
reg_mprj_io_1 = GPIO_MODE_USER_STD_INPUT_PULLUP;
```

##### Activation

GPIOを有効にするためのグローバル変数`reg_mprj_xfer`が存在する。これをHighにすれば上記の設定内容が適用され、GPIOが有効になるが、その際に以下の記述が必要である[^22]。
```c
 reg_mprj_xfer = 1;
 while (reg_mprj_xfer == 1);
```
一見無限ループだが、謎のテクノロジーでループを抜けるので機械的にコピペすればいい。

##### Data
GPIOを`GPIO_MGMT_MODE_STD_*`に設定した場合にのみ(未検証)、以下のレジスタを介してGPIOからSoC側にデータを入出力することが可能である。
* `reg_mprj_datal` : 最初の32本の値を入出力
* `reg_mprj_datah` : 残りの6本の値を入出力

#### User Project AreaとSoCをつなぐLogic Analyzer線
User Project AreaとSoCは128本のLogic Analyzer線(以下LA線)で接続されており、名前とは裏腹にデータのやり取りが可能である。

##### Configuration
各LA線の入出力は`reg_la0_oenb` ~ `reg_la3_oenb`と`reg_la0_ienb` ~ `reg_la3_oenb`という各32ビットのレジスタで設定できる。1でSoC側への入力、0でSoC側から出力と考えればよい[^23]。
```c
//  LA probes [31:0], [127:64] をCPUへの入力に設定 
// Configure LA probes [63:32] をCPUからの出力に設定
reg_la0_oenb = reg_la0_iena = 0xFFFFFFFF;    // [31:0]
reg_la1_oenb = reg_la1_iena = 0x00000000;    // [63:32]
reg_la2_oenb = reg_la2_iena = 0xFFFFFFFF;    // [95:64]
reg_la3_oenb = reg_la3_iena = 0xFFFFFFFF;    // [127:96]
```
一括だけでなく、各ビット毎に設定可能

##### Data
LA線を介したデータのやり取りは`reg_la0_data` ~ `reg_la3_data`というレジスタを介して行うことが出来る。以下の例では、上記の設定でLA線の`[63:32]`を出力に設定したため、SoC側からUser Project Areaへ値`0xdeadbeef`を出力している。
```c
buffer0 = reg_la0_data;    // [31:0]
reg_la1_data = 0xdeadbeef; // [63:32]
buffer2 = reg_la2_data;    // [95:64]
buffer3 = reg_la3_data;    // [127:96]
```
このLA線を使ってSoC側からUser Project Areaにいい感じに分周したクロックとかリセットを入力出来たりするらしい。需要があったら書く(つかれた)。

ここ[^15]らへんを読めば出来る

### 5.GDSIIのビルド[^4]

以下を実行すればgds以下にGDSIIが生える。
```shell
make <my_design>
make user_project_wrapper
```
### 6.プリチェックツールを走らせる[^9][^17]
以下のコマンドを実行する
```shell
make precheck // プリチェックツールのインストール
make run-precheck // プリチェックツールを走らせる
```
既にプリチェックツールがインストールされている場合は1つ目は必要ない、これで全部パスしたなら殆どの場合Tapeout出来る。

### footnote
[^1]:https://caravel-harness.readthedocs.io/en/latest/memory-mapped-io-summary.html
[^2]:実装を辿ってったら謎のマクロが生えてたので置いとく https://github.com/efabless/caravel-lite/blob/13f2590e4b3a74b910dac56a6b757f5a66fd5212/verilog/rtl/chip_io.v#L229
[^3]: https://github.com/efabless/caravel_user_project/blob/main/docs/source/index.rst#caravel-integration
[^4]: https://github.com/efabless/caravel_user_project/blob/main/docs/source/index.rst#hardening-the-user-project-macro-using-openlane
[^5]:https://caravel-harness.readthedocs.io/en/latest/quick-start.html#configuration
[^6]:https://github.com/efabless/openlane/tree/master/configuration
[^7]:https://twitter.com/Cra2yPierr0t/status/1427088118025392131
[^8]: https://youtu.be/jBrBqhVNgDo?t=1326
[^9]: 明記されてないけどefablessの動画[^8]でそうしてました
[^10]:各ドキュメントではefablessのOpenLaneを使っているが、MPW2が終わった直後のためefabless版は更新が止まっており``make pdk`` でコケるのでOpenROADを一時的に使う。(多分SKYWATER_COMMITの値を更新してないだけ)
[^11]:https://github.com/efabless/caravel_user_project
[^12]:https://caravel-harness.readthedocs.io/en/latest/index.html
[^13]:https://caravel-harness.readthedocs.io/en/latest/quick-start.html#adding-a-user-project
[^14]:https://caravel-harness.readthedocs.io/en/latest/quick-start.html#adding-a-new-design
[^15]:https://github.com/efabless/caravel_user_project/tree/main/verilog/dv#simulation-environment-setup
[^16]:これをファブに投げるとLSIが生えてくる(らしい)
[^17]:あったわ https://github.com/efabless/caravel_user_project/blob/main/docs/source/index.rst#running-open-mpw-precheck-locally
[^18]:https://github.com/konradwilk/caravel_user_project/blob/submission-mpw-two/openlane/user_project_wrapper/config.tcl
[^19]:https://github.com/efabless/caravel/blob/master/docs/source/_static/caravel_harness.png
[^20]:つまり`user_project_wrapper`がモジュール`wrapper1.v`と`wrapper2.v`で出来ている場合、`EXTRA_LEFS`には`wrapper1.lef`, `wrapper2.lef`と書き換え、`EXTRA_GDS_FILES`には`wrapper1.gds`, `wrapper2.gds`と書き換えてやる。どうしてもわからない場合、これ[^18]が参考になる。
[^21]:気軽にLSIが焼けたらうれしいので
[^22]:シフトレジスタになってるからどうとか
[^23]:1nputと0utputで覚えやすいね！！！
[^24]:よく知らないけどRTL記述レベルではなくてffとか論理回路のレベルまで変換した上でのシミュレーションだと思う。必要性はよくわからない。
[^25]:https://github.com/efabless/caravel_user_project/blob/mpw-3/verilog/rtl/user_proj_example.v#L86
[^26]:https://caravel-harness.readthedocs.io/en/latest/pinout.html
[^27]:https://www.youtube.com/watch?v=pPgnVBguNW8
[^28]:https://github.com/efabless/caravel_user_project/blob/mpw-3/verilog/rtl/user_project_wrapper.v
