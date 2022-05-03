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

    # グループ別の表示１（積み上げ、デフォルト）
    ggplot(df, aes(col1, col2, fill=col3)) +              # col3の値でfillを変える
        geom_col(position=position_stack(reverse=TRUE)) + # 積み上げを逆順にする
        guides(fill=guide_legend(reverse=TRUE))           # 凡例の表示を逆順にする

    # グループ別の表示２（100%積み上げ）
    ggplot(df, aes(col1, col2, fill=col3)) +              # col3の値でfillを変える
        geom_col(position="fill") +                       # position="fill"は100%積み上げ
        geom_text(aes(y=label_y, label=col2)) +           # 積み上げ棒グラフに値ラベル入れる場合は位置の列が必要
        scale_y_continuous(labels=scales::percent)        # スケールを100%表示

    # グループ別の表示３（並列）
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

    # グループ別の表示１（積み上げ、デフォルト）
    ggplot(df, aes(col1, col2, fill=col3)) +
        geom_area(colour=NA) +                              # 枠線が面の全てを囲まないようにするためNA
        scale_fill_brewer(palette="Blues") +
        geom_line(position="stack", colour="grey", size=.2) # 枠線

    # グループ別の表示２（100%積み上げ）
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

### R Tips  
as.integer(as.character(factor)): ファクターを表示の値で数値に変換する場合は先に文字列に変換しておく必要がある  
⇒ 直接数値に変換するとファクターのレベルに割り当てられた数値に変換されてしまうため  

---
　  

　  
## 第５章　散布図    
2つの連続変数の関係を表示  
geom_pointを使用  
1. 基本  
    ~~~
    # グループ別の表示１（離散変数でグルーピング）
    ggplot(df, aes(col1, col2, shape=col3, fill=col4)) +
        geom_point(size=2.5) +                              # 共通のサイズを指定
        scale_shape_manual(values=c(21, 24)) +              # グループ毎に形を指定
        scale_fill_manual(
            values=c(NA, "black"),                            # グループ毎に塗りつぶしを指定
            guide=guide_legend(override.aes=list(shape=21)))  # 凡例に表示する形を指定

    # グループ別の表示２（連続変数でグルーピング）
    ggplot(df, aes(col1, col2, fill=col3, size=col4)) +
        geom_point(shape=21) +             # 共通の形とサイズを指定
        scale_fill_gradient(               # 連続変数のスケールにグラデーションを使用
            low="black", high="white",       # 両端の色を設定
            breaks=seq(70, 170, by=20),      # 連続変数の目盛を設定
            guide=guide_legend()             # 凡例の表示を離散的にする
            ) +
        scale_size_area()                  # col4の値の大きさを点の面積に反映させる（デフォルトは半径） 
    ~~~
<img src="https://user-images.githubusercontent.com/51372161/157440331-d60154bf-81e1-4c93-b096-f64a4ff9aa07.png">  

2. オーバープロット１  
点の重なりを回避するための技法  
stat_bin2d、stat_binhex等の多次元ヒストグラムを使用  

    ~~~
    # 点に透明度を与える
    ggplot(df, aes(col1, col2)) +
        geom_point(alpha=.1)

    # ヒストグラム（長方形）を使用する
    ggplot(df, aes(col1, col2)) +
        stat_bin2d(bins=50) +
        scale_fill_gradient(low="lightblue", high="red", limits=c(0, 6000)) 

    # ヒストグラム（六角形）を使用する
    ggplot(df, aes(col1, col2)) +
        stat_binhex() +
        scale_fill_gradient(low="lightblue", high="red", limits=c(0, 6000))
    ~~~
<img src="https://user-images.githubusercontent.com/51372161/157861938-bd541bc8-fc83-4909-9aba-761600a72e20.png">  

3. オーバープロット２  
ジッターや箱ひげ図（geom_boxplot）を使用  

    ~~~
    # 散布図（ジッター）
    ggplot(ChickWeight, aes(Time, weight)) +
        geom_point(position=position_jitter(width=.5, height=0))

    # 箱ひげ図
    ggplot(ChickWeight, aes(Time, weight)) +
        geom_boxplot(aes(group=Time))
    ~~~
<img src="https://user-images.githubusercontent.com/51372161/157862654-dfe9eb2e-e2b8-47a7-a863-8672176edb0e.png">  

4. 傾向線１  
stat_smoothを使用  

    ~~~
    # グループ別の表示、注釈
    ggplot(df, aes(col1, col2, colour=col3)) +
        geom_point(shape=21, fill="white") + # 散布図
        stat_smooth(method=lm, se=FALSE, fullrange=TRUE) + # 線形モデルをフィッティング（信頼区間の表示なし、全データの範囲で予測値を外挿）
        annotate("text", x=16.5, y=52.5, label="f:r^2==0.43", parse=TRUE) + # 注釈を文字列の数式オブジェクトで与える場合
        annotate("text", x=16.5, y=51, label=expression(m:r^2==0.47)) # 注釈を直接数式オブジェクトで与える場合
  
    # 非線形
    ggplot(df, aes(col1, col2)) +
        geom_point(position=position_jitter(width=0.3, height=0.06)) +
        stat_smooth(method=glm, method.args=list(family=binomial)) # ロジスティック回帰
    ~~~
<img src="https://user-images.githubusercontent.com/51372161/158952530-b53b80d8-f4dc-4b00-a09d-de4162034591.png">  

5. 傾向線２  
別に作成したモデルの予測値をプロット  

    ~~~
    # 性別毎に線形モデルを構築
    mdls <- heightweight %>%
        nest(dt=!sex) %>%
        mutate(mdl=map(dt, ~lm(heightIn~ageYear, .))) %>%
        ungroup() %>%
        select(sex, mdl)

    # 各モデルの予測値を算出（データと列名を一致させる必要がある、predictvalsはモデルを入力とし予測値を出力する独自関数）
    preds <- mdls %>%
        mutate(pred=map(mdl, predictvals, xvar="ageYear", yvar="heightIn")) %>%
        select(sex, pred) %>%
        unnest(pred)

    # 性別毎に予測値をプロット
    ggplot(heightweight, aes(ageYear, heightIn)) +
        geom_point() + # データのプロット
        geom_line(data=preds) +　# 予測値（線）のプロット
        facet_grid(.~sex)
    ~~~
<img src="https://user-images.githubusercontent.com/51372161/158953994-939c4c01-51ff-4ef2-85b7-f05ae0182aa3.png">  

6. ラベリング１  
geom_textを使用  

    ~~~
    # 基本
    ggplot(df, aes(col1, col2)) +
        geom_point() +
        geom_text(aes(label=col3), size=3)

    # ラベルの位置を一律にシフト
    ggplot(df, aes(col1, col2)) +
        geom_point() +
        geom_text(aes(label=col3), 
            size=3, 
            hjust=0, # ラベルの左端を座標に合わせる
            position=position_nudge(x=80, y=-0.1)) # 一律にシフト
    ~~~
<img src="https://user-images.githubusercontent.com/51372161/159487864-9fd2eff7-401a-45dd-b91e-333f90d5ae80.png">  

7. ラベリング２  
ggrepelを使用  

    ~~~
    ggplot(df, aes(col1, col2)) +
        geom_point() +
        geom_text_repel(aes(label=col3), size=3)

    ggplot(df, aes(col1, col2)) +
        geom_point() +
        geom_label_repel(aes(label=col2), size=3)
    ~~~
<img src="https://user-images.githubusercontent.com/51372161/159678346-21232347-9d07-45e4-bc7c-033ebda6defe.png">  

8. その他１  
バルーンプロットによる情報付加  

    ~~~
    # バルーンプロット１
    ggplot(df, aes(col1, col2, size=col3)) +
        geom_point(shape=21, colour="black", fill="cornsilk") + # 散布図の設定
        scale_size_area(max_size=15) # 値を円の面積に反映させる（デフォルトは半径）

    # バルーンプロット２
    ggplot(df, aes(col1, col2)) +
        geom_point(aes(size=col3), shape=21, colour="black", fill="cornsilk") + # 散布図の設定
        scale_size_area(max_size=20, guide=FALSE) + # 値を円の面積に反映
        geom_text(aes(
            y=as.numeric(as.factor(col2))-sqrt(col3)/29, label=col3),
            vjust=1.3,
            colour="grey60",
            size=4) # ラベルを円の下端に表示
    ~~~
<img src="https://user-images.githubusercontent.com/51372161/159483255-a09c9e76-63e4-4559-80e8-43635e68f74d.png">  

9. その他２  
ラグによる情報付加（geom_rugの使用）  

    ~~~
    # ラグ（軸別の度数分布）
    ggplot(df, aes(col1, col2)) +
        geom_point() +
        geom_rug(position="jitter", size=0.2) # ラグの設定
    ~~~
<img src="https://user-images.githubusercontent.com/51372161/159483851-ef8ef5c2-6c8a-4378-b359-d1ef804eb2d3.png">  

10. その他３  
散布図行列  

    ~~~
    # 散布図行列
    pairs(df[, c(col1, col2, ...))
    ~~~
<img src="https://user-images.githubusercontent.com/51372161/159484089-3c454bb5-0e57-4e48-9be7-6cdf747c5fcf.png">  

### R Tips  
グラフ内で数式を用いる場合には特有の[数式オブジェクト](http://cse.naro.affrc.go.jp/takezawa/r-tips/r/56.html)を用いる  
数式オブジェクトの書式は`demo(plotmath)`でも確認可能  

---
　  

　  
## 第６章　データ分布の要約  

分布を可視化する方法  

1. ヒストグラム  
geom_histogramの使用  
ビンは下限値に閉じ、上限値に開いている  

    ~~~
    # ファセットで複数表示
    ggplot(df, aes(col1)) +
        geom_histogram(binwidth=150, fill="white", colour="black", boundary=750) + # ビンの幅を指定、boundaryはビンの左端
        facet_grid(col2~., scales="free") # 行指定（上下に並べる）
        facet_grid(~col2, scales="free")  # 列指定（左右に並べる）

    # 色を変えて複数表示
        ggplot(df, aes(col1, fill=col2)) +
        geom_histogram(bins=15, position="identity", alpha=0.4) # ビンの数を指定
    ~~~
<img src="https://user-images.githubusercontent.com/51372161/160262195-dc894cf3-a422-4529-b91b-1cb202a4c5e9.png">  

2. 密度曲線１  
geom_density、geom_line、geom_freqpolyを使用  

    ~~~
    # ヒストグラムと併せて表示（geom_density）
    ggplot(df, aes(col, ..density..)) + # 縦軸を百分比に統一
        geom_histogram(fill="cornsilk", colour="grey60", size=.2) + # ヒストグラムの表示
        geom_density(fill="blue", alpha=.1, colour=NA) + # 密度曲線の面を表示
        geom_line(stat="density") # 密度曲線の枠線を表示

    # 平滑化の度合別（geom_line）
    ggplot(df, aes(col)) +
        geom_line(stat="density") +
        geom_line(stat="density", adjust=.25, colour="red") +
        geom_line(stat="density", adjust=2, colour="blue")

    # 折れ線（geom_freqpoly）
    ggplot(df, aes(col)) +
        geom_freqpoly(binwidth=1, colour="blue")
    ~~~

<img src="https://user-images.githubusercontent.com/51372161/160624498-a6ff6bd0-18c0-4c2e-9f00-b59dfca59eda.png">  

3. 密度曲線２（複数）  
    ~~~
    # fill
    ggplot(df, aes(col1, fill=col2)) +
        geom_density(alpha=.3)

    # facet 
    ggplot      (df, aes(bwt, ..density..)) +
        geom_histogram(binwidth=200, fill="cornsilk", colour="grey60", size=.2) + # ヒストグラムを併せて表示
        geom_desity(alpha=.3) +
        facet_g rid(col2~.)
        lot      (df, aes(bwt, ..density..)) +
<img s      rc="https://user-images.githubusercontent.com/51372161/160827097-3c6bc74f-4582-4780-8e9c-9c88abb616f2.pn        g">  
        
4. 箱ひげ図  
geom_boxplotを使用  

    ~~~
    # 基本
    ggplot(df, aes(factor(col1), col2)) + # 数値をファクターとして扱う、x軸でグループ分けしない場合はx=1とする
        geom_boxplot(width=.5, outlier.size=1.5, outlier.shape=21)

    # 平均値を併せて表示
    ggplot(df, aes(factor(col1, col2)) +
        geom_boxplot() +
        stat_summary(fun="mean", geom="point", shape=23, fill="white") # 平均値
    ~~~
<img src="https://user-images.githubusercontent.com/51372161/160829503-e0a43845-eb60-40f2-b97b-3d496a0cce1b.png">  

5. バイオリンプロット  
デフォルトは各バイオリンの面積が等しくなるようにプロットされる  

    ~~~
    # 基本
    ggplot(df, aes(col1, col2)) +
        geom_violin(scale="count", adjust=.5) # バイオリンの面積をサンプル数に比例させる場合はscale="count"、adjustで平滑度をコントロール

    # 箱ひげ図を併せて表示
    ggplot(df, aes(col1, col2)) +
        geom_violin() +
        geom_boxplot(width=.1, fill="black", outlier=NA) +
        stat_summary(fun=median, geom="point", fill="white", shape=21, size=2.5)
    ~~~

<img src="https://user-images.githubusercontent.com/51372161/160831764-f0e83829-d015-473f-bf60-062b48ef4069.png">  

6. ドットプロット  
geom_dotplotを使用  

    ~~~
    # 箱ひげ図と重ねて表示
    ggplot(df, aes(col1, col2)) + # ドットプロット、箱ひげ図共通の設定
        geom_boxplot(outlier.colour=NA, width=.4) +
        geom_dotplot(binaxis="y", stackdir="center", fill=NA)　# stackdirでドットの積み上げ方を指定

    # 箱ひげ図と並べて表示
    ggplot(df1, aes(col1, col2)) +
        geom_boxplot(aes(as.numeric(col1)+0.2, group=col1), width=.25) + # 並べて表示するために横位置をずらす
        geom_dotplot(aes(x=as.numeric(col1)-.2, group=col1),
            binaxis="y",　# ビンをy軸方向に並べる
            binwidth=0.5, # ビンの幅を指定
            stackdir="center") +
        scale_x_continuous(breaks=1:nlevels(df$col1),
            labels=levels(df$col1))
    ~~~
<img src="https://user-images.githubusercontent.com/51372161/161200788-ff46e3b5-c86b-4c3a-b706-4fda569069e3.png">  


7. 二次元密度プロット  
stat_density2dを使用  

    ~~~
    # 等高線
    ggplot(df, aes(col1, col2)) +
    geom_point() + # 散布図と併せて表示
        stat_density2d(aes(colour=..level..))

    # 塗りつぶし
    ggplot(df, aes(col1, col2)) +
        # hで各軸の平滑化帯域幅を設定（大きいほど滑らかになる）
        stat_density2d(aes(fill=..density..), geom="raster", contour=F, h=c(.5, 5))

    # 透過度
    ggplot(df, aes(col1, col2)) +
        geom_point() + # 散布図と併せて表示
        stat_density2d(aes(alpha=..density..), geom="raster", contour=F)
    ~~~
<img src="https://user-images.githubusercontent.com/51372161/161356765-693c4cfa-8565-4dd3-8ddc-20b4e86dbc8a.png">  


### R Tips  
levels: カテゴリ変数のレベルを表示  
nlevels: カテゴリ変数の数を表示  

---
　  

## 第７章　注釈  
補助的要素の追加  

1. 注釈  
annotateを使用  
テキストの中心が指定した点に来るように表示される（デフォルトはhjust=0.5、vjust=0.5）  
hjust=0（1）でテキストの左端（右端）が、vjust=0（1）でテキストの下端（上端）が指定した点に来る  
    ~~~
    ggplot(faithful, aes(eruptions, waiting)) +
        geom_point() +
        annotate("text", x=-Inf, y=Inf, label="Upper left", hjust=0, vjust=1) +
        annotate("text", x=Inf, y=-Inf, label="Lower Right", hjust=1, vjust=0)
    ~~~
<img src="https://user-images.githubusercontent.com/51372161/161542218-a4fb6fca-61df-487e-9658-e3b5bb94d442.png">  

2. 補助線  
geom_hline、geom_vline、geom_ablineを使用  

    ~~~
    ggplot(df, aes(col1, col2, colour=col3)) +
        geom_point() +
        geom_hline(yintercept=60) +
        geom_vline(xintercept=14)

    ggplot(df, aes(col1, col2, colour=col3)) +
        geom_point() +
        geom_abline(intercept=37.4, slope=1.75)
    ~~~
<img src="https://user-images.githubusercontent.com/51372161/161755642-38d440a8-402b-439a-aafb-7a0a11b67ea3.png">  

3. 線分、網掛け  
annotateを使用  

    ~~~
    library(grid)
    ggplot(df, aes(col1, col2)) +
        geom_line() +
        # 矢印
        annotate("segment", x=1850, xend=1820, y=-0.8, yend=-.95, colour="blue", size=2, arrow=arrow()) +
        # 線分
        annotate("segment", x=1925, xend=1950, y=-.25, yend=-.25, arrow=arrow(ends="both", angle=90, length=unit(.2,"cm"))) +
        # 網掛け
        annotate("rect", xmin=1950, xmax=1980, ymin=-1, ymax=1, alpha=.1, fill="blue")
    ~~~
<img src="https://user-images.githubusercontent.com/51372161/161975566-89ae3ae3-1da3-4371-b24c-57b5748b36fa.png">  

4. エラーバー  
geom_errorbarを使用  

    ~~~
    # 棒グラフ
    ggplot(df, aes(col1, col2, fill=col3)) +
        geom_col(position="dodge") +
        # 棒グラフと同じだけエラーバーもずらす（col4にエラーが収録）
        geom_errorbar(aes(ymin=col2-col4, ymax=col2+colr), position=position_dodge(0.9), width=.2)

    # 折れ線グラフ
    ggplot(df, aes(col1, col2, colour=col3, group=col3)) +
        geom_errorbar(aes(ymin=col2-col4, ymax=col2+col4),
                        position=position_dodge(0.3), 
                        width=.2,
                        size=.25,
                        colour="black") +
        geom_line(position=position_dodge(0.3)) +
        geom_point(position=position_dodge(0.3), size=2.5)
    ~~~
<img src="https://user-images.githubusercontent.com/51372161/162194498-5d85a686-6587-4a4c-849d-e2f22a2a27db.png">  

5. ファセットに注釈を表示  
~~~
# データに対して数式となる文字列（Rフォーマット）を出力する関数を定義
lm_labels <- function(dat) {
  mod <- lm(hwy~displ, data=dat)
  # 回帰式
  formula <- sprintf("italic(y) == %.2f %+.2f * italic(x)",
                     round(coef(mod)[1], 2), round(coef(mod)[2], 2))
  r <- cor(dat$displ, dat$hwy)
  # 決定係数
  r2 <- sprintf("italic(R^2) == %.2f", r^2)
  data.frame(formula=formula, r2=r2, stringsAsFactors = F)

# グループ別のデータに関数を適用して数式を生成
labels <- mpg %>%
  select(drv, displ, hwy) %>%
  nest(dat=c(displ, hwy)) %>%
  mutate(label=map(dat, lm_labels)) %>%
  unnest(c(drv, label)) %>%
  select(drv, formula, r2)

# グループ別に表示
ggplot(mpg, aes(displ, hwy)) +
  geom_point() +
  geom_smooth(method=lm, se=F) +
  # 回帰式、ラベルデータを与える
  geom_text(data=labels, x=3, y=40, hjust=0, aes(label=formula), parse=T) +
  # 決定係数、ラベルデータを与える
  geom_text(data=labels, x=3, y=35, hjust=0, aes(label=r2), parse=T) +
  facet_grid(.~drv)
~~~
<img src="https://user-images.githubusercontent.com/51372161/162444661-5035e04e-4e86-4c93-8f90-84a5f18d4819.png">  

---
　  

## 第８章　軸  

1. 軸の反転、表示順の変更、スケール比の設定  
~~~
# 軸の反転、表示順の変更
ggplot(df, aes(col1, col2)) +
  geom_boxplot() +
  coord_flip() + # 軸の反転
  scale_x_discrete(limits=rev(levels(col1))) # 離散値の表示順はlimitsで指定できる
  scale_x_reverse() # 連続値の表示順の逆転
  xlim(6, 3) # 連続値は範囲の指定を逆にすることでも逆転できる

# スケール比の設定
ggplot(df, aes(col1, col2)) +
  geom_point() +
  coord_fixed(ratio=1/2) +
  scale_y_continuous(breaks=seq(0, 420, 30)) +
  scale_x_continuous(breaks=seq(0, 420, 15)) +
~~~
<img src="https://user-images.githubusercontent.com/51372161/163175296-b33e8d25-eba0-4a2d-b400-a3b47e72e53d.png">  

2. 軸の範囲（limits）  
~~~
# データ範囲の指定、指定の範囲だけを用いてグラフを作成
ggplot(df, aes(col1, col2)) +
  geom_boxplot() +
  ylim(0, 6)
  expand_limits(y=0) # 範囲を一方向に拡張
  scale_y_continuous(limits=c(0, 6), breaks=NULL) # 複数の設定が可能（breaksは目盛り線の設定）

# 表示範囲の指定、全データを用いて作成したグラフの表示範囲を指定
ggplot(df, aes(col1, col2)) +
  geom_boxplot() +
  coord_cartesian(ylim=c(5, 6.5))
~~~
<img src="https://user-images.githubusercontent.com/51372161/162552176-0bc93c82-c175-493e-8674-7bf29f13dc13.png">  


3. 目盛の表示設定（breaks: 目盛線、labels: 目盛ラベル）  
目盛線と目盛ラベルの表示内容はscaleで設定する（補助目盛線は目盛線の半分の幅で自動的に引かれる）  
scalesパッケージで提供されているフォーマッタ  

   |フォーマッタ|機能|  
   |---|---|  
   |comma|桁区切り|  
   |dollar|ドルマーク|  
   |percent|パーセント表示|  

    ~~~
    # 目盛線と目盛ラベルの設定
    ggplot(df, aes(col1, col2)) +
        geom_point() +
        scale_y_continuous(breaks=c(50, 56, 60, 66, 72),
                             labels=c("Tiny", "Really\nshort", "Short", "Medium", "Tallish"))
        scale_y_continuous(breaks=NULL) # 目盛線を非表示にする場合

    # 関数（フォーマッタ）を用いた表示単位の変換
    # 変換関数の設定
    footinch_formatter <- function(x) {
        foot <- floor(x/12)
        inch <- x %% 12
        return(paste(foot, "'", inch, "\"", sep=""))
    }

    # ラベルに作成した変換関数を設定する
    ggplot(df, aes(col1, col2)) +
        geom_point() +
        scale_y_continuous(labels=footinch_formatter)
    ~~~
<img src="https://user-images.githubusercontent.com/51372161/162736856-ad60899f-0ca5-4bab-b58f-329efe500913.png">  


4. 目盛ラベルの体裁、目盛記号の設定  
目盛ラベルの体裁、目盛記号の設定はthemeで行う  

    ~~~
    # 目盛ラベルと目盛記号の非表示
    ggplot(df, aes(col1, col2)) +
        geom_boxplot() +
        theme(axis.ticks=element_blank(), axis.text.y=element_blank())

    # 目盛ラベルの体裁変更
    ggplot(df, aes(col1, col2)) +
        geom_boxplot() +
        scale_x_discrete(
            breaks=c("fact1", "fact2", "fact3"),
            labels=c("Control", "Treatment 1", "Treatment 2")) +
        # hjust(vjust)=1は右（上）揃え、relでデフォルトからの倍率で文字サイズを指定する
        theme(axis.text.x=element_text(angle=30, hjust=1, vjust=1, size=rel(0.8)))
    ~~~
<img src="https://user-images.githubusercontent.com/51372161/162975056-cc1d80e2-e00e-4887-833c-e5e1daafa23e.png">  

5. 軸タイトル、軸線、枠線の設定  
~~~
# 軸タイトルの設定
ggplot(df, aes(col1, col2, colour=col3)) +
  geom_point() +
  labs(x="Age in years")
  scale_x_continuous(name="Age in years")

# 軸タイトルの書式設定
ggplot(df, aes(col1, col2)) +
  geom_point() +
  theme(axis.title.x=element_text(angle=30, face="italic", colour="darkred", size=14))
  theme(axis.title.x=element_blank()) # 軸タイトルを表示しない

# 軸線、枠線の設定
ggplot(df, aes(col1, col2)) +
  theme_bw() +
  geom_point() +
  theme(axis.line=element_line(colour="black"), # 軸線の設定
        panel.border=element_blank() # 枠線を非表示
        )
~~~
<img src="https://user-images.githubusercontent.com/51372161/163387635-bbd9f303-3813-4416-8a01-9f609a7d87fb.png">  


6. 対数軸の設定  
~~~
# 常用対数
ggplot(df, aes(col1, col2, label=col3)) +
  geom_text(size=3) +
  annotation_logticks() +                                            # 対数軸目盛を追加
  scale_x_log10(breaks=10^(-1:5),                                    # 目盛線の位置を指定
                minor_breaks=10^(-1:5)/2,                            # 補助目盛線の位置を指定
                labels=trans_format("log10", math_format(10^.x))) +  # ラベルの書式設定
  scale_y_log10(breaks=10^(0:3),
                labels=trans_format("log10", math_format(10^.x))) +
  theme_bw()

# それ以外
ggplot(df, aes(col1, col2, label=col3)) +
  geom_text(size=3) +
  scale_x_continuous(trans=log_trans(),
                     breaks=trans_breaks("log", function(x) exp(x)),
                     labels=trans_format("log", math_format(e^.x))) +
  scale_y_continuous(trans=log2_trans(),
                     breaks=trans_breaks("log2", function(x) 2^x),
                     labels=trans_format("log2", math_format(2^.x)))
~~~
<img src="https://user-images.githubusercontent.com/51372161/163659322-babdf2d8-d470-41ca-bf86-655fbc49bec0.png">  

7. 極座標  
coord_polarを使用  
    ~~~
    ggplot(df, aes(col1, fill=col2)) +
        geom_histogram(binwidth=15, boundary=-7.5, colour="black", size=.25) +
        coord_polar() +
        scale_x_continuous(limits=c(0, 360),
                     breaks=seq(0, 360, by=45),
                     minor_breaks = seq(0, 360, by=15))
    ~~~
<img src="https://user-images.githubusercontent.com/51372161/163695999-f1138af6-7762-4df8-8a21-78c80159e04d.png">  

8. 日付軸  
scale_x_dateを使用  
    ~~~ 
    ggplot(econ_mod, aes(date, psavert)) +
        geom_line() +
        scale_x_date(breaks=seq(as.Date("1992-6-1"), as.Date("1993-6-1"), by="2 month"),
                       labels=date_format("%Y / %m")) + # 独自関数（フォーマッタ）の使用も可能
        theme(axis.text.x=element_text(angle=30, hjust=1))
    ~~~

    |日付オプション|説明|  
    |---|---|  
    |%Y|年(YYYY)|  
    |%y|年(YY)|  
    |%m|月(MM)|  
    |%d|日(DD)|  
    |%a|曜日|  
    
<img src="https://user-images.githubusercontent.com/51372161/163806214-2fe970f5-b174-4330-b6bd-6394f04cc50e.png">  



### R Tips  
floor(x): xを越えない最大の整数  
round(x): 独自の基準で少数以下を丸める  
time(ts): タイムシリーズオブジェクトの時間のベクトルを出力する  
%+%: `グラフオブジェクト %+% df_new` でグラフで使用するデータセットを置き換える  
?strptime: 日付フォーマットオプションの一覧  

---
　 

## 第９章　グラフの全体的な体裁  

1. 各種テキストの書式設定    
~~~
# 軸タイトル
ggplot(df, aes(col1, col2)) +
  geom_point() +
  theme(axis.title.x=element_text(size=16, 
                                  lineheight=.9, # 行間隔を設定
                                  family="Times", 
                                  face="bold.italic", # 太字、イタリック等の文字効果
                                  colour="red"))

# グラフタイトル
ggplot(df, aes(col1, col2)) +
  geom_point() +
  labs(title="Age and Height\nof Schoolchildren") +
  theme(plot.title=element_text(size=16, lineheight=.9,
                                  family="Times", face="bold.italic",
                                  colour="red"))

# 凡例ラベル
ggplot(df, aes(col1, col2)) +
  geom_point(aes(colour=col3)) +
  theme(legend.text=element_text(family="Times", face="bold.italic", colour="red"))

# ファセットラベル
ggplot(df, aes(col1, col2)) +
  geom_point() +
  facet_wrap(~col3) +
  theme(strip.text.x=element_text(family="Times", face="bold.italic", colour="red"))
~~~
<img src="https://user-images.githubusercontent.com/51372161/164014806-d4207cd6-24fc-4cdd-8593-bfc34ec2b1ec.png">  

2. テーマ設定  
デフォルトのtheme_greyの他に幾つか既成のテーマが存在する  
    ~~~
    ggplot(df, aes(col1, col2)) +
        geom_point() +
        theme_classic()
        theme_minimal()
        theme_bw()
        theme_grey() # デフォルト
    ~~~

3. 各種オブジェクトの書式設定例  
~~~
# プロット領域（panel）
ggplot(df, aes(col1, col2, colour=col3)) +
    geom_point() +
    theme(panel.grid.major=element_line(colour="red"), # 主目盛線
            panel.grid.minor=element_line(colour="red", linetype="dashed", size=0.2), # 補助目盛線
            panel.background=element_rect(fill="lightblue"), # 背景
            panel.border=element_rect(colour="blue", fill=NA, size=2)) # 枠線

# 凡例（legend）
ggplot(df, aes(col1, col2, colour=col3)) +
    geom_point() +
    theme(legend.background=element_rect(fill="grey85", colour="red", size=1),
            legend.title=element_text(colour="blue", face="bold", size=14), # タイトル
            legend.text=element_text(colour="red"), # ラベル
            legend.key=element_rect(colour="blue", size=0.25)) # 各要素の枠線

# 軸（axis）、プロット全体（plot）
ggplot(df, aes(col1, col2, colour=col3)) +
    geom_point() +
    theme(axis.title.x=element_text(colour="red", size=14),
            axis.text.x=element_text(colour="blue"),
            axis.title.y=element_text(colour="red", size=14, angle=90), # タイトル
            axis.text.y=element_text(colour="blue"),
            plot.title=element_text(colour="red", size=20, face="bold"))　

# ファセット（strip）
ggplot(df, aes(col1, col2, colour=col3)) +
    geom_point() +
    facet_grid(col3~.) +
    theme(strip.background=element_rect(fill="pink"),
            strip.text.y=element_text(size=14, angle=-90, face="bold"))
~~~

### R Tips  
書式の拡張  
~~~
library(extrafont)
font_import()
loadfonts(device="win")
windowsFonts()　# 出力される表記でfamilyを設定すれば良い
~~~

---
　  

## 第１０章　凡例  

1. 凡例の位置、書式  
~~~
# 凡例の位置、書式
ggplot(df, aes(col1, col2, fill=col3)) +
  geom_boxplot() +
  theme(legend.position="bottom") # 既定のポジション選択
  theme(legend.position=c(1, 1),  # 中心位置の設定
        legend.justification=c(1, 1), # 中心位置に合わせる凡例領域の設定
        legend.background=element_rect(fill="white", colour="black"), # 背景色
        legend.key=element_blank()) # 項目別の境界線

# 凡例を表示しない
ggplot(df, aes(col1, col2, fill=col3)) +
  geom_boxplot() +
  guides(fill=F)
  scale_fill_discrete(guide=F)
  theme(legend.position="none")
~~~
<img src="https://user-images.githubusercontent.com/51372161/164814291-e58e8454-e0b8-462d-b618-0af77db4cb26.png">  

2. 凡例の表示順  
~~~
# 凡例の表示順（離散値のデータ範囲を望みの順序で設定する）
ggplot(df, aes(col1, col2, fill=col3)) +
  geom_boxplot() +
  scale_fill_grey(start=0, end=1, limits=c("trt1", "trt2", "ctrl"))
  
# 表示順の反転
ggplot(df, aes(col1, col2, fill=col3)) +
  geom_boxplot() +
  guides(fill=guide_legend(reverse=T))
  scale_fill_discrete(guide=guide_legend(reverse=T))
~~~

3. 凡例タイトル  
~~~
# 非表示の場合はNULLを与える
ggplot(df, aes(col1, col2, fill=col3)) +
  geom_boxplot() +
  labs(fill="Condition")
  scale_fill_discrete(name="Condition")
  guides(fill=guide_legend(title="Condition"))
~~~
<img src="https://user-images.githubusercontent.com/51372161/164814803-ce19f187-3d56-43e1-8f2f-9c71c7b30c60.png">  

4. 凡例ラベル  
~~~
# ラベルの設定
ggplot(df, aes(col1, col2, fill=col3)) +
  geom_boxplot() +
  scale_fill_discrete(labels=c("Control", "Treatment 1", "Treatment 2"))
  
# ラベルの書式設定
ggplot(df, aes(col1, col2, fill=col3)) +
  geom_boxplot() +
  theme(legend.text=element_text(
    face="italic",
    family="Times New Roman",
    colour="blue",
    size=14),
    legend.key.height=unit(1, "cm")) # 各要素の表示幅
~~~
<img src="https://user-images.githubusercontent.com/51372161/164951985-c4667937-5cf0-4c2e-8a8d-8b1723ee7dbb.png">  

---
　  

## 第１１章　ファセット  

サブプロットによる表示  

1. gridとwrap  
~~~
# gridは行~列で指定
ggplot(df, aes(col1, col2)) +
  geom_point() +
  facet_grid(col3~col4,
             labeller=label_parsed, # ラベルとして数式オブジェクトを用いる場合
             scales="free_y") # Y軸方向のスケール行毎に変える

# wrapは列方向にサブプロットを並べる
ggplot(df, aes(col1, col2)) +
  geom_point() +
  facet_wrap(~col3, ncol=4, # 列の数
             labeller=label_both) # 変数名も併せて表示する
~~~
<img src="https://user-images.githubusercontent.com/51372161/164952590-c264d9e7-b803-46f4-a6ad-5ec226be21e5.png">  

2. ファセットの書式設定  
~~~
ggplot(df, aes(col1, col2)) +
  geom_col() +
  facet_grid(.~col3) +
  theme(strip.text=element_text(face="bold", size=rel(1.5)), # ラベルの書式
        strip.background=element_rect(fill="lightblue", colour="black", size=1)) # プロット領域の書式
~~~
<img src="https://user-images.githubusercontent.com/51372161/164952642-dda57865-be95-492e-9066-eef895645fb7.png">

### R Tips  
ファセットのラベル名変更はデータ自体を変更しるしかない  

---
　  

## 第１２章　色を使う  

1. 特殊な色使い  
~~~
# viridis
ggplot(df, aes(col1, col2, fill=col3)) +
  geom_area() +
  scale_fill_viridis_d()          # 離散変数１
  scale_fill_viridis(discrete=T)  # 離散変数２
  scale_fill_viridis_c()          # 連続変数１
  scale_fill_viridis(discrete=F)  # 連続変数２

# brewer
ggplot(df, aes(col1, col2, fill=col3)) +
  geom_area() +
  scale_fill_brewer(palette="Oranges")
  
# グレースケール
ggplot(df, aes(col1, col2, colour=col3)) +
  geom_point() +
  scale_colour_grey(start=0.7, end=0)
~~~
<img src="https://user-images.githubusercontent.com/51372161/165509013-6e5f6e49-fa3f-45e2-8343-554a31754d62.png">  

2. 輝度  
~~~
ggplot(heightweight, aes(ageYear, heightIn, colour=sex)) +
  geom_point() +
  scale_color_discrete(l=45) # 0～100で指定
~~~
<img src="https://user-images.githubusercontent.com/51372161/165509402-c5d33de4-ed13-4a57-a126-262235b8940e.png">  

3. グラデーション  
~~~
# 離散値
ggplot(df, aes(col1, col2, colour=col3)) +
  geom_point() +
  scale_color_manual(values=c("red", "blue"))
  scale_color_manual(values=c(m="red", f="blue")) # 変数指定
  # #RRGGBB形式（RR: 赤色の強さ、GG: 緑色の強さ、BB: 青色の強さを二桁の16進数）で指定
  scale_color_manual(values=c("#440a54FF", "#FDE725FF")) 

# 連続値
ggplot(df, aes(col1, col2, colour=col3)) +
  geom_point(size=3) +
  #scale_color_gradient(low="black", high="white")                        # 二色
  scale_color_gradient2(low=muted("red"), 
                        mid="white",
                        high=muted("blue"), # mutedで彩度を落とす
                        midpoint=110)                                     # 三色
  scale_color_gradientn(colors=c("darkred", "orange", "yellow", "white")) # 四色以上
~~~
<img src="https://user-images.githubusercontent.com/51372161/166080408-8f70b07b-5b5c-4947-87c1-fbda51836791.png">  

### R Tips  
ggplotの色名一覧  
<img src="https://user-images.githubusercontent.com/51372161/165893106-b792eee0-006c-4eaa-ac5d-8dbfc951887b.png">  
ColorBrewerパレット`RColorBrewer:display.brewer.all()`
<img src="https://user-images.githubusercontent.com/51372161/165894132-2a1f35b3-4659-4da6-92e8-6bc58f072130.png">  

---
　  

## 第１３章　さまざまなグラフ  

1. 相関行列  
corrplotパッケージを使用  
~~~
# グラデーション設定
col <- colorRampPalette(c("#BB4444", "#EE9988", "#FFFFFF", "#77AADD", "#4477AA"))

# 相関行列の表示
corrplot(mcor, 
         method="shade",      # 相関行列のタイプ設定
         shade.col=NA,        # 府の値に対する斜線の有無
         tl.col="black",      # ラベルの色
         tl.srt=45,           # x軸ラベルの角度
         addCoef.col="black", # 値の表示色
         cl.pos="n",          # 凡例の有無
         order="AOE",         # 変数の順序
         col=col(200)         # カラーパレット
         )
~~~
<img src="https://user-images.githubusercontent.com/51372161/166126461-31b39118-8ca4-4114-8fff-b74c27101d84.png" width="500px">  


2. 関数プロット  
デフォルトのデータポイント数は101  
~~~
ggplot(data.frame(x=c(-3, 3)), aes(x)) +
  stat_function(fun=dt,          # 関数
                geom="line",     # 幾何学オブジェクト
                args=list(df=2), # 関数に与えるパラメータ 
                n=200)           # データポイント数
~~~

3. ネットワークグラフ（ggplotではない）  
~~~
library(igraph)
g <- graph.data.frame(df, directed=T) # グラフオブジェクトの作成（ペアのデータから）
par(mar=c(0, 0, 0, 0))

# グラフ関数で設定
plot(g, 
     layout=layout.fruchterman.reingold, # ネットワークの種類
     vertex.size=5,                      # ノードのサイズ
     edge.arrow.size=0.4,                # 矢印のサイズ
     vertex.label=V(g)$name,             # 各ノードのラベル
     vertex.label.cex=1.0,               # ノードのサイズ
     vertex.label.dist=0.4,              # ラベルの点からのオフセット
     vertex.label.color="black")         # ラベルの色

# グラフオブジェクトに設定することもできる
V(g)$size <- 4                           # ノードのサイズ
V(g)$label <- V(g)$name                  # 各ノードのラベル
E(g)[c(2, 11, 19)]$label <- "M"          # 特定の辺のラベル
E(g)$color <- "grey70"                   # 各辺の色
E(g)[c(2, 11, 19)]$color <- "red"        # 特定の辺の色
g$layout <- layout.fruchterman.reingold  # ネットワークの種類
plot(g)
~~~
<img src="https://user-images.githubusercontent.com/51372161/166391080-41ad36d3-3a09-4fcd-98dc-d9097ed8a531.png">  









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

### R Tips  
scale(df): 各列を正規化  
merge(tbl1, tbl2, by.x=BMONTH, by.y=TMONTH): 結合（join）と同じ  
クロージャー（関数を含んだ関数オブジェクト）作成関数  
~~~
# 関数の出力範囲に制限するクロージャーを出力
clfunc <- function(fun, min, max) {
  function(x) {
    y <- fun(x)
    y[x<min|x>max] <- NA
    return(y)
  }
}
~~~
ネットワークの種類: `?igraph::layout`




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

