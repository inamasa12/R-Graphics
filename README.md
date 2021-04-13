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
# 塗りつぶしでグループ化、塗りつぶしの色設定変更
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

* Tips  
RColorBrewer::display.brewer.all(): R Color Brewerの全パレット表示  
top_n(n, col): 上位n個のデータを抽出  
reorder(col, col2): col1の順序をcol2の順序に設定
