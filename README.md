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





* Tips  
グループ化は文字列かファクターで行い、数値の場合はファクターに変更してから使用する  
shape: 0～20はcolourだけ指定、21～25はfillも指定  


