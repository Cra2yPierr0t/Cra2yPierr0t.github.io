---
layout: default
title: 3.デザインをCaravelに載せる
---
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
```shell
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
