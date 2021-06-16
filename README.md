# R-Graphics

[正誤表]  
[英語](https://www.oreilly.com/library/view/r-graphics-cookbook/9781491978597/)  
[日本語](https://www.oreilly.co.jp/books/9784873118925/)  


## 付録A ggplot2を理解する  

グラフ化 ⇒ データの視覚属性（軸、色等）へのマッピング  
縦持ち形式（各行が観測値、各列が変数）のデータフレームが前提  
スケール: データ空間からエステティック空間への投影の仕方を定義するもの  
ガイド: 目盛り、ラベル、凡例等  

* 基本操作  
~~~
#棒グラフ、軸の入替

ggplot(simpledat_long, aes(Aval, value, fill=Bval)) +
  geom_col(position="dodge")

ggplot(simpledat_long, aes(Bval, value, fill=Aval)) +
  geom_col(position="dodge")

# 折れ線グラフ

ggplot(simpledat_long, aes(Aval, value, group=Bval, colour=Bval)) +
  geom_line()

# 幾何オブジェクトのエステティック空間へのマッピング
p + geom_point(aes(colour=group))

# 幾何オブジェクトの設定
p + geom_point(colour="blue")

# スケールの設定（軸）
p + 
  geom_point(colour="blue") +
  scale_x_continuous(limits=c(0, 8))

# スケールの設定（色）
p + 
  geom_point(aes(colour=group)) +
  scale_colour_manual(values=c("orange", "forestgreen"))
~~~


　  
## 第１章　Rの基本  

~~~
# 役に立つデータセット
install.packages("gcookbook")
~~~

~~~
# パッケージのアップデイト
update.packages()
update.packages(ask=FALSE) #一括
~~~

* データのサマリー
~~~
# 単変量
morley %>%
  filter(Expt==1) %>%
  summary()

# 共変量
morley %>%
  filter(Expt==1) %>%
  pairs()
~~~


　  
## 第２章　データの基本的なプロット  

baseグラフィックスとの比較  
基本的にggplotはいかなるグラフでも統一的な処理が可能  

* 散布図  
~~~
# base
plot(mtcars$wt, mtcars$mpg)

# ggplot
ggplot(mtcars, aes(wt, mpg)) +
  geom_point()
~~~

* 折れ線グラフ  
~~~
# base
plot(pressure$temperature, pressure$pressure, type="l")
points(pressure$temperature, pressure$pressure)
lines(pressure$temperature, pressure$pressure/2, col="red")
points(pressure$temperature, pressure$pressure/2, col="red")

# ggplot
ggplot(pressure) +
  geom_line(aes(temperature, pressure)) +
  geom_point(aes(temperature, pressure)) +
  geom_line(aes(temperature, pressure/2), colour="red") +
  geom_point(aes(temperature, pressure/2), colour="red")
~~~

* 棒グラフ  
~~~
# base
# 生値
barplot(BOD$demand, names.arg=BOD$Time)
# 集計値
barplot(table(mtcars$cyl))

# ggplot
# 生値  
# 連続値
ggplot(BOD, aes(Time, demand)) +
  geom_col()
# 離散値
ggplot(BOD, aes(factor(Time), demand)) +
  geom_col()
# 集計値
# 連続値
ggplot(mtcars, aes(cyl)) +
  geom_bar()
# 離散値
ggplot(mtcars, aes(factor(cyl))) +
  geom_bar()
aes(temperature, pressure/2), colour="red") +
  geom_point(aes(temperature, pressure/2), colour="red")
~~~

* ヒストグラム  
~~~
# base
# ビンの数を指定
hist(mtcars$mpg, breaks=10)

# ggplot
# ビンの幅を指定
ggplot(mtcars, aes(mpg)) +
  geom_histogram(binwidth=4)
~~~

* 箱ひげ図  
~~~
# base
plot(ToothGrowth$supp, ToothGrowth$len)
boxplot(len~supp+dose, data=ToothGrowth)

# ggplot
ggplot(ToothGrowth, aes(interaction(supp, dose), len)) +
  geom_boxplot()
~~~

* 関数曲線  
~~~
# 関数定義
myfun1 <- function(xvar) {
  1 / (1 + exp(- xvar + 10))
}
myfun2 <- function(xvar) {
  1 - 1 / (1 + exp(- xvar + 10))
}

# base
curve(myfun1(x), from=0, to=20)
curve(1-myfun1(x), add=TRUE, col="red")

# ggplot
ggplot(tibble(x=c(0, 20)), aes(x)) +
  stat_function(fun=myfun1, geom="line") + 
  stat_function(fun=myfun2, geom="line", color="red")
~~~

* Tips  
search(): ロード済みのパッケージを確認  
interaction(col1, col2): 合成ファクターの生成  


　  
## 第３章　棒グラフ    

ある区分に対応する値を示す  

* 基本  
~~~
# fill: 塗りつぶしの色、colour: 枠線の色
ggplot(pg_mean, aes(group, weight)) +
  geom_col(fill="lightblue", colour="black")
~~~

* グループ化  
~~~
# 塗りつぶしでグループ化、塗りつぶしの色設定変更（brwerのパレットを使用）
# positionはデフォルトが積み上げ、"dodge"はグラフを重ねない
ggplot(cabbage_exp, aes(Date, Weight, fill=Cultivar)) +
  geom_col(position="dodge", colour="black") +
  scale_fill_brewer(palette="Pastel1")
~~~

* 要約量  
~~~
# geom_barを用いる
ggplot(diamonds, aes(cut)) +
  geom_bar()
~~~

* 色付きのグラフ  
~~~
# reorderでファクターの順序を変更、塗りつぶしの色設定を個別に変更
ggplot(upc, aes(reorder(Abb, Change), Change)) +
  geom_col(aes(fill=Region), colour="black") +
  scale_fill_manual(values=c("#669933", "#FFCC66")) +
  xlab("State")
~~~

* グラフの色分け  
~~~
# sizeは枠線の幅、guide=Fで凡例を非表示
ggplot(climate_sub, aes(Year, Anomaly10y)) +
  geom_col(aes(fill=pos), colour="black", size=0.25) +
  scale_fill_manual(values=c("#CCEEFF", "#FFDDDD"), guide=F)
~~~

* 棒の幅  
~~~
# width=0.9がデフォルト、position_dodge(x)で棒グラフ中心間の幅を設定
ggplot(cabbage_exp, aes(Date, Weight)) +
  geom_col(aes(fill=Cultivar), width=0.5, position=position_dodge(0.7))
~~~

* 積み上げ棒グラフ  
~~~
# デフォルト
ggplot(cabbage_exp, aes(Date, Weight)) +
  geom_col(aes(fill=Cultivar))

# 逆順
ggplot(cabbage_exp, aes(Date, Weight)) +
  geom_col(aes(fill=Cultivar), position=position_stack(reverse=T), colour="black") +
  scale_fill_brewer(palette="Pastel1") +
  guides(fill=guide_legend(reverse=T))
~~~

* 100%積み上げ棒グラフ  
~~~
# スケールの設定でラベルをパーセント表示
ggplot(cabbage_exp, aes(Date, Weight)) +
  geom_col(aes(fill=Cultivar), position="fill", colour="black") +
  scale_y_continuous(labels=scales::percent) +
  scale_fill_brewer(palette="Pastel1")
~~~

* ラベル  
~~~
# vjustで位置の上下を微調整、ラベルを加味して表示範囲を調整
ggplot(cabbage_exp, aes(interaction(Date, Cultivar), Weight)) +
  geom_col() +
  geom_text(aes(label=Weight), vjust=-0.2) +
  ylim(0, max(cabbage_exp$Weight)*1.05)

# 位置を直接指定、表示範囲は自動調整
ggplot(cabbage_exp, aes(interaction(Date, Cultivar), Weight)) +
  geom_col() +
  geom_text(aes(y=Weight+0.1, label=Weight))

# 要約量
# geom_barを用い、statとlabelをcountに指定
ggplot(mtcars, aes(factor(cyl))) +
  geom_bar() +
  geom_text(aes(label=..count..), stat="count", vjust=1.5, colour="white")

# グループ化
# geom_textではpositionの簡易表示"dodge"は使用できない
ggplot(cabbage_exp, aes(Date, Weight, fill=Cultivar)) +
  geom_col(position="dodge") +
  geom_text(aes(label=Weight),
            vjust=1.5,
            colour="white", size=4, 
            position=position_dodge(0.9))

# 積み上げ棒グラフ

# 表示順序にデータを並べる
ce <- cabbage_exp %>%
  arrange(Date, rev(Cultivar))

# ラベルの表示位置を定める列を作成
ce <- ce %>%
  group_by(Date) %>%
  mutate(label_y=cumsum(Weight)-0.5*Weight)

# ラベルの表示形式を追加的に設定
ggplot(ce, aes(Date, Weight, fill=Cultivar)) +
  geom_col(colour="black") +
  geom_text(aes(label=paste(format(Weight, nsmall=2), "kg"), y=label_y), 
            size=4) +
  scale_fill_brewer(palette="Pastel1")
~~~

* ドットプロット  
~~~
# テーマの設定
# x軸の目盛り線を消す、y軸目盛りの書式を設定
ggplot(tophit, aes(avg, reorder(name, avg))) +
  geom_point(size=3) +
  theme_bw() +
  theme(
    panel.grid.major.x=element_blank(),
    panel.grid.minor.x=element_blank(),
    panel.grid.major.y=element_line(colour="grey60", linetype="dashed")
  ) 

# 軸の入替、目盛りラベルの書式設定
ggplot(tophit, aes(avg, reorder(name, avg))) +
  geom_point(size=3) +
  theme_bw() +
  theme(
    panel.grid.major.x=element_blank(),
    panel.grid.minor.x=element_blank(),
    panel.grid.major.y=element_line(colour="grey60", linetype="dashed"),
    axis.text.x = element_text(angle=60, hjust=1)
  ) +
  coord_flip()

# グループ分け１
# 先に指定順のファクター変数を用意
# geom_segmentでデータから線分を引く
# データの表示順に合わせてlimitsで色の割当順を設定
# 目盛りと凡例の書式設定
tophit %>%
  mutate(nameorder=tophit$name[order(tophit$lg, tophit$avg)]) %>%
  mutate(name2=factor(name, levels=nameorder)) %>%
  select(name2, lg, avg, nameorder) %>%
  ggplot(aes(avg, name2)) +
    geom_point(aes(colour=lg), size=3) +
    geom_segment(aes(yend=name2), xend=0, colour="grey50") +
    scale_colour_brewer(palette="Set1", limits=c("NL", "AL")) +
    theme_bw() +
    theme(
      panel.grid.major.y=element_blank(),
      legend.position=c(1, 0.5),
      legend.justification=c(1, 0.5)
    )

# グループ分け２
# 凡例なし、guide=FALSE
# ファセット毎にスケールを変える（scales）
# ファセット毎に大きさを変える（space）
ggplot(tophit, aes(avg, reorder(name, avg))) +
  geom_point(size=3, aes(colour=lg)) +
  geom_segment(aes(yend=name), xend=0, colour="grey50") +
  scale_colour_brewer(palette="Set1", guide=FALSE, limits=c("NL", "AL")) +
  facet_grid(lg~., scales="free_y", space="free_y") +
  theme_bw() +
  theme(panel.grid.major.y=element_blank())
~~~

* Tips  
RColorBrewer::display.brewer.all(): R Color Brewerの全パレット表示  
top_n(n, col): 上位n個のデータを抽出  
reorder(col, col2): col1をファクターに変換し、col2の順序で整列
rev(col): ベクトルを逆順にする  
desc(col): 符号を反転させる  
format(x, nsmall=2): 少数第二位まで表示  


　  
## 第４章　折れ線グラフ    

連続変数もしくはカテゴリ変数に対して、連続値がどのように推移するかを表示  

* 基本  
~~~
# 連続値
# 縦軸の範囲を設定
ggplot(BOD, aes(Time, demand)) +
  geom_line() +
  ylim(0, max(BOD$demand))

# 離散値
# 変数がファクターの場合は各値のグループを明示的に設定する必要がある
# 　⇒ 設定がない場合、全ての値が独立した値と見做され、折れ線グラフでは表示できなくなる
# 縦軸の範囲を設定
ggplot(BOD1, aes(Time, demand, group=1)) +
  geom_line() +
  expand_limits(y=0)
~~~

* 点線グラフ  
~~~
# geom_pointを重ねる、縦軸を対数目盛に変換
ggplot(worldpop, aes(Year, Population)) +
  geom_line() +
  geom_point() +
  scale_y_log10()

# 点の重なりを無くす、shapeは点の形状を指定
ggplot(tg, aes(dose, length, shape=supp)) +
  geom_line(position=position_dodge(0.2)) +
  geom_point(size=4, position=position_dodge(0.2))

# fillは点の塗りつぶしを指定、colourは点の外形の色を指定、fillは塗りつぶしの色を指定
ggplot(tg, aes(dose, length, fill=supp)) +
  geom_line() +
  geom_point(size=4, shape=21, colour="darkred", fill="pink")
~~~

* 線の形状  
~~~
# linetypeで指定
ggplot(BOD, aes(Time, demand)) +
  geom_line(size=1, colour="blue", linetype="dashed")

# 色を指定
ggplot(tg, aes(dose, length, colour=supp)) +
  geom_line(size=0.5) +
  scale_colour_brewer(palette="Set1")
~~~

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
  guides(fill=guide_legend(reverse=T)) +
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





