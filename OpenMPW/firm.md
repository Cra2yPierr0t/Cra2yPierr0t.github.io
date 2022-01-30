---
layout: default
title: 4.テストベンチ/ファームウェアを書く
---
### 4.テストベンチ/ファームウェアを書く

オレオレLSIが完成したら、それが正しく動作するか確かめる必要がある。そういうわけでテストベンチ又はファームウェアの書き方を解説する。サンプルは`verilog/dv/`以下に色々入っているので参考にするとよい。

`verilog/dv/`以下で作業を行う。

#### General
ここではCaravelでファームウェア又はテストベンチの実行方法を解説する。

コンパイルとシミュレーション環境に関してはDockerイメージが提供されている。
```shell
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
```c=
reg_mprj_io_0 = GPIO_MODE_USER_STD_OUTPUT;
reg_mprj_io_1 = GPIO_MODE_USER_STD_INPUT_PULLUP;
```

##### Activation

GPIOを有効にするためのグローバル変数`reg_mprj_xfer`が存在する。これをHighにすれば上記の設定内容が適用され、GPIOが有効になるが、その際に以下の記述が必要である[^22]。
```c=
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
