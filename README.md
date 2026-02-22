
# UniTalker: 統一モデルによる音声駆動3Dフェイシャルアニメーションのスケールアップ [ECCV2024]

## 関連リンク

<div align="center">
    <a href="https://x-niper.github.io/projects/UniTalker/" class="button"><b>[Homepage]</b></a> &nbsp;&nbsp;&nbsp;&nbsp;
    <a href="https://arxiv.org/abs/2408.00762" class="button"><b>[arXiv]</b></a> &nbsp;&nbsp;&nbsp;&nbsp;
    <a href="https://www.youtube.com/watch?v=oUUh67ECzig" class="button"><b>[Video]</b></a> &nbsp;&nbsp;&nbsp;&nbsp;
</div>

<div align="center">
<img src="https://github.com/X-niper/X-niper.github.io/blob/master/projects/UniTalker/assets/unitalker_architecture.png" alt="UniTalker Architecture" width="80%" align=center/>
</div>


> UniTalkerは、クリーンな音声やノイズを含む多言語音声、テキスト読み上げ生成音声、さらにはBGM付きのノイズを含む歌声など、さまざまな音声ドメインからリアルな顔のモーションを生成します。
>
> UniTalkerは複数のアノテーション形式で出力が可能です。
>
> 新しいアノテーション形式を持つデータセットに対しては、UniTalkerに新しいヘッドをプラグインし、既存のデータセットまたは新しいデータセットのみで学習させることができ、リトポロジーが不要です。

## インストール

```bash
uv sync
```

## 推論

### チェックポイント、PCAモデル、テンプレートリソースのダウンロード

[UniTalker-B-[D0-D7]](https://drive.google.com/file/d/1PmF8I6lyo0_64-NgeN5qIQAX6Bg0yw44/view?usp=sharing): 論文のベースモデルです。ダウンロードして `./pretrained_models` に配置してください。

[UniTalker-L-[D0-D7]](https://drive.google.com/file/d/1sH2T7KLFNjUnTM-V1eRMM1Tytxd2sYAp/view?usp=sharing): 論文のデフォルトモデルです。まずベースモデルでパイプラインの動作確認を行ってください。

[Unitalker-data-release-V1](https://drive.google.com/file/d/1Un7TB0Z5A1CG6bgeqKlhnSOECFN-C6KK/view?usp=sharing): 公開データセット、PCAモデル、データ分割用JSONファイル、IDテンプレートのnumpy配列です。ダウンロードしてこのリポジトリ内で解凍してください。

[FLAME2020](https://flame.is.tue.mpg.de/download.php): FLAME 2020をダウンロードし、generic_model.pklを resources/binary_resources/flame.pkl に移動してください。

<!-- [PCA models](https://drive.google.com/file/d/1e0sG2vvdrtAMgwD5njctifhX0ai4eu3g/view?usp=sharing): download the pca models and unzip it in "./unitalker_data_release" -->

`git lfs pull` を実行して `./resources.zip` と `./test_audios.zip` を取得し、このリポジトリ内で解凍してください。

最終的に、以下のようなファイル構成になります:

```text
├── pretrained_models
│   ├── UniTalker-B-D0-D7.pt
│   ├── UniTalker-L-D0-D7.pt
├── resources
│   ├── binary_resources
│   │   ├── 02_flame_mouth_idx.npy
│   │   ├── ...
│   │   └── vocaset_FDD_wo_eyes.npy
│   └── obj_template
│       ├── 3DETF_blendshape_weight.obj
│       ├── ...
│       └── meshtalk_6172_vertices.obj
└── unitalker_data_release_V1
│   ├── D0_BIWI
│   │   ├── id_template.npy
│   │   └── pca.npz
│   ├── D1_vocaset
│   │   ├── id_template.npy
│   │   └── pca.npz
│   ├── D2_meshtalk
│   │   ├── id_template.npy
│   │   └── pca.npz
│   ├── D3D4_3DETF
│   │   ├── D3_HDTF
│   │   └── D4_RAVDESS
│   ├── D5_unitalker_faceforensics++
│   │   ├── id_template.npy
│   │   ├── test
│   │   ├── test.json
│   │   ├── train
│   │   ├── train.json
│   │   ├── val
│   │   └── val.json
│   ├── D6_unitalker_Chinese_speech
│   │   ├── id_template.npy
│   │   ├── test
│   │   ├── test.json
│   │   ├── train
│   │   ├── train.json
│   │   ├── val
│   │   └── val.json
│   └── D7_unitalker_song
│       ├── id_template.npy
│       ├── test
│       ├── test.json
│       ├── train
│       ├── train.json
│       ├── val
│       └── val.json
```

### デモ

```bash
uv run python -m main.demo --config config/unitalker.yaml test_out_path ./test_results/demo.npz
uv run python -m main.render ./test_results/demo.npz ./test_audios ./test_results/
```

## 学習

### データのダウンロード
[Unitalker-data-release-V1](https://drive.google.com/file/d/1qRBPsTdOWp72ty04oD1Q_ivtwMjrACLH/view?usp=sharing) にはD5、D6、D7が含まれています。データセットは処理済みで、train、validation、testに分割されています。まずこの3つのデータセットで学習ステップをお試しください。
D0〜D7の全データセットでモデルを学習させる場合は、以下のリンクからデータセットをダウンロードしてください:
[D0: BIWI](https://github.com/Doubiiu/CodeTalker/blob/main/BIWI/README.md).
[D1: VOCASET](https://voca.is.tue.mpg.de/).
[D2: meshtalk](https://github.com/facebookresearch/meshtalk?tab=readme-ov-file).
[D4,D5: 3DETF](https://github.com/psyai-net/EmoTalk_release).

### 設定の変更と学習
準備したデータセットに合わせて、`config/unitalker.yaml` 内の `dataset` と `duplicate_list` を変更してください。両方のリストの長さが一致するようにしてください。


```bash
uv run python -m main.train --config config/unitalker.yaml
```
