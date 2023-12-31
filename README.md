# 概要
対症的放射線治療を予後を予測するためのツールです。<br>
<a href="https://huggingface.co/spaces/s-okazaki/prognosis_palliativeRT">HuggingFace</a>に登録していますので、そちらから使用してください。<br>
# 背景
<ul>
  <li>放射線治療は根治目的だけでなく、がんによる疼痛や神経障害、出血、醜形などの症状を緩和する目的でも行われる。</li>
  <li>実臨床において進行がん患者の約40%が対症的放射線治療を受けていると言われている。</li>
  <li>対症的放射線治療を行う際には患者の予後を見据えて照射範囲や投与線量、治療期間などを考慮する必要がある。</li>
  <li>対象とする患者や病状が幅広く、経験豊富な臨床医であっても予後を見積もることが難しい。</li>
  <li>簡便かつ有用な予後予測ツールを開発し使用することで、より適切な放射線治療方針の決定に繋がる。</li>
</ul>

# 学習データ
<ul>
  <li>2016年1月～2023年7月に対症的放射線治療を行った進行がん患者516症例 (単施設症例)</li>
  <li>治療開始時期の患者因子 (性別、年齢、身長・体重、performance status、チャールソン併存疾患指数) や腫瘍学的因子 (疾患名、病理診断名、照射部位、原発巣制御、肺・脳・肝・骨・リンパ節転移、脊髄横断症状、ステロイド併用の有無、オピオイド投与量)、血液検査データ、生存期間を遡及的に調査。</li>
  <li>【年齢】中央値：69歳 (範囲：31～93歳)</li>
  <li>【性別】男性：55.4%、女性：44.6%</li>
  <li>【Performance Status】0:3.7%、1:32.6%、2:34.9%、3:25.6%、4:3.3%</li>
  <li>【疾患の内訳】肺がん:36.0%、乳がん:11.6%、結腸直腸がん:8.7%、胃がん:6.6%、前立腺がん:5.6%、非ホジキンリンパ腫:4.7%、食道がん:3.5%、腎がん:3.5%、子宮頸がん:3.1%</li>
  <li>【病理組織の内訳】adenocarcinoma:34.9%、squamous cell carcinoma:13.0%、invasive ductal carcinoma:9.9%、病理診断なし:6.8%</li>
  <li>【照射部位の内訳】骨:52.9%、脳:17.4%、リンパ節:10.9%、肺:4.1%、軟部組織:3.5%、胃:2.9%</li>
  <li>【血液検査データ】測定頻度の高い項目について、治療6か月前～治療後1週間の範囲で1週間毎の平均値を算出した。欠損データは前後に値があれば線形補間、前後どちらかに値があれば前または後の値で置換、値が全くなければ極小値(10<sup>-12</sup>)で置換した。</li>
  <li>【欠損データ】身長：2症例、体重：3症例→データ中央値で置換した。
</ul>

# 学習方法
<ul>
  <li>ホールド・アウト法を採用した。学習データを7:3の割合で訓練データとテストデータに分割し、さらに訓練データを7:3の割合で訓練データと検証データに分割した。</li>
  <li>Keras APIを用いてTensorflowで深層ニューラルネットワークモデルの構築を行った。</li>
  <li>最初に、各特徴量を数値データ、カテゴリデータ、および血液検査データの3つの異なるカテゴリに分割した。それぞれのデータカテゴリに対して個別のニューラルネットワークを構築した。数値データと血液検査データに対しては、データの標準化を最初に実行した。それぞれのニューラルネットワークでは、バッチ正規化を適用してから全結合層に接続した。活性化関数としてReLU関数を採用し、全結合層の後にDropOut層を挿入した。血液検査データのニューラルネットワークには、リカレントニューラルネットワークの1つであるGRUを組み込んだ。そして、各ニューラルネットワークを結合し、6つの全結合層を経て最終的な出力を生成するモデルを構築した。</li>
  <img src="https://raw.githubusercontent.com/okazaki-7296/rt_prognosis/main/img/nn_model.png" widht="300px" alt="ニューラルネットワークの概略図">
  <li>損失関数として平均二乗対数誤差、最適化アルゴリズムとしてRMSPropを採用した。ミニバッチ法(バッチサイズ：10)を用いて学習を実行した。最大エポック数を500と設定し、アーリーストッピング法を用いて損失関数の最小化を得られた時点で学習終了とした。</li>
  <li>モデル学習後にSHAPライブラリを用いて各特徴量のShapley値を算出し、平均Shapley値の高い10項目を抽出して、再度ニューラルネットワークモデルの構築を行った。学習終了後のモデルデータを本ツールに組み込んでいる。</li>
</ul>

# 結果
## すべての特徴量を使用したモデル
<ul>
  <li>エポック数：108回で学習終了した。最終的な損失関数の値は訓練データで1.1366、検証データで1.9761と、わずかに過学習の傾向があった。</li>
  <img src="https://raw.githubusercontent.com/okazaki-7296/rt_prognosis/main/img/loss_normalmodel.png">
  <li>予測値と実測値の差が大きい症例が少なからずいたが、約4割の症例では±50日の精度で予後を予測した。(散布図では青色：訓練データ、橙色：テストデータを示している)</li>
  <img src="https://raw.githubusercontent.com/okazaki-7296/rt_prognosis/main/img/scatter_normalmodel.png">
  <img src="https://raw.githubusercontent.com/okazaki-7296/rt_prognosis/main/img/histogram_normalmodel.png">
  <li>高Shapley値を得たのはPerformance Statusや非ホジキンリンパ腫であること、リンパ節転移の有無、性別などであった。</li>
  <img src="https://raw.githubusercontent.com/okazaki-7296/rt_prognosis/main/img/shap_result.png">
</ul>

## 特徴量を絞った軽量モデル (本ツールで使用しているモデル)
<ul>
  <li>エポック数：173回で学習終了した。最終的な損失関数の値は訓練データで1.4153、検証データで1.4478と、両者に差は見られなかった。</li>
  <img src="https://raw.githubusercontent.com/okazaki-7296/rt_prognosis/main/img/loss_lightmodel.png">
  <li>やや過小評価して予測値を出力する傾向があるが概ね予測精度は保たれていると考えられた。(散布図では青色：訓練データ、橙色：テストデータを示している)</li>
  <img src="https://raw.githubusercontent.com/okazaki-7296/rt_prognosis/main/img/scatter_lightmodel.png">
  <img src="https://raw.githubusercontent.com/okazaki-7296/rt_prognosis/main/img/histogram_lightmodel.png">
</ul>
