# [Japanese] YoloTrainDataGenerate
動画からYoloV2独自学習データを半機械的に自動生成するための手順とツール<br>
Procedures and tools for semi-mechanically automatically generating YoloV2 original learning data from video<br>
https://qiita.com/PINTO/items/d5645734ca9c95b1c395

　
# 環境
* CPU：第3世代 Intel Core i7-3517U(1.9GHz)
* MEM：16GB
* GPU：Geforce GT 650M (VRAM:2GB)
* OS：Ubuntu 16.04 LTS
* OpenCV 3.4.1
* ffmpeg
* Samba
* pyrenamer

　
# おおまかな流れ
1. 適当に動画撮影
2. 動画から機械的に静止画を大量生成
3. 大量の静止画から物体部分を機械的に抽出して背景が透過した物体画像生成
4. 別途用意した背景静止画と 3. で生成した物体静止画をランダムに回転・縮小・拡大・配置・ノイズ追加しながら合成して大量に水増し

　
# 動画→静止画変換
下記コマンドにより動画から生成したpngファイルを「YoloTrainDataGenerate/images_org/」へコピー<br>
`$ ffmpeg -i xxxx.mp4 -vcodec png -r 10 image_%04d.png`

　
# 複数静止画ファイル、複数物体周囲を機械的に一括透過加工
* 背景が白色に近い色・物体が白色／灰色以外の配色で構成されている場合のみ動作
* １画像内に複数物体が写っている場合は物体数分の画像ファイルへ分割して加工
* 入力画像が長方形であっても最終生成画像は物体を含む96×96の正方形
* エッジ抽出の都合上、重なり合っている物体は１つと認識される
* 検出された物体の面積が1000pxに満たない場合は当該物体を抽出対象から除外
* 最終生成された画像内に物体が存在しないと判断される場合はファイルを生成しない
```
$ cd YoloTrainDataGenerate
$ python3 object_extraction.py
```
(1) 編集元画像 1920x1080<br>
&nbsp;&nbsp;&nbsp;&nbsp;![1.png](https://github.com/PINTO0309/YoloTrainDataGenerate/blob/master/media/1.png)<br>
(2) 元画像の背景白色化 1920x1080<br>
&nbsp;&nbsp;&nbsp;&nbsp;![2.png](https://github.com/PINTO0309/YoloTrainDataGenerate/blob/master/media/2.png)<br>
(3) 物体検出 1920x1080<br>
&nbsp;&nbsp;&nbsp;&nbsp;![3.png](https://github.com/PINTO0309/YoloTrainDataGenerate/blob/master/media/3.png)<br>
(4) 背景透過処理後PNGファイル２枚 96x96<br>
&nbsp;&nbsp;&nbsp;&nbsp;![4.png](https://github.com/PINTO0309/YoloTrainDataGenerate/blob/master/media/4.png)&nbsp;&nbsp;&nbsp;&nbsp;![5.png](https://github.com/PINTO0309/YoloTrainDataGenerate/blob/master/media/5.png)![6.png](https://github.com/PINTO0309/YoloTrainDataGenerate/blob/master/media/6.png)
<br>

# 画像の前処理
images_org配下のファイル名をpyrenamer等を利用して 「(ラベル名)_xxxx.png」 に一括変更
* xxxx の箇所は同一ラベル名で重複しないように連番なり、文字列なり、自由に設定 (４桁でなくても良い)

```（例）.
　labelA_0001.png　→　「labelA」に集約
　labelA_0002.png　→　「labelA」に集約
　labelA_0003.png　→　「labelA」に集約
　labelB_0001.png　→　「labelB」に集約
　labelB_0002.png　→　「labelB」に集約
　labelC_0001.png　→　「labelC」に集約
　　　：
```

```（例）.ラベルがswitchとremoconの場合
　switch_0001.png　→　「switch」に集約
　switch_0002.png　→　「switch」に集約
　switch_0003.png　→　「switch」に集約
　switch_0004.png　→　「switch」に集約
　remocon_0001.png　→　「remocon」に集約
　remocon_0002.png　→　「remocon」に集約
　　　：
```
<br>

# Yolo学習用画像データ他の自動生成

* ランダムに回転・縮小・拡大・配置・ノイズ追加を繰り返して画像を水増し生成
  * コントラスト変換
  * 平滑化
  * ヒストグラム均一化
  * ガウシアンノイズ付加
  * Salt&Pepperノイズ付加
  * 画像反転
  * 画像回転
* 任意の物体画像と任意の背景画像を自由に合成
* 前処理で生成した連番付き画像ファイル名の連番部を無視し、複数画像をひとつのラベルへ集約
* アノテーションを含む train.txt、test.txt、label.txt が生成される
* images配下に水増し済みの静止画像が生成される
* デフォルトの水増し枚数は10,000枚、変更する場合は generate_sample.py の 「train_images = 10000」 を修正する


下記コマンドを実行
```
$ cd YoloTrainDataGenerate
$ sudo chmod +x *.sh
$ ./setup.sh
$ python3 generate_sample.py
```
<br>
<br>
<hr>

# [English] YoloTrainDataGenerate
Procedures and tools for semi-mechanically automatically generating YoloV2 original learning data from video<br>
https://qiita.com/PINTO/items/d5645734ca9c95b1c395

　
# Environment
* CPU：Intel Core i7-3517U(1.9GHz)
* MEM：16GB
* GPU：Geforce GT 650M (VRAM:2GB)
* OS：Ubuntu 16.04 LTS
* OpenCV 3.4.1
* ffmpeg
* Samba
* pyrenamer

　
# Rough flow
1. Suitable movie shooting
2. Massively generate still images mechanically from motion pictures
3. Mechanical extraction of object parts from a large amount of still images, generation of object images through which background passes
4. We created a background static image separately prepared and the object still image generated in 3 randomly add, rotate, reduce, enlarge, arrange, add noise and synthesize it and inflate a large amount

　
# Movie → Still image conversion
Copy the png file generated from the movie by the following command to "YoloTrainDataGenerate/images_org/"<br>
`$ ffmpeg -i xxxx.mp4 -vcodec png -r 10 image_%04d.png`

　
# Multiple still image files, Machining of multiple object perimeter through mechanical processing
* Only when the color is close to white · the object is composed only of white / gray color scheme
* When there are multiple objects in one image, it is divided into image files for the number of objects and processed
* Even if the input image is a rectangle, the final generated image is a 96×96 square including the object
* For convenience of edge extraction, one overlapping object is recognized as one
* When the area of the detected object is less than 1000px, the object is excluded from the extraction target
* If it is determined that there is no object in the final generated image, no file is generated
```
$ cd YoloTrainDataGenerate
$ python3 object_extraction.py
```
(1) Original image to edit 1920x1080<br>
&nbsp;&nbsp;&nbsp;&nbsp;![1.png](https://github.com/PINTO0309/YoloTrainDataGenerate/blob/master/media/1.png)<br>
(2) Background whitening of the original image 1920x1080<br>
&nbsp;&nbsp;&nbsp;&nbsp;![2.png](https://github.com/PINTO0309/YoloTrainDataGenerate/blob/master/media/2.png)<br>
(3) Object detection 1920x1080<br>
&nbsp;&nbsp;&nbsp;&nbsp;![3.png](https://github.com/PINTO0309/YoloTrainDataGenerate/blob/master/media/3.png)<br>
(4) Background PNG file after transparent processing 96x96<br>
&nbsp;&nbsp;&nbsp;&nbsp;![4.png](https://github.com/PINTO0309/YoloTrainDataGenerate/blob/master/media/4.png)&nbsp;&nbsp;&nbsp;&nbsp;![5.png](https://github.com/PINTO0309/YoloTrainDataGenerate/blob/master/media/5.png)![6.png](https://github.com/PINTO0309/YoloTrainDataGenerate/blob/master/media/6.png)
<br>

# Image preprocessing
Change file name under images_org to "(label name)_xxxx.png" collectively using pyrenamer etc.
* The part of xxxx is the same label name and it is a serial number so that it does not overlap, it becomes a character string, it is set freely (it does not have to be 4 digits)

```(Example1)
　labelA_0001.png　→　「labelA」Summarize
　labelA_0002.png　→　「labelA」Summarize
　labelA_0003.png　→　「labelA」Summarize
　labelB_0001.png　→　「labelB」Summarize
　labelB_0002.png　→　「labelB」Summarize
　labelC_0001.png　→　「labelC」Summarize
　　　：
```

```(Example2) switch and remocon
　switch_0001.png　→　「switch」Summarize
　switch_0002.png　→　「switch」Summarize
　switch_0003.png　→　「switch」Summarize
　switch_0004.png　→　「switch」Summarize
　remocon_0001.png　→　「remocon」Summarize
　remocon_0002.png　→　「remocon」Summarize
　　　：
```
<br>

# Yolo image data for learning automatic generation

* Repetitive rotation, reduction, enlargement, placement, and noise addition are repeated at random to create inflated images
  * Contrast transformation
  * Smoothing
  * Histogram equalization
  * Gaussian noise addition
  * Salt & Pepper noise addition
  * Image reversal
  * Image rotation
* Free combination of arbitrary object image and arbitrary background image
* Ignoring the serial number part of the image file name with sequential number generated in preprocessing, aggregate multiple images on one label
* Train.txt, test.txt, label.txt containing annotations are generated
* An inflated still image is generated under images folder
* The default number of padded sheets is 10,000, and if you change it, modify "generate_sample.py" train_images = 10000


Execute the following command
```
$ cd YoloTrainDataGenerate
$ sudo chmod +x *.sh
$ ./setup.sh
$ python3 generate_sample.py
```
