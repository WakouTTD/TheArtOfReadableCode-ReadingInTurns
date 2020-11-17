---
marp: true
header: 'リーダブルコード輪読会　５・６章'
footer: 'The Art of Readable Code - Simple and Practical Techniques for Writing Better Code -'

---
<!-- $theme: gaia -->
<!-- $size: 16:9 -->
<!-- page_number: true -->
<!-- paginate: true -->
<!-- backgroundColor: floralwhite -->

# リーダブルコード輪読会　５・６章

***WakouTTD***

![bg right:40%](https://images-na.ssl-images-amazon.com/images/I/814plqPaZNL.jpg)

---

## リーダブルコード

ー より良いコードを書くためのシンプルで実践的なテクニック ー
  Dustin Bosewell, Trevor Foucher 著
  角 征典　訳

- [2018年12月12日時点で10万部](https://twitter.com/kdmsnr/status/1072677018335145984?s=20)

- [日本ITエンジニアの人口が大体100万人](https://www.meti.go.jp/policy/it_policy/jinzai/gaiyou.pdf)

図書館や会社で購入されていることを踏まえると、半数くらいは読んでいるかもしれない？

---

### 目次

```text
- 5章　コメントすべきことを知る
    - 5.1　コメントするべきでは「ない」こと
        - コメントのためのコメントをしない
        - ひどい名前はコメントをつけずに名前を変える
    - 5.2　自分の考えを記録する
        - 「監督のコメンタリー」を入れる
        - コードの欠陥にコメントをつける
        - 定数にコメントをつける
    - 5.3　読み手の立場になって考える
        - 質問されそうなことを想像する
        - ハマりそうな罠を告知する
        - 「全体像」のコメント
        - 要約コメント
    - 5.4　ライターズブロックを乗り越える
    - 5.5　まとめ
```

---

```text
コメントの目的は、「コードの動作を説明する」ことだと思っているかもしれない。
でもそれは、目的のごく一部でしかない。
```

たとえば、OSSのフレームワークのようなコードであれば、git上に「コードの動作を説明する」コメントのみで良いように思うが、プロダクトコードとしては、動作の説明のみではないように思う。

```text
コメントの目的は、書き手の意図を読み手に知らせることである。


コメントを読むとその分だけコードを読む時間がなくなる。
コメントは画面を占領してしまう。
```

---

コメントには価値が無い例

```java
// The class definition for Account
class Account {
   public:
   // Constructor
   Account();
   // Set the profit member to a new value
   void SetProfit(double profit);
   // Return the profit from this Account
   double GetProfit();
};
```

```text
コードからすぐにわかることをコメントに書かない。
```

---

価値がある例

```python
# remove everything after the second '*'
name = '*'.join(line.split('*')[:2])
```

---

コメント改善

改善前
```java
// Find the Node in the given subtree, with the given name, using the given depth.
Node* FindNodeInSubtree(Node* subtree, string name, int depth);
```

改善後

```java
// Find a Node with the given 'name' or return NULL.
// If depth <= 0, only 'subtree' is inspected.
// If depth == N, only 'subtree' and N levels below are inspected.
Node* FindNodeInSubtree(Node* subtree, string name, int depth);
```

・・・引数名と返り値の型がわかってるから、Findだけでも良さそうかも

---

ひどい名前はコメントをつけずに名前を変える

```java
// Enforce limits on the Reply as stated in the Request,
// such as the number of items returned, or total byte size, etc.
void CleanReply(Request request, Reply reply);
```

このコメントはCleanの意味をわかりやすく説明しているだけだ。
制限を課す(enforce limit)という言葉を関数名に入れた方が良い。

```java
// Make sure 'reply' meets the count/byte/etc. limits from the 'request'
void EnforceLimitsFromRequest(Request request, Reply reply);
```

この関数名は`自己文書化` “self-documenting.” されている

---

改善前

```java
// Releases the handle for this key. This doesn't modify the actual registry.
void DeleteRegistry(RegistryKey* key);
```

改善後　`自己文書化` “self-documenting.”

```java
void ReleaseRegistryHandle(RegistryKey* key);
```

たしかにコメント考えてて、実コードの方を変えた方が良いねと思うことは多い。

---

### good code > bad code + good comments.

### 優れたコメントというのは考えを記録する

```java
//このデータだとハッシュテーブルよりもバイナリーツリーの方が40%早かった。
//左右の比較よりもハッシュの計算コストのほうが高いようだ。
```

```java
//ヒューリスティックだと単語が漏れることがあるが仕方ない。100%は難しい。
```

※ヒューリスティック(heuristic)
必ず正しい答えを導けるわけではないが、ある程度のレベルで正解に近い解を得ることができる方法

私も何か調査したり、検討したりしたものはコメントいれます。

---

誰かに修正を促すコメント例

```java
// このクラスは汚くなってきている。
// サブクラス'ResourceNode'を作って整理したほうがいいかもしれない。
```

私だったら、下記のようにコメントにTODOつけて、技術的負債としてのチケット切ります。

```java
// TODO サブクラス'ResourceNode'を作って整理したほうがいいかもしれない。
```

---

と、思ったら、ちょうど次が

```java
// TODO: もっと高速なアルゴリズムを使う
```

```java
// TODO(ダスティン): JPEG以外のフォーマットに対応する
```

- プログラマがよく使う記法

|  記法 | 典型的な意味 |
| ---- | ---- |
| TODO:  | あとで手をつける |
| FIXME:  | 既知の不具合があるコード |
| HACK:  | あまり綺麗じゃない解決策 |
| XXX:  | 危険！　大きな問題がある |

---

```text
大きな問題は大文字TODO
小さな問題は小文字todoや、maybe-later(あとで直す)を使うなど。
```

IDEでは、TODO以外の記法でリスト表示する機能を見た記憶ないです。
javaだと@deprecatedで、非推奨のコードを使用を知らせる機能があります。

---

### 定数のコメント

どういう意図で8という定数にしたかを記録している良い例

```
NUM_THREADS = 8 # 値は「>= 2 * num_processors
```

```java
// 現実的な限界値。人間はこんなに読めない。
const int MAX_RSS_SUBSCRIPTIONS = 1000;
```

```
image_quality = 0.72; // 0.72ならユーザはファイルサイズと品質の面で妥協できる。
```

---

なかには名前が明確なのでコメントが必要ない定数もある

```java
SECOND_PER_DAY = 86400 # 60 sec/min * 60 min/hr * 24 hr/day
```

そういえば、cccmkに参画して間も無い頃に
`SLEEP_TIME`という定数を見つけて`INTERVAL_SECONDS`にリファクタしたことが有ります。

---

### 5.3 読み手の立場になって考える

```text
本書で扱っている技法は、他の人にコードがどのように見えるかを想像するものだ。
他の人というのは、プロジェクトのことを君のように熟知していない人のことである。
```

- 質問されそうなことを想像する

---

```text
どうして単純にdata.clear()せずに空のvectorをスワップするんだ？
```

```java
struct Recorder {
 vector<float> data;
 ...
 void Clear() {
 vector<float>().swap(data); // Huh? Why not just data.clear()?
 }
};
```

```text
ベクタのメモリを開放してメモリアロケータに戻す方法がこれしかないからだ。
```

---

```text
ここでコメントをつけるべきなのだ。
```

```java
// ベクタのメモリを開放する(「STL swap技法」で検索してみよう)
vector<float>().swap(data);
```

私も長い説明があった方が良いなぁと思ったら、ドキュメントツール(Confluenceとか、jiraのチケット、esa)のURLを貼ったりすることあります。

---

### ハマりそうな罠を告知する

```text
このコードを見て、びっくりすることは何だろう？どんなふうに間違えて使う可能性があるだろう？
```

`ユーザにメールを送信するコード例`

```java
void SendEmail(string to, string subject, string body);
```

```text
この関数の実装では、外部のメールサービスに接続している。
その接続には１秒かかる。このことを知らないウェブアプリケーション開発者が、
HTTPリクエストの処理中に誤ってこの関数を呼び出してしまうかもしれない。
メールサービスがダウンしていると、ウェブアプリケーションがハングしてしまう。
このような不幸を防ぐには、「実装の詳細」についてコメントを書くべきだ。
```

```java
// メールを送信する外部サービスを呼び出している(1分でタイムアウト)
void SendEmail(string to, string subject, string body);
```
---

`閉じタグのない壊れたHTMLを修復するための関数`

```java
def FixBrokenHtml(html): ...
```

```text
間違いなく動くが、タグのネストが深いと処理に時間がかかる。
HTMLによっては、何分もかかることもある。
ユーザーが使った「あと」で気づくよりも、使う「前」に告知するほうがいい。
```

```java
// 実行時間は０(タグの数　* タグの深さの平均)なので、ネストの深さに気をつける。
def FixBrokenHtml(html): ...
```

---
### 「全体像」のコメント

```text
新しいチームメンバにとって、最も難しいのは「全体像」の理解である。
クラスはどのように連携しているのか。
データはどのようにシステムを流れているのか。
エントリーポイントはどこにあるのか。
```

対して

```text
システムを設計した人は、こうしたことについてコメントを書かないことが多い。
あまりにも密接にシステムに関わりすぎているからだ。
```

---

`ある思考実験`

```text
新しくチームに参加した人がいるとする。
彼女は君の隣に座っている。
彼女にはコードに慣れてもらわなければならない。

君は彼女にコードのことを教える。
ファイルやクラスを指して、こんなことを言うだろう

「これはビジネスロジックとデータベースをつなぐグルーコード(glue code)です。」

「アプリケーションから直接使ってはいけません。」

「このクラスは複雑に見えますけど、単なるキャッシュです。
システムのことは関知していません。」

簡単な会話だけど、ソースコードを読んだだけでは得られない情報が手に入った。
```

***これは高レベルのコメントに書くべき情報だ。***

---

```text
// このファイルには、ファイルシステムに関する便利なインターフェースを提供
// するヘルパー関数が含まれています。ファイルのパーミッションなどを扱います。
```

***短い適切な文書で構わない。何もないよりはマシだ。***

---

## 要約コメント

```java
# 顧客が自分で購入した商品を検索する
for customer_id in all_customers:
    for sale in all_sales[customer_id].sales:
        if sale.recipient == customer_id:
 ...
```

```
コメントがないとコードを読んでいる途中で意味がわからなくなる
このような要約コメントは、関数の内部にある大きな「塊」 “chunks” につけても良い。
```

※ ファイル形式やデータ転送でのchunkではなくて、認知心理学の用語で、人間が記憶する際の単位としての意味だと思います。

---

```text
def GenerateUserReport():
 # このユーザーのロックを獲得する
 ...
 # ユーザの情報をDBから読み込む
 ...
 # Write info to a file
 ...
 # Release the lock for this user
```

```text
ひどいコードに優れたコメントをつけるより、優れたコードの方がいい
```

---

### WHAT・WHY・HOW をコメントに書くべきか

```text
「コメントにはWHATではなく（あるいはHOWではなく）WHYを書こう」
というアドバイスを耳にしたことがあるかもしれない。
たしかにキャッチーな言葉だけど、少し単純化しすぎていて、
人によって受け取り方が違うかもしれない。

　ぼくたちからアドバイスはこうだ。
「コードを理解するのに役立つものなら何でもいいから書こう」
これなら、WHATでもHOWでもWHYでも(あるいは３つ全部)でもかける
```

※ 私は下記のようなコーディングを見たことあるので「コードを理解するのに役立つものなら何でもいいから書こう」のみでは反対です。

- 詳細仕様書の文言をそのまま貼り付けたコメント
- リファクタリングでプロダクトコードと乖離してしまったコメント

---

### ライターズブロックを乗り越える

```text
プログラマの多くはコメントを書きたがらない。
コメントをうまく書くのは大変だと思っているかもしれない。
こうしたライターズブロック("Writer’s Block")を乗り越えるには、
とにかく書き始めるしかない。
自分の考えていることをとりあえず書き出してみよう。
生煮えであっても構わない。

たとえば、ある関数を作っていて、
「ヤバい、これはリストに重複があったら面倒なことになる」と思ったとする。
それをそのまま書き出せば良い
```

```text
// ヤバい。これはリストの重複があったら面倒なことになる。
```

```text
// 注意：このコードはリストの重複を処理できません(実装が難しいので。)
```

---

```text
・頭のなかにあるコメントをとにかく書き出す。

・コメントを読んで（どちらかと言えば）改善が必要なものを見つける。

・改善する。
```

---

## 5章まとめ

### コメントすべきでは「ない」こと
```text
・コードからすぐに抽出できること

・ひどいコード(例えば、ひどい名前の関数)を補う「補助的なコメント」。
　コメントを書くのではなくコードを修正する。
```

### 記録すべき自分の考え

```text
・なぜコードが他のやり方ではなくこうなっているのか(監督コメンタリー)
　“director commentary”

・コードの血管をTODO:やXXX:などの記法を使って示す。

・定数の値にまつわる背景

```

---

### 読み手の立場になって考える；

```text
・コードを読んだ人が「えっ？」と思うところを予想してコメントをつける。

・平均的な読み手が驚くような動作は文書化しておく。

・ファイルやクラスには「全体像」のコメントを書く。

・読み手が細部に捕われないように、コードブロックにコメントをつけて概要をまとめる。

```

---

この章のみの個人的な感想としては

- 新しく参画した人がコードを読んだ時にどう思うかの想像が大事
- self-documentationが大事
- CIテストを充足させていれば、エラーケース含めたその関数やクラスの使い方がわかるし、リファクタリングでコメントとコードが乖離するような現象が発生しない

---

## 6章　コメントは正確で簡潔に

```text
- 6章　コメントは正確で簡潔に
    - 6.1　コメントを簡潔にしておく
    - 6.2　あいまいな代名詞を避ける
    - 6.3　歯切れの悪い文章を磨く
    - 6.4　関数の動作を正確に記述する
    - 6.5　入出力のコーナーケースに実例を使う
    - 6.6　コードの意図を書く
    - 6.7　「名前付き引数」コメント
    - 6.8　情報密度の高い言葉を使う
    - 6.9　まとめ
```

---



```text
前章では、何をコメントに書くべきかを説明した。
本章では、どうすればコメントを正確で簡潔に欠けるかを説明する。
```

`鍵となる考え`

```text
コメントは領域に対する情報の比率が高くなければいけない。
```

※ 私にとっては不明瞭な日本語に感じたのですが、
Comments should have a high information-to-space ratio.
「短い文章で多くの情報を伝えなければなりません」ってことだと思います。

---

### コメントを簡潔にしておく

3行も要らない

`改善前`

```c++
// CategoryTypeはint
// ペアの最初のfloatは、score
// 2つめは、weight
typedef hash_map<int, pair<float, float> > ScoreMap;
```

`改善後`

```c++
// CategoryType -> (score, weight)
typedef hash_map<int, pair<float, float> > ScoreMap;
```
---

### 6.2　あいまいな代名詞を避ける

```text
読み手は代名詞を「還元」しなければならない
```

※　It takes extra work for the reader to “resolve” a pronoun.
読み手が代名詞を「解決」するのは、余分な作業です。

```text
場合によっては、「それ」や「これ」が何を指しているのかよくわからないこともある。
```

日本人エンジニアだとそもそもコメントで「それ」「これ」使わないかも。
使うべきではないけど。

---

改善前

```text
// データをキャッシュに入れる。ただし、先のそのサイズをチェックする。
// Insert the data into the cache, but check if it's too big first.
```

改善後①

```text
// データをキャッシュに入れる。ただし、先にデータのサイズをチェックする
// Insert the data into the cache, but check if the data is too big first.
```

改善後②

```text
// データが十分にちいさければ、それをキャッシュに入れる
// If the data is small enough, insert it into the cache.
```

---


### 6.3　歯切れの悪い文章を磨く

改善前

```text
# これまでにクロールしたURLかどうかによって優先度を変える。
```

改善後

```text
これまでにクロールしてないURLの優先度を高くする。
```

---

### 6.4　関数の動作を正確に記述する

改善前

```c++
// このファイルに含まれる行数を返す
int CountLines(string filename) { ... }
```

コーナーケース(見逃されそうなケース)

```text
・ ""(空のファイル)は、0行なのか１行なのか                                       
・ "hello"は、0行なのか１行なのか
・ "hello\n"は、0行なのか１行なのか
・ "hello\n world"は、1行なのか2行なのか
・ "hello\n\r cruel\n world\r"は、2行なのか3行なのか4行なのか
```

改善後

```text
// このファイルに含まれる改行文字(\n)を数える
int CountLines(string filename) { ... }
```

---

### 6.5　入出力のコーナーケースに実例を使う

```text
慎重に選んだ入出力の実例をコメントに書いておけば、それは千の文字に等しいと言える。
```

```c++
// 'src'の先頭や末尾にある'chars'を除去する
String Strip(String src, String chars) { ... }
```

```text
・ charsは、除去する文字列なのか、順序のない文字集合なのか？
・ srcの末尾に複数のcharsがあったらどうなるのか？
```

使用例をコメントに入れる

```c++
// ...
// 実例: Strip("abba/a/ba", "ab") returns "/a/"
```

```c++
// ...
// 実例: Strip("ab", "a") returns "b"
```

---

```c++
// 'v' の「要素 < pivot」が、「要素 >= pivot」の前に来るように配置し直す。
// それから、「v[i] < pivot」になる最大の'i'を返す(なければ-1を返す)。
int Partition(vector<int>* v, int pivot);
```

```c++
// ...
// 実例: Partition([8 5 9 8 2], 8) の結果は [5 2 | 8 9 8] となり、1を返す。
int Partition(vector<int>* v, int pivot);
```

`この実例にした理由`

```text
・ pivotをベクタの要素と同じにして、エッジケースを示している。
・ ベクタに重複要素(8)を入れることで、これが入力値として受け入れられることを示している。
・ 結果のベクタはソートしてない。ここでソートしていたら、ソートされているはずと読み手が誤解する可能性がある。
・ 戻り値が１になるのでベクタの要素に1を入れてない。
```

---

### 6.6 コードの意図を書く

`改善前`

```c++
void DisplayProducts(list<Product> products) {
   products.sort(CompareProductByPrice);
   // listを逆順にイテレートする
   for (list<Product>::reverse_iterator it = products.rbegin(); it != products.rend();
     ++it)
     DisplayPrice(it->price);
 ...
}
```
`改善後`　プログラムの動作を高レベルから説明

```c++
   // 値段の高い順に表示する
   for (list<Product>::reverse_iterator it = products.rbegin(); it != products.rend();
     ++it)
```

---

```text
このプログラムにはバグがある。
CompareProductByPrice関数(ここには書いてない)が、既に値段の高い順にソートしている。
改善後のコメントによって、発覚した。
コメントが冗長検査(redundancy check)の役割を果たした。
```

```text
究極的には、ユニットテストが最高の冗長検査になるだろう
(「14章　テストと読みやすさ」参照)
だからといって、プログラムの意図を説明するコメントをつけるのが無駄になるわけではない。
```

---

### 6.7　「名前付き引数」コメント

`呼び出し方例`

```c++
Connect(10, false);
```

`呼び出す関数`

```c++
def Connect(timeout, use_encryption): ...
```

`呼び出し方例　python改善後`

```c++
# Call the function using named parameters
Connect(timeout = 10, use_encryption = False)
```


---

`呼び出し方例　java,c++改善後`

```c++
// 引数にコメントをつけて関数を呼び出す
Connect(/* timeout_ms = */ 10, /* use_encryption = */ false);
```

```text
最初の引数名がtimeoutではなくtimeout_msになっていることに注意してほしい。
本来であれば、仮引数の名前をtimeout_msにすべきだけど、
何らかの理由で変更できないのであれば、このように手っ取り早く名前を「改善」できるのだ。
```

`悪い「名前付き引数」コメント`

```c++
// Don't do this!
Connect( ... , false /* use_encryption */);
// Don't do this either!
Connect( ... , false /* = use_encryption */);
```

---

### 6.8　情報密度の高い言葉を使う

`改善前`

```text
// このクラスには大量のメンバがある。同じ情報はデータベースにも保管されている。
// ただし、速度の面からここにも保管しておく。
// このクラスを読み込むときには、メンバが存在しているかどうかを先に確認する。
// もし存在していれば、そのまま返す。
// 存在していなければ、データベースから読み込んで、次回のためにデータをフィールドに保管する。
```

`改善後`

```text
このクラスの役割は、データベースのキャッシュ層である。
```

---

`改善前`

```text
所在地から余分な空白を除去する。それから「Avenue」を「Ave.」にするなどの整形を施す。
こうすれば、表記がわずかに違う所在地でも同じものであると判別できる。
```

`改善後`

```text
所在地を正規化する(例: "Avenu" -> "Ave.")。
```

`多くの意味を含んだ言葉や表現例`


- [ヒューリスティック](https://ja.wikipedia.org/wiki/%E3%83%92%E3%83%A5%E3%83%BC%E3%83%AA%E3%82%B9%E3%83%86%E3%82%A3%E3%82%AF%E3%82%B9)　“heuristic”
必ず正しい答えを導けるわけではないが、ある程度のレベルで正解に近い解

- [ブルートフォース](https://ja.wikipedia.org/wiki/%E7%B7%8F%E5%BD%93%E3%81%9F%E3%82%8A%E6%94%BB%E6%92%83)　“brute force” 総当たり攻撃
- [ナイーブ](https://www.itmedia.co.jp/business/articles/1905/13/news064.html)ソリューション　 “naive solution” 未熟な解決法

---

### 6.9　まとめ

```text
本章では、小さな領域にできるだけ多くの情報を詰め込んだコメントを書くことについて説明した。
具体的なヒントを以下に挙げる。

・複数のものをサス可能性がある「それ」や「これ」などの代名詞を避ける。

・関数の動作はできるだけ正確に説明する。

・コメントに含める入出力の実例を慎重に選ぶ。

・コードの意図は、詳細レベルではなく、高レベルで記述する。

・よくわからない引数にはインラインコメントを使う
（例:Function(/* arg= */ ...)）

・多くの意味が詰め込まれた言葉や表現を使って、コメントを簡潔に保つ。

```
