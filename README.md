# groupstudy_Haskell  

#Learn You a Haskell for Great Good!を翻訳します。

##Pattern matching の復習

パターンマッチングとは、あるデータが従うべきパターンを指定して、そのパターンに従ってデータを分解するために使うものである。Haskellだと数値、文字、リスト、タプルなどの多くのデータ型でパターンマッチにつかうことができる。

例えば渡された数が7かどうかを調べる関数

    lucky :: (Integral a) => a -> String  
    lucky 7 = "LUCKY NUMBER SEVEN!"  
    lucky x = "Sorry, you're out of luck, pal!" 

そして1~5が渡されたかどうかを確認する関数
パターンマッチなしだとif/then/elseを使う羽目になるが…

    sayMe :: (Integral a) => a -> String  
    sayMe 1 = "One!"  
    sayMe 2 = "Two!"  
    sayMe 3 = "Three!"  
    sayMe 4 = "Four!"  
    sayMe 5 = "Five!"  
    sayMe x = "Not between 1 and 5"  

パターンマッチ使えば簡単に書けます。
最後のパターンを最初に持ってくると、常に"Not between 1 and 5"を表示する関数になってしまうので注意(他のパターンに落ちなくなってしまう)

そしてnの階乗をパターンマッチングを使えば再帰的に定義することができます。
0の階乗を1と定義し、nの階乗の関数を書くと、

    factorial :: (Integral a) => a -> a  
    factorial 0 = 1  
    factorial n = n * factorial (n - 1) 

このようになりますね。
パターンマッチングも失敗したりするんです。次のような関数がそれです。

    charName :: Char -> String  
    charName 'a' = "Albert"  
    charName 'b' = "Broseph"  
    charName 'c' = "Cecil" 

普通に動きそうだけど予期していない値が入力されると…

    ghci> charName 'a'  
    "Albert"  
    ghci> charName 'b'  
    "Broseph"  
    ghci> charName 'h'  
    "*** Exception: tut.hs:(53,0)-(55,21): Non-exhaustive patterns in function charName 

こんな感じでエラー吐いちゃいますよ。網羅的でないパターンとエラー吐いているので、パターンを作るときはすべてに合致するパターンを最後に入れておいてね。そうすればプログラムがクラッシュせずにすむからね。

##Guards, guards!
パターンマッチングはタプルで利用することができる。仮にあなたが2次元上の2つのベクトルを足し合わせる関数を作りましょう。2つのベクトルの足し算するとき、x成分とy成分をそれぞれ足し合わせます。ここでパターンマッチングを知らなかった場合こう書くだろう。

    addVectors :: (Num a) => (a, a) -> (a, a) -> (a, a)  
    addVectors a b = (fst a + fst b, snd a + snd b)

これでも動きますけど、でももっといい方法があるんですよ。パターンマッチングを使って書き換えていきますかね。

    addVectors :: (Num a) => (a, a) -> (a, a) -> (a, a)  
    addVectors (x1, y1) (x2, y2) = (x1 + x2, y1 + y2)

こっちの方がわかりやすいよね。パラメータとなる2つのペアが引数となるのは保障されてるってこと。
Fstとsndはペアの要素を引き抜くものですね。しかしトリプルの場合どうなる？そういうのはないけど独自で定義をすることは可能です。

    first :: (a, b, c) -> a  
    first (x, _, _) = x  
      
    second :: (a, b, c) -> b  
    second (_, y, _) = y  
      
    third :: (a, b, c) -> c  
    third (_, _, z) = z  

この_はどうでもいいので使い捨ての変数を表すために使ってますよ。

リスト内包表記でもパターンマッチが使えますよ。

     ghci> let xs = [(1,3), (4,3), (2,4), (5,3), (5,6), (3,1)]  
     ghci> [a+b | (a,b) <- xs]  
     [4,7,6,8,11,4] 

リスト内包表記のパターンマッチは失敗したら次の要素に進みます。

普通のリストもパターンマッチで使えますよ。空リスト「」、または：を含むパターンと合致させることができる。ただし、[1,2,3]は1:2:3:[]の構文糖衣(読み書きしやすいために導入される構文)ということである。x:xsのようなパターンは、リストの先頭要素をxに縛り付け、残りをxsに縛り付ける。リストの要素が1つならxsは空のリストですよ。


###ノート
**x:xsというパターンは特に再帰関数と一緒によく使われている。しかし:を含むパターンは長さが1以上のリストにのみ合致しません。**

**もし3つの先頭要素を縛り付けたいなら、x:y:z:zsと書けばできますよ。**

それじゃあ今覚えたパターンマッチを使って独自のhead関数を作りましょう。

    head' :: [a] -> a  
    head' [] = error "Can't call head on an empty list, dummy!"  
    head' (x:_) = x 

これをロードするとこう動きますよ。

    ghci> head' [4,5,6]  
    4  
    ghci> head' "Hello"  
    'H'

定義にあるように、複数の変数に束縛したいときは丸括弧で囲まないとシンタックスエラーを吐きますよ。

Errorという関数を使ってることにも気づいてください。Erroe関数は文字列を引数にとって、その文字列を使ってランタイムエラーを表示させる。これはプログラムをクラッシュさせるものなので、使うのはよろしくないですよ。まぁ空リストは受け取れないから多少はね？

そんじゃもう1つ、不便な英語形式であるリストの最初の要素を出力する関数を作ってみましょう

    tell :: (Show a) => [a] -> String  
    tell [] = "The list is empty"  
    tell (x:[]) = "The list has one element: " ++ show x  
    tell (x:y:[]) = "The list has two elements: " ++ show x ++ " and " ++ show y  
    tell (x:y:_) = "This list is long. The first two elements are: " ++ show x ++ " and " ++ show y 

この関数は空リスト、1つの要素、2つの要素、それ以上の要素をもつリストを管理しているので安全です(エラー吐かないよ)。(x:[])と(x:y:[])は、[x],[x,y]と書くこともできます。しかし(x:y:_)は角括弧を使って書き直すことはできない。このパターンは要素の長さが2以上のリストに合致するからやで。

私たちはすでにリストの内包表記を利用して、length関数を実装しました。それじゃ今からパターンマッチと再帰を少し使ってlength関数を作ってみましょう。

    length' :: (Num b) => [a] -> b  
    length' [] = 0  
    length' (_:xs) = 1 + length' xs  

さっき書いた階乗の関数と似てますね。まず空リストを0と定義します。次にリストを先頭と尾の部分に分解していき、リストを離れさせて2つのリストを取る。今私たちにとって先頭のリストが何かであるかはどうでもいいので、先頭を統一するために_を使う。また、私たちはリストのすべて可能なパターンを管理していたことに注意してください。最初のパターンが空リストと一致し、2つ目のパターンが空リストでないものと一致する。

例えば"ham"というリストを使ってlength'を呼び出したとき何が起きているかを確認しましょう。まず空リストであるかどうかを確認、まぁそんなことはありえないから次にパターンに行こう。次は先頭の要素を切り捨てることになり、長さは

`1 + length'  "am"`

となります。同じように続けていくと、

`1 + ( 1 + length' "m" )、1 + ( 1 + ( 1 + length' " " ) )`

となり、先頭要素が切り捨てられない空リストになるので、length' "" は0となる。
よって

`1 + ( 1 + ( 1 + 0) )`

となり、"ham"の長さは3となる。

次に和"sum"を実装してみましょう。空リストの合計が0であることはわかる。そして先頭部分と残りの部分の足し算がリストの合計であることもわかる。

    sum' :: (Num a) => [a] -> a  
    sum' [] = 0  
    sum' (x:xs) = x + sum' xs 

これをasパターンと呼ぶ。asパターンは、パターンに従って値を分解し、パターンマッチの対象になった値自信を参照したいときに使います。asパターンを作るときは、普通のパターンの前に名前と@を追加します。
例えば、xs@(x:y:ys)のようなasパターンを作りますよ。このパターンは、x:y:ysに合致するものとまったく同じものに合致しつつ、xsで元のリスト全体にアクセスすることもできる。x:y:ysとタイプしなくてもいいのよ。これが簡単な例です。

    capital :: String -> String  
    capital "" = "Empty string, whoops!"  
    capital all@(x:xs) = "The first letter of " ++ all ++ " is " ++ [x]

ロードしてみましょう

    ghci> capital "Dracula"  
    "The first letter of Dracula is D" 

私たちは関数本体に再び全体を利用する必要があるとき、大きなパターンと合致する際にパターンの繰り返しを避けるためにasパターンを使います。

あと1つあってですね、パターンマッチでは++は使えないんですよ。(++は2つのリストをつなげる演算子)　例えば(xs ++ ys)に対して合致させようとしても、リストのどの部分をxsに合致させて、どの部分をysに合致させればいいかわからないわけですよ。(xs ++ [x,y,z])や(xs ++ [x])というパターンなら、リストの性質から合致できそうですけど実際そんな賢い合致はできませんよ。

##Where!?
前のセクションで，私達はBMI計算関数を定義しましたがそれはこんな感じでした：


    bmiTell :: (RealFloat a) => a -> a -> String  
    bmiTell weight height  
        | weight / height ^ 2 <= 18.5 = "You're underweight, you emo, you!"  
        | weight / height ^ 2 <= 25.0 = "You're supposedly normal. Pffft, I bet you're ugly!"  
        | weight / height ^ 2 <= 30.0 = "You're fat! Lose some weight, fatty!"  
        | otherwise                   = "You're a whale, congratulations!" 


私達はここで3回も繰り返しを行っている事に気付いて下さい．私達は3回も(ry．プログラミング中に(三回も)繰り返すのは頭痛がする程望ましいです．つまり望ましくないです．私達は同じ式を三回も繰り返したけど，もし１回のみの計算でOKならばそれは理想的であるので，何か変数名を付けてやり，そしてそれを式の替わりに使いましょう．よし，私達は関数をこんな感じで修正できます：


    bmiTell :: (RealFloat a) => a -> a -> String  
    bmiTell weight height  
        | bmi <= 18.5 = "You're underweight, you emo, you!"  
        | bmi <= 25.0 = "You're supposedly normal. Pffft, I bet you're ugly!"  
        | bmi <= 30.0 = "You're fat! Lose some weight, fatty!"  
        | otherwise   = "You're a whale, congratulations!"  
        where bmi = weight / height ^ 2  


ガードの後にwhereっていうキーワードを置き(普通はガードの記号|がインデントされている個数分，whereもインデントしたほうがいいよ)，そしていくつかの変数とか関数を定義します．これらの変数はガード内から使用可能であり，そして繰り返しから逃れられます．もし少し違うBMIの計算方法に変えたいのなら，１回のみ変更すれば済みます．変数名を与えることで可読性も増し，プログラムの実行速度も上がります．もっと外に出せばこんな感じになります：


    bmiTell :: (RealFloat a) => a -> a -> String  
    bmiTell weight height  
        | bmi <= skinny = "You're underweight, you emo, you!"  
        | bmi <= normal = "You're supposedly normal. Pffft, I bet you're ugly!"  
        | bmi <= fat    = "You're fat! Lose some weight, fatty!"  
        | otherwise     = "You're a whale, congratulations!"  
        where bmi = weight / height ^ 2  
              skinny = 18.5  
              normal = 25.0  
              fat = 30.0


関数のwhere節で定義した変数名ははその関数のみでしか使えないので，他の関数の名前空間を汚す心配はしなくて大丈夫です．全ての変数の宣言はインデントが揃っている事に注意して下さい．もし，いい感じに一列に並べないと，Haskellは混乱し，ブロックの範囲が分からなくなってしまいます．

where節は同じ関数内でも，違うパターンマッチからは見ることが出来ません．もし，一つの関数内の幾つかのパターンマッチから変数にアクセスしたいのなら，グローバルに定義が必要です．

where節内部でもパターンマッチは可能です．先程の関数のwhere節をこんな感じで書き直す事が出来ます：


    ...  
    where bmi = weight / height ^ 2  
          (skinny, normal, fat) = (18.5, 25.0, 30.0)  


もう一つのかなりありふれた、ファーストネームとラストネームを受け取ってイニシャルを返す関数を作ってみましょう．


    initials :: String -> String -> String  
    initials firstname lastname = [f] ++ ". " ++ [l] ++ "."  
        where (f:_) = firstname  
              (l:_) = lastname


関数の引数部分で行うパターンマッチでもこれを実現する事はできますが(実際そっちのほうが短いし明らかですね)，でもこれはwhere節で同様の事ができるって見せているだけなのですよ．

ここまで定数をwhere節内で定義して来た様に，関数も定義できます．健康がテーマのプログラミングの続きで，体重と身長がペアのリストを受け取ってBMIを返す関数を作りましょう．


    calcBmis :: (RealFloat a) => [(a, a)] -> [a]  
    calcBmis xs = [bmi w h | (w, h) <- xs]  
        where bmi weight height = weight / height ^ 2


できました！この例でbmiを関数としたのは，calcBmisの引数から複数のBMIを計算するからです．身長と体重がペアになったリストを関数に渡し，それぞれのペアに基づいた異なったBMIのリストが返ってくるのを試してみましょう．


whereは入れ子構造にもできます．関数を作り，その関数内部のwhere節で「関数を助けるお助け関数1」を作り，そのお助け関数1のwhere節内部でもさらに「お助け関数1を助けるお助け関数2」を作る事ができます．


##Let it be 
let it beアルバムより、ジョージ・ハリスンの似顔絵ですが似てますね…w

whereと非常によく似たものでletというものがあります．whereは構文構造であり，関数の最後にて変数を束縛し，whereを用いた関数内ではガードを含めwhere節内部を見れました．letはどこであっても変数の束縛が可能であり，それ自身は式となります(ここからlet式と呼びます)．でも，宣言した変数は局所的な変数となり，ガード間での共有はできません．値を変数へ束縛するHaskellの構文と同様，let式でもパターンマッチングが使えます．実際に見てみましょう．次の関数は円柱の表面積を高さと半径から求める関数です．


    cylinder :: (RealFloat a) => a -> a -> a  
    cylinder r h = 
        let sideArea = 2 * pi * r * h  
            topArea = pi * r ^2  
        in  sideArea + 2 * topArea  


let式の書き方は「let <束縛> in <式>」です．let式で定義した変数はその後のin部分の<式>にてのみアクセスする事ができます．ご覧のとおり，これはwhereを使っても再現する事ができます．変数の宣言の際にインデントが揃っていることに注意して下さい．じゃぁ、whereとlet式の違いは何なんでしょう？この段階だと，let式は最初に束縛を行ってその後に式が来るという書き方で，whereはそれの逆に思えます．

let式とwhereの違いは，let式の場合だとそれ自身が式だと言うことです．whereは構文構造です．ifについて勉強した時，if〜elseは式であり，どこでも使えると説明された事を思い出して下さい．


    ghci> [if 5 > 3 then "Woo" else "Boo", if 'a' > 'b' then "Foo" else "Bar"]  
    ["Woo", "Bar"]  
    ghci> 4 * (if 10 > 5 then 10 else 0) + 2  
    42  


letでも同様です．


    ghci> 4 * (let a = 9 in a + 1) + 2  
    42  


ローカルスコープにて関数を作ることもできます．


    ghci> [let square x = x * x in (square 5, square 3, square 2)]  
    [(25,9,4)]


幾つもの変数を一行で束縛したいのなら，改行をしてインデントを揃えるという事はできません．そこでセミコロンを使って区切ります．


    ghci> (let a = 100; b = 200; c = 300 in a*b*c, let foo="Hey "; bar = "there!" in foo ++ bar)  
    (6000000,"Hey there!")  


「let a = 100; b = 200; c = 300 in a*b*c; in a*b*c」みたいに最後にセミコロンを付ける必要はないけど，したいならできます．以前言ったように，let式ではパターンマッチができます．次の例は，素早くタプルを分解し，変数に束縛する例です．


    ghci> (let (a,b,c) = (1,2,3) in a+b+c) * 100  
    600


letはリスト内包表記内でも使えます．先程の体重と身長がペアになったリストを受け取ってBMIのリストを返す関数を書きなおしてみましょう．


    calcBmis :: (RealFloat a) => [(a, a)] -> [a]  
    calcBmis xs = [bmi | (w, h) <- xs, let bmi = w / h ^ 2]  	


リスト内包表記内部でのlet式で束縛した変数は「|」の前部分と，束縛後の「,」の後ろでのみ使えます．つまり，「(w, h) <- xs」この部分では使えません．肥満な人のBMIのみを返す関数はこんな感じです．


    calcBmis :: (RealFloat a) => [(a, a)] -> [a]  
    calcBmis xs = [bmi | (w, h) <- xs, let bmi = w / h ^ 2, bmi >= 25.0] 


リスト内包表記内部にてlet式を用いた束縛を行った際，inは削りましたが，これは変数を使う範囲が既にそこで決められていたからです(リスト内でのみ使用すると分かっている)．しかしながら，私達はlet〜inによる束縛を決まった場所で行い，束縛した変数はその決まった場所でのみ使います．in部分はGHCi上での関数や変数の宣言の際も省略可能です．inを省略した場合，定義した名前はそれ以降の対話全体から見えるようになります．


    ghci> let zoot x y z = x * y + z  
    ghci> zoot 3 9 2  
    29  
    ghci> let boot x y z = x * y + z in boot 3 4 2  
    14  
    ghci> boot  
    <interactive>:1:0: Not in scope: `boot' 


let式による束縛がカッコいいという事なら，何故whereによる束縛の替わりにしないのでしょうか？それはですね，let式はそれ自体が式であり，そこでの束縛は局所的なものになりますし，ガードを跨いでの共有はできません．ある人達はwhereによる束縛を好み，それは関数で使用した名前が後に来るからという事が理由です．whereを使えば，関数の本体が関数の名前や引数の定義に近づきますし，可読性も上がります．


##Case expressions

CとかC++とかJavaと言った沢山の命令形言語はcase文を持っているので，こういった言語に触れたことがあるのならば，case分がどんなものか知っているかもしれません．caseは変数を取り，ある特定の値に対してのコードのブロックを実行し，当てはまらなかった場合のブロックも含みます．

Haskellのcase式もそのコンセプトを含み，かつさらに一歩前進したものとなっています．case式とあるように，case式はlet式やif式等と同様，式となっています．各値によって計算する式を変えられるのみで無く，パターンマッチングも使えます．うーーん，変数を受け取って，パターンマッチングを行って，それに伴ってコードを実行するって，かつて何処かで聞いたことありましたっけ？そうなんです，関数の定義の際の引数部分でのパターンマッチングがそれです！はい，それは実際の所，case式の糖衣構文となっています．以下の２つのコードは同じことをしています:


    head' :: [a] -> a  
    head' [] = error "No head for empty lists!"  
    head' (x:_) = x  


    head' :: [a] -> a  
    head' xs = case xs of [] -> error "No head for empty lists!"  
                      (x:_) -> x  


ご覧のとおり，case式の文法は凄い簡単です．


    case expression of pattern -> result  
                       pattern -> result  
                       pattern -> result  
                       ...  


「expression」がパターンに対応していきます．ここでのパターンマッチングの動作は期待通りのものであり，「expression」に一番最初にマッチしたものが使われます．もし全てのパターンを通りすぎても，マッチするものが見つからなかったら，ランタイムエラーが起こります．

関数の引数部分でのパターンマッチは関数の宣言時のみでしか使えませんが，case式ならどこでも使えます．こんな感じです：


    describeList :: [a] -> String  
    describeList xs = "The list is " ++ case xs of [] -> "empty."  
                                                   [x] -> "a singleton list."   
                                                   xs -> "a longer list."  


case式は何か式の途中でパターンマッチングを行う際に便利です．ちなみに上の例は次の様にもできます．


    describeList :: [a] -> String
    describeList xs = "The list is " ++ what xs
        where what [] = "empty."
              what [x] = "a singleton list."
              what xs = "a longer list."


##説明に使った幾つかの関数
where失敗な関数 whereFail.hs
    whereFail 1 = failMessage1
    whereFail 2 = failMessage2
    whereFail a = "number!"
        where failMessage1 = "fail1"
              failMessage2 = "fail2"

お助けな関数nest_where.hs
    関数 :: Int -> Int -> Int
    関数 a b = お助け関数1 a b
       where temp0 = 100
             お助け関数1 a' b' = お助け関数2 a' b' + temp0
                 where temp1 = 10
                       temp2 = 20
                       お助け関数2 a'' b'' = temp1 + temp2 + a'' + b''

リッチな？関数 rich.hs

    hoge 1 = tmp1
      where tmp1 = "aaa"
    
    hoge 2 = myfunc tmp
      where myfunc "fizz" = "fizz"
            myfunc "buzz" = "buzz"
            tmp = "fizz"
    
    hoge 3 =
      let foo = "foo"
          bar = "bar"
      in foo ++ bar
    
    hoge a
      | a == 10  = let aaa="aaa" in "10 " ++ aaa ++ " " ++ tmp
      | a == 20  = "20"
      | otherwise = "other" ++ tmp ++ tmp1 ++ case a of 21 -> " 21"
                                                        22 -> " 22"
                                                        otherwise -> " other"
      where tmp = "fuga"
            tmp1 = "moge"

