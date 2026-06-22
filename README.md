# AIエンジニアリング研修 導入 (Introduction)

このフォルダは、研修で使うNotebookと実演用データを管理する場所です。各回ごとにフォルダを分けます。

## フォルダ構成

```text
README.md
pyproject.toml
01_AIの仕組み/
  01_neural_network_learning_demo.ipynb
  data/
    diagnostician_routing/
      doctor_master.csv
      routing_requests.csv
```

今後、第2回以降のNotebookを追加する場合は、回ごとの内容が分かるフォルダ名で追加します。

## 第1回 Notebook

`01_AIの仕組み/01_neural_network_learning_demo.ipynb` は、画像診断依頼をどの提携診断士へ紹介するかを題材に、1層FCと2層FCの違いを確認するNotebookです。

扱うのは診断そのものではなく、非臨床の業務ルーティングです。画像の中身、所見、診断名、治療判断は使いません。

入力データはNotebook内で生成せず、次のCSVを読み込みます。

- `01_AIの仕組み/data/diagnostician_routing/doctor_master.csv`: 診断士候補 `D1`〜`D6` の6名分の特徴量
- `01_AIの仕組み/data/diagnostician_routing/routing_requests.csv`: 画像診断依頼1800件と教材用CSVの教師ラベル `selected_doctor_id`

`selected_doctor_id` は、モデルが学習・評価で合わせにいく教材用の教師ラベルです。実運用で医学的・業務的に最適な診断士を検証した結果ではありません。
accuracyは医学的正解率ではなく、予測1位の診断士IDがこのCSVラベルと一致した割合です。

依頼側は、画像種別、部位、枚数規模、優先度に、スライス厚、施設の月間件数、過去レポート有無を加えた構造です。
診断士側は、得意画像、得意部位、大量対応、速さ、負荷に、専門年数、直近品質スコア、月間シフト枠を加えた構造です。
スライス厚、施設の月間件数、過去レポート有無、専門年数、直近品質スコア、月間シフト枠はモデル入力には含めますが、教材用ラベルの判断には使いません。

教材用ラベルは、同じ判断軸を持つ3つの独立判定で `D1`〜`D6` から1名を選び、多数票を採ったものです。
3判定が全員別の候補を選んだ場合だけ、依頼内容と候補特徴を表で説明しやすい1名を採っています。

## uv環境のセットアップ

前提として `uv` が必要です。未インストールの場合は公式手順でインストールしてください。

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

このリポジトリのルートで環境を作成します。

```bash
uv sync
```

Jupyter Labを起動します。

```bash
uv run jupyter lab
```

ブラウザでJupyter Labが開いたら、`01_AIの仕組み/01_neural_network_learning_demo.ipynb` を開いてください。

## 実行確認

Notebookは `01_AIの仕組み/` フォルダを基準に、相対パスで `data/diagnostician_routing/` を読み込みます。Jupyter Lab上でNotebookを開いて上から順に実行すれば、CSV読込、特徴量化、1層FCの学習と判定例、2層FCの学習と判定例、lossグラフ、最終比較、検証セルまで実行できます。

最後のセルで次のメッセージが表示されれば、想定どおりに動いています。

```text
検証OK: CSV読込、診断士候補6名、主要特徴量、業務補助列、CSV教師ラベル、R1287の説明表、20epochごとの学習履歴、lossグラフ、1層FC/2層FCの判定例、2層FCの改善を確認しました。
```
