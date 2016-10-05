<properties
	pageTitle="Machine Learning Studio におけるサンプル データセットの使用 | Microsoft Azure"
	description="ML Studio に含まれるサンプル モデルで使用されるデータセットの説明です。実験には、これらのサンプル データセットを使用できます。"
	services="machine-learning"
	documentationCenter=""
	authors="garyericson"
	manager="jhubbard"
	editor="cgronlun"/>

<tags
	ms.service="machine-learning"
	ms.workload="data-services"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/16/2016"
	ms.author="garye"/>


# Azure Machine Learning Studio におけるサンプル データセットの使用

[top]: #machine-learning-sample-datasets

Azure Machine Learning で新しいワークスペースを作成する場合、既定でさまざまなサンプルのデータセットと実験が含まれています。こうしたサンプル データセットの多くは、[Cortana Intelligence ギャラリー](http://gallery.cortanaintelligence.com/)のサンプル モデルで使用されています。その他にも Machine Learning で使用するさまざまな種類のデータの例が含まれています。

データセットの一部は、Azure BLOB ストレージで使用できます。次の表ではこれらのデータセットへの直接のリンクを提供します。これらのデータ セットは、[データのインポート][import-data] モジュールを使用して実験で使用できます。

ML Studio で新しい実験を開くか作成する場合、サンプル データセットの残りは、実験キャンバスの左側にあるモジュール パレットの **[保存されたデータセット]** に一覧表示されます。実験キャンバスにこれらのデータセットのいずれかをドラッグすると、独自の実験で使用できます。


<!--
For a list of sample experiments available in ML Studio, see [Machine Learning Sample Experiments][sample-experiments].

[sample-experiments]: machine-learning-sample-experiments.md
-->

[AZURE.INCLUDE [machine-learning-free-trial](../../includes/machine-learning-free-trial.md)]


<table>

<tr>
  <th align=left>データセットの名前</th>
  <th align=left>データセットの説明</th>
</tr>

<tr>
  <td valign=top>米国国勢調査局提供の、成人収入に関する二項分類データセット</td>
  <td valign=top>
調整後の所得指数が 100 を超える就労成人男性 (16 歳以上) を対象とした、米国国勢調査局のデータベース (1994 年) のサブセットです。<p> </p><b>使用法:</b> 人口統計データを使用して対象の人々を分類し、個人が 1 年間に 50,000 ドル以上の年収を得られるかどうかを予測します。<p> </p><b>関連の研究:</b> Kohavi, R.、Becker, B. (1996 年)。UCI Machine Learning Repository <a href="http://archive.ics.uci.edu/ml">http://archive.ics.uci.edu/ml</a>。カリフォルニア州アーバイン: カリフォルニア大学、情報・コンピューター サイエンス学部
  </td>
</tr>

<tr ID=airport-codes-dataset>
  <td valign=top>空港コードのデータセット</td>
  <td valign=top>
米国の空港コード。<p> </p>このデータセットは、米国の各空港ごとに 1 行という構成になっており、各行には、空港の ID 番号、名前、所在する都市および州が記載されています。
  </td>
</tr>

<tr>
  <td valign=top>自動車価格データ (生データ)</td>
  <td valign=top>
メーカー/モデル別にまとめた自動車の情報です。価格、シリンダー数、燃費、保険リスク スコアなどの情報が含まれます。<p> </p>リスク スコアはまず車体価格に関連付けられた後、アクチュアリー (保険数理士) の間でシンボリングと呼ばれるプロセスによって、実際のリスクに対して調整されます。+3 は自動車のリスクが高く、-3 は非常に高い安全性が見込まれることを示しています。<p> </p><b>使用法:</b> 回帰または多変量分類を使用して、機能別のリスク スコアを予測します。<p> </p><b>関連の研究:</b> Schlimmer, J.C.(1987 年)。UCI Machine Learning Repository <a href="http://archive.ics.uci.edu/ml">http://archive.ics.uci.edu/ml</a>。カリフォルニア州アーバイン: カリフォルニア大学、情報・コンピューター サイエンス学部
  </td>
</tr>

<tr ID=bike-rental-uci-dataset>
  <td valign=top>自転車レンタルの UCI データセット</td>
  <td valign=top>
UCI Bike Rental データセットを使用します。このデータセットは、ワシントン D.C. で自転車のレンタル ネットワークを管理している Capital Bikeshare 社の実際のデータに基づいています。<p> </p>このデータセットは、2011 年と 2012 年の各日の 1 時間ごとに 1 行という構成になっており、全体で 17,379 行あります。毎時間の自転車レンタル数の範囲は 1 ～ 977 です。

  </td>
</tr>

<tr ID=bill-gates-rgb-image>
  <td valign=top>Bill Gates の RGB 画像</td>
  <td valign=top>
CSV データに変換された、公開の画像ファイル。<p> </p>画像を変換するコードは、<strong>K-平均法クラスタリングを使用した色の量子化</strong>モデルの詳細ページで提供されています。
  </td>
</tr>

<tr>
  <td valign=top>献血データ</td>
  <td valign=top>
台湾新竹市にある輸血サービス センターの献血者データベースから提供されたデータのサブセットです。<p> </p>献血データには、最後の献血からの経過月数、頻度、献血の合計回数、最後の献血からの経過期間、献血量が含まれます。<p> </p><b>使用法:</b> 目標は、2007 年 3 月に献血者が献血を実施したかどうかの分類に基づいて予測を実行することです。1 は対象期間に献血をした人、0 はしなかった人を示しています。<p> </p><b>関連の研究:</b> Yeh, I.C. (2008 年)。UCI Machine Learning Repository <a href="http://archive.ics.uci.edu/ml">http://archive.ics.uci.edu/ml</a>。カリフォルニア州アーバイン: カリフォルニア大学、情報・コンピューター サイエンス学部 <p> </p>Yeh, I-Cheng、Yang, King-Jang、Ting, Tao-Ming、「Knowledge discovery on RFM model using Bernoulli sequence」、Expert Systems with Applications、2008 年、<a href="http://dx.doi.org/10.1016/j.eswa.2008.07.018">http://dx.doi.org/10.1016/j.eswa.2008.07.018</a>
  </td>
</tr>

<tr ID=book-reviews-from-amazon>
  <td valign=top>Amazon の書評</td>
  <td valign=top>
ペンシルベニア大学の研究者によって Amazon.com Web サイトから取得された Amazon での本のレビュー (<a href="http://www.cs.jhu.edu/~mdredze/datasets/sentiment/">Sentiment</a>)。John Blitzer、Mark Dredze、Fernando Pereira による次の研究論文を参照してください: 「Biographies, Bollywood, Boom-boxes and Blenders: Domain Adaptation for Sentiment Classification (伝記、ボリウッド、ラジオカセットレコーダ、ミキサー: 感想の分類に対する領域適応)」、コンピューター言語学会 (ACL)、2007 年。<p> </p>元のデータセットには、1、2、3、4、5 の 5 段階評価付きのレビューが 975,000 件含まれています。レビューは英語で書かれており、対象期間は、1997 ～ 2007 年です。このデータセットは、ダウンサンプリングされており、10,000 件のレビューが含まれています。
  </td>
</tr>

<tr>
  <td valign=top>乳がんデータ</td>
  <td valign=top>
機械学習に関する文献で頻繁に登場する腫瘍学研究所が提供した、3 つのがん関連データセットの 1 つです。診断情報と、研究所が分析した 300 個の細胞組織サンプルの特徴が組み合わせられています。<p> </p><b>使用法:</b> がんの種類を 9 つの属性 (線形、カテゴリなど) に基づいて分類します。<p> </p><b>関連の研究:</b> Wohlberg, W.H.、Street, W.N.、Mangasarian, O.L.(1995 年)。UCI Machine Learning Repository <a href="http://archive.ics.uci.edu/ml">http://archive.ics.uci.edu/ml</a>。カリフォルニア州アーバイン: カリフォルニア大学、情報・コンピューター サイエンス学部
  </td>
</tr>

<tr ID=breast-cancer-features>
  <td valign=top>乳がん特徴
  <td valign=top>
それぞれ 117 個の特徴によって説明される、約 102,000 件の疑わしい領域 (対象) を示す X 線画像に関する情報が含まれます。特徴は独占所有されており、データセット作成元 (Siemens Healthcare) は特徴の意味を公開していません。
  </td>
</tr>

<tr ID=breast-cancer-info>
  <td valign=top>乳がん情報</td>
  <td valign=top>
データセットには、X 線画像の疑わしい領域それぞれについての追加情報が含まれています。それぞれの例に、乳がんの特徴データセット内の対応する行番号について、ラベル、患者 ID、画像全体に対する患部の座標などの情報が提供されています。1 人の患者に複数の例あります。がんを発症している患者については、いくつかの例は陽性で、いくつかの例は陰性です。がんでない患者の場合、すべての例は陰性です。データセットには約 102,000 件の例が含まれています。データセットには偏りがあり、陽性は全ポイントの 0.6% で、残りは陽性です。データセットは Siemens Healthcare によって提供されています。
  </td>
</tr>

<tr ID=crm-appetency-labels-shared>
  <td valign=top>CRM 強い欲求ラベルの共有</td>
  <td valign=top>
KDD Cup 2009 顧客間関係の予測に関する課題のラベル (<a href="http://www.sigkdd.org/site/2009/files/orange_small_train_appetency.labels">orange_small_train_appetency.labels</a>)。
  </td>
</tr>

<tr ID=crm-churn-labels-shared>
  <td valign=top>CRM 離反ラベルの共有</td>
  <td valign=top>
KDD Cup 2009 顧客間関係の予測に関する課題のラベル (<a href="http://www.sigkdd.org/site/2009/files/orange_small_train_churn.labels">orange_small_train_churn.labels</a>)。
  </td>
</tr>

<tr ID=crm-dataset-shared>
  <td valign=top>CRM データセットの共有</td>
  <td valign=top>
このデータは、KDD Cup 2009 顧客関係の予測に関する課題から取得しています (<a href="http://www.sigkdd.org/kdd-cup-2009-customer-relationship-prediction - orange_small_train.data.zip">orange_small_train.data.zip</a>)。<p> </p>データセットには、French Telecom company Orange の顧客 50,000 人のデータが含まれます。各顧客には匿名化された特徴が 230 あり、その中の 190 が数値で、40 がカテゴリです。特徴はきわめて疎です。
  </td>
</tr>

<tr ID=crm-upselling-labels-shared>
  <td valign=top>CRM アップセリング ラベルの共有</td>
  <td valign=top>
KDD Cup 2009 顧客間関係の予測に関する課題のラベル (<a href="http://www.sigkdd.org/site/2009/files/orange_large_train_upselling.labels">orange_large_train_upselling.labels</a>)。
  </td>
</tr>

<tr>
  <td valign=top>エネルギー効率回帰データ</td>
  <td valign=top>
12 種類の建物形状に基づいてシミュレーションされたエネルギー プロファイルのコレクションです。それぞれの建物は、ガラス部分、ガラス部分の配置、方向など、8 種類の特徴によって区別されています。<p> </p><b>使用法:</b> 回帰または分類を使用して、2 つの実数値のうち 1 つを応答としてエネルギー効率を評価します。マルチクラス分類の場合、応答変数が最も近い整数に丸められます。<p> </p><b>関連の研究:</b> Xifara, A.、Tsanas, A.(2012 年)。UCI Machine Learning Repository <a href="http://archive.ics.uci.edu/ml">http://archive.ics.uci.edu/ml</a>。カリフォルニア州アーバイン: カリフォルニア大学、情報・コンピューター サイエンス学部
  </td>
</tr>

<tr ID=flight-delays-data>
  <td valign=top>フライト遅延データ</td>
  <td valign=top>
米国運輸省の TranStats データ コレクションから取得した旅客機の定時運航データ (<a href="http://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236&DB_Short_Name=On-Time">定時</a>)。<p> </p>データセットには、2013 年 4 月から 10 月までの期間のデータが含まれます。Azure Machine Learning Studio へのアップロードの前に、データセットを次のように処理します。<ul><li>米国本土の混雑度が上位 70 位までの空港を含めるように、データセットをフィルター処理します。</li><li>キャンセルされたフライトは 15 分超の遅延として分類します。</li><li>迂回したフライトをフィルターで外します。</li><li>Year、Month、DayofMonth、DayOfWeek、Carrier、OriginAirportID、DestAirportID、CRSDepTime、DepDelay、DepDel15、CRSArrTime、ArrDelay、ArrDel15、Cancelled の各列が選択されています。</li></ul>
</td>
</tr>

<tr>
  <td valign=top>定刻フライト実績 (生データ)</td>
  <td valign=top>
2011 年 10 月の米国における航空機のフライトの発着記録です。<p> </p><b>使用法:</b> フライトの遅延を予測します。<p> </p><b>関連の研究:</b> 米国運輸省 <a href="http://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236&DB_Short_Name=On-Time">http://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236&amp;DB_Short_Name=On-Time</a>。
  </td>
</tr>

<tr>
  <td valign=top>森林火災データ</td>
  <td valign=top>
ポルトガル北東地域を対象とする、不快指数や風速などの気象データと森林火災の記録が含まれます。<p> </p><b>使用法:</b> 森林火災の被害地域を予測する回帰を実行するという、難しい作業となります。<p> </p><b>関連の研究:</b> Cortez, P.、Morais, A.(2008 年)。UCI Machine Learning Repository <a href="http://archive.ics.uci.edu/ml">http://archive.ics.uci.edu/ml</a>。カリフォルニア州アーバイン: カリフォルニア大学、情報・コンピューター サイエンス学部 <p> </p>[Cortez and Morais、2007 年] P. Cortez、A. Morais.「A Data Mining Approach to Predict Forest Fires using Meteorological Data」J. Neves、M. F.Santos、J. Machado Eds。「New Trends in Artificial Intelligence」、Proceedings of the 13th EPIA 2007 - Portuguese Conference on Artificial Intelligence、December、Guimarães、Portugal、pp. 512-523, 2007。APPIA,、SBN-13 978-989-95618-0-9。<a href="http://www.dsi.uminho.pt/~pcortez/fires.pdf">http://www.dsi.uminho.pt/~pcortez/fires.pdf</a> で入手可能。
  </td>
</tr>

<tr ID=german-credit-card-uci-dataset>
  <td valign=top>ドイツのクレジット カード UCI データセット</td>
  <td valign=top>
german.data ファイルを使用した、UCI Statlog (ドイツのクレジット カード) データセット (<a href="http://archive.ics.uci.edu/ml/datasets/Statlog+(German+Credit+Data)">Statlog+German+Credit+Data</a>)。<p> </p>データセットは、低信用リスクまたは高信用リスクとして属性のセットで表現された人々を分類します。サンプルはそれぞれ人を表します。20 の特徴があり、数値とカテゴリの両方と、二項のラベル (信用リスク値) で構成されます。高信用リスクのエントリにはラベル 2、低信用リスクのエントリにはラベル 1 が付きます。低信用リスクのサンプルを高信用リスクとして誤って分類した場合のコストは 1 ですが、高信用リスクのサンプルを低信用リスクとして誤って分類した場合のコストは 5 です。
  </td>
</tr>

<tr ID=imdb-movie-titles>
  <td valign=top>IMDB 映画のタイトル</td>
  <td valign=top>
Twitter のツイートで評価された映画に関する情報として、IMDB 登録 ID、映画の名称とジャンル、制作年が含まれます。約 17,000 件の映画の情報があります。データセットは、S. Dooms、T. De Pessemier、L. Martens による論文「MovieTweetings: a Movie Rating Dataset Collected From Twitter。Workshop on Crowdsourcing and Human Computation for Recommender Systems, CrowdRec at RecSys 2013」で紹介されました。
  </td>
</tr>

<tr>
  <td valign=top>あやめの 2 クラス データ</td>
  <td valign=top>
パターン認識の文献でおそらく最も多く使用されているデータベースです。3 種類のあやめの品種について 50 例の花弁を測定した、比較的小さなデータセットです。<p> </p><b>使用法:</b> あやめの品種を測定値から予測します。 <p> </p><b>関連の研究:</b> Fisher, R.A.(1988 年)。UCI Machine Learning Repository <a href="http://archive.ics.uci.edu/ml">http://archive.ics.uci.edu/ml</a>。カリフォルニア州アーバイン: カリフォルニア大学、情報・コンピューター サイエンス学部
  </td>
</tr>

<tr ID=movie-tweets>
  <td valign=top>ムービーのツイート</td>
  <td valign=top>
このデータセットは、ムービー ツイート データセットの拡張バージョンです。Twitter で適切に構成されたツイートから抽出した、約 170,000 件の評価が含まれます。各インスタンスは 1 つのツイートを表し、ユーザー ID、IMDB 登録 ID、評価、タイプスタンプ、このツイートのお気に入りの数、そして、このツイートのリツイートの数を組み合わせたものです。データセットは、A. Said、S. Dooms、B. Loni、D. Tikk によって Recommender Systems Challenge 2014 のために提供されました。
  </td>
</tr>

<tr>
  <td valign=top>さまざまな自動車の燃費データ</td>
  <td valign=top>
カーネギーメロン大学 StatLib 研究所が提供したデータセットを多少変更したものです。このデータセットは 1983 年に米国統計学会で使用された実績があります。<p> </p>さまざまな車体の燃料消費量 (ガロンあたりのマイル数) と、シリンダー数、エンジン排気量、馬力、総重量、加速度などの情報が記載されています。<p> </p><b>使用法:</b> 3 つの多値離散属性と 5 つの連続属性に基づいて燃費効率を予測します。<p> </p><b>関連の研究:</b> カーネギーメロン大学 StatLib (1993 年)。UCI Machine Learning Repository <a href="http://archive.ics.uci.edu/ml">http://archive.ics.uci.edu/ml</a>。カリフォルニア州アーバイン: カリフォルニア大学、情報・コンピューター サイエンス学部
  </td>
</tr>

<tr>
  <td valign=top>ピマ族の糖尿病に関する二項分類データセット</td>
  <td valign=top>
国立糖尿病・消化器・腎疾病研究所のデータベースから提供されたデータのサブセットです。データセットはフィルタリングされ、ピマ族の女性患者だけが対象とされています。データには、血糖値やインスリン値などの医療データと、生活習慣要因が含まれています。<p> </p><b>使用法:</b> 対象者が糖尿病にかかっているかどうかを予測します (二項分類)。<p> </p><b>関連の研究:</b> Sigillito, V.(1990 年)。UCI Machine Learning Repository <a href="http://archive.ics.uci.edu/ml">http://archive.ics.uci.edu/ml</a>。カリフォルニア州アーバイン: カリフォルニア大学、情報・コンピューター サイエンス学部
  </td>
</tr>

<tr>
  <td valign=top>レストラン顧客データ</td>
  <td valign=top>
人口統計データや嗜好など、顧客に関するメタデータのセットです。<p> </p><b>使用法:</b> このデータセットを、レストランに関する他の 2 つのデータセットと組み合わせて使用して、レコメンダー システムの調整とテストを実施します。<p> </p><b>関連の研究:</b> Bache, K.、Lichman, M.(2013 年)。UCI Machine Learning Repository <a href="http://archive.ics.uci.edu/ml">http://archive.ics.uci.edu/ml</a>。カリフォルニア州アーバイン: カリフォルニア大学、情報・コンピューター サイエンス学部
  </td>
</tr>

<tr>
  <td valign=top>レストラン特徴データ</td>
  <td valign=top>
料理の種類、食事スタイル、場所など、レストランとその特徴に関するメタデータのセットです。<p> </p><b>使用法:</b> このデータセットを、レストランに関する他の 2 つのデータセットと組み合わせて使用して、レコメンダー システムの調整とテストを実施します。<p> </p><b>関連の研究:</b> Bache, K.、Lichman, M.(2013 年)。UCI Machine Learning Repository <a href="http://archive.ics.uci.edu/ml">http://archive.ics.uci.edu/ml</a>。カリフォルニア州アーバイン: カリフォルニア大学、情報・コンピューター サイエンス学部
  </td>
</tr>

<tr>
  <td valign=top>レストランの評価</td>
  <td valign=top>
ユーザーによるレストランの評価 (0 ～ 2) が含まれます。<p> </p><b>使用法:</b> このデータセットを、レストランに関する他の 2 つのデータセットと組み合わせて使用して、レコメンダー システムの調整とテストを実施します。<p> </p><b>関連の研究:</b> Bache, K.、Lichman, M.(2013 年)。UCI Machine Learning Repository <a href="http://archive.ics.uci.edu/ml">http://archive.ics.uci.edu/ml</a>。カリフォルニア州アーバイン: カリフォルニア大学、情報・コンピューター サイエンス学部
  </td>
</tr>

<tr>
  <td valign=top>鉄鋼のアニール処理に関するマルチクラス データセット</td>
  <td valign=top>
このデータセットには、鉄鋼のアニール処理試験に関する一連の記録が含まれています。幅、厚み、種類 (コイル、シートなど) など、処理後のさまざまな物理的属性も含まれます。<p> </p><b>使用法:</b> 2 つの数値クラス属性 (硬度と強度) のいずれかを予測します。属性間の相関関係を分析することも可能です。<p> </p>鉄鋼のグレードは、SAE その他の機関が定義した標準に準拠します。特定の "グレード"(クラス変数) を検索して、必要な値を把握する必要があります。<p> </p><b>関連の研究:</b> Sterling, D.、Buntine, W. (年度なし).UCI Machine Learning Repository <a href="http://archive.ics.uci.edu/ml">http://archive.ics.uci.edu/ml</a>。カリフォルニア州アーバイン: カリフォルニア大学、情報・コンピューター サイエンス学部 <p> </p>鉄鋼グレードの有用なガイドがこちらにあります。<a href="http://www.outokumpu.com/SiteCollectionDocuments/Outokumpu-steel-grades-properties-global-standards.pdf">http://www.outokumpu.com/SiteCollectionDocuments/Outokumpu-steel-grades-properties-global-standards.pdf</a>
  </td>
</tr>

<tr>
  <td valign=top>望遠鏡データ</td>
  <td valign=top>
高エネルギー ガンマ線バーストとバックグラウンド ノイズをモンテカルロ法でシミュレーションした記録です。<p> </p>シミュレーションの目的は大気地上チェレンコフ ガンマ望遠鏡の精度向上です。期待する信号 (チェレンコフ放射) と上層大気の宇宙線から発生するバックグラウンド ノイズ (ハドロン放射) を、統計的手法によって区別します。<p> </p>データを前処理することによって、長軸がカメラの中心を向く伸長クラスターが作成されています。この楕円のさまざまな特性は通常 Hillas パラメーターと呼ばれ、これらが画像のパラメーターに表れることによって区別が可能になります。<p> </p><b>使用法:</b> 放射の画像が信号とバックグラウンド ノイズのどちらを表しているかを予測します。<p> </p><b>注:</b> バックグラウンド イベントを信号に分類することは、シグナル イベントをバックグラウンドに分類することよりも大きな悪影響を及ぼすため、シンプルな分類の正確度は、このデータでは意味を持ちません。異なる分類器の比較には ROC グラフを使用する必要があります。バックグラウンド イベントを信号として許容する場合の確度は、次のしきい値のいずれを下回る必要があります: 0.01、0.02、0.05、0.1、0.2。<p> </p>また、バックグラウンド イベントの数 (ハドロン放射の場合 h) は過小評価されますが、実際の測定では、h またはノイズ クラスがイベントの大部分を占めています。<p> </p><b>関連の研究:</b> Bock, R.K.(1995 年)。UCI Machine Learning Repository <a href="http://archive.ics.uci.edu/ml">http://archive.ics.uci.edu/ml</a>。カリフォルニア州アーバイン: カリフォルニア大学、情報学部
  </td>
</tr>

<tr ID=weather-dataset>
  <td valign=top>天候データセット</td>
  <td valign=top>
NOAA の 1 時間ごとの陸上の気象観測 (<a href="http://cdo.ncdc.noaa.gov/qclcd_ascii/, merged data from 201304 to 201310">2013 年 4 月から 2013 年 10 月までのデータをマージ</a>)。<p> </p>気象データには、2013 年 4 月から 10 月までの期間の航空気象観測所での観測が含まれます。Azure Machine Learning Studio へのアップロードの前に、データセットを次のように処理します。<ul><li>気象観測所 ID を対応する空港 ID にマッピングします。</li><li>混雑度が上位 70 位の空港に関連付けられていない気象観測所をフィルターで外します。</li><li>Date 列を Year、Month、Day の列に分割します。</li><li>AirportID、Year、Month、Day、Time、TimeZone、SkyCondition、Visibility、WeatherType、DryBulbFarenheit、DryBulbCelsius、WetBulbFarenheit、WetBulbCelsius、DewPointFarenheit、DewPointCelsius、RelativeHumidity、WindSpeed、WindDirection、ValueForWindCharacter、StationPressure、PressureTendency、PressureChange、SeaLevelPressure、RecordType、HourlyPrecip、Altimeter の各列が選択されています。</li></ul>
  </td>
</tr>

<tr ID=wikipedia-sp-500-dataset>
  <td valign=top>Wikipedia SP 500 データセット</td>
  <td valign=top>
データは、S&amp;P 500 企業それぞれの記事に基づいて Wikipedia (<a href="http://www.wikipedia.org/">http://www.wikipedia.org/</a>) から取得され、XML データとして格納されています。<p> </p>Azure Machine Learning Studio へのアップロードの前に、データセットを次のように処理します。<ul><li>特定の企業のテキスト コンテンツを抽出します。</li><li>Wiki の書式設定を削除します。</li><li>英数字以外の文字を削除します。</li><li>すべてのテキストを小文字に変換します。</li><li>既知の会社のカテゴリを追加します。</li></ul><p> </p>いくつかの企業の記事が見つからないため、レコード数は 500 未満であることに注意してください。
  </td>
</tr>





<tr ID=direct-marketing>
  <td valign=top><a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/direct_marketing.csv">direct_marketing.csv</a></td>
  <td valign=top>
顧客データと、ダイレクト メール キャンペーンに対する顧客の反応を示す指標が含まれます。1 行につき 1 件の顧客を表します。データセットには、ユーザーの人口統計と過去の行動に関する 9 個の特徴と、3 つのラベル列 (訪問、コンバージョン、支出) が含まれます。訪問は、顧客がマーケティング キャンペーン後に訪問したことを示すバイナリ列です。コンバージョンは、顧客が何かを購入したことを示します。支出は、使われた金額です。データセットは Kevin Hillstrom によって、MineThatData 電子メール分析とデータ マイニングの課題用に提供されています。
  </td>
</tr>

<tr ID=lyrl2004-tokens-test>
  <td valign=top><a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/lyrl2004_tokens_test.csv">lyrl2004_tokens_test.csv</a></td>
  <td valign=top>
RCV1-V2 Reuters ニュースのデータセットにあるテスト例の特徴。データセットには約 781,000 本のニュース記事が、それぞれの記事の ID (データセットの最初の列) と共に含まれます。各記事はトークン化され、ストップワード処理と語幹処理が行われています。データセットは DavidD. Lewis によって提供されています。
  </td>
</tr>

<tr ID=lyrl2004-tokens-train>
  <td valign=top><a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/lyrl2004_tokens_train.csv">lyrl2004_tokens_train.csv</a></td>
  <td valign=top>
RCV1-V2 Reuters ニュースのデータセットにあるトレーニング例の特徴。データセットには約 23,000 本のニュース記事が、それぞれの記事の ID (データセットの最初の列) と共に含まれます。各記事はトークン化され、ストップワード処理と語幹処理が行われています。データセットは DavidD. Lewis によって提供されています。
  </td>
</tr>

<tr ID=intrusion-detection>
  <td valign=top><a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/network_intrusion_detection.csv">network_intrusion_detection.csv</a><br></td>
  <td valign=top>
KDD Cup 1999 ナレッジ検出とデータ マイニング ツールのコンペティションのデータセット (<a href="http://kdd.ics.uci.edu/databases/kddcup99/kddcup99.html">kddcup99.html</a>)。<p> </p>データセットは、Azure BLOB ストレージにダウンロードされ、格納されています (<a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/network_intrusion_detection.csv">network_intrusion_detection.csv</a>)。データセットには、トレーニングとテストの両方のデータセットが含まれます。トレーニングのデータセットには、ラベルを含む、約 126,000 行と 43 列が含まれます。ラベル情報の一部である 3 列と、数値、文字列、カテゴリによる特徴で構成される 40 列がモデルのトレーニングに使用できます。テスト データには約 22,500 個のテスト例があります (トレーニング データと同じ 43 列が含まれます)。

  </td>
</tr>

<tr ID=rcv1-v2-topics-qrels>
  <td valign=top><a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/rcv1-v2.topics.qrels.csv">rcv1-v2.topics.qrels.csv</a></td>
  <td valign=top>
RCV1-V2 Reuters ニュース データセット内のニュース記事のトピック割り当てです。1 つのニュース記事を複数のトピックに割り当てることができます。各行の形式は "<topic name> <document id> 1" です。データセットには、260 万件のトピック割り当てが含まれます。データセットは DavidD. Lewis によって提供されています。
  </td>
</tr>

<tr ID=student-performance>
  <td valign=top><a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/student_performance.txt">student_performance.txt</a></td>
  <td valign=top>
このデータは、KDD Cup 2010 学生の成績評価に関する課題 (<a href="http://www.kdd.org/kdd-cup-2010-student-performance-evaluation">student performance evaluation</a>) から取得しています。使用されるデータは、Algebra_2008_2009 トレーニング セット (Stamper, J.、Niculescu-Mizil, A.、Ritter, S.、Gordon, G.J.、Koedinger, K.R.(2010)、Algebra I 2008-2009 です。KDD Cup 2010 データ マイニングの学習課題のデータセットは、<a href="http://pslcdatashop.web.cmu.edu/KDDCup/downloads.jsp">downloads.jsp</a> または <a href="http://www.kdd.org/sites/default/files/kddcup/site/2010/files/algebra_2008_2009.zip">algebra_2008_2009.zip</a> にあります。<p> </p>データセットは、Azure BLOB ストレージにダウンロードされ、格納されています (<a href="https://azuremlsampleexperiments.blob.core.windows.net/datasets/student_performance.txt">student_performance.txt</a>)。データセットには、学生のチュータリング システムのログ ファイルが含まれます。提供される特徴には、問題 ID とその簡潔な説明、学生 ID、タイムスタンプ、学生が問題を正しい方法で解くまでにかかった回数があります。元のデータセットには、8,900,000 件のレコードが含まれますが、このデータセットは、最初の 100,000 行にダウンサンプリングされています。このデータセットには、23 個のタブで区切られた、さまざまなタイプ (数値、カテゴリ、およびタイムスタンプ) の列が含まれます。

  </td>
</tr>




</table>


<!-- Module References -->
[import-data]: https://msdn.microsoft.com/library/azure/4e1b0fe6-aded-4b3f-a36f-39b8862b9004/

<!---HONumber=AcomDC_0921_2016-->