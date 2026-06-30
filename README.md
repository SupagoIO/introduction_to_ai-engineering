# AIエンジニアリング研修 Beginners Guide

このフォルダは、研修で使うNotebookと実演用データを管理する場所です。各回ごとにフォルダを分けます。

## フォルダ構成

```text
ai-engineering/
  beginners-guide/
    README.md
    pyproject.toml
    01_AIの仕組み/
      01_neural_network_learning_demo.ipynb
      data/
        diagnostician_routing/
          doctor_master.csv
          routing_requests.csv
    02_LLMの生成原理/
      02_rnn_next_token_prediction.ipynb
      data/
        mhlw_ika_fee_schedule_pdf_excerpt.txt
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

## 第2回 Notebook

`02_LLMの生成原理/02_rnn_next_token_prediction.ipynb` は、厚生労働省が公開している「医科診療報酬点数表」PDFから抽出したテキストを題材に、次トークン予測の学習データを機械的に作る流れを確認するNotebookです。

入力データはNotebook内で生成せず、次のテキストを読み込みます。

- `02_LLMの生成原理/data/mhlw_ika_fee_schedule_pdf_excerpt.txt`: 厚生労働省PDF `https://www.mhlw.go.jp/content/12400000/001686842.pdf` の全400ページを `pdftotext -layout` で抽出したテキスト

このNotebookの目的は、RNNで高性能な文章生成を行うことではありません。既存テキストをトークン化し、`ここまでのトークン列 -> 次のトークン` の学習ペアを大量に作れること、全ペアを学習候補プールにしてmini-batchをサンプリングできること、Embedding 64次元・hidden 128次元のAdam付きRNNで学習中にlossの途中経過が更新されること、固定プロンプトから続きを生成できることを観察します。

生成結果が制度上正しいか、実在制度の算定可否、医療上の判断、実務での請求判断は扱いません。PDF由来のテキストを使いますが、Notebook内の生成結果や小さなRNNの出力は、制度上の正誤、最新版、完全性、法的解釈を保証するものではありません。

## uv環境のセットアップ

前提として `uv` が必要です。未インストールの場合は公式手順でインストールしてください。

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

環境を作成します。

```bash
cd /Users/kanam/Business/Supago/lectures/ai-engineering/beginners-guide
uv sync
```

Jupyter Labを起動します。

```bash
uv run jupyter lab
```

ブラウザでJupyter Labが開いたら、各回のNotebookを開いてください。

## 実行確認

Notebookは `01_AIの仕組み/` フォルダを基準に、相対パスで `data/diagnostician_routing/` を読み込みます。Jupyter Lab上でNotebookを開いて上から順に実行すれば、CSV読込、特徴量化、1層FCの学習と判定例、2層FCの学習と判定例、lossグラフ、最終比較、検証セルまで実行できます。

最後のセルで次のメッセージが表示されれば、想定どおりに動いています。

```text
検証OK: CSV読込、診断士候補6名、主要特徴量、業務補助列、CSV教師ラベル、R1287の説明表、20epochごとの学習履歴、lossグラフ、1層FC/2層FCの判定例、2層FCの改善を確認しました。
```

第2回Notebookは `02_LLMの生成原理/` フォルダを基準に、相対パスで `data/mhlw_ika_fee_schedule_pdf_excerpt.txt` を読み込みます。Jupyter Lab上でNotebookを開いて上から順に実行すれば、テキスト読込、簡易トークン化、語彙表、次トークン予測ペア、全ペアを候補プールにしたAdam付きRNNの学習、同じ表示枠で更新される学習中の数値表、学習後に1回描画されるlossグラフ、固定プロンプトからの生成例、検証セルまで実行できます。

最後のセルで次のメッセージが表示されれば、想定どおりに動いています。

```text
検証OK: MHLW PDF由来テキストの読込、簡易トークン化、語彙表、ここまでのトークン列 -> 次のトークン の全ペアを学習候補プール化、Embedding 64次元・hidden 128次元のAdam付きRNN、非クリア型の学習中数値更新、学習後1回描画のlossグラフ、loss低下、固定プロンプトからの生成例、制度上の正誤を扱わない注意を確認しました。
```
