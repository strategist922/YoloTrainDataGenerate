# [Japanese] YoloTrainDataGenerate
YoloV2独自学習データの生成＋Movidius Neural Compute Stick向け学習データコンバート

YoloV2 Generate original learning data + Learning data conversion for Movidius Neural Compute Stick

　
## 環境
(1)【学習用PC】 GIGABYE U2442F

  ・MEM：16GB

  ・CPU：第3世代 Intel Core i7-3517U(1.9GHz)

  ・GPU：Geforce GT 650M (VRAM:2GB)

  ・OS：Ubuntu 16.04 LTS (Windows10とのデュアルブート)

  ・CUDA 8.0.61

  ・cuDNN v6.0

  ・Caffe

  ・OpenCV 3.4.0

  ・Samba

(2)【実行環境】 Raspberry Pi 3 ModelB

  ・Raspbian Stretch

  ・NCSDK v1.12.00

  ・Intel Movidius Neural Compute Stick

  ・OpenCV 3.4.0

  ・Samba

　
## おおまかな流れ
  1. 適当に動画撮影
  2. 動画から機械的に静止画を大量生成
  3. 大量の静止画から物体部分を機械的に抽出して背景が透過した物体画像生成
  4. 別途用意した背景静止画と 3. で生成した物体静止画をランダムに回転・縮小・拡大・配置・ノイズ追加しながら合成して大量に水増し
  5. 学習
  6. Intel Movidius Neural Compute Stick 用学習データ(graph)へ変換
  7. Raspberry Pi上で 6. を使用してtinyYoloによる複数動体検知
  
 
## 【学習用PC】 動画→静止画変換

`$ ffmpeg -i xxxx.mp4 -vcodec png -r 10 image_%04d.png`
　
　
## 【学習用PC】 指定フォルダ内の複数静止画ファイル、複数物体周囲をまとめて機械的に透過加工

* 背景が白色に近い色・物体が白色／灰色以外の配色で構成されている場合のみ動作
* １画像内に複数物体が写っている場合は物体数分の画像ファイルへ分割して加工
* 入力画像が長方形であっても最終生成画像は物体を含む96×96の正方形
* エッジ抽出の都合上、重なり合っている物体は１つと認識される
* 検出された物体の面積が1000pxに満たない場合は当該物体を抽出対象から除外
* 最終生成された画像内に物体が存在しないと判断される場合はファイルを生成しない




