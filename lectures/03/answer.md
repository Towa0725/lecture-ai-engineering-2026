# 第3回 解答

## 演習

- 第1回（素のLLM）とエージェントの回答の違い：抽象的な言葉に対して、一般的な情報ではなく、持っている情報を提示してきた
- `--trace` で見えたツール呼び出し（ツール名と引数）：
[tool call]   search_glossary({'query': 'LoRA'})
[tool call]   search_glossary({'query': 'QLoRA'})
[tool return] search_glossary -> [{"term": "LoRA", "lecture": 2, "definition": "事前学習済みの重みは固定したまま、重みの更新分を2つの小さな行列の積（低ランク行列）で表し、その小さな行列だけを学習するPEFT手法。学習するパラ…
[tool return] search_glossary -> [{"term": "QLoRA", "lecture": 2, "definition": "ベースモデルの重みを4ビットなどに量子化してメモリを節約したうえで、LoRAで学習する手法。より大きなモデルを小さなGPUで扱える。本講座の第2…
[usage] RunUsage(cost=Decimal('0.0008113'), details={'text_prompt_tokens': 1196}, input_text_tokens=1196, input_tokens=1196, output_tokens=181, requests=2, tool_calls=2)
- 1つの質問での LLM 呼び出し回数（`requests`）：２
- 追加したツールと、docstring に書いた説明：ツール名（関数名）: make_quiz
"""指定された講義回の用語から、復習用の確認問題を1問作る。

確認問題を求められたときに使用する。
"""

## 宿題

- 取り組んだレベル：基礎 
- やったこと：
- 結果：

変更前
query には調べたい用語を1つだけ入れる。
uv run python -m app.agent --trace "LoRAとQLoRAの違いは？"
[model] gemini-3.5-flash-lite
LoRAとQLoRAの違いは、**ベースモデルの重みをそのまま使うか、量子化してメモリをさらに節約するかにあります**。

* **LoRA（Low-Rank Adaptation）**
  事前学習済みの重みは固定したまま、重みの更新分を小さな低ランク行列で近似し、その部分だけを学習するPEFT（Parameter-Efficient Fine-Tuning）手法です。学習するパラメータ数と保存サイズを大幅に削減できます。
* **QLoRA（Quantized LoRA）**
  LoRAの仕組みに加えて、ベースモデルの重みを4ビットなどに**量子化**（精度を落としてデータ量を圧縮）してから学習する手法です。これにより、さらにメモリ消費量を抑えられ、より大きなモデルを比較的小さなGPUでも扱えるようになります。

関連する回：第2回

--- trace ---
[tool call]   search_glossary({'query': 'LoRA'})
[tool call]   search_glossary({'query': 'QLoRA'})
[tool return] search_glossary -> [{"term": "LoRA", "lecture": 2, "definition": "事前学習済みの重みは固定したまま、重みの更新分を2つの小さな行列の積（低ランク行列）で表し、その小さな行列だけを学習するPEFT手法。学習するパラ…
[tool return] search_glossary -> [{"term": "QLoRA", "lecture": 2, "definition": "ベースモデルの重みを4ビットなどに量子化してメモリを節約したうえで、LoRAで学習する手法。より大きなモデルを小さなGPUで扱える。本講座の第2…
[usage] RunUsage(cost=Decimal('0.0009516'), details={'text_prompt_tokens': 1322}, input_text_tokens=1322, input_tokens=1322, output_tokens=222, requests=2, tool_calls=2)


変更後
query には関連する用語をまとめて入れる。
uv run python -m app.agent --trace "LoRAとQLoRAの違いは？"
[model] gemini-3.5-flash-lite
LoRAとQLoRAの違いは、ベースモデルの重みをそのまま使うか、量子化してメモリを節約するかにあります。

* **LoRA（Low-Rank Adaptation）**
  事前学習済みの重みを固定したまま、重みの更新分を低ランク行列で表し、その小さな行列だけを学習するPEFT（Parameter-Efficient Fine-Tuning）手法です。学習パラメータが大幅に少なくなります。
* **QLoRA（Quantized LoRA）**
  ベースモデルの重みを4ビットなどに量子化（Quantize）してメモリ消費量をさらに劇的に削減したうえで、LoRAによる学習を行う手法です。これにより、より大きなモデルを限られたGPUリソースでも扱えるようになります。

関係する回：第2回

--- trace ---
[tool call]   search_glossary({'query': 'LoRA QLoRA'})
[tool return] search_glossary -> [{"term": "LoRA", "lecture": 2, "definition": "事前学習済みの重みは固定したまま、重みの更新分を2つの小さな行列の積（低ランク行列）で表し、その小さな行列だけを学習するPEFT手法。学習するパラ…
[usage] RunUsage(cost=Decimal('0.0007783'), details={'text_prompt_tokens': 1011}, input_text_tokens=1011, input_tokens=1011, output_tokens=190, requests=2, tool_calls=1)




結果比較
ツール呼び出し回数の半減
変更前は、モデルが指示通りに LoRA と QLoRA でツールを個別に2回呼び出していましたが、変更後は "LoRA QLoRA" という1つの文字列に結合して1回で呼び出す形に変化しました。

コンテキストサイズ（トークン数）の縮小
ツールの呼び出し回数が減ったことで、ツール定義のやり取りやツール戻り値（JSON形式）のメタデータ重複が削減されました。その結果、入力トークン数は 1,322 から 1,011 へ約23.5%（311トークン）減少 し、出力トークン数も 222 から 190 へ減少 しています。

実行コストの抑制
トークン消費量の減少に伴い、1リクエストあたりのコストは $0.0009516 から $0.0007783 へ約18.2%削減 されました。

回答品質の維持
出力された最終的な回答文は、どちらもLoRAとQLoRAの定義、差分、関連講義回を過不足なく網羅しており、品質に実質的な差はありませんでした。


- 気づき・疑問：LLMエージェントにおいて、ツールの呼び出し定義を生成する処理は推論時間の遅延に直結します。呼び出しが1回にまとまったことで、初回推論でモデルが生成すべきトークン量が削減され、応答速度（レイテンシ）の向上につながっていると考える。
