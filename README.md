# CRAFT
論文:https://arxiv.org/abs/1904.01941

# _テキスト検出のための文字領域認識_

Youngmin Baek, Bado Lee, Dongyoon Han, Sangdoo Yun, and Hwalsuk Lee∗ 
Clova AI Research, NAVER Corp.

## 要旨
ニューラルネットワークを用いたシーンテキスト検出法が登場し、これまでの研究では、文字領域を任意の形状で表現することに限界があることが示されていましたが、本稿では、文字レベルの各アノテーションの欠如を克服するために、合成画像の文字レベルのアノテーションと、学習された中間モデルで得られた実像の文字レベルの真理の推定の両方を利用する新たなシーンテキスト検出法を提案しました。このネットワークは、文字間の親和性を推定するために、新たに提案された親和性の表現を訓練されています。自然画像に高度に湾曲したテキストを含むTotalTextおよびCTW-1500のデータセットを含む、6つのベンチマークに関する広範な実験で、我々の文字レベルの検出は、最先端の検出器を大きく上回っていることが示されています。この結果によれば、本発明者らが提案した方法は、任意に配向された、湾曲された、または変形されたテキストのような複雑なシーンテキスト画像を検出する際の高いフレキシビリティを保証する。

## 1.序章
シーンテキスト検出は、瞬時翻訳、画像検索、シーン構文解析、地理位置解析、ブラインドナビゲーションなどの応用が多く、コンピュータの視野で注目されています。最近では、深層学習によるシーンテキスト検出器(8,40,21,4,11,10,12,12,17,17,24,25,32,26)が有望な性能を示しています。これらの方法は、主に、ワードレベルの境界ボックスを局在化するためにネットワークを訓練するものですが、曲線、変形、極端に長いテキストなど、単一の境界ボックスでは検出しにくい難易度の高い場合に苦労することがあります。

既存のテキストデータセットのほとんどは文字レベルの注釈を提供せず、文字レベルのグラウンドトゥルースを得るために必要な作業は多すぎます。

![Figure1](https://github.com/inouetaka/CRAFT/blob/master/images/figure1.png)

## 2.関連作業
深層学習が登場する前のシーンテキスト検出の主な傾向は、基本的な構成要素としてMSER [27] やSWT [5] のような手作りの特徴が多く用いられるボトムアップであり、最近では、SSD [20]、Faster R-CNN [30]、FCN [23] のような一般的なオブジェクト検出/セグメンテーション方法を採用することで、深層学習に基づくテキスト検出器が提案されている。
### 回帰ベースのテキスト探知器
一般的なオブジェクト検出器に適合したボックス回帰を用いた様々なテキスト検出器が提案されている。一般的なオブジェクトとは異なり、テキストは様々なアスペクト比を有する不規則な形状で提示されることが多い。この問題に対処するために、テキストボックス[18]修正コンボリューションカーネル及びアンカーボックスを修正し、様々なテキスト形状を効果的に捕捉する。DMPNet[22]は、四角スライドウィンドウを組み込むことにより、問題をさらに低減しようと試みた。最近では、回帰フィルタを能動的に回転させることにより回転不変の特徴を十分に利用する回転感受性回帰検出器(RSDD)[19]が提案されているが、このアプローチを使用する際には、野生に存在する可能な形状を全て捕捉する。

### セグメンテーションベースのテキスト探知機
もう一つの一般的なアプローチは、画素レベルでテキスト領域を探索することを目的としたセグメンテーションを扱う作業に基づくもので、複数スケールのFCN[7]、ホリスティック予測[37]、PixelLink[4]のようなワード境界領域を推定することでテキスト領域を検出するアプローチも、セグメンテーションをベースとして提案されている。SSTD[8]は、アテンションメカニズムを用いて、フィーチャーレベルでのバックグラウンド干渉を低減することによりテキスト関連領域を強化することで、回帰とセグメンテーションの両方のアプローチから利益を得ようと試みている。

### エンドツーエンドのテキスト探知機
FOTS [21] とEAA [10] は、一般的な検出と認識の方法を連結し、エンドツーエンドの方法でトレーニングしている。マスクテキストスポッター [25] は、認識タスクを意味的なセグメンテーションの問題として扱うために、それらの統一モデルを利用している。認識モジュールを用いたトレーニングは、テキスト検出器がテキストのようなバックグラウンドのクラッターに対してより堅牢であるのを助けることは明らかである。
ほとんどの方法は単語を単位としてテキストを検出するが、単語を検出するための単語の範囲を定義することは、単語の意味、空間、色などの様々な基準によって区切ることができるため、決して簡単ではない。また、単語分割の境界は厳密に定義することができないので、単語セグメント自体は明確な意味を持たないので、単語注釈のあいまいさは、回帰アプローチとセグメンテーションアプローチの両方の根拠となる真実の意味を希薄化する。

### 文字レベルのテキスト探知機
Zhangら[39]は、MSER[27]によって蒸留されたテキストブロック候補を用いて文字レベル検出器を提案した。MSERを用いて、個々の文字を識別することは、低コントラスト、曲率、光反射のシーンなど、特定の状況下での検出の頑強性を制限するという事実を示した。Yaoら[37]は、文字レベルの注釈を必要とするテキスト領域のマップおよびリンク配向と共に、文字の予測マップを用いた。明示的な文字レベルの予測ではなく、Seglink[32]はテキストグリッド(部分テキストセグメント)をハントし、これらのセグメントを追加のリンク予測と関連付けた。
この研究は、文字レベル検出器を訓練するために弱い教師のフレームワークを使用するWordSup[12]のアイデアに触発されているが、Wordsupの欠点は、文字表現が矩形アンカーで形成され、カメラの視点を変えることによって誘発される文字の斜視変形に対して脆弱であり、また、バックボーン構造の性能(すなわち、SSDを使用し、アンカーボックスの数と大きさによって制限される)に拘束されることである。

![Figure2](https://github.com/inouetaka/CRAFT/blob/master/images/figure2.png)

## 3.方法論
私たちの主な目的は、個々の個性を自然映像の中に正確に位置づけることであり、そのためには、文字領域や文字間の親和性を予測する深いニューラルネットワークを訓練し、公的な文字レベルのデータセットが存在しないため、モデルを弱い教師の下で訓練することです。

### 3.1.構成
VGG-16 [34] に基づく完全コンボリューション型ネットワークアーキテクチャをバックボーンとして採用し、復号部にはU-net [31] と同様のスキップ接続を採用し、低レベルの機能を集約しています。最終出力は、領域スコアとアフィニティースコアの2つのチャネルをスコアマップとして示しています。ネットワークアーキテクチャを図2に図示します。

### 3.2.学習  

### 3.2.1 グラウンドトゥルースラベル生成
それぞれのトレーニング画像に対して、領域スコアのグラウンド真実ラベルと、文字レベルの境界ボックスを持つアフィニティースコアを生成し、領域スコアは所与の画素が文字の中心である確率を表し、アフィニティースコアは隣接する文字間のスペースの中心確率を表す。

各画素を個別にラベル付けするバイナリセグメンテーションマップとは異なり、文字中心の確率をガウスヒートマップで符号化するヒートマップは、厳密に境界のないグラウンドトゥルース領域を扱う際の柔軟性が高いことから、ポーズ推定[1,29]などの他の応用例で使用されており、領域スコアとアフィニティースコアの両方を学習するためにヒートマップ表現を使用しています。

図3は、合成画像のラベル生成パイプラインをまとめたものであり、境界ボックス内の各画素のガウス分布値を直接計算するのは非常に時間がかかりますが、画像上の文字の境界ボックスは、一般的に透視投影で歪んでいるため、領域スコアとアフィニティースコアの両方について、近似的に地理的真実を生成するために、1)2次元等方性ガウスマップを作成し、2)ガウスマップ領域と各文字ボックスとの間の透視変換を計算し、3)縦軸ガウスマップをボックス領域に生成します。

アフィニティースコアのグラウンドトゥルースについては、図3に示すように隣接する文字ボックスを用いてアフィニティーボックスを定義し、対角線を描いて各文字ボックスの反対の角を結ぶことで、2つの三角形(上下の文字の三角形)を生成し、隣接する文字ボックスのペアごとに、上下の三角形の中心をボックスの隅に設定することでアフィニティーボックスが生成されます。

提案されているグラウンドトゥルースの定義は、小さな受容野を用いても、モデルが十分に大長いテキストインスタンスを検出することを可能にするが、一方、ボックス回帰のような従来のアプローチは、大きな受容野を必要とし、我々の文字レベル検出は、テキストインスタンス全体ではなく、文字内及び文字間のみに焦点を当てることを可能にする。

![Figure3](https://github.com/inouetaka/CRAFT/blob/master/images/figure3.png)

### 3.2.2 弱教師学習
合成データセットとは異なり、通常、データセット内の実像はワードレベル注釈を有するが、ここでは、図4に要約したように、弱教師された形で、各ワードレベル注釈から文字ボックスを生成する。ワードレベル注釈を有する実像を提供する場合、学習された中間モデルは、クロップされたワード画像の文字領域スコアを予測して文字レベル結合ボックスを生成する。中間モデルの予測の信頼性を反映するために、各ワードボックス上の信頼マップの値は、検出された文字数を訓練中の学習ウェイトに使用されるグラウンドトゥルース文字数で除した値に比例して計算される。

![Figure4](https://github.com/inouetaka/CRAFT/blob/master/images/figure4.png)  

図6は、文字分割の全手順を示しています。第1に、単語レベルの画像を元の画像から切り取ること、第2に、現在までに訓練されたモデルが領域スコアを予測すること、第3に、文字境界ボックスを構成する文字領域を分割するために使用される分水界アルゴリズム[35]を使用すること、最後に、キャラクタボックスの座標を、クロッピングステップからの逆変換を使用して元の画像座標に戻すこと、そして、領域スコアとアフィニティースコアに対する擬似グラウンドトゥルース(擬似GTs)を、得られた四辺形文字レベルの境界ボックスを使用して、図3に記載されたステップにより生成することができます。
![figure6](https://github.com/inouetaka/CRAFT/blob/master/images/figure6.png)  

このモデルを弱教師下で訓練すると、不完全な擬似GTを訓練する必要があります。もしモデルが不正確な領域スコアで訓練されれば、文字領域内の出力がぼやけてしまう可能性があります。これを防ぐために、モデルによって生成された擬似GTの品質を測定します。幸いなことに、文字の長さであるテキスト注釈には非常に強力な手がかりがあります。ほとんどのデータセットでは、単語の転写が提供され、単語の長さが疑似GTの信頼性を評価するために使用できます。　　
　　
  
トレーニングデータの単語レベルの注釈付きサンプルwについては、R(w)とl(w)をそれぞれ境界ボックス領域、およびサンプルwの単語長とし、文字分割プロセスを通じて推定文字の境界ボックスとそれに対応する文字の長さlc(w)を求め、サンプルwの信頼スコアsconf(w)を次のように計算する。

![SconF(w)](https://github.com/inouetaka/CRAFT/blob/master/images/SconF(w).png)


画像の画素単位の信頼マップScは以下のように計算される。

![Sc(p)](https://github.com/inouetaka/CRAFT/blob/master/images/Sc(p).png)
  
ここで、pは領域R(w)の画素を表し、目的Lは以下のように定義される。

![L](https://github.com/inouetaka/CRAFT/blob/master/images/L.png)

ここで、Sr※(p)とSa※(p)はそれぞれ擬似グラウンド領域スコアとアフィニティーマップ、Sr(p)とSa(p)はそれぞれ予測領域スコアとアフィニティースコアを表し、合成データを用いたトレーニングでは真のグラウンドトゥルースを得ることができるので、Sc(p)は1に設定されます。

学習が行われるにつれて、CRAFTモデルは文字をより正確に予測することができ、信頼スコアsconf(w)も徐々に向上していきます。図5は、学習中の文字領域スコアマップを示しています。学習の初期段階では、自然画像の中で見慣れない文字に対しては、領域スコアは比較的低く、不規則なフォントなどの新しいテキストの外観を学び、SynthTextデータセットとは異なるデータ分布を持つ合成テキストの外観を学びます。

信頼スコアsconf(w)が0.5を下回る場合は、モデルの学習時に悪影響があるため、推定された文字境界ボックスは無視すべきであるが、この場合、個々の文字の幅は一定であると仮定し、単語領域R(w)を文字数l(w)で割るだけで文字レベル予測を計算し、sconf(w)を0.5に設定してテキストの見えない外観を学習する。

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

__エンドツーエンド方式との比較__  我々の方法は、グラウンドトゥルースボックスのみを検出するために訓練されているが、テーブル3に示すように、他のエンドツーエンドの方法と同等であり、失敗事例の分析から、我々のモデルは、特にグラウンドトゥルースの単語が視覚的な手がかりではなく意味論によって分離されている場合に、認識結果から利益を得ることが期待される。

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
[5] B.Epshtein,E.Ofek,andY.Wexler.Detectingtextinnatural scenes with stroke width transform. In CVPR, pages 2963–2970. IEEE, 2010. 2  
[6] A. Gupta, A. Vedaldi, and A. Zisserman. Synthetic data for text localisation in natural images. In CVPR, pages 2315-2324, 2016. 6  
[7] D.He,X.Yang,C.Liang,Z.Zhou,G.Alexander,I.Ororbia, D. Kifer, and C. L. Giles. Multi-scale fcn with cascaded instance aware segmentation for arbitrary oriented word spotting in the wild. In CVPR, pages 474–483, 2017. 2  
[8] P. He, W. Huang, T. He, Q. Zhu, Y. Qiao, and X. Li. Single shot text detector with regional attention. In ICCV, volume 6, 2017. 1, 2, 6  
[9] T. He, W. Huang, Y. Qiao, and J. Yao. Accurate text localization in natural image with cascaded convolutional text network. arXiv preprint arXiv:1603.09423, 2016. 11  
[10] T. He, Z. Tian, W. Huang, C. Shen, Y. Qiao, and C. Sun. An end-to-end textspotter with explicit alignment and attention. In CVPR, pages 5020–5029, 2018. 1, 2, 6, 7  
[11] W. He, X.-Y. Zhang, F. Yin, and C.-L. Liu. Deep direct regression for multi-oriented scene text detection. In CVPR, pages 745–753, 2017. 1, 6  
[12] H. Hu, C. Zhang, Y. Luo, Y. Wang, J. Han, and E. Ding. Wordsup: Exploiting word annotations for character based text detection. In ICCV, 2017. 1, 2, 5, 6  
[13] Y. Jiang, X. Zhu, X. Wang, S. Yang, W. Li, H. Wang, P. Fu, and Z. Luo. R2cnn: rotational region cnn for orientation robust scene text detection. arXiv preprint arXiv:1706.09579, 2017. 1, 6  
[14] D. Karatzas, L. Gomez-Bigorda, A. Nicolaou, S. Ghosh, A. Bagdanov, M. Iwamura, J. Matas, L. Neumann, V. R.
Chandrasekhar, S. Lu, et al. Icdar 2015 competition on robust reading. In ICDAR, pages 1156–1160. IEEE, 2015. 1  
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
