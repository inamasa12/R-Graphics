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


search(): ロード済みのパッケージを確認  



