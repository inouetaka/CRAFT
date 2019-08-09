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

## 1.序章
瞬間翻訳、画像検索、地中検索、盲点検など、数多くのアプリケーションがあり、コンピュータ視野で注目されています。
最近、ディープラーニングをベースにしたテキストデテクター(8、40、40、21、10、12、24、25、32、26)は、ワードライブボックスのローカライズを主にしています。
しかし、単一バウンディングボックスでは検知しにくい曲線、変形、極端に長いテキストなど、難しい場合には苦しみます。
代わりに、文字をボトムアップで結びつけることでテキストにチャレンジすると、文字レベル認識は多くのメリットがあります。
既存のテキストデータセットの大部分は文字レベルの注釈ではなく、文字レベルのグランドトゥルースを得るために必要な作業は高すぎます。
本稿では個々の文字領域をローカライズし、検知文字をテキスト例にリンクするテキスト検知器を提案します。
文字領域認知のためのCRAFTと呼ばれる枠文字領域認知のために、文字領域スコアと親和スコアを出して複雑なニューラルネットワークで設計します。
領域スコアはイメージの個々の文字をローカライズし、親和スコアは各文字を一例にまとめて使います。
文字レベルの注釈の欠如を補うために、既存のリアルワードレベルデータセットの文字レベルグランドトゥルースを見積もる弱教師あり学習枠を提案します。  
![Figure1](https://github.com/inouetaka/CRAFT/blob/master/images/figure1.png)

## 2.関連作業
ディープラーニング登場前のシーン検知の主なトレンドはボトムアップであり、手作りの機能が主に使われていた(MSER[27]やSWT[5])。最近はSSD[20]、Faster R-CNN[30]、FCN[23]のような人気物体検知/セグメンテーション法を採用してディープラーニングベースのテキスト検知器が提案されている。
### 回帰ベースのテキスト探知器
人気物検知器を用いた様々なテキストレグレージョン適応が提案されています。一般的な物とは違い、様々なアスペクト比で不規則な形でテキストが提示されることが多いです。この問題に対処するために、テキストボックス[18] 変形コンカーネルとアンカーボックスは様々なテキスト形状を効果的に捕捉するために改良されています。DMPNet[22] は四角いスライディングウィンドウを取り入れることで問題のさらなる減らしに努めました。最近では、回転感覚回帰検知器(RSDD) [19] ボレーションフィルターを積極的に回転させることで不変の特徴を最大限に活用することが提案されました。

### セグメンテーションベースのテキスト探知機
もう一つの共通アプローチは、ピクセルレベルでテキスト領域を探求するセグメンテーションの手法です。マルチスケールFCN[7]、ホリスティック予測[37]、PixelLink[4]などの単語境界領域を推定してテキストを検出するアプローチはセグメンテーションをベースに提案しました。SSTD[8]は特徴レベルの背景干渉を減らして関連領域を強化するアテンションメカニズムを用いて回帰とセグメンテーションの両方の恩恵を受けようとしました。最近、テキストスネーク[24]は幾何学属性と共にテキスト領域とセンターラインを予測してテキストインスタンスを検知することを提案しました。

### エンドツーエンドのテキスト探知機
認識結果を活用して検知精度を高めるために、一斉検知と認識モジュールを同時に教育します。FOTS[21]とEAA[10]の一般検知と認識方法を組み合わせて、エンドツーエンドでトレーニングします。マスクテキストスポッター[25]は、統一モデルを利用して認識作業を意味論セグメンテーション問題として扱いました。認識モジュールでのトレーニングは、テキスト感知器がテキストのような背景の雑然としたものになるのを助けるのは明白です。
ほとんどの方法では単語でテキストを検知しますが、検知のために単語にするのは些細なことではありません。なぜなら単語は意味、空間、色など様々な基準で分けることができるからです。また、単語のセグメンテーションの境界は厳密に定義できないので、単語のセグメント自体は明確な意味合いがありません。この単語の注釈の曖昧さは回帰とセグメンテーションの両方のアプローチのためのグランドトゥルースの意味を希釈します。

### 文字レベルのテキスト探知機
チャンら[39]は、MSERで蒸留したテキストブロック候補で文字レベル感知器を提案しました。[27]では個々の文字を識別するためにMSERを使用することで、対照シーンなどの特定の状況下での感知力強さを制限しています。　　
ヤオら[37]は、文字領域の地図と、文字レベルの注釈が必要な連動方向の予測マップを使いました。明示文字レベル予測の代わりに、セグリンク[32]はテキストグリッド(一部テキストセグメント)を狩り、これらのセグメントを連想させて追加リンク予測を行います。マスクテキストスポッター[25]は文字レベル確率マップを予測しますが、個々の文字を見分ける代わりにテキスト記録に使用しました。
弱教師あり学習の枠組みで文字レベル感知器の学習を行うWordSup[12]の発想にインスパイアされた作品です。しかし、Wordsupの欠点は、文字の表現が長方形のアンカーで形成されていることで、カメラの見方によって誘導される文字のパース変形に弱いことです。また、バックボーン構造の性能(SSDを使用し、アンカーボックスの数と大きさで制限されていること)に縛られています。  
![Figure2](https://github.com/inouetaka/CRAFT/blob/master/images/figure2.png)

## 3.方法論
主な目的は、個々の文字を自然画像で正確にローカライズすることです。そのために、文字領域や文字間の親和性を予測する深層ネットワークを学習しています。公的文字レベルのデータセットがないので、弱教師あり学習を受けています。

### 3.1.構成
バッチノーマライゼーションでVGG-16[34]をベースとした全結合ネットワーク構造をバックボーンに採用しました。低レベルの特徴を集約する点でU-net[31]に似たデコーディング部分はスキップ接続があります。最終出力はスコアマップとして2チャンネルあります。リージョンスコアと親和スコアです。ネットワーク構造は図2で計画的に図示しています。

### 3.2.学習  

### 3.2.1 グランドトゥルースラベル生成
学習イメージ毎に、領域スコアのグランドトゥルースラベルと文字境界ボックスの親和スコアを作成します。領域スコアは与えられたピクセルが文字の中心である確率を表し、親和スコアは隣接文字の間隔のセンター確率を表します。
各ピクセルに個別にラベルを貼る連続セグメンテーションマップと違い、ガウシアンヒートマップで文字センターの確率をエンコードします。グランドトゥルースにこだわらない領域に対処する際の高柔軟性のため、ポーズ推定ワーク[1,29]などの他の用途で使用しています。領域スコアと親和スコアの両方を習得するためにヒートマップ表現を使用します。
図3は合成イメージのラベル生成パイプラインを要約します。境界ボックス内の各ピクセルの直接的な正規分布の値を計算するのはとても時間がかかります。イメージ上の文字の境界ボックスは概ね歪んでいるので、領域スコアと親和スコアの両方のグランドトゥルースをおおよそに生成します。1)2次元等方性ガウス図を準備します。2)ガウシアン図と各文字ボックスの間のパースチェンジ。3)ウォープガウシアンマップをボックスエリアにします。
親和スコアの地盤部分は、図3のように隣接文字ボックスで親和性ボックスを定義します。それぞれのキャラクターボックスの対角に対角になるように対角線を描くことで、上と下の三角形と呼ぶ三角形を生成します。次に、隣接する文字ボックスペアでは、上と下の三角形の中心を箱の隅に設けて親和性ボックスを作ります。
グランドトゥルースの定義を提案すると、小さい受信場ではあるが、長さの大きい文章や長さのある文章を十分に検知できるようになっている。一方、回帰ボックスのような従来のアプローチは、そのような場合には受容場が大きい必要がある。我々の文字レベル検知は、文章例全体ではなく、入り組んだフィルターとインターキャラクターのみに集中することができるようになっている。
![Figure3](https://github.com/inouetaka/CRAFT/blob/master/images/figure3.png)

### 3.2.2 弱教師付き学習
合成データセットとは異なり、データセットのリアル画像は通常単語レベルの注釈があります。ここでは、図4に要約したように、統計的に各単語レベルの注釈から、弱い教師で文字ボックスを作ります。単語レベルの注釈でリアルイメージを設けた場合、学習中間モデルは、クロップされた単語イメージの文字領域スコアを予測して、文字レベルの境界ボックスを作ります。中間モデルの予測の信頼性を反映するため、トレーニング中の重さ学習に使われるグランドトゥルース文字の数で割った検出文字の数に比例して、各単語ボックス上の信頼図の値を算出します。
![Figure4](https://github.com/inouetaka/CRAFT/blob/master/images/figure4.png)  

図6は文字の割り振りの全工程を示しています。まず、オリジナルイメージから言葉レベルのイメージを描きます。第2に、最新のトレーニングを受けたモデルは領域のスコアを予測します。第3に、文字のバウンディングボックスを形成するために使われるキャラクタ領域の分割に使います。最後に、文字ボックスのコーディネートは、刈り込み段階から変換することでオリジナルイメージコーディネートに戻します。領域のスコアは擬似グランドトゥルース(pseudoGTs)と親和スコアは、図3の手順で作成できます。獲得した四角い文字レベルのバウンディングボックスで行います。
![figure6](https://github.com/inouetaka/CRAFT/blob/master/images/figure6.png)  

モデルのトレーニングは不完全な擬似GTでトレーニングします。もしモデルが不正確な領域スコアでトレーニングされれば、文字領域で出力がぼやけてしまうかもしれません。これを防ぐために、モデルで発生する擬似GTの品質を測ります。幸いにも文字の長さである注釈には非常に強いキューがあります。ほとんどのデータセットでは単語の書き継ぎが設けられており、単語の長さは偽GTの信頼度を評価するのに使えます。　　
　　
  
トレーニングデータの単語レベル注釈サンプルwは、R(w)とl(l)をそれぞれ標本wの境界領域と語長にします。字割り工程で、推定文字の境界ボックスと対応文字の長さlc(w)を入手できます。そして標本wの確信スコアsconf(w)を計算します。
![SconF(w)](https://github.com/inouetaka/CRAFT/blob/master/images/SconF(w).png)


画像のピクセルに関するスキャンは、次のように表現されています。  
![Sc(p)](https://github.com/inouetaka/CRAFT/blob/master/images/Sc(p).png)
  
ここでPとは領域R(w)のピクセルを意味し、目標Lは次のように定義します。
![L](https://github.com/inouetaka/CRAFT/blob/master/images/L.png)

Sr∗(p)とSa∗(p)が擬地グランドトゥルーススコアを表し、親和図がそれぞれSr(p)とSa(p)が予測領域スコアと親和スコアを表します。合成データで訓練すると真のグランドトゥルースが得られるのでSc(p)は1に設定します。

トレーニングを行うことで、CRAFTモデルは文字をより正確に予測でき、sconf(w)も徐々に増加していきます。図5はトレーニング中の文字領域のスコアマップです。トレーニングの初期段階では、自然画像で見慣れないテキストは領域のスコアが比較的低いです。モデルは不規則フォントやSynthTextデータセットとは異なるデータ分布を持つ合成テキストなどの新しいテキストの外観を学習します。

確信得点sconf(w)が0.5以下ならモデルのトレーニング時に悪影響があるので推定文字境界ボックスは無視してください。この場合、個々の文字の幅は一定であると仮定し、l(w)の文字数で単純にRegion(w)を分けるだけで文字レベルの予測を計算します。そしてsconf(w)を0.5にして目に見えないテキストの外観を学習します。
![figure5](https://github.com/inouetaka/CRAFT/blob/master/images/figure5.png)

## 推論
推論段階では、最終出力は単語ボックスや文字ボックス等様々な形で格納でき、さらに多角的に格納できます。ICDAR等のデータセットは、評価プロトコルは単語レベルの交差点U(IoU)なので、予測されたSrとSaからの単語境界ボックスの作り方を、シンプルかつ効果的な後処理段階で記述します。

境界ボックス発見後の処理としては、1. Sr(p)>τrまたはSa(p)>τaで画像を覆うバイナリマップMを1に初期化し、τrを領域閾値、τaをアフィニティ閾値とする。2. Mで接続されたコンポーネントラベル(CCL)を行う。3. QuadBoxは、各ラベルに対応する接続されたコンポーネントを囲む最小の領域で回転した矩形を検出することにより得られる、というように要約され、OpenCVが提供するコネクテッドコンポーネントやminAreactのような機能を適用することができる。

なお、CRAFTの利点は、非最大サプレッション(NMS)のように、それ以上の後処理方法を必要としないことである。我々は、CCLによって分離されたワード領域のイメージブロブを有するので、単語の境界ボックスは、単に単一の囲い込み矩形によって定義される。別の点では、我々のキャラクタリンク処理は、明示的にテキストコンポーネント間の関係を検索することに依存する他のリンクベースの方法[32,12]とは異なる画素レベルで実行される。

また、図7に示すように、文字領域全体を有効に扱うために、多角形を生成することができます。多角形生成の手順は、図7に示すように、走査方向に沿った文字領域の局所最大線を見つけることです。図7に示すように、局所最大線の長さを青色の矢印で示しています。局所最大線の長さを等しく設定して、最終的な多角形の結果が不均一にならないようにします。局所最大線のすべてを結ぶ線は、黄色で示した中心線と呼ばれます。次に、局所最大線は、赤色の矢印で示した文字の傾斜角を反映するように中心線に対して垂直に回転します。局所最大線の終点は、テキスト多角形の制御点の候補です。
![figure7](https://github.com/inouetaka/CRAFT/blob/master/images/figure7.png)

## 4.実験
### 4.1.データセット
__ICDAR2013(IC13)__ は、高解像度画像、学習画像229枚と英語テキストを含むテスト画像233枚からなる、フォーカスシーンテキスト検出のための「Robust Reading Competition for Focusted spence text」の中でリリースされ、注釈は矩形ボックスを使用した単語レベルである。

__ICDAR2015(IC15)__ は、ICDAR 2015「偶発的なシーンテキスト検出のためのロバスト・リーディング・コンクール」に導入されました。このコンクールは学習1000枚とテスト画像500枚から構成され、いずれも英語のテキストで構成されています。注釈は、4角枠を使用した単語レベルで表示されます。

__ICDAR2017(IC17)__ では、学習画像7,200枚と検証画像1,800枚、多言語シーンテキスト検出のために9言語のテキストを用いたテスト画像9,000枚が収録されており、IC15と同様に、IC17のテキスト領域にも四角形の4つの頂点が注釈付けされています。

__MSRA-TD500(TD500)__ は、ポケットカメラを用いて室内・屋外で撮影した学習画像300枚とテスト画像200枚に分割された500枚の自然画像を収録し、英語・中国語の台本を入れ、文字領域を回転矩形で注釈付けしています。

__TotalText(TotalText)__ は、ICDAR 2017で最近発表された。学習1255枚とテスト画像300を含み、特に、ポリゴンと単語レベルの転写によって注釈付けされた曲線テキストを提供します。

__CTW-1500(CTW)__ は1学習画像000枚とテスト画像500枚で構成されていますが、いずれの画像にも曲線のテキストインスタンスがあり、14の頂点を持つ多角形で注釈が付いています。

## 4.2.学習戦略
トレーニング手順には、まずSynthText dataset [6] を使用して50k回繰り返しネットワークを訓練し、各ベンチマークデータセットをモデルの微調整に使用します。一部の「DO NOT CARE 2015」および「ICDAR 2017」データセットは、sconf (w) を設定してトレーニングでは無視されます。すべてのトレーニングプロセスでADAM [16] オプティマイザを使用します。マルチGPUトレーニングでは、トレーニングと監視GPUが分離され、メモリーに保存されます。SynthTextデータセットは、1:5の割合で、文字領域が確実に分離されるように使用されます。自然なシーンでテクスチャのようなテクスチャを除去するために、On-line Hard Negative Mining [33] が1:3の割合で適用されます。また、作物、ローテーション、および/またはカラーバリエーションなどの基本的なデータ増強技術が適用される。

弱教師ありでのトレーニングには、2種類のデータが必要です。IC13、IC15、IC17の単語の長さを計算するための単語画像と転写の四辺形注釈。MSRA-TD500、TotalText、CTW-1500などのその他のデータセットは要求事項を満たしていません。MSRA-TD500は転写を提供せず、TotalTextとCTW-1500はポリゴン注釈のみを提供します。そのため、ICDARデータセットのみでCRAFTをトレーニングし、微調整を行わずに他のデータセットでテストしました。ICDARデータセットでは2種類のモデルをトレーニングします。第1のモデルはIC15のみを評価するためにIC15でトレーニングを受けます。第2のモデルはIC13とIC17の両方でトレーニングを受け、他の5つのデータセットを評価するために使用します。余分な画像はトレーニングに使用しません。微調整のための反復回数は25kに設定しています。

## 4.3.実験結果
__四辺形データセット(ICDAR、MSRA-TD500)__ は、1つの解像度で実験を行います。IC13、IC15、IC17、MSRA-TD500は、それぞれ960、2240、2560、1600にサイズ変更されます。テーブル1は、ICDAR、MSRA-TD500のさまざまな方法のh平均スコアを示しています。エンドツーエンドの方法との公正な比較のため、オリジナルの論文を参考に検出のみの結果を取り入れています。全てのデータセットで最先端の性能を実現しています。また、IC13のデータセットでは、後処理が簡単で効果的なため、CRAFTは8.6 FPSで比較的高速に動作します。
![table1](https://github.com/inouetaka/CRAFT/blob/master/images/table1.png)

MSRA-TD500では、枠内の単語間のスペースを含め、行レベルで注釈が与えられます。前述のように、ワードボックスを結合する後処理ステップが適用されます。一方のボックスの右側と他のボックスの左側が十分に近い場合、2つのボックスが結合されます。TD500トレーニングセットで微調整は行われませんが、表1に示すように、CRAFTは他のすべての方法よりも優れています。

ポリゴン型データセット(TotalText, CTW-1500)注釈は多角形であるため、TotalTextとCTW-1500のモデルを直接訓練することは困難です。
そのため、IC13及びIC17のトレーニング画像のみを用いて、これらのデータセットから得られるトレーニング画像を微調整することなく、領域スコアからの多角形生成後処理を用いて、提供された多角形タイプの注釈に対処した。

TotalText、CTW-1500内の画像の長辺をそれぞれ1280、1024にサイズ変更し、テーブル2に多角形型データセットの実験結果を示します。CRAFTの個々の文字局在化能力により、他の方法と比較して任意の形状のテキストを検出することがより強固で優れた性能を発揮することができます。特に、TotalTextデータセットには図8に示すような曲線テキストを含む様々な変形があり、四角形ベースのテキスト検出器による適切な推論が不可能なため、これらのデータセットでは非常に限られた数の方法で評価することができます。

![table2](https://github.com/inouetaka/CRAFT/blob/master/images/table2.png)

## 4.4.考察
スケール分散に対する頑健性テキストの大きさは非常に多様であるにもかかわらず、我々は全てのデータセットに対して単一スケールの実験を行った。これは、スケール分散問題を扱うために複数スケールのテストに依存する他の多くの方法とは異なる。この利点は、テキスト全体ではなく、個々の文字をローカライズする我々の方法の特性からもたらされる。比較的小さな受容フィールドは、大きな画像の中の単一文字をカバーするのに十分であり、スケールの変形テキストを検出するにはCRAFTは堅牢である。

__Multi-language issue IC17データセット__ には、合成テキストデータセットに含まれていないバングラ文字、アラビア文字が含まれています。また、両言語とも、それぞれの文字が曲がりくねった形で書かれているため、バングラ文字、アラビア文字、ラテン語、韓国語、中国語、日本語を区別することができません。東アジアの文字の場合は、幅を一定にすることができ、弱教師あり学習で高性能化を図ることができます。

__エンドツーエンド方式との比較__  我々の方法は、グランドトゥルースボックスのみを検出するために訓練されているが、テーブル3に示すように、他のエンドツーエンドの方法と同等であり、失敗事例の分析から、我々のモデルは、特にグランドトゥルースの単語が視覚的な手がかりではなく意味論によって分離されている場合に、認識結果から利益を得ることが期待される。
![table3](https://github.com/inouetaka/CRAFT/blob/master/images/table3.png)

__一般化能力__ 私たちの手法は、3つの異なるデータセットで最先端の性能を達成し、さらに微調整することなく、特定のデータセットにオーバーフィットするのではなく、テキストの一般的な特性を捉えることができることを実証しました。

![Figure8](https://github.com/inouetaka/CRAFT/blob/master/images/figure8.png)

## 5.結論	
本稿では、文字レベルの注釈がなくても個々の文字を検出できるCRAFTという新しいテキスト検出器を提案しました。本提案の方法は、文字領域のスコアとキャラクタアフィニティスコアをボトムアップで完全にカバーするものです。文字レベルの注釈を備えた実際のデータセットはまれなので、中間モデルから擬似グラウンドトラスを生成する、弱教師学習方法を提案しました。CRAFTは、ほとんどの公開データセットで最先端の性能を示し、微調整することなくこれらの性能を示すことで一般化能力を実証します。今後の研究として、CRAFTの性能、堅牢性、一般化性が、より一般的な設定で適用可能な、より良いシーンテキストスポットシステムに変換されるかどうかを、認識モデルを用いて、エンドツーエンドで訓練したいと考えています。

__謝辞__
著者らは、Beomjorned Kim、Daehyun Nam、およびDonghyun Kimに、広範な実験の手助けをしてくれたことに感謝したい。

References
[1] Z. Cao, T. Simon, S.-E. Wei, and Y. Sheikh. Realtime multiperson 2d pose estimation using part affinity fields. In CVPR, pages 1302–1310. IEEE, 2017. 3
[2] L.-C. Chen, G. Papandreou, I. Kokkinos, K. Murphy, and A. L. Yuille. Deeplab: Semantic image segmentation with deep convolutional nets, atrous convolution, and fully connected crfs. PAMI, 40(4):834–848, 2018. 11
[3] C. K. Ch’ng and C. S. Chan. Total-text: A comprehensive dataset for scene text detection and recognition. In ICDAR, volume 1, pages 935–942. IEEE, 2017. 2
[4] D. Deng, H. Liu, X. Li, and D. Cai. Pixellink: Detecting scene text via instance segmentation. In AAAI, 2018. 1, 2, 6
[5] B.Epshtein,E.Ofek,andY.Wexler.Detectingtextinnatural
scenes with stroke width transform. In CVPR, pages 2963–
2970. IEEE, 2010. 2
[6] A. Gupta, A. Vedaldi, and A. Zisserman. Synthetic data for
text localisation in natural images. In CVPR, pages 2315–
2324, 2016. 6
[7] D.He,X.Yang,C.Liang,Z.Zhou,G.Alexander,I.Ororbia,
D. Kifer, and C. L. Giles. Multi-scale fcn with cascaded instance aware segmentation for arbitrary oriented word spotting in the wild. In CVPR, pages 474–483, 2017. 2
[8] P. He, W. Huang, T. He, Q. Zhu, Y. Qiao, and X. Li. Single shot text detector with regional attention. In ICCV, volume 6, 2017. 1, 2, 6
[9] T. He, W. Huang, Y. Qiao, and J. Yao. Accurate text localization in natural image with cascaded convolutional text network. arXiv preprint arXiv:1603.09423, 2016. 11
[10] T. He, Z. Tian, W. Huang, C. Shen, Y. Qiao, and C. Sun. An end-to-end textspotter with explicit alignment and attention. In CVPR, pages 5020–5029, 2018. 1, 2, 6, 7
[11] W. He, X.-Y. Zhang, F. Yin, and C.-L. Liu. Deep direct regression for multi-oriented scene text detection. In CVPR, pages 745–753, 2017. 1, 6
[12] H. Hu, C. Zhang, Y. Luo, Y. Wang, J. Han, and E. Ding. Wordsup: Exploiting word annotations for character based text detection. In ICCV, 2017. 1, 2, 5, 6
[13] Y. Jiang, X. Zhu, X. Wang, S. Yang, W. Li, H. Wang, P. Fu, and Z. Luo. R2cnn: rotational region cnn for orientation robust scene text detection. arXiv preprint arXiv:1706.09579, 2017. 1, 6
[14] D. Karatzas, L. Gomez-Bigorda, A. Nicolaou, S. Ghosh, A. Bagdanov, M. Iwamura, J. Matas, L. Neumann, V. R. Chandrasekhar, S. Lu, et al. Icdar 2015 competition on robust reading. In ICDAR, pages 1156–1160. IEEE, 2015. 1
[15] D. Karatzas, F. Shafait, S. Uchida, M. Iwamura, L. G. i Bigorda, S. R. Mestre, J. Mas, D. F. Mota, J. A. Almazan, and L. P. De Las Heras. Icdar 2013 robust reading competition. In ICDAR, pages 1484–1493. IEEE, 2013. 1
[16] D. P. Kingma and J. Ba. Adam: A method for stochastic optimization. In ICLR, 2015. 7
[17] M. Liao, B. Shi, and X. Bai. Textboxes++: A single-shot oriented scene text detector. Image Processing, 27(8):3676– 3690, 2018. 1, 6
[18] M. Liao, B. Shi, X. Bai, X. Wang, and W. Liu. Textboxes: A fast text detector with a single deep neural network. In AAAI, pages 4161–4167, 2017. 2
[19] M. Liao, Z. Zhu, B. Shi, G.-s. Xia, and X. Bai. Rotationsensitive regression for oriented scene text detection. In CVPR, pages 5909–5918, 2018. 2, 6
[20] W. Liu, D. Anguelov, D. Erhan, C. Szegedy, S. Reed, C.-Y. Fu, and A. C. Berg. Ssd: Single shot multibox detector. In ECCV, pages 21–37. Springer, 2016. 2
[21] X. Liu, D. Liang, S. Yan, D. Chen, Y. Qiao, and J. Yan. Fots: Fast oriented text spotting with a unified network. In CVPR, pages 5676–5685, 2018. 1, 2, 6, 7
[22] Y. Liu and L. Jin. Deep matching prior network: Toward tighter multi-oriented text detection. In CVPR, pages 3454– 3461, 2017. 2
[23] J. Long, E. Shelhamer, and T. Darrell. Fully convolutional networks for semantic segmentation. In CVPR, pages 3431– 3440, 2015. 2
[24] S. Long, J. Ruan, W. Zhang, X. He, W. Wu, and C. Yao. Textsnake: A flexible representation for detecting text of ar- bitrary shapes. arXiv preprint arXiv:1807.01544, 2018. 1, 2, 6
[25] P. Lyu, M. Liao, C. Yao, W. Wu, and X. Bai. Mask textspotter: An end-to-end trainable neural network for spotting text with arbitrary shapes. arXiv preprint arXiv:1807.02242, 2018. 1, 2, 6, 7
[26] P. Lyu, C. Yao, W. Wu, S. Yan, and X. Bai. Multi-oriented scene text detection via corner localization and region segmentation. In CVPR, pages 7553–7563, 2018. 1, 6
[27] J. Matas, O. Chum, M. Urban, and T. Pajdla. Robust wide-baseline stereo from maximally stable extremal regions. Im- age and Vision Computing, 22(10):761–767, 2004. 2
[28] N. Nayef, F. Yin, I. Bizid, H. Choi, Y. Feng, D. Karatzas, Z. Luo, U. Pal, C. Rigaud, J. Chazalon, et al. Icdar2017 robust reading challenge on multi-lingual scene text detection and script identification-rrc-mlt. In ICDAR, volume 1, pages 1454–1459. IEEE, 2017. 1
[29] A. Newell, K. Yang, and J. Deng. Stacked hourglass networks for human pose estimation. In ECCV, pages 483–499. Springer, 2016. 3
[30] S. Ren, K. He, R. Girshick, and J. Sun. Faster r-cnn: towards real-time object detection with region proposal net- works. PAMI, (6):1137–1149, 2017. 2
[31] O. Ronneberger, P. Fischer, and T. Brox. U-net: Convolutional networks for biomedical image segmentation. In MIC- CAI, pages 234–241. Springer, 2015. 3
[32] B. Shi, X. Bai, and S. Belongie. Detecting oriented text in natural images by linking segments. In CVPR, pages 3482– 3490. IEEE, 2017. 1, 2, 5, 6
[33] A. Shrivastava, A. Gupta, and R. Girshick. Training regionbased object detectors with online hard example mining. In CVPR, pages 761–769, 2016. 7
[34] K. Simonyan and A. Zisserman. Very deep convolutional networks for large-scale image recognition. In ICLR, 2015. 3
[35] L. Vincent and P. Soille. Watersheds in digital spaces: an efficient algorithm based on immersion simulations. PAMI, (6):583–598, 1991. 4
[36] C. Yao, X. Bai, W. Liu, Y. Ma, and Z. Tu. Detecting texts of arbitrary orientations in natural images. In CVPR, pages 1083–1090. IEEE, 2012. 2
[37] C. Yao, X. Bai, N. Sang, X. Zhou, S. Zhou, and Z. Cao. Scene text detection via holistic, multi-channel prediction. arXiv preprint arXiv:1606.09002, 2016. 2, 6
[38] L.Yuliang,J.Lianwen,Z.Shuaitao,andZ.Sheng.Detecting curve text in the wild: New dataset and new solution. arXiv preprint arXiv:1712.02170, 2017. 2, 6, 11
[39] Z. Zhang, C. Zhang, W. Shen, C. Yao, W. Liu, and X. Bai. Multi-oriented text detection with fully convolutional networks. In CVPR, pages 4159–4167, 2016. 2, 6
[40] X. Zhou, C. Yao, H. Wen, Y. Wang, S. Zhou, W. He, and J. Liang. East: an efficient and accurate scene text detector. In CVPR, pages 2642–2651, 2017. 1, 6
