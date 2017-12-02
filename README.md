# 3つのレコメンド系アルゴリズム
1. 協調フィルタリング
2. fasttextでの購買時系列を考慮したアイテムベースのproduct2vec(skipgram)
3. アイテムベースのtfidfなどの類似度計算を利用したレコメンド

# 1. 協調フィルタリング
協調フィルタリング自体は簡潔なアルゴリズムで、実装しようと思えば、簡単にできる類のものであるように思えるのですが、製品と製品の類似度を計算するのに、その製品を購入したユーザをベクトル列としてみなすと割と簡単に計算できます[5]。世の中のコンテンツはユーザの関連殿の計算の方が多い気がしますが、今回はアイテムにひもづくユーザをベクトルにします　

例えば、今回はbookmeter.comさまのユーザの読んだ本情報を用いて、一人のユーザを一つのユニークな特徴量としてみなすことで、本同士の関連度が計算可能になります  

Albertさんなどのブログなどを参考し、今回の問題に当てはめると、このようなことであると言えそうです。  

今期は要素間距離の計算方法に、cosine similarityを利用していますが、ジャッカードという距離の取り方や、色々あるので、実際に業務で使う時にはあるべきレコメンドリストを作って定量評価すると良いです。  

<p align="center">
  <img width="700px" src="https://user-images.githubusercontent.com/4949982/33258093-c83903ca-d39b-11e7-8c4d-0ca9622f6d91.png">
</p>
<div align="center"> 図1. 今回用いた協調フィルタリング </div>

今回用いさせていただいた、bookmeter.comさんから作成したデータセットは[こちら](https://storage.googleapis.com/nardtree/bookmeter-scraping-20171127/htmls.tar.gz)です。27GByte程度あるので、覚悟してダウンロードしてください（なんどもスクレイピングするのは気がひけるので、初期調査はこのデータセットを公開しますので、有効活用していただけると幸いです。用途はアカデミックな用途に限定されると思います）  

また、必要なユーザと読んだ本とその時のタイムスタンプの情報のみをまとめたものは、[こちら](https://storage.googleapis.com/nardtree/bookmeter-scraping-20171127/mapped.jsonp)からダウンロードできます。  

## 学習
Google Cloud Strageから、mapped.jsonpを当プロジェクトにダウンロードしたという前提です
```console
$ wget https://storage.googleapis.com/nardtree/bookmeter-scraping-20171127/mapped.jsonp
$ python3 reduce.py --fold1
$ cd collaborative-filtering-itembase 
$ python3 make_matrix.py --step1
$ python3 make_matrix.py --step2 # <- とても重いので、計算に数日みてください
```

## 結果.
**氷菓(1)(角川コミックス・エース)** との類似度  
当然、その本地身とシリーズが出て欲しいのですが、例えば、コミック版氷菓を読むユーザは、「我妻さんは俺のヨメ」シリーズも読むことがわかりました。  
氷菓が好きな人は、このシリーズもきっとおすすめです
```console
氷菓(1)(角川コミックス・エース) 1.0
氷菓(3)(角川コミックス・エース) 0.6324555320336759
氷菓(2)(角川コミックス・エース) 0.565685424949238
氷菓(4)(角川コミックス・エース) 0.5477225575051661
氷菓(5)(角川コミックス・エース) 0.4743416490252569
我妻さんは俺のヨメ（８） 0.4472135954999579
キン肉マン60(ジャンプコミックスDIG… 0.4472135954999579
弱虫ペダル　51(少年チャンピオン・コミッ… 0.4472135954999579
魔法遣いに大切なこと～夏のソラ～(1)(角… 0.4472135954999579
人生はまだ長いので(FEELCOMICS… 0.4472135954999579
キン肉マン59(ジャンプコミックスDIG… 0.4472135954999579
お前はまだグンマを知らない　1巻(バンチコ… 0.4472135954999579
我妻さんは俺のヨメ（５） 0.4472135954999579
トリニティセブン　７人の魔書使い(1)(ド… 0.4472135954999579
このお姉さんはフィクションです！？：4… 0.4472135954999579
我妻さんは俺のヨメ（６）(週刊少年マガジン… 0.4472135954999579
ナナマルサンバツ(7)(角川コミックス・… 0.4472135954999579
夏の前日（４） 0.4472135954999579
我妻さんは俺のヨメ（９）(週刊少年マガジン… 0.4472135954999579
お前はまだグンマを知らない　2巻(バンチコ… 0.4472135954999579
塩田先生と雨井ちゃん２ 0.4472135954999579
```
**ガガガ文庫　羽月莉音の帝国10** との類似度  
完全に趣味ですが、一時期、どハマりしていた「羽月莉音の帝国」シリーズです。お金と欲と権力に振り回される少年少女とディープステートたちは熱かった  

この系統の書籍が好きでしたが、当時は定量的に解析するすべを知らず、今このように類似度を計算すると感慨深いものがあります  
```console
ガガガ文庫　羽月莉音の帝国10（イラスト完全… 1.0
ガガガ文庫　羽月莉音の帝国9（イラスト完全版） 1.0
群れない生き方(SB文庫) 0.7071067811865475
ハチワンダイバー100万円争奪・地下将棋!… 0.7071067811865475
ガガガ文庫　羽月莉音の帝国3（イラスト完全版） 0.7071067811865475
大人のＡＤＨＤ　――もっとも身近な発達障害… 0.7071067811865475
ハチワンダイバー闇将棋集団・鬼将会を追え編… 0.7071067811865475
ガガガ文庫　羽月莉音の帝国5（イラスト完全版） 0.7071067811865475
奥の細道(21世紀によむ日本の古典15) 0.7071067811865475
ハチワンダイバー恋の“賭け将棋”にダイブ!… 0.7071067811865475
ハチワンダイバー激アツ修行!24時間将棋編… 0.7071067811865475
世界の端っことあんずジャム（２）(デザート… 0.7071067811865475
No.1理論―ビジネスで、スポーツで、受験… 0.7071067811865475
腸が嫌がる食べ物、喜ぶ食べ物　40歳を過ぎた… 0.7071067811865475
UXの時代―IoTとシェアリングは産業を… 0.7071067811865475
図解・速算の技術　一瞬で正確に計算するための… 0.7071067811865475
マンガでわかる有機化学　結合と反応のふしぎか… 0.7071067811865475
頭の体操　第１集～パズル・クイズで脳ミソを鍛… 0.7071067811865475
ガガガ文庫　羽月莉音の帝国7（イラスト完全版） 0.7071067811865475
コンなパニック（１）(なかよしコミックス) 0.7071067811865475
ガガガ文庫　羽月莉音の帝国8（イラスト完全版） 0.7071067811865475
ガガガ文庫　羽月莉音の帝国4（イラスト完全版） 0.5
シンギュラリティ・ビジネス　ＡＩ時代に勝ち残… 0.5
```
**ハーモニー〔新版〕(ハヤカワ文庫JA)**  に類似する本  
私が唯物論者になるきっかけで、ビッグデータと機械学習による意思決定のその先に置けそうな社会として、「意識」が消失するという仮説は大変魅力的な小説です  
伊藤計劃三部作の中で一番好きです。  
```console
ハーモニー〔新版〕(ハヤカワ文庫JA) 1.0
虐殺器官〔新版〕(ハヤカワ文庫JA) 0.4568027835606774
屍者の帝国(河出文庫) 0.2727051065113576
虐殺器官(ハヤカワ文庫JA) 0.14109861338756574
TheIndifferenceEngin… 0.12909944487358058
アンドロイドは電気羊の夢を見るか?(ハヤカ… 0.11756124576907943
PSYCHO-PASSGENESIS1… 0.11603391514496726
PSYCHO-PASSGENESIS2… 0.11489125293076057
know(ハヤカワ文庫JA) 0.11199737837836385
一九八四年[新訳版](ハヤカワepi文庫) 0.10744565033846916
PSYCHO-PASSASYLUM1(… 0.10264528445948555
PSYCHO-PASSGENESIS3… 0.09331569498236164
PSYCHO-PASSASYLUM2(… 0.09203579866168446
ニューロマンサー(ハヤカワ文庫SF) 0.0900003231847342
伊藤計劃記録II(ハヤカワ文庫JA) 0.08882967062031503
いなくなれ、群青(新潮文庫nex) 0.08679025878292934
[映]アムリタ(メディアワークス文庫の… 0.08626212458659283
華氏451度〔新訳版〕(ハヤカワ文庫SF) 0.08464673190724618
その白さえ嘘だとしても(新潮文庫nex) 0.08191004034460432
```
PSYCHO-PASSも好きですが、小説でてたんですか！良いですね。買います！←こんな感じのユースケースを想定しています

## データセットのダウンロード
1. [アイテムベースの協調フィルタリング結果](https://storage.googleapis.com/nardtree/bookmeter-collaborative-filtering-results/book_book.tar.gz)
2. [ユーザベースの協調フィルタリング結果](https://storage.googleapis.com/nardtree/bookmeter-collaborative-filtering-results/user_user.tar.gz)

# 2. fasttextでのアイテムベースのproduct2vec(skipgram)

一部でproduct2vecと呼ばれる技術のようですが、同名でRNNを用いた方法も提案されており、購買鼓動を一連の時系列として文章のように捉えることで、似た購買行動をするユーザの購買製品が似たようなベクトルになるという方法を取っているようです  

このベクトルとベクトルのコサイン距離か、ユークリッド距離を取れば、購買行動が似た商品ということができそうです  

リクルートではこのアリゴリズムは動作しているよう[4]なのですが、効果のほどはどうなんですかね？知りたいです  
<p align="center">
  <img width="700px" src="https://user-images.githubusercontent.com/4949982/33325568-fc19a280-d495-11e7-9240-482aaf383aeb.png">
</p>

## 期待される結果
- 流行があり、時代と時期によって読まれやすい本などが存在している場合、同じ時代に同じ流れで、読まれやすい本のレコメンド
- 本のコンテンツの類似度ではなく、同じような本を読む人が同じ時代にどういった方も、また読んでいたか、という解釈
- 時系列的な影響を考慮した協調フィルタリングのようなものとして働くと期待できる

## 学習アルゴリズム
- fasttext
- skipgram
- 512次元
- n-chargramは無効化

## 前処理
bookmeterさんからスクレイピンしたデータからユーザ名とIDで読んだ本を時系列順に紐づけます
```console
$ python3 parse_user_book.py --map1
$ python3 parse_user_book.py --fold1
```
bookmeterさんのデータをpythonで扱うデータ型に変換します
```console
$ python3 reduce.py --fold1
$ python3 reduce.py --label1 > recoomender-fasttext/dump.jsonp
```
fasttext(skipgramを今回計算するソフト)で処理できる形式に変換します
```console
$ cd recoomender-fasttext
$ python3 mkdataset.py
```
## SkipGramでベクトル化と、本ごとのcosime similarityの計算
```console
$ sh run.sh
$ python3 ranking.py --to_vec
$ mkdir sims
$ python3 ranking.py --sim
```


## 定性的な結果
1. 近年、本は大量に出版されて、その時に応じて売れ行きなどが変化するため、その時代に同じような本を買う傾向がある人が同じような買うというプロセスで似た傾向の本を買うと仮定ができそうである  
2. 本は、趣味嗜好の内容が似ている系列で似る傾向があり、コンテンツの内容では評価されない  

以上の視点を持ちながら、私が知っている書籍では理解しやすいので、いくつかピックアップした  

**聖☆おにいさん(11)(モーニングKC)** と同じような購買の行動で登場する本
```json
聖☆おにいさん(11)(モーニングKC) 聖☆おにいさん(11)(モーニングKC) 1.0 
聖☆おにいさん(11)(モーニングKC) 富士山さんは思春期(1)(アクションコミッ… 0.8972133709829763
聖☆おにいさん(11)(モーニングKC) 血界戦線4―拳客のエデン―(ジャンプコ… 0.8880415759167462
聖☆おにいさん(11)(モーニングKC) 闇の守り人2(Nemuki+コミックス) 0.874690584963135
聖☆おにいさん(11)(モーニングKC) 高台家の人々1(マーガレットコミックス) 0.8739713568492584
聖☆おにいさん(11)(モーニングKC) よりぬき青春鉄道(MFコミックスジーンシ… 0.8654563200372407
聖☆おにいさん(11)(モーニングKC) MUJIN―無尽―2巻(コミック(Y… 0.864354621889988
聖☆おにいさん(11)(モーニングKC) 日日べんとう8(オフィスユーコミックス) 0.8642825499104115
聖☆おにいさん(11)(モーニングKC) Baby,ココロのママに!(1)(ポラリ… 0.8633512045595658
聖☆おにいさん(11)(モーニングKC) 僕とおじいちゃんと魔法の塔（１）(怪CO… 0.8599349055581674
```

**ボールルームへようこそ(5)** と同じような購買の行動で登場する本
```json
ボールルームへようこそ(5)(講談社コミッ… ボールルームへようこそ(5)(講談社コミッ… 0.9999999999999999
ボールルームへようこそ(5)(講談社コミッ… 乙女座・スピカ・真珠星―タカハシマコ短編集… 0.9209695753657776
ボールルームへようこそ(5)(講談社コミッ… 微熱×発熱(少コミフラワーコミックス) 0.9200136951145198
ボールルームへようこそ(5)(講談社コミッ… 銀魂-ぎんたま-52(ジャンプコミックス) 0.9192139559710213
ボールルームへようこそ(5)(講談社コミッ… ボールルームへようこそ(3)(講談社コミッ… 0.9185994617714857
ボールルームへようこそ(5)(講談社コミッ… リメイク5(マッグガーデンコミックスE… 0.9178462840921753
ボールルームへようこそ(5)(講談社コミッ… カラダ探し2(ジャンプコミックス) 0.9133800737256051
ボールルームへようこそ(5)(講談社コミッ… エンジェル・ハート1STシーズン4(ゼノ… 0.9131301530550904
ボールルームへようこそ(5)(講談社コミッ… モテないし・・・そうだ、執事を召喚しよう。… 0.912900126682059
ボールルームへようこそ(5)(講談社コミッ… IPPO1(ヤングジャンプコミックス) 0.9127181876031607
```

**バーナード嬢曰く。(REXコミックス)** と同じような購買の行動で登場する本
```json
バーナード嬢曰く。(REXコミックス) バーナード嬢曰く。(REXコミックス) 0.9999999999999999
バーナード嬢曰く。(REXコミックス) 宝石の国(7)(アフタヌーンKC) 0.8471396900305308
バーナード嬢曰く。(REXコミックス) 宝石の国(5)(アフタヌーンKC) 0.8382072129764407
バーナード嬢曰く。(REXコミックス) 里山奇談 0.816403634050381
バーナード嬢曰く。(REXコミックス) ファイブスター物語(12)(ニュータイプ… 0.8158735339937038
バーナード嬢曰く。(REXコミックス) 汐の声(山岸凉子スペシャルセレクション2) 0.813237955152229
バーナード嬢曰く。(REXコミックス) ゴールデンカムイ5(ヤングジャンプコミッ… 0.8124194507560973
バーナード嬢曰く。(REXコミックス) サンドマン(1)(DCCOMICSV… 0.8063562786141018
バーナード嬢曰く。(REXコミックス) ナチュン(5)(アフタヌーンKC) 0.8062792077739434
バーナード嬢曰く。(REXコミックス) めぞん一刻15(ビッグコミックス) 0.8050153883069142
```

34万冊にも及ぶ購買行動関連スコアを計算したので、きっとあなたの好きな本を広げるにも役に立つはずです。参考にしていただければ幸いです  

[Google Cloud Strageにtar.gzで固めたファイルを置いてある](https://storage.googleapis.com/nardtree/bookmeter-fasttext-sims-20171127/sims.tar.gz)ので、どの本が何の本と関連度がどれくらい持ちたい方は参考にしてみてください

# アイテムベースのtfidfの類似度を利用したレコメンド
今更、tfidfなのかという気持ちもあるのですが、ユーザの購買行動ログが蓄積されていない時には有効な手段です  
skipgramにせよRNNのEncoder-Decoderモデルを利用するにせよ、ログがある程度あることを前提とするので、コンテンツを示す情報が何かしら適切に表現されていれば、類似度の計算が可能です。  
例えば、本の概要情報をBag Of Wordsとして文字列のベクトルとしてみなすことで、レコメンドが可能になります。  

## 前処理
本の概要情報をtfidfでベクトル化します
```console
$ python3 tfidf.py --make
```
## 類似度を計算
```console
$ python3 tfidf.py --similarity
```
## 具体例
**その女アレックス (文春文庫)**  
```json
{ 
  "その女アレックス (文春文庫)": 1.0,
  "死のドレスを花婿に (文春文庫)": 0.217211675720195,
  "蟻の菜園 ―アントガーデンー (宝島社文庫 『このミス』大賞シリーズ)": 0.17019688046138778,
  "ゴーストマン 時限紙幣": 0.15007411620369276, 
  "使命と魂のリミット (角川文庫)": 0.11353386562607512,
  "ネトゲの嫁は女の子じゃないと思った? (電撃文庫)": 0.10053605201421331,
  "絶叫": 0.09928094382921815, 
  "限界点": 0.09839970550260285,
  "探偵が早すぎる (上) (講談社タイガ)": 0.09710240347327244,
  "何様ですか? (宝島社文庫 『このミス』大賞シリーズ)": 0.0967122631023104
}
```
**ビブリア古書堂の事件手帖 (6) ~栞子さんと巡るさだめ~ (メディアワークス文庫)**  
```json
{
  "ビブリア古書堂の事件手帖 (6) ~栞子さんと巡るさだめ~ (メディアワークス文庫)": 1.0,
  "ビブリア古書堂の事件手帖4 ~栞子さんと二つの顔~ (メディアワークス文庫)": 0.14235848517270094,
  "ビブリア古書堂の事件手帖 (5) ~栞子さんと繋がりの時~ (メディアワークス文庫)": 0.12690320576737468,
  "ビブリア古書堂の事件手帖 2 栞子さんと謎めく日常 (メディアワークス文庫)": 0.10860850572897893,
  "ビブリア古書堂の事件手帖3 ~栞子さんと消えない絆~ (メディアワークス文庫)": 0.09155840600578283,
  "0能者ミナト〈3〉 (メディアワークス文庫)": 0.08957995681421002,
  "ビブリア古書堂の事件手帖7 ~栞子さんと果てない舞台~ (メディアワークス文庫)": 0.07945929682993862,
  "きみと暮らせば (徳間文庫)": 0.07897713721113855,
  "ビブリア古書堂の事件手帖―栞子さんと奇妙な客人たち (メディアワークス文庫)": 0.07818237184568279,
  "天使たちの課外活動２ - ライジャの靴下 (C・NOVELSファンタジア)": 0.07661947755379202
}
```

# オプションデータ
## 月ごとの本棚に登録された本の累計（その月での）数
<p align="center">
  <img width="750px" src="https://user-images.githubusercontent.com/4949982/33263544-9a98ddd4-d3ad-11e7-9649-bd8743758763.png">
</p>

## 読んでる人が多い本ランキング
読んでる人が多い本(2017年11月時点)
```json
永遠の0(講談社文庫) 6805
ビブリア古書堂の事件手帖―栞子さんと奇妙な客… 6551
舟を編む 6500
イニシエーション・ラブ(文春文庫) 6158
火花 5988
阪急電車(幻冬舎文庫) 5979
夜は短し歩けよ乙女(角川文庫) 5964
君の膵臓をたべたい 5504
コンビニ人間 5134
ビブリア古書堂の事件手帖2栞子さんと謎め… 5075
レインツリーの国(新潮文庫) 4994
その女アレックス(文春文庫) 4792
ぼくは明日、昨日のきみとデートする(宝島社… 4631
氷菓(角川文庫) 4606
ビブリア古書堂の事件手帖3~栞子さんと消え… 4590
```
6ヶ月以上、連続で本を読んでいる人の読んでる本ランキング
```json
1 舟を編む 5419 
1 永遠の0(講談社文庫) 5296
1 ビブリア古書堂の事件手帖―栞子さんと奇妙な客… 5277
1 火花 5081 
1 イニシエーション・ラブ(文春文庫) 4784
1 阪急電車(幻冬舎文庫) 4570
1 夜は短し歩けよ乙女(角川文庫) 4427
1 君の膵臓をたべたい 4422
1 コンビニ人間 4225
1 その女アレックス(文春文庫) 4186
```
6ヶ月間、連続で本を読むことができなかった人の読んでる本ランキング
```json
0 夜は短し歩けよ乙女(角川文庫) 1537
0 永遠の0(講談社文庫) 1509
0 阪急電車(幻冬舎文庫) 1409
0 イニシエーション・ラブ(文春文庫) 1374
0 ビブリア古書堂の事件手帖―栞子さんと奇妙な客… 1274
0 ぼくは明日、昨日のきみとデートする(宝島社… 1143
0 レインツリーの国(新潮文庫) 1120
0 君の膵臓をたべたい 1082
0 舟を編む 1081
0 西の魔女が死んだ(新潮文庫) 1004
```

## 参考
[1] [Instacart Product2Vec & Clustering Using word2vec](https://www.kaggle.com/goodvc/instacart-product2vec-clustering-using-word2vec)  
[2] [MRNet-Product2Vec: A Multi-task Recurrent Neural Network for Product Embeddings](https://arxiv.org/pdf/1709.07534.pdf)  
[3] [Deep Learning at AWS: Embedding & Attention Models](https://www.slideshare.net/AmazonWebServices/deep-learning-at-aws-embedding-attention-models)  
[4] [リクルートのデータで世界へ挑む　組織を率いるサイエンティストの仕事観](http://logmi.jp/134774)
[5] [推薦システムのアルゴリズム](http://www.kamishima.net/archive/recsysdoc.pdf)
