# R-Graphics

[正誤表]  
[英語](https://www.oreilly.com/library/view/r-graphics-cookbook/9781491978597/)  
[日本語](https://www.oreilly.co.jp/books/9784873118925/)  

## 付録A ggplot2を理解する  

ggplot概要  

グラフ化とは各データを視覚属性（軸、色等）にマッピングすること  
入力として縦持ち形式（各行が観測値、各列が変数）のデータフレームを想定  
データをマッピングする前に何らかの要約が必要な場合には統計関数（stat_）を用いる    
グラフの全体的な体裁をテーマとして統一的にコントロールすることができる    

**用語説明**  
データ: 変数の集合  
幾何オブジェクト: グラフのタイプ（どのように表現するか）  
エステティック属性: 視覚属性  
スケール: データ空間からエステティック空間への投影の仕方を定義するもの  
ガイド: 目盛り、ラベル、凡例等  
⇒ 幾何オブジェクトを決め、データをエステティック属性にマッピングし、スケールとガイドを定義する  

---
　  
　  
## 第１章　Rの基本  

割愛  

### R Tips
gcookbookパッケージには多数のサンプルデータが含まれる  

### Other Functions
`update.packages()`: パッケージの更新、ask=FALSEで確認なしで更新  
`summary(df)`: 単変量の統計量（全ての列）  
`pairs(df)`: 共変量の統計量（各列の全ての組み合わせ）  

---
　  

## 第２章　データの基本的なプロット  

主なグラフ（幾何学オブジェクト）  

~~~
ggplot(df, aes(col1, col2)) +
    geom_point()   # 散布図
    geom_line()    # 折れ線グラフ
    geom_col()     # 棒グラフ（値を表示）
    geom_boxplot() # 箱ひげ図
  
ggplot(df, aes(col)) +
    geom_bar()                      # 棒グラフ（個数を集計）
    geom_histogram()                # ヒストグラム
    stat_function(fun, geom="line") # 関数曲線
~~~

### Other Functions  
`interaction(col1, col2)`: 二つのファクターを合成し、新しいファクターを合成  

---
　  

## 第３章　棒グラフ    

ある区分に対応する値を示す  
表示する値が要約値か生値かを明確に区別する必要がある  

1. 値を表示  
geom_colを使用  
~~~
# 基本
ggplot(df, aes(col1, col2)) +
    geom_col(fill="lightblue", colour="black") + # fill: 塗りつぶしの色、colour: 枠線の色
    geom_text(aes(label=col2), vjust=1.5) +      # 値ラベル、vjustは下側にシフト
    xlab("x")                                    # x軸のタイトル

# グループ別の表示①（積み上げ、デフォルト）
ggplot(df, aes(col1, col2, fill=col3)) +              # col3の値でfillを変える
    geom_col(position=position_stack(reverse=TRUE)) + # 積み上げを逆順にする
    guides(fill=guide_legend(reverse=TRUE))           # 凡例の表示を逆順にする

# グループ別の表示②（100%積み上げ）
ggplot(df, aes(col1, col2, fill=col3)) +              # col3の値でfillを変える
    geom_col(position="fill") +                       # position="fill"は100%積み上げ
    geom_text(aes(y=label_y, label=col2)) +           # 積み上げ棒グラフに値ラベル入れる場合は位置の列が必要
    scale_y_continuous(labels=scales::percent)        # スケールを100%表示

# グループ別の表示③（並列）
ggplot(df, aes(col1, col2, fill=col3)) +              # col3の値でfillを変える
    geom_col(position="dodge", colour="black") +      # position="dodge"は棒グラフの重なりを無くす
    scale_fill_brewer(palette="Pastel1")              # fillの既成パターンを設定
    scale_fill_manual(values=c("#669933", "#FFCC66")) # fillをマニュアル指定    
~~~

<img src="https://user-images.githubusercontent.com/51372161/155881597-d2b8aa7f-00e4-4966-bbd7-6e36dde15a96.png">  


2. 要約値の表示  
geom_barを使用  
~~~
# 指定の列の値の個数を表示 ⇒ 列の値が離散値: 棒グラフ、連続値: ヒストグラム
ggplot(df, aes(col1, fill=col2)) +
    # widthで棒の幅（棒の間隔）を指定（デフォルト0.9、最大1.0）
    # position_dodgeでグループ間の棒の間隔（互いの中心からの距離）を指定（デフォルト0.9）
    geom_bar(width=0.5, position=position_dodge(0.7))
    geom_text(aes(label=..count.., stat="count"), position=position_dodge(0.7), size=3)  # 要約値を値ラベルで表示、サイズのデフォルトは5
~~~

<img src="https://user-images.githubusercontent.com/51372161/156161725-9a6b8d6c-b118-4263-b951-33f7332577bd.png">  

3. 表示の工夫  
値の正負でグラフを色分け ⇒ col3に正負を表す論理値等を設定  
~~~
ggplot(df, aes(col1, col2)) +
    geom_col(aes(fill=col3), colour="black", size=0.25) +
    scale_fill_manual(values=c("#CCEEFF", "#FFDDDD"), guide=F) # guide=Fで凡例を非表示
~~~

<img src="https://user-images.githubusercontent.com/51372161/156162343-98511e10-d9ae-47ab-94ea-1f131f32dfe4.png">  


4. ドットプロット  
~~~
ggplot(tophit, aes(avg, name)) +
  geom_point(size=3, aes(colour=lg)) + 
  geom_segment(aes(yend=name), xend=0, colour="grey50") + # (avg, name)から(0, name)に線分を引く
  theme_bw() + # グラフ全体のテーマを設定
  theme(panel.grid.major.y=element_blank()) + # 罫線はthemeで扱う
  facet_grid(lg~., scales="free_y", space="free_y") # spaceを指定しない場合、各図の大きさは同じに設定される
~~~

<img src="https://user-images.githubusercontent.com/51372161/156549905-db73ddbc-7117-4d98-b2b5-2204bae2e752.png">  


### R Tips  
RColorBrewer::display.brewer.all(): R Color Brewerの全パレット表示  
ファクターのレベル順序の作り方  
~~~
lorder <- col1[order(col2, col3)]    # 好みの順序に並べ替え
col1 <- factor(col1, levels=lorder)  # 並べ替えたベクトルをレベルとしてファクター化
~~~

### Other Functions
`slice(df, 1:10)`: 指定の行を抽出  
`rev(col)`: 逆順にする  
`top_n(n, col)`: 上位n個のデータを抽出  

---
　  

## 第４章　折れ線グラフ    

連続変数もしくはカテゴリ変数に対して、連続値がどのように推移するかを表示  

1. 折れ線グラフ  
geom_lineを使用  
~~~
# 基本
ggplot(df, aes(col1, col2, group=1)) + # x軸がファクターの場合は全てのデータが同じグループであることを明示するために「group=1」を指定する  
    geom_line(colour="blue") + # 共通の色を設定
    geom_point() +             # 点を追加する場合
    ylim(0, max(df$col))       # y軸の範囲を指定
    expand_limits(y=0)         # y軸に含める値を指定
    scale_y_log10()            # 対数グラフの場合

# グループ別の表示
ggplot(df, aes(col1, col2, linetype=col3, group=col3)) + #x軸がファクターの場合はgroupを明示的に指定する
    geom_line(position=position_dodge(0.2)) + # 点の重複を避けたい場合はposition_dodgeでずらす
    geom_point(size=4, shape=21, fill="pink") # サイズのデフォルトは2
~~~

<img src="https://user-images.githubusercontent.com/51372161/156872927-e593a801-397a-4d47-9d8e-864b056dfc1f.png">  

2. 面グラフ  
geom_areaを使用  
~~~
# 基本（網掛け領域付のグラフ）
ggplot(df, aes(col1, col2)) +
  geom_area(fill="blue", alpha=.2) + # 領域の色と透明度を指定
  geom_line()                        # 枠線

# グループ別の表示①（積み上げ、デフォルト）
ggplot(df, aes(col1, col2, fill=col3)) +
  geom_area(colour=NA) +                              # 枠線が面の全てを囲まないようにするためNA
  scale_fill_brewer(palette="Blues") +
  geom_line(position="stack", colour="grey", size=.2) # 枠線

# グループ別の表示②（100%積み上げ）
ggplot(df, aes(col1, col2, fill=col3)) +
  geom_area(colour=NA, position="fill") +             # 枠線が面の全てを囲まないようにするためNA
  scale_fill_brewer(palette="Pastel1") +
  geom_line(position="fill", colour="gray", size=.2)  # 枠線
  scale_y_continuous(labels=scales::percent)          # y軸をパーセント表示
~~~

<img src="https://user-images.githubusercontent.com/51372161/156921425-062ac4ce-1a0a-41d4-bb7e-102228786351.png">  

3. 信頼区間の表示  
geom_ribbonを使用  
~~~
# 指定領域を網掛する
ggplot(df, aes(col1, col2)) +
  geom_ribbon(aes(ymin=col2 - col3, ymax=col2 + col3), alpha=.2) +
  geom_line()
~~~

<img src="https://user-images.githubusercontent.com/51372161/156921881-6f5c5df0-0b98-4ccb-aee0-c244c6940308.png">  


* 面グラフ
~~~
# 点同様の設定方法
ggplot(sunspotyear, aes(Year, Sunspots)) +
  geom_area(fill="blue", alpha=0.2, colour="black")

# デフォルトは棒グラフ同様積み上げ
ggplot(uspopage, aes(Year, Thousands, fill=Ageroup)) +
  geom_area(alpha=0.4, colour="black", size=0.2) +
  scale_fill_brewer(palette="Blues")

# 100%積み上げはposition="fill"（棒グラフ同様）
ggplot(uspopage, aes(Year, Thousands, fill=AgeGroup)) +
  geom_area(position="fill", alpha=0.4, colour="black", size=0.2) +
  scale_fill_brewer(palette="Blues") +
  scale_y_continuous(labels=scales::percent)
~~~

* 信頼区間の表示  
~~~
# geom_ribbonを用いる
ggplot(climate_mod, aes(Year, Anomaly10y)) +
  geom_line() +
  geom_ribbon(aes(ymin=Anomaly10y-Unc10y,
                  ymax=Anomaly10y+Unc10y,
                  alpha=0.2))
~~~

　  
* Tips  
as.integer(as.character(factor)): ファクター変数を表示の値で数値に変換する場合は、一旦文字列に変換する必要がある  
　⇒ 直接数値に変換するとファクターのレベルに割り当てられた数値に変換されるため  
group: 系列をグルーピング、指定が無ければエステティック属性で使用した列が自動的にグルーピングに使用される  


　  
## 第５章　散布図    

* 基本  
~~~
ggplot(heightweight, aes(ageYear, heightIn)) +
  geom_point(aes(shape=sex, colour=sex)) +
  scale_shape_manual(values=c(1, 2)) +
  scale_colour_brewer(palette="Set1")
~~~

* 形と塗りつぶしで組み合わせを表現
~~~
# 凡例の形状していの意図は不明？
ggplot(hw, aes(ageYear, heightIn, shape=sex, fill=weightgroup)) +
  geom_point(size=2.5) +
  scale_shape_manual(values=c(21, 24)) +
  scale_fill_manual(values=c(NA, "black"),
                    guide=guide_legend(override.aes=list(shape=21)))
~~~

* 連続変数  
~~~
# 色
ggplot(heightweight, aes(ageYear, heightIn, colour=weightLb)) +
  geom_point()
# 大きさ（点の半径）  
ggplot(heightweight, aes(ageYear, heightIn, size=weightLb)) +
  geom_point()
# 大きさ（点の面積）
ggplot(heightweight, aes(ageYear, heightIn, size=weightLb)) +
  geom_point() +
  scale_size_area()
# グラデーションの指定、凡例の表示
ggplot(heightweight, aes(ageYear, heightIn, fill=weightLb)) +
  geom_point(shape=21, size=2.5) +
  scale_fill_gradient(low="white", high="black",
                      breaks=seq(70, 170, by=20),
                      guide=guide_legend())
# 組み合わせ
ggplot(heightweight, aes(ageYear, heightIn, colour=sex, size=weightLb)) +
  geom_point(alpha=0.5) +
  scale_colour_brewer(palette="Set1") +
  scale_size_area()
~~~

* オーバープロット  
多くの点が重なるケース  
~~~
# 透明度を上げる
diamonds_sp +
  geom_point(alpha=0.01)

# 色で分布を表現する、グラデーションのメリハリを付ける
diamonds_sp +
  stat_bin2d(bins=50) +
  scale_fill_gradient(low="lightblue", high="red", limits=c(0, 6000))

# ジッター
cw_sp +
  geom_point(position=position_jitter(width=.5, height=0))

# 箱ひげ図
cw_sp +
  geom_boxplot(aes(group=Time))
~~~

* 回帰線  

~~~
# 信頼区間の設定
hw_sp +
  geom_point() +
  stat_smooth(method=lm, level=0.99)

# 色の調整、信頼区間の無効化
hw_sp +
  geom_point(colour="grey60") +
  stat_smooth(method=lm, se=F, colour="black")

# デフォルトの局所加重多項式でパラメータを設定
hw_sp +
  geom_point() +
  stat_smooth(method=loess, method.args=list(degree=1))

# ロジスティック回帰でフィッティング
ggplot(biopsy_mod, aes(V1, classn)) +
  geom_point(
    position=position_jitter(width=0.3, height=0.06),
    size=1.5,
    alpha=0.4,
    shape=21
  ) +
  stat_smooth(method=glm, method.args=list(family=binomial))

# グループ化している場合はグループ毎に曲線が引かれる
# 外挿して曲線を両端まで表示
ggplot(heightweight, aes(ageYear, heightIn, colour=sex)) +
  geom_smooth(method=lm, fullrange=T)

# 曲線のデータを作成して追加
# 変数名を揃えておく必要がある
hw_sp +
  geom_line(data=lm_predicted, colour="red", size=0.8) +
  geom_line(data=loess_predicted, colour="blue", size=0.8)
~~~

* アノテーション  
~~~
# Rの数式表現を使用する場合
hw_sp +
  annotate("text", x=16.5, y=52, label="r^2==0.42", parse=T)
~~~

* ラグ（一次元の分布を軸上に表示）  
~~~
ggplot(faithful, aes(eruptions, waiting)) +
  geom_point() +
  geom_rug(position="jitter", size=0.2)
~~~

* ラベル  
~~~
# justは点を中心にした位置調整、(0.5, 0.5)が点の中心で0が左（下）揃え、1が右（上）揃え
# aesでプロット位置をオーバーライド、更にposition_nudgeで調整
countries_sp +
  geom_text(aes(x=healthexp + 100, label=Name), size=3, hjust=0,
            position=position_nudge(x=100, y=0))

# ラベルの重複を無くす（ggrepel）
countries_sp +
  geom_text_repel(aes(label=Name), size=3)

# ラベルに囲みを追加（ggrepel）
countries_sp +
  geom_label_repel(aes(label=Name), size=3)
~~~

* バルーンプロット  
~~~
# scale_size_areaで値を面積に比例させる
# 半径の大きさに応じてラベルの位置を調整
ggplot(hec, aes(Eye, Hair)) +
  geom_point(aes(size=count), shape=21, colour="black", fill="cornsilk") +
  scale_size_area(max_size=20, guide=F) +
  geom_text(aes(label=count, y=as.numeric(as.factor(Hair))-sqrt(count)/34),
            colour="grey60",
            vjust=1.3,
            size=4)
~~~

* 散布図行列  
~~~
# pairsをそのまま用いるのが楽
c2009 <- countries %>%
  filter(Year==2009) %>%
  select(Name, GDP, laborrate, healthexp, infmortality) %>%
  pairs()
~~~

　  
* Tips  
グループ化は文字列かファクターで行い、数値の場合はファクターに変更してから使用する  
shape: 0～20はcolourだけ指定、21～25はfillも指定  
range: 最大値と最小値  
recode(col, x1=y1, x2=y2): col列のxをyに置き換える  
ungroup: group_byの解除  
特定のデータグループに関数を適用する方法  
~~~
# 入れ子データの列を作成し、当該要素にmapで関数を適用する
heightweight %>%
  group_by(sex) %>%
  nest() %>%
  mutate(model=map(data, ~lm(heightIn~ageYear, data=.))) %>%
  ungroup()

#doでグループ化したデータフレーム毎に関数を適用する
heightweight %>%
  group_by(sex) %>%
  do(model=lm(heightIn~ageYear, .)) %>%
  ungroup()
~~~


　  
## 第６章　データ分布の要約  

* ヒストグラム  
~~~
# binwidthはビンの幅、binsはビンの数、boundaryは集計の起点
# ビンは下限値には閉じていて、上限値には開いている  
ggplot(faithful, aes(waiting)) +
  geom_histogram(binwidth=8, fill="white", colour="black", boundary=35)
 
 # データフレームではなくベクトルから作成
 ggplot(NULL, aes(vec)) +
  geom_histogram()
~~~

* 複数のヒストグラム  
~~~
# facetを用いる
ggplot(birthwt_mod, aes(bwt)) +
  geom_histogram(fill="white", colour="black") +
  facet_grid(smoke~.) #上下に並べる（行指定）
  #facet_grid(~smoke) #左右に並べる（列指定）

# グルーピングを用いる、identityを指定しないと積み上げグラフになる
ggplot(birthwt_mod, aes(bwt, fill=smoke)) +
  geom_histogram(alpha=0.4, position="identity") 
~~~

* 密度曲線
推定量である点に注意  

~~~
# geom_density
ggplot(faithful, aes(waiting)) +
  geom_density(colour="red") +
  geom_density(adjust=.25, colour="red") +
  geom_density(adjust=2, colour="blue")

# geom_line
ggplot(faithful, aes(waiting)) +
  geom_line(stat="density") +
  geom_line(stat="density", adjust=.25, colour="red") +
  geom_line(stat="density", adjust=2, colour="blue")

# 色掛けはgeom_densityのみで可能
ggplot(faithful, aes(waiting)) +
  geom_density(alpha=.2, fill="blue") +
  xlim(35, 105)
  #expand_limits(x=c(35, 105))

# 実際の分布と重ねる
# 実際の分布はy=..density..で百分比に揃える必要がある
ggplot(faithful, aes(waiting, y=..density..)) +
  geom_histogram(fill="cornsilk", colour="grey60", size=.2) +
  geom_density() +
  xlim(35, 105)

# 複数の密度曲線  
# facetで複数のグラフに分けるのが見やすく最善
ggplot(birthwt_mod, aes(bwt, ..density..)) +
  geom_histogram(binwidth=200, fill="cornsilk", colour="grey60", size=.2) +
  geom_density() +
  facet_grid(smoke ~ .)
~~~

* 頻度の折れ線グラフ  
~~~
ggplot(faithful, aes(waiting)) +
  geom_freqpoly(binwidth=15)
~~~

* 箱ひげ図  
~~~
# 箱の横幅、外れ値の形と大きさを指定
ggplot(birthwt, aes(factor(race), bwt)) +
  geom_boxplot(width=.5, outlier.size=1.5, outlier.shape=21)

# 一変数のみのケース、xに1を指定
# x軸の補助線とタイトルを消す
ggplot(birthwt, aes(1, bwt)) +
  geom_boxplot() +
  scale_x_continuous(breaks=NULL) +
  theme(axis.title.x=element_blank())

# ノッチ、中央値の信頼区間を表す
ggplot(birthwt, aes(factor(race), bwt)) +
  geom_boxplot(notch=T)

# 平均値、stat_summaryで要約量を算出して表示
ggplot(birthwt, aes(factor(race), bwt)) +
  geom_boxplot() +
  stat_summary(fun.y="mean", geom="point", shape=23, size=3, fill="white")
~~~

* バイオリンプロット  
~~~
# プロット内に箱ひげ図を表示、中央値を別途表示
# バイオリンプロットの範囲を推定値の端点まで表示（trim=F、デフォルトは最大値と最小値の範囲を表示）
hw_p +
  geom_violin(trim=F) +
  geom_boxplot(width=.1, fill="black", outlier.colour=NA) +
  stat_summary(fun.y=median, geom="point", fill="white", shape=21, size=2.5)

# 分布の面積を観測数に比例させる（デフォルトは全ての面積を等しくする）
hw_p +
  geom_violin(scale="count")

# 分布の平滑度合を設定
hw_p +
  geom_violin(adjust=2)
~~~

* ドットプロット
~~~
# デフォルト
# 下端基準、ビンはデフォルトで設定（dot-density）
c2009_p +
  geom_dotplot(binwidth=.25) +
  geom_rug() +
  scale_y_continuous(breaks=NULL) +
  theme(axis.title.y=element_blank())

# ビンを等間隔に設定
c2009_p +
  geom_dotplot(method="histodot", binwidth=.25) +
  geom_rug() +
  scale_y_continuous(breaks=NULL) +
  theme(axis.title.y=element_blank())

# 中央基準
c2009_p +
  geom_dotplot(binwidth=.25, stackdir="center") +
  geom_rug() +
  scale_y_continuous(breaks=NULL) +
  theme(axis.title.y=element_blank())

# ドットの縦位置を揃える
c2009_p +
  geom_dotplot(binwidth=.25, stackdir="centerwhole") +
  geom_rug() +
  scale_y_continuous(breaks=NULL) +
  theme(axis.title.y=element_blank())

# 箱ひげ図と重ねて表示
ggplot(heightweight, aes(sex, heightIn)) +
  geom_boxplot(outlier.colour=NA, width=.4) +
  geom_dotplot(binaxis="y", binwidth=.5, stackdir="center", fill=NA)

# 箱ひげ図と並べて表示
# 横軸の位置をずらす、横軸で使用する列を変換して使用すると改めてグルーピングを指定する必要がある
ggplot(heightweight, aes(sex, heightIn)) +
  geom_boxplot(aes(as.numeric(sex) + .2, group=sex), width=.25) +
  geom_dotplot(aes(as.numeric(sex) - .2, group=sex),
    binaxis="y", binwidth=.5, stackdir="center") +
  scale_x_continuous(
    breaks=1:nlevels(heightweight$sex),
    labels=levels(heightweight$sex))
~~~

* 密度プロット  
~~~
# 等高線
faithful_p +
  geom_point() +
  stat_density2d()

# 色付き
faithful_p +
  stat_density2d(aes(colour=..level..))

# 塗りつぶし、色に反映させる
faithful_p +
  stat_density2d(aes(fill=..density..), geom="raster", contour=F)

# 塗りつぶし、透過率に反映させる
faithful_p +
  geom_point() +
  stat_density2d(aes(alpha=..density..), geom="tile", contour=F)

#カーネル推定の範囲を調節、x軸とy軸の両方のパラメータを指定
faithful_p +
  stat_density2d(aes(fill=..density..), 
                 geom="raster", 
                 contour=F,
                 h=c(.5, 5))
~~~

* Tips  
levels: カテゴリ変数のレベルを表示  
recode_factor: カテゴリ変数の変換  


## 第７章　注釈  

* 注釈  
~~~
# 書式等を指定
p +
  annotate("text", x=3, y=48, label="Group 1",
           family="serif", fontface="italic", colour="darkred", size=3) +
  annotate("text", x=4.5, y=66, label="Group 2",
           family="serif", fontface="italic", colour="darkred", size=3)

# geom_textはデータの数だけ指定のテキストを表示するので注意
p +
  annotate("text", x=3, y=48, label="Group 1", alpha=.1) +
  geom_text(x=4.5, y=66, label="Group 2", alpha=.1)

# テキストは指定の点を中心に設定されるので、hjust、vjustで微調整する
p +
  annotate("text", x=-Inf, y=Inf, label="Upper left", 
           hjust=-.2, vjust=2) +
  annotate("text", x=mean(range(faithful$eruptions)), y=-Inf, label="Bottom middle",
           vjust=-0.4)

# 数式
p + annotate("text", x=2, y=0.3, parse=T,
             label="'Function: '*frac(1, sqrt(2*pi))*e~{-x^2/x}")
~~~


* 補助線    
~~~
# 軸に平行な線
hw_plot +
  geom_hline(yintercept=60) +
  geom_vline(xintercept=14)

# 傾きのある線
hw_plot +
  geom_abline(intercept=37.4, slope=1.75)

# 複数の線  
hw_plot +
  geom_hline(
    data=hw_means,
    aes(yintercept=heightIn, colour=sex),
    linetype="dashed",
    size=1
  )

# ファクター値を切片とする場合
pg_plot +
  geom_vline(xintercept=which(levels(PlantGrowth$group)=="ctrl"))
~~~

* その他  
~~~
# 線分、endに向かって矢印が出る
p +
  annotate("segment", x=1950, xend=1980, y=-.25, yend=-.25,
           arrow=arrow(ends="both", angle=90, length=unit(.2,"cm"))) +
  annotate("segment", x=1850, xend=1820, y=-.8, yend=-.95,
           colour="blue", size=2, arrow=arrow())

# 長方形の網掛
p +
  annotate("rect", xmin=1950, xmax=1980, ymin=-1, ymax=1,
           alpha=.1, fill="blue")

# 要素の強調１、列を準備
ggplot(pg_mod, aes(group, y=weight, fill=hl)) +
  geom_boxplot() +
  scale_fill_manual(values=c("grey85", "#FFDDCC"), guide=F)

# 要素の強調２、要素ごとに設定
ggplot(PlantGrowth, aes(group, y=weight, fill=group)) +
  geom_boxplot() +
  scale_fill_manual(values=c("grey85", "grey85", "#FFDDCC"), guide=F)
  
# 折れ線グラフのエラーバー
ggplot(ce_mod, aes(Date, Weight)) +
  geom_line(aes(group=1)) +
  geom_point(size=4) +
  geom_errorbar(aes(ymin=Weight-se, ymax=Weight+se), width=.2)

# 棒グラフのエラーバー
# グループ表示する場合は棒グラフと同じ幅だけドッジする必要がある
ggplot(cabbage_exp, aes(Date, Weight, fill=Cultivar)) +
  geom_col(position="dodge") +
  geom_errorbar(aes(ymin=Weight-se, ymax=Weight+se),
                width=.2, position=position_dodge(0.9))
~~~

* ファセット    
~~~
# ファセットに対応した表示データを用意
f_labels <- data.frame(drv=c("4", "f", "r"), label=c("4wd", "Front", "Rear"))
ggplot(mpg, aes(displ, hwy)) +
  geom_point() +
  facet_grid(.~drv) +
  geom_text(x=6, y=40, aes(label=label), data=f_labels)

#annotateはデータ、対応関係を問わずに同じ値を表示
facet_plot +
  annotate("text", x=6, y=42, label="label text")
~~~


* Tips  
?plotmath: 数式表現のヘルプ  
library(grid): 矢印を記載する場合に必要  


## 第８章　軸  

* 基本  
~~~
# 軸の入替、カテゴリ変数の表示順序を逆にする
ggplot(PlantGrowth, aes(group, weight)) +
  geom_boxplot() +
  coord_flip() +
  scale_x_discrete(limits=rev(levels(PlantGrowth$group)))
~~~


* 範囲の変更と目盛りの非表示  
~~~
# 複数の変更を行う場合はscaleで同時に設定する
pg_plot +
  scale_y_continuous(limits=c(0, 10), breaks=NULL)

# 悪い例１、最後の設定だけが有効となり範囲の変更は無視される
pg_plot +
  ylim(0, 10) +
  scale_y_continuous(breaks=NULL)
  
# 悪い例１、最後の設定だけが有効となり目盛りの非表示は無視される
hhpg_plot +
  scale_y_continuous(breaks=NULL) +
  ylim(0, 10)

# 範囲の拡大
pg_plot +
  expand_limits(y=0)

#座標変換（ズームイン、ズームアウト）
pg_plot +
  coord_cartesian(ylim=c(5, 6.5))
~~~

* 軸の順序  
~~~
# 軸の順序を反転
し、範囲も変更
ggplot(PlantGrowth, aes(group, weight)) +
  geom_boxplot() +
  scale_y_reverse(limits=c(8, 0))

# ylimでも変更が可能
ggplot(PlantGrowth, aes(group, weight)) +
  geom_boxplot() +
  ylim(6.5, 3.5)

# カテゴリカル変数の順序指定
pg_plot +
  scale_x_discrete(limits=c("trt1", "ctrl", "trt2"))

# カテゴリカル変数の順序反転
pg_plot +
  scale_x_discrete(limits=rev(levels(PlantGrowth$group)))
~~~

* レイアウト  
~~~
# 縦横比を揃えて、同じ幅の区切りを設定
m_plot +
  #coord_fixed() +
  scale_y_continuous(breaks=seq(0, 420, 30)) +
  scale_x_continuous(breaks=seq(0, 420, 30))

# 縦横比を1:2にして、同じ幅の区切りを設定
m_plot +
  coord_fixed(ratio=1/2) +
  scale_y_continuous(breaks=seq(0, 420, 30)) +
  scale_x_continuous(breaks=seq(0, 420, 30))
~~~

* 目盛り  
~~~
# 主目盛りを指定（補助目盛は主目盛りの半分になる）
ggplot(PlantGrowth, aes(group, weight)) +
  geom_boxplot() +
  scale_y_continuous(breaks=c(4, 4.25, 4.5, 5, 6, 8))

# 目盛り線の消去、目盛り記号の消去、目盛りラベルの消去
pg_plot +
  scale_y_continuous(breaks=NULL) +
  theme(axis.ticks=element_blank(), axis.text.y=element_blank())

# ラベルの変更１
hw_plot +
  scale_y_continuous(breaks=c(50, 56, 60, 66, 72),
                     labels=c("Tiny", "Really\nshort", "Short", "Medium", "Tallish"))

# ラベルの変更２，独自変換関数を使用
hw_plot +
  scale_y_continuous(breaks=seq(48, 72, 4), labels=footinch_formatter)

# ラベルの変更３，既存変換関数を使用（scalesパッケージ）
# comma: 桁区切り、dollar: ドル表示、percent: パーセント表示
hw_plot +
  scale_y_continuous(breaks=seq(48, 72, 4), labels=comma)

# ラベルの角度、justは文字揃え（0, 0.5, 1で指定）
pg_plot +
  theme(axis.text.x=element_text(angle=30, hjust=1, vjust=1))

# ラベルの詳細設定
pg_plot +
  theme(axis.text.x=element_text(family="Arial", 
                                 face="italic", 
                                 colour="darkred",
                                 size=rel(0.9)))
~~~


* 軸  
~~~
# 軸タイトルの指定１
hw_plot +
  xlab("Age in years") +
  ylab("Height in inches")

# 軸タイトルの指定２
hw_plot +
  labs(x="Age in years", y="Height in inches")

# 軸タイトルの指定３
hw_plot +
  scale_x_continuous(name="Age\n(years)")

# 軸タイトルの非表示１、タイトル表示部分を完全になくす
pg_plot +
  xlab(NULL)

# 軸タイトルの非表示２、１と同じ
pg_plot +
  theme(axis.title.x=element_blank())

# 軸タイトルの非表３、タイトル部分をブランク
pg_plot +
  xlab("")

# 軸タイトルを回転させない（デフォルトは90度左に回転）
hw_plot +
  ylab("Height\n(inches)") +
  theme(axis.title.y=element_text(face="italic", size=14, angle=0))

# 軸を明示
hw_plot +
  theme(axis.line=element_line(colour="black"))

# 太軸、交点を揃える
hw_plot +
  theme_bw() +
  theme(panel.border=element_blank(),
        axis.line=element_line(colour="black", size=4, lineend="square"))

# 対数グラフ、簡易バージョン
animals_plot +
  scale_x_log10(breaks=10^(-1:5),
                labels=trans_format("log10", math_format(10^.x))) +
  scale_y_log10(breaks=10^(0:3),
                labels=trans_format("log10", math_format(10^.x)))

# データ自体を変換
ggplot(Animals, aes(log10(body), log10(brain),
                    label=rownames(Animals))) +
         geom_text(size=3)

# 対数グラフ、詳細設定
animals_plot +
  scale_x_continuous(
    trans=log_trans(),   # 軸を自然対数変換
    breaks=trans_breaks("log", function(x) exp(x)),  # 目盛りの位置を変換
    labels=trans_format("log", math_format(e^.x))    # 表記の指定
  ) +
  scale_y_continuous(
    trans=log2_trans(),  # 軸を対数変換（底２）
    breaks=trans_breaks("log2", function(x) 2^x),    # 目盛りの位置を変換
    labels=trans_format("log2", math_format(2^.x))   # 表記の指定
)

# 片方だけ対数軸
ggplot(aapl, aes(date, adj_price)) +
  geom_line() +
  scale_y_log10(breaks=c(2, 10, 50, 250))

# 目盛り幅を関数で指定
# 目盛りをグラフ内で表示（annotation_logticks）
breaks_log10 <- function(x) {
  low <- floor(log10(min(x)))
  high <- ceiling(log10(max(x)))
  10^(seq.int(low, high))
}

ggplot(Animals, aes(body, brain, label=rownames(Animals))) +
  geom_text(size=3) +
  scale_x_log10(breaks=breaks_log10, 
                labels=trans_format(log10, math_format(10^.x))) +
  scale_y_log10(breaks=breaks_log10, 
                labels=trans_format(log10, math_format(10^.x))) +
  annotation_logticks() # 対数目盛り
~~~


* 円形グラフ  
デフォルトではデータの最小値が座標の中心になる  
デフォルトでは回転軸の始点と終点が同じになる  
データは一周しかできない  
~~~
# -7.5から幅15でbinをスタートし、0、15、30、、、にセンタリング
# 凡例は逆順に表示、スタートは45度反時計回りにずらす
ggplot(wind, aes(DirCat, fill=SpeedCat)) +
  geom_histogram(binwidth=15, boundary=-7.5, colour="black", size=0.25) +
  guides(fill=guide_legend(reverse=T)) + # 凡例の表示順を逆にする
  coord_polar(start=-45*pi/180) +
  scale_x_continuous(limits=c(0, 360),
                     breaks=seq(0, 360, by=45),
                     minor_breaks=seq(0, 360, by=15)) +
  scale_fill_brewer()
~~~

* 日付  
~~~
# date_formatを使用するにはscalesのロードが必要
datebreaks <- seq(as.Date("1992-06-01"), as.Date("1993-06-01"), by="2 month")
econ_plot +
  scale_x_date(breaks=datebreaks, labels=date_format("%Y %b")) +
  theme(axis.text.x=element_text(angle=30, hjust=1))

# 表示形式を独自関数で設定
timeHM_formatter <- function(x) {
  h <- floor(x/60)
  m <- floor(x %% 60)
  lab <- sprintf("%d:%02d", h, m)
  return(lab)
}

ggplot(www, aes(minute, users)) +
  geom_line() +
  scale_x_continuous(
    name="time",
    breaks=seq(0, 100, by=10),
    labels=timeHM_formatter
  )

# 目盛りラベルを直接指定
ggplot(www, aes(minute, users)) +
  geom_line() +
  scale_x_continuous(
    name="time",
    breaks=seq(0, 100, by=20),
    labels=c("0:00", "0:20", "0:40", "1:00", "1:20", "1:40")
  )
~~~

* Tips  
seq(f, t, b): fromからtoまでをb区切りで数列化  
floor(x): xを越えない最大の整数ound(x): 独自の基準で少数以下を丸める
rownames(tbl): 行名のベクトルを出力  
colnames(tbl): 列名のベクトルを出力  
%+%: グラフオブジェクト %+% df で既存のグラフのデータを新しいものに置き換えることができる  
?strptime: 日付フォーマットオプション  
time(ts): タイムシリーズオブジェクトの時間のベクトルを出力する  


## 第９章　グラフの全体的な体裁  

軸ラベル、凡例、各タイトル、ファセットの書式設定  

* グラフタイトル  
~~~
# 上部余白
hw_plot +
  ggtitle("Age and Height of Schoolchildren", "11.5 to 17.5 years old")

# プロット内、vjust
hw_plot +
  ggtitle("Age and Height of Schoolchildren") +
  theme(plot.title=element_text(vjust=-8))

# プロット内、annotate
hw_plot +
  annotate("text", x=mean(range(heightweight$ageYear)), y=Inf,
           label="Age and Height of Schoolchildren", size=4.5, vjust=1.5)
~~~

* 書式設定  
タイトル、ラベル、凡例、ファセットラベルの書式を設定する
~~~
# 拡張書式の設定
library(extrafont)
font_import()
loadfonts(device="win")
windowsFonts()　# 出力される表記でfamilyを設定する

# 軸タイトル
# lineheightは行の間隔
hw_plot +
  theme(axis.title.x=element_text(size=16,
                                  lineheight=.9,
                                  family="serif",
                                  face="bold.italic",
                                  colour="red"))

# 表タイトル
hw_plot +
  ggtitle("Age and Height\nof Schoolchildren") +
  theme(plot.title=element_text(size=rel(1.5),
                                lineheight=.9,
                                family="serif",
                                face="bold.italic",
                                colour="red"))

# アノテーション
hw_plot +
  annotate("text", x=15, y=53, label="Some text",
           size=7, family="serif", fontface="bold.italic", colour="red")

# データ毎
hw_plot +
  geom_text(aes(label=weightLb), size=4, family="serif", colour="red")
~~~

* テーマ設定  

デフォルトはtheme_grey
theme_bw()、theme_minimal()、theme_classic()、theme_void()等  

~~~
各オブジェクトのテーマ設定

#プロット領域
hw_plot +
  theme(panel.grid.major=element_line(colour="red"),
        panel.grid.minor=element_line(colour="red", linetype="dashed", size=0.2),
        panel.background=element_rect(fill="lightblue"),
        panel.border=element_rect(colour="blue", fill=NA, size=2))

# 凡例
hw_plot +
  theme(legend.background=element_rect(fill="grey85", colour="red", size=1),
        legend.title=element_text(colour="blue", face="bold", size=14),
        legend.text=element_text(colour="red"),
        legend.key=element_rect(colour="blue", size=0.25)
        )

#タイトル、ラベル
hw_plot +
  ggtitle("Plot title here") +
  theme(
    axis.title.x=element_text(colour="red", size=14),
    axis.text.x=element_text(colour="blue"),
    axis.title.y=element_text(colour="red", size=14, angle=90),
    axis.text.y=element_text(colour="blue"),
    plot.title=element_text(colour="red", size=20, face="bold")
  )

# ファセット
hw_plot +
  facet_grid(sex~.) +
  theme(
    strip.background=element_rect(fill="pink"),
    strip.text.y=element_text(size=14, angle=-90, face="bold")
  )

# 独自の書式設定を関数化
mytheme <- theme_bw() +
  theme(
    text=element_text(colour="red"),
    axis.title=element_text(size=rel(1.25))
  )
)

hw_plot + mytheme

# 目盛線の非表示
hw_plot +
  theme(
    panel.grid.major=element_blank(),
    panel.grid.minor=element_blank()
  )
~~~


## 第１０章　凡例  

~~~
# 非表示
pg_plot +
  guides(fill=F)

pg_plot +
  scale_fill_discrete(guide=F)

pg_plot +
  theme(legend.position="none")
  
# グラフ領域の右下に凡例の右下を合わせる
pg_plot + 
  theme(legend.position=c(1, 0), legend.justificatio=c(1, 0))

# 凡例の背景を操作
pg_plot + 
  theme(legend.position=c(0.85, 0.2)) +
  theme(legend.background=element_rect(fill="white", colour="black"))

# 凡例の背景と項目の背景を非表示
pg_plot + 
  theme(legend.position=c(0.85, 0.2)) +
  theme(legend.key=element_blank()) +
  theme(legend.background=element_blank())
~~~

* 表示順の変更
~~~
# limitsで順番に指定
pg_plot +
  scale_fill_discrete(limits=c("trt1", "trt2", "ctrl"))

pg_plot +
  scale_fill_grey(start=.5, end=1, limits=c("trt1", "trt2", "ctrl"))

pg_plot +
  scale_fill_brewer(palette="Pastel2", limits=c("trt1", "trt2", "ctrl"))

# 逆順
pg_plot +
  guides(fill=guide_legend(reverse=T))

pg_plot +
  scale_fill_discrete(guide=guide_legend(reverse=T))
~~~

* タイトル  
~~~
# labs
pg_plot +
  labs(fill="Condition")

# scale
pg_plot +
  scale_fill_discrete(name="Condition")

# guide
pg_plot +
  guides(fill=guide_legend(title="Condition"))

# ２つある場合
hw_plot +
  labs(colour="Male/Female", size="Weight\n(pounds)")

# 書式設定
pg_plot +
  theme(legend.title=element_text(face="italic",
                                  family="serif",
                                  colour="red",
                                  size=14))
# 非表示
pg_plot +
  guides(fill=guide_legend(title=NULL))

pg_plot +
  scale_fill_discrete(name=NULL)
~~~

* ラベル  
~~~
# 指定順に上から表示
pg_plot +
  scale_fill_discrete(labels=c("Control", "Treatment 1", "Treatment 2"))

# 図に表示する順も合わせて表示する
pg_plot +
  scale_fill_discrete(limits=c("trt1", "trt2", "ctrl"),
    labels=c("Treatment 1", "Treatment 2", "Control"))

# 複数の属性にマッピングされている場合は両方変更する
ggplot(heightweight, aes(ageYear, heightIn, shape=sex, colour=sex)) +
  geom_point() +
  scale_shape_discrete(labels=c("Female", "Male")) +
  scale_colour_discrete(labels=c("Female", "Male"))

# 体裁、theme
pg_plot +
  theme(legend.text=element_text(
    face="italic",
    family="Times New Roman",
    colour="red",
    size=14
  ))

# 体裁、guides
pg_plot +
  guides(fill=guide_legend(label.theme=element_text(
    face="italic",
    family="Times New Roman",
    colour="red",
    size=14
  )))

# 凡例要素の高さを変更、key
pg_plot +
  scale_fill_discrete(labels=c("Control", "Type1\ntreatment", 
                      "Type2\ntreatment")) +
  theme(legend.text=element_text(lineheight=.8),
        legend.key.height=unit(1, "cm"))
~~~

## 第１１章　ファセット  

サブプロット  

* 基本  
~~~
# 行、列の順に指定
mpg_plot +
  facet_grid(drv~cyl)

# 行もしくは列の数を指定
mpg_plot +
  facet_wrap(~class, nrow=2)

# y軸の範囲を行もしくは列ごとに変える
mpg_plot +
  facet_grid(drv~cyl, scales="free_y")

# 軸の範囲を行もしくは列ごとに変える
mpg_plot +
  facet_grid(drv~cyl, scales="free")

# ラベルに変数名を入れる
mpg_mod <- mpg %>%
  mutate(drv=recode(drv, "4"="4wd", "f"="Front", "r"="Rear"))

ggplot(mpg_mod, aes(displ, hwy)) +
  geom_point() +
  facet_grid(drv~., labeller=label_both)

# ラベルに数式を用いる
mpg_mod <- mpg %>%
  mutate(drv=recode(drv, 
                    "4"="4^{wd}", 
                    "f"="- Front %.% e^{pi*i}", 
                    "r"="4^{wd}-Front"))

ggplot(mpg_mod, aes(displ, hwy)) +
  geom_point() +
  facet_grid(drv~., labeller=label_parsed)

# ラベルの書式設定  
ggplot(cabbage_exp, aes(Cultivar, Weight)) +
  geom_col() +
  facet_grid(.~Date) +
  theme(
    strip.text=element_text(face="bold", size=rel(1.5)),
    strip.background=element_rect(fill="lightblue", colour="black",
                                  size=1)
  )

~~~

* Tips  
ファセットのラベル名変更はできない（変数名を変更するしかない）  


## 第１２章　色を使う  

* viridisパレット  
~~~
# 離散値の場合１
uspopage_plot +
  scale_fill_viridis_d()

# 離散値の場合２
uspopage_plot +
  scale_fill_viridis(discrete=T)

# magma, plasma, inferno, cividis等
uspopage_plot +
  scale_fill_viridis_d(option="cividis")
~~~

* その他  
~~~
# brewer
uspopage_plot +
  scale_fill_brewer()

# grey
hw_splot +
  scale_colour_grey(start=0.7, end=0)

# 輝度の操作
hw_splot +
  scale_colour_discrete(l=45)
  
# マニュアル設定

# 色指定
hw_plot +
  scale_colour_manual(values=c("red", "blue"))

# RGB値
# R、G、Bをそれぞれ二桁の16進数（0～F）で表現
# 二桁目はほとんど影響しないため、同じ数値にすることが多い
hw_plot +
  scale_colour_manual(values=c("#CC6666", "#7777DD"))

# 変数指定
hw_plot +
  scale_colour_manual(values=c(m="blue", f="red"))


# 連続変数

# 色指定
hw_plot +
  scale_colour_gradient(low="black", high="white")

# 三色
hw_plot +
  scale_colour_gradient2(
    low=muted("red"),  # 彩度を落とす
    mid="white",
    high=muted("blue"),
    midpoint=110  # 真ん中になる値
  )

# 複数色
hw_plot +
  scale_colourgradientn(colours=c("darkred", "orange", "yellow", "white"))

# viridis
hw_plot +
  scale_colour_viridis_c(option="magma")

# 領域
# 離散変数を作成
climate_mod <- climate %>%
  filter(Source=="Berkeley") %>%
  mutate(valence=if_else(Anomaly10y>=0, "pos", "neg"))

# 離散変数に色を割り当て
ggplot(climate_mod, aes(Year, Anomaly10y)) +
  geom_area(aes(fill=valence)) +
  geom_line() +
  geom_hline(yintercept=0)
~~~

* Tips  
colors(): 色名一覧  
demo("colors"): 色デモ  
viridis(5): viridisを5分類した時の各色をRGB値で返す  

## 第１３章　さまざまなグラフ  

* 相関行列  
~~~
library(corrplot)

# 基本設定
corrplot(mcor, 
         method="shade",  # パネル表示
         shade.col=NA,    # 正負を表す斜線は使用しない 
         tl.col="black",  # ラベルの色
         tl.srt=45        # ラベルの表示角度
         )

# 詳細設定
col <- colorRampPalette(c("#BB4444", "#EE9988", "#FFFFFF", "#77AADD", "#4477AA"))
corrplot(mcor, method="shade", shade.col=NA, tl.col="black", tl.srt=45,
         addCoef.col="black", # 数字の追加
         cl.pos="n",          # バーの非表示
         order="AOE",         # 相関の高い要素を隣接させる
         col=col(200)         # 色のタイプ
         )
~~~

* 関数表示  
~~~
# 既存関数、パラメータの有無
p + stat_function(fun=dnorm)
p + stat_function(fun=dt, args=list(df=2))

# 自作関数
ggplot(data.frame(x=c(0, 20)), aes(x)) +
  stat_function(fun=myfunc, n=200)

# 網掛け
# 関数の範囲を限定する関数
limitRange <- function(fun, min, max) {
  function(x) {
    y <- fun(x)
    y[x<min|x>max] <- NA
    return(y)
  }
}
# 範囲を限定された関数を併せて表示
p +
  stat_function(fun=dnorm) +
  stat_function(fun=limitRange(dnorm, 0, 2), geom="area", fill="blue", alpha=.2)
~~~

* ネットワーク（標準作図関数）  
~~~
# 第一列から第二列への有向グラフ
g <- graph.data.frame(madmen2, directed=T)
plot(g, layout=layout.fruchterman.reingold, vertex.size=8,
     edge.arrow.size=0.5, vertex.label=NA)

# 無向グラフ
g <- graph.data.frame(madmen, directed=F)
plot(g, layout=layout.circle, vertex.size=8, vertex.label=NA)

# ラベルの設定１
# グラフで設定
plot(g, layout=layout.fruchterman.reingold,
     vertex.size=4,
     vertex.label=V(g)$name,
     vertex.label.cex=0.8,
     vertex.label.dist=0.4,
     vertex.label.color="black")

# ラベルの設定２
# 頂点の属性として設定
V(g)$size <- 4
V(g)$label <- V(g)$name
V(g)$label.cex <- 0.8
V(g)$label.dist <- 0.4
V(g)$label.color <- "black"
g$layout <- layout.fruchterman.reingold
plot(g)

# 辺の設定
E(g)[c(2, 11, 19)]$label <- "M"
E(g)$color <- "grey70"
E(g)[c(2, 11, 19)]$color <- "red"
plot(g)


# レイアウトの種類確認
?igraph:: layout
~~~

* ヒートマップ  

~~~
# 基本  
ggplot(pres_rating, aes(year, quarter, fill=rating)) + 
  geom_tile()

# 描画効率が良い
p + geom_raster()

# 詳細設定
p + 
  geom_tile() +
  scale_y_reverse(expand=c(0, 0)) + # 逆順にし、上下の余白を除く
  scale_x_continuous(breaks=seq(1940, 1976, by=4), expand=c(0, 0)) + # 目盛り設定
  scale_fill_gradient2(midpoint=50, mid="grey50", limits=c(0, 100)) # グラデーションの設定
~~~

* 三次元プロット  
~~~
install.packages("rgl")
library(rgl)

plot3d(mtcars$wt, mtcars$disp, mtcars$mpg, 
       typ="s",  # 球状
       size=0.75, 
       lit=FALSE) # 光沢なし


# 背景操作

# 線分（端点の組み合わせ）のベクトルを作成
interleave <- function(v1, v2) as.vector(rbind(v1, v2))

# 点のみ
plot3d(mtcars$wt, mtcars$disp, mtcars$mpg, 
       xlab="", ylab="", zlab="",
       axes=F,
       typ="s", size=0.75, lit=FALSE)

# 線分
segments3d(interleave(mtcars$wt, mtcars$wt),
           interleave(mtcars$disp, mtcars$disp),
           interleave(mtcars$mpg, min(mtcars$mpg)),
           alpha=0.4, col="blue")

# 3D領域
rgl.bbox(color="grey50")

# 軸ラベルの追加
axes3d(edges=c("x--", "y++", "z--"), # 軸の位置
       ntick=6, # 目盛りの数
       cex=0.75) # ラベルの大きさ

# 軸タイトルの追加
mtext3d("Weight", edge="x--", line=2) # lineは軸とラベルの距離
mtext3d("Displacement", edge="y++", line=3) # lineは軸とラベルの距離
mtext3d("MPG", edge="z--", line=3) # lineは軸とラベルの距離


# 予測面の追加、調整

# 実測値
plot3d(mtcars$wt, mtcars$disp, mtcars$mpg,
     xlab="", ylab="", zlab="",
     axes=F,
     size=.5, type="s", lit=F)

# 予測値
spheres3d(m$wt, m$disp, m$pred_mpg, alpha=0.4, type="s", size=0.5,
          lit=F)

# 実測値から予測値への線分
segments3d(interleave(m$wt, m$wt),
           interleave(m$disp, m$disp),
           interleave(m$mpg, m$pred_mpg),
           alpha=0.4, col="red")

# 予測面
surface3d(mpgrid_list$wt, mpgrid_list$disp, mpgrid_list$mpg,
          alpha=0.4, 
          front="lines", # グリッド面
          back="lines"
)

# 軸平面
rgl.bbox(color="grey50", # 表面の色
         emission="grey50", # 発行色
         xlen=0, ylen=0, zlen=0
         )

# 軸ラベルの追加
rgl.material(color="black")
axes3d(edges=c("x--", "y++", "z--"),
       ntick=6,
       cex=.75)

# 軸タイトル
mtext3d("Weight", edge="x--", line=2)
mtext3d("Displacement", edge="y++", line=3)
mtext3d("MPG", edge="z--", line=3)
~~~

* 保存  
~~~
#表示されているグラフの保存
rgl.snapshot('3dplot.png', fmt='png')
rgl.postscript('3dplot.pdf', fmt='pdf')
rgl.postscript('3dplot.ps', fmt='ps')


#現在の視点を保存
view <- par3d("userMatrix")

#視点を再現
par3d(userMatrix=view)

#視点を出力
dput(view)

#視点を設定
View <- structure(c(0.0885247588157654, 0.297822892665863, -0.950507640838623, 
                    0, -0.970648050308228, -0.188437670469284, -0.149443686008453, 
                    0, -0.223619237542152, 0.935837864875793, 0.272399872541428, 
                    0, 0, 0, 0, 1), .Dim = c(4L, 4L))

#視点を再現
par3d(userMatrix=view)

# アニメーション
# 回転速度、時間を設定
plot3d(mtcars$wt, mtcars$disp, mtcars$mpg, type="s", size=0.75, lit=F)
play3d(spin3d(axis=c(1, 0, 0), rpm=4), duration=20)

# 保存、フレーム数を設定
library(migick)
movie3d(spin3d(axis=c(0, 0, 1), rpm=4), duration=15, fps=50,
        movie="C:/Users/yh_in/learning/R-Graphics/anime",
        convert=F)
~~~

* その他  
~~~
# 樹形図
# distで各点の距離を計測、hclustでクラスター
hc <- hclust(dist(c3))
plot(hc, hang=-1)


# ベクトルフィールド
# 基本
ggplot(islicesub, aes(x, y)) +
  geom_segment(aes(xend=x+vx/50, yend=y+vy/50, alpha=speedxy), # ベクトルの終点を設定
               arrow=arrow(length=unit(0.1, "cm")),
               size=0.6)

# 地図に重ねる
usa <- map_data("usa")
ggplot(islicesub, aes(x, y)) +
  geom_segment(aes(xend=x+vx/50, yend=y+vy/50, colour=speedxy),
               arrow=arrow(length=unit(0.1, "cm")),
               size=0.6) +
  scale_colour_continuous(low="grey80", high="darkred") +
  geom_path(aes(x=long, y=lat, group=group), data=usa) + #地図の表示
  coord_cartesian(xlim=range(islicesub$x), ylim=range(islicesub$y)) #ズームイン

# ファセット
ggplot(isub, aes(x, y)) +
  geom_segment(aes(xend=x+vx/50, yend=y+vy/50, colour=speed),
               arrow=arrow(length=unit(0.1, "cm")),
               size=0.5) +
  scale_colour_continuous(low="grey80", high="darkred") +
  facet_wrap(~z)


# Q-Qプロット、対応する正規分布と合わせて表示
ggplot(heightweight, aes(sample=ageYear)) +
  geom_qq_line() +
  geom_qq()


# 累積分布 
ggplot(heightweight, aes(heightIn)) +
  stat_ecdf()


# モザイクプロット  
# 基本、インプットは多次元分割表
library(vcd)
mosaic(~Admit + Gender + Dept, data=UCBAdmissions)

# 応用
mosaic(~Dept + Gender + Admit, data=UCBAdmissions,
       highlighting="Admit",
       highlighting_fill=c("lightblue", "pink"),  # 指定のカテゴリで色分け
       direction=c("v", "h", "v")) # v:垂直線で分割、h: 水平線で分割

# 円グラフ  
# 分割表（table）を入力
pie(fold)

# 直接指定
pie(c(99, 18, 120), labels=c("L on R", "Neither", "R on L"))


# 地図

# 地図データの取得
states_map <- map_data("state")
# 塗りつぶしあり
ggplot(states_map, aes(long, lat, group=group)) +
  geom_polygon(fill="white", colour="black")
# 塗りつぶしなし
ggplot(states_map, aes(long, lat, group=group)) +
  geom_path() +
  coord_map("mercator")

# 特定の地域
east_asia <- map_data("world", region=c("Japan", "China", "North Korea", "South Korea"))
ggplot(east_asia, aes(long, lat, group=group, fill=region)) +
  geom_polygon(colour="black") +
  scale_fill_brewer(palette="Set2")


# 地図に値を割り当てて表示１（geom_polygonを使用）

# 割り当て
crime_map <- merge(states_map, crimes, by.x="region", by.y="state")

# 値をfillに設定
crime_p <- ggplot(crime_map, aes(long, lat, group=group, fill=Assault)) +
  geom_polygon(colour="black") +
  expand_limits(x=states_map$long, y=states_map$lat) # 範囲の拡張（この場合変わらない）
  coord_map("polyconic")

# グラデーション１
crime_p +
  scale_fill_gradient2(low="#559999", mid="grey90", high="#BB650B",
                      midpoint=median(crimes$Assault))

# グラデーション２
crime_p +
  scale_fill_viridis_c()


# 地図に値を割り当てて表示２（geom_mapを使用）
# mapに与えるデータフレームはgeom_polygonで用いた、lat、long、regionの列を持つ必要がある
# map_idにはregionに対応する変数を与える
ggplot(crimes, aes(map_id=state, fill=Assault_q)) +
  geom_map(map=states_map, colour="black") +
  scale_fill_manual(values=pal) +
  expand_limits(x=states_map$long, y=states_map$lat) + # geom_mapの時は必ず必要
  coord_map("polyconic") +
  labs(fill="Assault Rate\nPercentile") +
  theme_void() # 背景を消す場合


# シェープファイルから作図
# https://gadm.org/index.html から旧バージョンのshファイルをダウンロード

library(sf)
taiwan_shp <- st_read("TWN_adm2.shp")
ggplot(taiwan_shp_mod) +
  geom_sf(aes(fill=ENGTYPE_2))
~~~

* Tips  
scale(df): 各列を正規化  
merge(tbl1, tbl2, by.x=BMONTH, by.y=TMONTH): 結合（join）と同じ



## 第１４章　文書用に図を出力する  

* 出力  
~~~
# PDFファイルへの出力

# 標準
pdf("myplot.pdf", width=4, height=4)
plot(mtcars$wt, mtcars$mpg)
ggplot(mtcars, aes(wt, mpg)) + geom_point()
dev.off()

# ggplot
plot1 <- ggplot(mtcars, aes(wt, mpg)) + geom_point()
ggsave("myplot.pdf", plot1, width=8, height=8, units="cm")


# svgファイルへの出力

# 標準
install.packages("svglite")
library(svglite)
svglite("myplot.svg", width=4, height=4)
plot(mtcars$wt, mtcars$mpg)
dev.off()

# ggplot
ggplot(mtcars, aes(wt, mpg)) + geom_point()
ggsave("myplot.svg", width=8, height=8, units="cm")


# wmfファイルへの出力

# 標準
win.metafile("myplot.wmf", width=4, height=4)
plot(mtcars$wt, mtcars$mpg)
dev.off()

# ggplot
ggplot(mtcars, aes(wt, mpg)) + geom_point()
ggsave("myplot.wmf", width=8, height=8, units="cm")


# PNGファイル（ビットマップファイル）への出力

# 標準（複数ファイル）
# 幅と高さは画素数（ピクセル）、解像度はインチあたりピクセルで指定
# ⇒ 解像度に合わせて幅と高さを増減させないとグラフが縮小、拡大される
png("myplot-%d.png", width=400, height=400, dpi=100)
plot(mtcars$wt, mtcars$mpg)
ggplot(mtcars, aes(wt, mpg)) + geom_point()
dev.off()

# ggplot
ggplot(mtcars, aes(wt, mpg)) + geom_point()
ggsave("myplot_gg2.png", width=8, height=8, unit="cm", dpi=300)


# フォント指定

library(extrafont)
loadfonts("win")

ggplot(mtcars, aes(wt, mpg)) +
  geom_point() +
  ggtitle("Title text goes here") +
  theme(text=element_text(size=16, family="Georgia", face="italic"))

ggsave("myplot.png", width=4, height=4, dpi=300)


# 図の結合
library(patchwork)
plot1 + plot2 + plot_layout(ncol=1, heights=c(1, 4))
~~~


## 第１６章　データの前処理  

* tidyverse  
~~~
# 列の削除
ToothGrowth %>%
  select(-len)

# 列名の変更
ToothGrowth %>%
  rename(length=len)

# 列の順序を変える
# everything()は残りの列全てを表す
ToothGrowth %>%
  select(dose, everything())

# 行の選択（インデックス指定）
ToothGrowth %>%
  slice(1:5)

# カテゴリ変数の順序変更
# stats/reorder
iss %>%
  ggplot(aes(x=reorder(spray, count, FUN=mean), count)) +
  geom_boxplot()

# forcats/fct_reorder
iss %>%
  ggplot(aes(x=fct_reorder(spray, count, .fun=median), count)) +
  geom_boxplot()

# カテゴリ変数のレベル名の操作
# base
levels(sizes) <- list(S="small", M="medium", L="large")
# forcats
fct_recode(sizes, S="small", M="medium", L="large")

# カテゴリ変数で使用していないレベルを削除する
# base
droplevels(sizes)
# forcats
fct_drop(sizes)

# 文字列ベクトルの置き換え、baseとtidyverseで設定の右辺と左辺が逆になる点に注意
# dplyr
# recodeはインプットと同じ型を返す
recode(sizes, small="S", medium="M", large="L")
# forcats
# fct_recodeは常にファクタを返す
fct_recode(sizes, S="small", M="medium", L="large")

# カテゴリ変数の変換も文字列ベクトルの置き換え同様
# 文字列をそのまま返す
recode(pg$group, ctrl="No", trt1="Yes", trt2="Yes")
# ファクターに変換して返す
fct_recode(pg$group, No="ctrl", Yes="trt1", Yes="trt2")
~~~

* tidyverseを用いた関数（標準誤差の算出）
~~~
summarySE <- function(data=NULL, measurevar, groupvars=NULL, na.rm=F,
                      conf.interval=.95, .drop=T){
  
  #有効データ数
  length2 <- function(x, na.rm=F) {
    if(na.rm) sum(!is.na(x))
    else length(x)
  }
  
  #関数に与えた値、ベクトルを、
  #tidyverseの中でそれぞれ単独列変数、グループ列変数として扱えるようにする
  groupvars <- rlang::syms(groupvars)
  measurevar <- rlang::sym(measurevar)
  
  datac <- data %>%
    dplyr::group_by(!!!groupvars) %>%
    dplyr::summarize(
      N=length2(!!measurevar, na.rm=na.rm),
      sd=sd(!!measurevar, na.rm=na.rm),
      !!measurevar:=mean(!!measurevar, na.rm=na.rm),
      se=sd/sqrt(N),
      ci=se*qt(conf.interval/2+.5, N-1)
    ) %>%
    dplyr::ungroup() %>%
    dplyr::select(seq_len(ncol(.)-4), ncol(.)-2, sd, se, ci)
  
  datac
}
~~~

* 時系列オブジェクトの変換
~~~
# 年次データ
pres_rating <- data.frame(
  year=as.numeric(time(presidents)),
  rating=as.numeric(presidents)
)

# 四半期データ
pres_rating <- data.frame(
  year=as.numeric(floor(time(presidents))),
  quarter=as.numeric(cycle(presidents)),
  rating=as.numeric(presidents)
)
~~~

* Tips  
fct_relevel(fct, "A", "B, "C"): ファクター列のカテゴリ順序を変更する
interaction(col1, col2): 列の値を結合して新しい列の値を作る
cut(col, breaks=閾値ベクトル, labels=ラベルベクトル): 連続変数をカテゴリ変数に変換する
complete(col1, col2): 不足する組み合わせ行を追加する
gather(tbl, factor_colname, val_colname, gathered_col1, gathered_col1): 横持ちから縦持ちへの変換
spread(tbl, factor_colname, val_colname): 縦持ちから横持への変換
unite(new_colname, col1, col2): 列同士を結合した列を加えたtblを返す

