---
layout: default
title: footnote
---
\[^1]:https://caravel-harness.readthedocs.io/en/latest/memory-mapped-io-summary.html<br>
[^2]:実装を辿ってったら謎のマクロが生えてたので置いとく https://github.com/efabless/caravel-lite/blob/13f2590e4b3a74b910dac56a6b757f5a66fd5212/verilog/rtl/chip_io.v#L229<br>
[^3]: https://github.com/efabless/caravel_user_project/blob/main/docs/source/index.rst#caravel-integration<br>
[^4]: https://github.com/efabless/caravel_user_project/blob/main/docs/source/index.rst#hardening-the-user-project-macro-using-openlane<br>
[^5]:https://caravel-harness.readthedocs.io/en/latest/quick-start.html#configuration<br>
[^6]:https://github.com/efabless/openlane/tree/master/configuration<br>
[^7]:https://twitter.com/Cra2yPierr0t/status/1427088118025392131<br>
[^8]: https://youtu.be/jBrBqhVNgDo?t=1326<br>
[^9]: 明記されてないけどefablessの動画[^8]でそうしてました<br>
[^10]:各ドキュメントではefablessのOpenLaneを使っているが、MPW2が終わった直後のためefabless版は更新が止まっており``make pdk`` でコケるのでOpenROADを一時的に使う。(多分SKYWATER_COMMITの値を更新してないだけ)<br>
[^11]:https://github.com/efabless/caravel_user_project<br>
[^12]:https://caravel-harness.readthedocs.io/en/latest/index.html<br>
[^13]:https://caravel-harness.readthedocs.io/en/latest/quick-start.html#adding-a-user-project<br>
[^14]:https://caravel-harness.readthedocs.io/en/latest/quick-start.html#adding-a-new-design<br>
[^15]:https://github.com/efabless/caravel_user_project/tree/main/verilog/dv#simulation-environment-setup<br>
[^16]:これをファブに投げるとLSIが生えてくる(らしい)<br>
[^17]:あったわ https://github.com/efabless/caravel_user_project/blob/main/docs/source/index.rst#running-open-mpw-precheck-locally<br>
[^18]:https://github.com/konradwilk/caravel_user_project/blob/submission-mpw-two/openlane/user_project_wrapper/config.tcl<br>
[^19]:https://github.com/efabless/caravel/blob/master/docs/source/_static/caravel_harness.png<br>
[^20]:つまり`user_project_wrapper`がモジュール`wrapper1.v`と`wrapper2.v`で出来ている場合、`EXTRA_LEFS`には`wrapper1.lef`, `wrapper2.lef`と書き換え、`EXTRA_GDS_FILES`には`wrapper1.gds`, `wrapper2.gds`と書き換えてやる。どうしてもわからない場合、これ[^18]が参考になる。<br>
[^21]:気軽にLSIが焼けたらうれしいので<br>
[^22]:シフトレジスタになってるからどうとか<br>
[^23]:1nputと0utputで覚えやすいね！！！<br>
[^24]:よく知らないけどRTL記述レベルではなくてffとか論理回路のレベルまで変換した上でのシミュレーションだと思う。必要性はよくわからない。<br>
[^25]:https://github.com/efabless/caravel_user_project/blob/mpw-3/verilog/rtl/user_proj_example.v#L86<br>
[^26]:https://caravel-harness.readthedocs.io/en/latest/pinout.html<br>
[^27]:https://www.youtube.com/watch?v=pPgnVBguNW8<br>
[^28]:https://github.com/efabless/caravel_user_project/blob/mpw-3/verilog/rtl/user_project_wrapper.v<br>
