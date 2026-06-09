pth choose?/data preprocess?(network input/ output )，dataloader

## 1. データセットの配置と確認

ダウンロードした4つのデータセット（約49GB、14,782ファイル）を以下のディレクトリへ移行しました。

- **配置先:** `/home/huang/code/cst_ai/input/data_0607/`
    
- **解凍について:** データはすべて直接利用可能な形式（.tif, .tsv, .txt, .pdf）でダウンロードされていたため、zip等からの解凍ステップは不要です。ディレクトリ構造も元ファイル名のまま保持しています。
    

**【Battery TIFFデータの特性（参考）】**

- **フォーマット:** シングルチャネル uint16（LZW圧縮）
    
- **解像度:** 約 1280×1148
    
- **ダイナミックレンジ:** 0 ～ 65535（露光設定により変動）
    
- **構成:** 110kV + uAの条件を変えて撮影した同一オブジェクトの画像10枚
    

## 2. 環境設定（DevContainer / Docker）のアップデート

Cursorでスムーズに開発できるよう、設定ファイルを以下の通り更新しました。

- **マウントパスの整備:**
    
    - 共有データ盤: `/data` (ホスト側の `/mnt/data`)
        
    - 生データディレクトリ: `/input` (ホスト側の `cst_ai/input`)
        
- **自動セットアップ (`setup.sh` の新設):**
    
    - コンテナの初回起動時に、`uv` を用いたPython 3.11仮想環境の構築、PyTorch (cu124)、およびBasicSRの依存関係インストールを自動実行します。
        
- **Dockerfileの改修:**
    
    - OpenCVに必要なシステムライブラリ（`libgl1` など）を追加しました。コンテナ起動時に自動で学習スクリプトは走らず、手動での実行を前提とする設計（`CMD ["bash"]`）にしています。
        

## 3. コンテナの利用手順

今後は以下の手順で開発・検証を進められます。

1. Cursorで `/home/huang/code/cst_ai/BasicSR` を開く。
    
2. コマンドパレットから **「Dev Containers: Reopen in Container」** を実行。
    
3. 初回起動時は `postCreateCommand` による自動環境構築を待つ（数分程度）。
    
4. 構築完了後、コンテナ内のターミナルで直接スクリプトを実行可能。
    

**【実行コマンド例】**

Bash

```
# スモークテストの手動実行（環境変数はコンテナ内で設定済み）
python scripts/smoke_test_swinir_gray_dn_battery.py

# BasicSRの学習コマンド例
PYTHONPATH=./ python basicsr/train.py -opt options/train/...
```

**【コンテナ内の主要パス対応表】**

- 電池TIFF生データ: `/input/data_0607/260520_1-battery_data/260520_battery image tiff data`
    
- 共有データ出力先: `/data`
    
- 事前学習済み重み: `/workspace/experiments/pretrained_models/SwinIR/`


| **比較項目**    | swinIR                                                      | cursor生成                                                          |
| ----------- | ----------------------------------------------------------- | ----------------------------------------------------------------- |
| **全体フロー**   | クリーンなグレースケール画像を読み込み、オンラインでAWGNノイズ（noise/255）を付加してからノイズ除去へ入力 | 実際のノイズ付き電池TIFF画像を読み込み、直接推論（Inference）を実行                          |
| **画像読み込み**  | `cv2.imread(..., GRAYSCALE) / 255`                          | PILでuint16のTIFFを読み込み + p1–p99パーセンタイル正規化（Percentile Normalization） |
| **入力データ**   | 合成ノイズ画像（`img_lq`）                                           | 前処理済みの実測画像（SEM画像など）                                               |
| **評価指標**    | PSNR / SSIM を算出（GT: 正解データと比較）                               | 指標評価なし（パイプラインが完走し、画像が出力されることの検証のみ）                                |
| **パディング**   | `torch.flip` を用いた反転結合パディング                                  | `F.pad(..., 'reflect')` によるパディング                                  |
| **Tile 推論** | `--tile` オプションをサポート（分割推論）                                   | 未実装（画像全体を一括で推論）                                                   |
| **出力・保存**   | `cv2.imwrite` にてBGR形式で保存                                    | グレースケールPNG形式で保存                                                   |