# SBV2-API

> [!CAUTION]
> 本バージョンはアルファ版です。
> 
> 安定版を利用したい場合は[こちら](https://github.com/neodyland/sbv2-api/tree/v0.1.x)をご覧ください。

> [!CAUTION]
> オプションの辞書はLGPLです。
> 
> オプションの辞書を使用する場合、バイナリの内部の辞書部分について、LGPLが適用されます。

> [!NOTE]
> このレポジトリはメンテナンスの都合上、[tuna2134](https:://github.com/tuna2134)氏の所属する[Neodyland](https://neody.land/)へとリポジトリ所在地を移動しました。
> 
> 引き続きtuna2134氏がメインメンテナとして管理しています。

## プログラミングに詳しくない方向け

[こちら](https://github.com/tuna2134/sbv2-gui)を参照してください。

コマンドやpythonの知識なしで簡単に使えるバージョンです。(できることはほぼ同じ)

## このプロジェクトについて

このプロジェクトは Style-Bert-ViTS2 を ONNX 化したものを Rust で実行するのを目的としたライブラリです。

JP-Extra しか対応していません。(基本的に対応する予定もありません)

## 変換方法

[こちら](https://github.com/neodyland/sbv2-api/tree/main/scripts/convert)を参照してください。

## Todo

- [x] REST API の実装
- [x] Rust ライブラリの実装
- [x] `.sbv2`フォーマットの開発
- [x] PyO3 を利用し、 Python から使えるようにする
- [ ] 組み込み向けにCライブラリの作成
- [x] GPU 対応(CUDA)
- [x] GPU 対応(DirectML)
- [x] GPU 対応(CoreML)
- [x] WASM 変換
- [x] arm64のdockerサポート
- [x] aivis形式のサポート
- [ ] MeCabを利用する

## 構造説明

- `crates/sbv2_api` - 推論用 REST API
- `crates/sbv2_core` - 推論コア部分
- `scripts/docker` - docker ビルドスクリプト
- `scripts/convert` - onnx, sbv2フォーマットへの変換スクリプト

## プログラミングある程度できる人向けREST API起動方法

### models をインストール

https://huggingface.co/neody/sbv2-api-assets/tree/main/deberta
から`tokenizer.json`,`debert.onnx`
https://huggingface.co/neody/sbv2-api-assets/tree/main/model
から`tsukuyomi.sbv2`
を models フォルダに配置

### .env ファイルの作成

```sh
cp .env.sample .env
```

### 起動

CPUの場合は
```sh
docker run -it --rm -p 3000:3000 --name sbv2 \
-v ./models:/work/models --env-file .env \
ghcr.io/neodyland/sbv2-api:cpu
```

<details>
<summary>Apple Silicon搭載のMac(M1以降)の場合</summary>
docker上で動作させる場合、.envのADDRをlocalhostから0.0.0.0に変更してください。

```yaml
ADDR=0.0.0.0:3000
```

CPUの場合は
```bash
docker run --platform linux/amd64 -it --rm -p 3000:3000 --name sbv2 \
-v ./models:/work/models --env-file .env \
ghcr.io/neodyland/sbv2-api:cpu
```
</details>

CUDAの場合は
```sh
docker run -it --rm -p 3000:3000 --name sbv2 \
-v ./models:/work/models --env-file .env \
--gpus all \
ghcr.io/neodyland/sbv2-api:cuda
```

### 起動確認

```sh
curl -XPOST -H "Content-type: application/json" -d '{"text": "こんにちは","ident": "tsukuyomi"}' 'http://localhost:3000/synthesize' --output "output.wav"
curl http://localhost:3000/models
```

## 開発者向けガイド

### Feature flags

`sbv2_api`、`sbv2_core`共に
- `cuda` featureでcuda
- `cuda_tf32` featureでcudaのtf32機能
- `tensorrt` featureでbert部分のtensorrt利用
- `dynamic` featureで手元のonnxruntime共有ライブラリを利用(`ORT_DYLIB_PATH=./libonnxruntime.dll`などで指定)
- `directml` featureでdirectmlの利用ができます。
- `coreml` featureでcoremlの利用ができます。

### 環境変数

以下の環境変数はライブラリ側では適用されません。

ライブラリAPIについては`https://docs.rs/sbv2_core`を参照してください。

- `ADDR` `localhost:3000`などのようにサーバー起動アドレスをコントロールできます。
- `MODELS_PATH` sbv2モデルの存在するフォルダを指定できます。
- `RUST_LOG` おなじみlog levelです。
- `HOLDER_MAX_LOADED_MODElS` RAMにロードされるモデルの最大数を指定します。

## 謝辞

- [litagin02/Style-Bert-VITS2](https://github.com/litagin02/Style-Bert-VITS2) - このコードを書くにあたり、ベースとなる部分を参考にさせていただきました。
- [Googlefan](https://github.com/Googlefan256) - 彼にモデルを ONNX ヘ変換および効率化をする方法を教わりました。
- [Aivis Project](https://github.com/Aivis-Project/AivisSpeech-Engine) - 辞書部分
