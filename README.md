# CRAFT

# _テキスト検出のための文字領域認識_

Youngmin Baek, Bado Lee, Dongyoon Han, Sangdoo Yun, and Hwalsuk Lee∗ 
Clova AI Research, NAVER Corp.

## 要旨
ニューラルネットワークに基づくシーンテキスト検知方法が最近浮上し、有望な結果を得ました。
従来の方法では、文字間の関係性と文字間の親和性を検出するために新しいシーン検知方法を提案します。
個々の文字レベルの注釈の欠如を克服するために、学習した中間モデルで獲得した実像のための文字レベルの注釈と文字レベルのグランドトゥルースの両方を利用します。
文字間の親和性を推定するために、ネットワークは新規提案された親和性のための表現で訓練されます。
自然画像の中の高カーブテキストを含む6つのベンチマークの拡張実験と、文字レベル検出が最先端検知器を大幅に上回ることを実証します。
その結果、提案手法は恣意的、カーブ、変形文などの複雑なシーン画像の検知において、高柔軟性を保証します。

## 序章
瞬間翻訳、画像検索、地中検索、盲点検など、数多くのアプリケーションがあり、コンピュータ視野で注目されています。
最近、ディープラーニングをベースにしたテキストデテクター(8、40、40、21、10、12、24、25、32、26)は、ワードライブボックスのローカライズを主にしています。
しかし、単一バウンディングボックスでは検知しにくい曲線、変形、極端に長いテキストなど、難しい場合には苦しみます。
代わりに、歴代キャラクターをボトムアップで結びつけることでテキストにチャレンジすると、キャラクターレベル認識は多くのメリットがあります。
既存のテキストデータセットの大部分は文字レベルの注釈ではなく、文字レベルのグランドトゥルースを得るために必要な作業は高すぎます。
本稿では個々の文字領域をローカライズし、検知文字をテキスト例にリンクするテキスト検知器を提案します。
文字領域認知のためのCRAFTと呼ばれる枠文字領域認知のために、文字領域スコアと親和スコアを出して複雑なニューラルネットワークで設計します。
地域スコアはイメージの個々の文字をローカライズし、親和スコアは各文字を一例にまとめて使います。
文字レベルの注釈の欠如を補うために、既存のリアルワードレベルデータセットの文字レベルグランドトゥルースを見積もる弱管理学習枠を提案します。
