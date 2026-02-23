# Data Cleansing using BBO and Quantum Annealing

このリポジトリは、量子アニーリングと代理モデルベースのブラックボックス最適化（BBO）を組み合わせて、ノイズの多いデータセットから誤ラベル（Mislabeled instances）を特定・除去する手法を簡易的に再現したものです。

## 概要
実世界のデータセットには、ヒューマンエラーなどの予期せぬ理由で誤ったラベル（ノイズ）が含まれることが多々あり、これがモデルの汎化性能を著しく低下させる。

本プロジェクトでは、以下の論文の手法に基づき、「検証ロス（Validation Loss）」を直接最小化するようにトレーニングデータの最適なサブセットを選択します。

**ベース論文:**
> Otsuka, M., Kodama, K., Morita, K., & Ohzeki, M. (2025). *Filtering out mislabeled training instances using black-box optimization and quantum annealing.* [arXiv:2501.06916v2](https://arxiv.org/abs/2501.06916v2)

## 主な特徴
* **検証ロスの直接最適化**: 汎化性能の代理指標である検証ロスを直接最小化するサブセット選択を実行
* **代理モデル (Surrogate Model)**: 2次形式の代理モデルを用いて、膨大な組み合わせ空間（$`2^n`$）を近似的に探索
* **量子サンプリング**: 量子トンネル効果を活用し、多様で質の高い解を効率的にサンプリング

## 実験タスク: Noisy Majority Bit Task
9ビットの入力ベクトルに対する「多数決（Majority Bit）」を予測するタスク
* **Real (Clean)**: 正解ラベルが付与されたデータ（64件） 
* **Fake (Noisy)**: ラベルを反転させた汚染データ（64件）
* **目標**: 128件の混在データからFakeのみを除去し、テスト精度を回復させる

## 構成
* `nqc2025.ipynb`: 全工程を網羅したJupyter Notebook
  * データの生成
  * Neal (Simulated Annealing) を用いた32 SEED分の最適化実行
  * 結果の可視化（Selection Pattern & Accuracy Comparison）

## セットアップ
必要なライブラリのインストール:
```bash
pip install numpy scikit-learn matplotlib openjij dwave-neal tqdm
```

## 使用方法
1. nqc2025.ipynb を開き、全てのセルを実行します
2. デフォルト設定では neal.SimulatedAnnealingSampler が使用されます
3. 初期化フェーズ（64ステップ）と最適化フェーズ（256ステップ）を経て、計320ステップの探索が行われます

## 再現結果の解釈
<img width="841" height="560" alt="Image" src="https://github.com/user-attachments/assets/59fc832f-2ba1-464f-99df-f3f35409e307" />

* 選択パターン: ヒートマップにおいて、Fake領域（後半64件）が優先的に除外（黒色）されます。これは、低エントロピーなノイズほどモデルへの悪影響が強く、優先的に除去されるという論文の知見と一致します。

<img width="536" height="528" alt="Image" src="https://github.com/user-attachments/assets/79045195-bf2f-480a-b5f6-2c9e117377c4" />

Average Accuracy (Before): 0.4934

Average Accuracy (After) : 0.8044

* 精度の向上: ノイズ除去前はランダム予測ですが、最適化後はテスト精度が大幅に向上し、モデルが正しく「多数決」の法則を学習できたことが示されます。

## ライセンス
このプロジェクトは MIT License の下で公開されています。
