# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

UniTalker は音声から3Dフェイシャルアニメーションを生成する統合ニューラルネットワークモデル（ECCV 2024）。複数のアノテーション形式（頂点座標、ブレンドシェイプ、FLAMEパラメータ）と8つのデータセット（D0〜D7）を統一的に扱う。

## 環境構築

```bash
uv sync
```

## コマンド

```bash
# 推論
uv run python -m main.demo --config config/unitalker.yaml test_out_path ./test_results/demo.npz

# レンダリング
uv run python -m main.render ./test_results/demo.npz ./test_audios ./test_results/

# 学習
uv run python -m main.train --config config/unitalker.yaml

# 設定のオーバーライド（コマンドライン引数でYAML設定を上書き可能）
uv run python -m main.train --config config/unitalker.yaml lr 0.0001 batch_size 2
```

## アーキテクチャ

### パイプライン

```
音声(16kHz) → AudioEncoder(Wav2Vec2/WavLM) → Decoder(TCN/Transformer) + StyleEmbedding → OutputHead → [PCA Decoder] → 出力
```

### 主要モジュール構成

- **`models/unitalker.py`** - メインモデル。音声エンコーダ・デコーダ・データセット別出力ヘッドを統合
- **`models/unitalker_decoder.py`** - TCNデコーダ（`UniTalkerDecoderTCN`）とTransformerデコーダ（`UniTalkerDecoderTransformer`）
- **`models/wav2vec2.py`, `wavlm.py`** - HuggingFace事前学習モデルのラッパー
- **`models/pca.py`** - 高次元頂点データのPCA圧縮/復元（D0,D1,D2で使用）
- **`loss/loss.py`** - アノテーション型別の損失関数（`VerticesLoss`, `BlendShapeLoss`, `FlameParamLossForDADHead`）
- **`dataset/dataset.py`** - `AudioFaceDataset`（単一）と`MixAudioFaceDataset`（複数データセット混合）
- **`dataset/dataset_config.py`** - D0〜D7のデータセット定義（アノテーション型・次元・スケール等）
- **`config/unitalker.yaml`** - 全設定パラメータ（学習率、データセット選択、モデル構造等）

### データセットとアノテーション型の対応

| データセット | アノテーション型 | PCA使用 |
|---|---|---|
| D0 BIWI | BIWI_23370_vertices (70110次元) | Yes |
| D1 VOCASET | FLAME_5023_vertices (15069次元) | Yes |
| D2 MeshTalk | meshtalk_6172_vertices (18516次元) | Yes |
| D3 HDTF, D4 RAVDESS | 3DETF_blendshape_weight (52次元) | No |
| D5 FaceForensics++ | flame_params_from_dadhead (413次元) | No |
| D6 中国語音声, D7 歌 | inhouse_blendshape_weight (51次元) | No |

### 設計上の重要ポイント

- **マルチヘッド構造**: アノテーション型ごとに独立した出力ヘッドを持ち、新しいアノテーション型の追加はヘッドのプラグインで対応
- **アイデンティティ条件付け**: 学習可能なスタイル埋め込み（64次元）で話者ごとの特徴を表現。`random_id_prob`で汎化を促進
- **エンコーダ凍結戦略**: 最初のエポックで音声エンコーダを凍結（`fix_encoder_first`）し学習を安定化
- **長音声推論**: 25秒ウィンドウで分割し5秒オーバーラップで重み付きブレンディング
- **D5はFLAMEモデルが必要**: `resources/binary_resources/flame.pkl`が無い場合、`demo_dataset`からD5を除外する

### 設定システム

`utils/config.py`の`CfgNode`クラスでYAML設定を管理。コマンドライン引数で`key value`ペアとしてオーバーライド可能。設定値は属性アクセス（`args.lr`）で参照。

## 必要なリソースファイル

- `pretrained_models/UniTalker-*.pt` - 学習済みモデル
- `resources/` - テンプレートメッシュ、口領域インデックス等（`git lfs pull`で取得後解凍）
- `unitalker_data_release_V1/` - データセット・PCAモデル・JSONマニフェスト
- `test_audios/` - テスト用音声ファイル（`git lfs pull`で取得後解凍）
