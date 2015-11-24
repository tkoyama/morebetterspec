# 基本的な指針

- describe, context, exampleのメッセージだけを読んで読める
- まず日本語で書いてみる
- BDDであることを意識する

# describe/context/exampleをきちんと書く

まずこれがきちんと書けるようになるだけで、一気に読みやすいテストになる

### 用語

- describe
  - テスト対象
- context
  - テストする状況
- example
  - テスト(itやspecify)


### 重要なこと

『aaaはbbbの時cccになる』
というテストを書くときは

```ruby
describe 'aaa' do
  context 'when bbb' do
    it 'should eq ccc'
  end
end
```

このようになる
重要なのは、

- describe/context/exampleを読んだだけでテストの内容がわかるようになっていること
  - レビューする人・次さわる人が、他の部分のコード(特にテスト対象の実装)を読まなくても理解できるのが理想
- そうなるように、describe/context/exampleをきちんと分けること
- 1つのexampleの中で複数のことをテストしたりしないこと
  - いろんな理由で、1つのexampleの中で複数のことをテストすることはある
  - その時はそうだとわかるようにexampleのメッセージを工夫すること

### 日本語で書いてみる

decribe/context/exampleがうまくわけれない時は
とりあえず日本語で書き出してみてから英語に直すと良い

- decribe/context/exampleを日本語でまず全部書き出してみる
  - この時にexampleの中身やletなどは書かない
- 全部書き出し終わって違和感がなくなったら、letなどを書いて埋めていく
  - (必要であれば)日本語->英語もこの時にする
- はじめに書いたdecribe/context/exampleに固辞せず臨機応変に変える
  - はじめに書いたdecribe/context/exampleでは書きづらかったり過不足があったりすることがある


# shared_context, shared_examples_for

DRYを意識して無闇矢鱈に使われることがあるけど、
その殆どが読みづらさにつながることが多い
ので、あまり使わないほうが良い。使うとしても節度を持って使うこと

読みづらくなる原因は色んな物をsharedに詰め込んでしまって、
適切なメッセージがつかなくて中身を読まないと何をしているのかわからなくなってしまうから

### shared_contextはcontextをshareするもの

いろんな使われ方をするが、
基本的には文字通り、contextをshareするもの、というのを意識する

```
aaaはxxxの時cccになる
dddはxxxの時eeeになる
```

というテストを書きたい時には以下の様なテストになる

```ruby
shared_context "xxx"

describe 'aaa' do
  context 'when xxx' do
    include_context "xxx"
    it 'should eq ccc'
  end
end

describe 'ddd' do
  context 'when xxx' do
    include_context "xxx"
    it 'should eq eee'
  end
end
```

この使い方が一番ベーシック

### shared_examples_forはit_behaves_likeでメッセージ決める

`shared_examples_for 'hoge'`でメッセージを考えるよりも
`it_behaves_like 'hoge'`で考えるほうが考えやすい可能性がある

```ruby
describe 'aaa' do
  context 'when bbb' do
    it_behaves_like 'hoge'
  end
end
```
例えばこういうテストがあった時に
『aaaはbbbの時にhogeのように振る舞う』と読める
これが違和感無いようにメッセージを決めると良い

『hogeのように振る舞う』なので

```ruby
it_behaves_like 'a collection'
it_behaves_like 'a listable resource'
it_behaves_like 'a joinable room'
```
のように名詞が入るのが普通 # ラスタムさんチェックおねがいするるるるるるるるるるるるるるる

# let/let!どっちを使うのか

個人的には基本的に`let`で必要なときに`let!`を使うようにしている
が、その逆の使い方をする人もいる
そのメリットもわからなくはないので
まだ悩み中

# 必要なときに必要な分だけ用意する

その時に必要な分だけ必要なデータを用意すること

### 良くないケース

```ruby
# もっと具体的なspecにする必要あるなーーー
descirbe "hoge" do
  let!(:a1) { create :a }
  let!(:a2) { create :a }
  let!(:a3) { create :a }

  context "foo" do
    # a1, a2, a3を使うテスト
  end

  context "bar" do
    # a2, a3を使うテスト
  end

  context "baz" do
    # a3を使うテスト
  end
end
```

各contextでどうせ使うから、と言ってcontextの外で定義することがよくある
例えば`baz`のときに`a3`だけが必要なのか、ほかの`a1`や`a2`も影響しているのか、
がテストの中身を読み解かないと理解できないことが多い
また、共通で使われるので汎用的な名前になってしまうし結果として読みにくいテストとなってしまう

### 必要なときに必要なデータを用意したケース

```ruby
# もっと具体的なspecにする必要あるなーーー
descirbe "hoge" do
  context "foo" do
    # ちゃんと名前つける------
    let!(:a1) { create :a }
    let!(:a2) { create :a }
    let!(:a3) { create :a }
  end

  context "bar" do
    let!(:a2) { create :a }
    let!(:a3) { create :a }
  end

  context "baz" do
    let!(:a3) { create :a }
  end
end
```

必要なときに必要な分だけ用意するので、余計なこと考えなくて済むし、
適切な名前がつけられるので理解しやすさに繋がる


# 良いFactoryGirl

# どこまでテストするのか

### mockはあまり使わない

### privateメソッド

### 意図的なskip(pending)


# 良くないspecの兆候

- contextが多い
- shared_context, shared_examples_forが欲しくなる
=> テスト対象が複雑だったりDRYじゃなかったりする

# 良くないspec=>見やすいspecにリファクタ

ガッツリコード書いて説明したい