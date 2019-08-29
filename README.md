# 論文
Englesh -> [Paper](https://arxiv.org/abs/1904.01941)  
日本語 -> [CRAFT論文の日本語訳](https://github.com/inouetaka/CRAFT/wiki/論文-日本語訳)

# 依存関係のインストール
requirements
* PyTorch>=0.4.1
* torchvision>=0.2.1
* opencv-python>=3.4.2
詳しい依存関係はrequirements.txtを確認

`pip install -r requirements.txt`

## トレーニング
現在調査中

## 事前学習済みモデル
* [学習済みパラメータ](https://drive.google.com/open?id=1Jk4eGD7crsqCCg9C9VjCLkMN3ze8kutZ)
* 学習済みパラメータを使ったテスト
`python test.py --trained_model=[WEIGHT_FILE] --test_folder=[FOLDER PATH TO TEST IMAGES] --cuda==[True or False]`
* ./resultフォルダーが自動生成されて結果が保存される

## 引数
* --trained_model:学習済みモデル
* --text_threshold:テキスト信頼閾値
* --low_text:テキストの下限スコア
* --link_threshold:リンク信頼閾値
* --canvas_size:推論のための最大画像サイズ
* --cuda:GPUの使用有無
* --mag_ratio:画像倍率
* --poly:ポリゴンタイプを有効にする結果
* --show_time:show処理時間
* --test_folder:画像を入力するためのフォルダパス
