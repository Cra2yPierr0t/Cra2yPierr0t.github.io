---
layout: default
title: OpenLANE
---
# OpenLANEと半導体設計入門
質問、修正案、その他連絡は@Cra2yPierr0tマデ

Github : [https://github.com/The-OpenROAD-Project/OpenLane](https://github.com/The-OpenROAD-Project/OpenLane)<br>
ドキュメント : [https://openlane.readthedocs.io/en/latest/](https://openlane.readthedocs.io/en/latest/)

## OpenLANEについて
OpenLANEとはOSSなRTL-to-GDSIIコンパイラであり、20個くらいのOSSを組み合わせて作られている。PDKとRTLとコンフィグを揃えて実行するとGDSIIが生えてくる。本記事ではOpenLANEの使い方及び各フローの説明を行いながら、現代の半導体設計を学んでいく。

OpenLANEの全体的なフローは次のようになっている。[^1]
1. Synthesis
    1. [`yosys`](https://github.com/YosysHQ/yosys) - RTLを論理合成
    2. [`abc`](https://github.com/YosysHQ/yosys) - PDKにマッピング
    3. [`OpenSTA`](https://github.com/The-OpenROAD-Project/OpenSTA) - 静的タイミング解析
2. Floorplan and PDN
    1. [`init_fp`](https://github.com/The-OpenROAD-Project/OpenROAD/tree/master/src/ifp) - コア領域の定義
    2. [`ioplacer`](https://github.com/The-OpenROAD-Project/OpenROAD/tree/master/src/ppl) - 入出力ポートの設置
    3. [`pdn`](https://github.com/The-OpenROAD-Project/OpenROAD/tree/master/src/pdn) - 給電ネットワークの生成
    4. [`tapcell`](https://github.com/The-OpenROAD-Project/OpenROAD/tree/master/src/tap) - weltapとデキャップセルの挿入
3. Placement
    1. [`RePLace`](https://github.com/The-OpenROAD-Project/RePlAce) - グローバル配置
    2. [`Resizer`](https://github.com/The-OpenROAD-Project/OpenROAD/tree/master/src/rsz) - 最適化
    3. [`OpenDP`](https://github.com/The-OpenROAD-Project/OpenROAD/tree/master/src/dpl) - ローカル配置
4. CTS
    1. [`TritonCTS`](https://github.com/The-OpenROAD-Project/OpenROAD/tree/master/src/cts) - クロック供給ネットワーク(クロックツリー)の合成
5. Routing
    1. [`FastRoute`](https://github.com/The-OpenROAD-Project/OpenROAD/tree/master/src/grt) - グローバル配線
    2. [`CU-GR`](https://github.com/cuhk-eda/cu-gr) - グローバル配線(プランB)
    3. [`TritonRoute`](https://github.com/The-OpenROAD-Project/TritonRoute) - ローカル配線
    4. [`SPEF_Extractor`](https://github.com/HanyMoussa/SPEF_EXTRACTOR) - 寄生フォーマット"SPEF"の摘出
6. GDSII Generation
    1. [`Magic`](https://github.com/RTimothyEdwards/magic) - routed defからのGDSIIファイル生成
    2. [`Klayout`](https://github.com/KLayout/klayout) - GDSIIファイル生成(バックアップ)
7. Checks
    1. `Magic` - DRCチェックとアンテナチェック
    2. `Klayout` - DRCチェック
    3. [`Netgen`](https://github.com/RTimothyEdwards/netgen) - LVSチェック
    4. [`CVC`](https://github.com/d-m-bailey/cvc) - 回路妥当性検証
    
![](https://github.com/The-OpenROAD-Project/OpenLane/blob/master/docs/_static/openlane.flow.1.png?raw=true)

出典：[https://openlane.readthedocs.io/en/latest/#openlane-architecture](https://openlane.readthedocs.io/en/latest/#openlane-architecture)

以上を見たところで半導体設計もOpenLANEも完全に理解出来る訳が無いので、これから各項目について説明していく。楽しみましょう。

## 1. Synthesis
勉強中
## 2. Floorplan and PDN
勉強中
## 3. Placement
勉強中
## 4. CTS
勉強中
## 5. Routing
勉強中
## 6. GDSII Generation
勉強中
## 7. Checks
勉強中

[^1]: https://openlane.readthedocs.io/en/latest/#openlane-design-stages
