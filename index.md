---
layout: default
title: 20行でわかる！OpenMPWとOpenLANEとSkywaterPDKの様々
---
## 20行でわかる！OpenMPWとOpenLANEとSkywaterPDKの様々
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

