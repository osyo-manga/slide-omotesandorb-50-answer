## 表参道.rb #50
- - -

## いろいろ回答

---

#### 自己紹介
- - -

* なまえ  : おしょー
* Twitter : [@pink_bangbi](https://twitter.com/pink_bangbi)
* github  : [osyo-manga](https://github.com/osyo-manga)
* ブログ  : [Secret Garden(Instrumental)](http://secret-garden.hatenablog.com)
* Rails 歴 1年半
* 趣味で Ruby にパッチを投げています
* 最近のトレンド
  * GitHub Actions
  * Rubocop の運用方法
  * RSpec もっと綺麗に書きたい

---

#### Q. to_curryとか
#### パイプラインオペレーターとか
#### 関数ぽいAPIが提供され出しているけど
#### 実際にproductionでどのような使われ方が想定されるんでしょう

---

### A. どういう機能があるかみてみよう

---

#### パイプラインオペレータとは
- - -

* |> を使って hoge(a, b).foo を hoge(a, b) |> foo とかける演算子       <!-- .element: class="fragment" -->
  * . 演算子と同等の演算子
  * . 演算子よりも優先順位が低いので
  * hoge(a, b).foo を hoge a, b |> foo とかけるのが利点
* いわゆる関数型言語にあるようなパイプラインオペレーターとは用途がちょっと違う       <!-- .element: class="fragment" -->
  * 例えば Elixir の場合は
  * f() |> g(a, b) は g(f(), a, b) と同じ意味になる
  * |> の右辺を左辺の関数の第一引数に置き換える
* 詳しくは         <!-- .element: class="fragment" -->
  * [パイプライン演算子の歴史 - まめめも](https://mametter.hatenablog.com/entry/2019/06/15/192311)   

---

# なお

---

### パイプラインオペレータは Ruby 2.7 で入る予定でしたが
### 様々な事情により見送られる事になりました
### ＼(^o^)／

---

#### Proc#<< と Proc#>>
- - -

* Proc や Method で関数合成を行うメソッド
* f >> g は `proc { |*a| g.call(f.call(*a)) }`

```ruby
to_camel = :capitalize.to_proc
add_header = -> val {"Title: " + val }

pp add_header.call(to_camel.call("homu"))
# => "Title: Homu"

pp (to_camel >> add_header).call("homu")
# => "Title: Homu"

pp %w(homu mami mado).map { |it| add_header.call(to_camel.call(it)) }
# => ["Title: Homu", "Title: Mami", "Title: Mado"]

pp %w(homu mami mado).map &to_camel >> add_header
# => ["Title: Homu", "Title: Mami", "Title: Mado"]
```


>>>

#### Symbol#to_proc を使うと
- - -

* Symbol#to_proc を利用してメソッドチェーン

```ruby
pp proc { |it| it.to_i.chr }.call "72"
# => "H"

pp (:to_i.to_proc >> :chr.to_proc).call "72"
# => "H"

pp %w(72 101 108 108 111).map { |it| it.to_i.chr }
# => ["H", "e", "l", "l", "o"]

pp %w(72 101 108 108 111).map &:to_i.to_proc >> :chr.to_proc
# => ["H", "e", "l", "l", "o"]

# 最近 @1 から _1 に変わったナンパラ使うともっと簡潔に…
pp %w(72 101 108 108 111).map { _1.to_i.chr }
```

---

#### .: 演算子
- - -

* Ruby 2.7 で .: 演算子が追加される予定
* obj.method(:hoge) が obj.:hoge とかける

```ruby
pp Math.method(:sqrt).call 256
# => 16.0

pp Math.:sqrt.call 256
# => 16.0

pp [4, 16, 64, 256].map &Math.method(:sqrt)
# => [2.0, 4.0, 8.0, 16.0]

pp [4, 16, 64, 256].map &Math.:sqrt
# => [2.0, 4.0, 8.0, 16.0]

%w(72 101 108 108 111)
  .map(&:to_i)
  .each(&self.:pp)     # メソッドチェーンの間に出力を割り込む
  .map(&:chr)
```


---

### Q. なぜ変数名に
### クエスチョンマークが使えないの？

---

## A. 実装が難しいから

---

#### 経緯
- - -

* bugs.ruby に issues があった         <!-- .element: class="fragment" -->
  * [Feature #15991: Allow questionmarks in variable names - Ruby master - Ruby Issue Tracking System](https://bugs.ruby-lang.org/issues/15991)
* 前回の開発者会議で提案してみたが、実装が難しいとのこと         <!-- .element: class="fragment" -->
  * 条件演算子や複数割り当てなどと競合する可能性がある
* ちなみにインスタンス変数に関しては matz が否定している         <!-- .element: class="fragment" -->
  * [Feature #5781: Query attributes (attribute methods ending in `?` mark) - Ruby master - Ruby Issue Tracking System](https://bugs.ruby-lang.org/issues/5781)


---

#### Q. rspec で let/let!/before の使い分け、
#### context の分け方・nest のさせ方等
#### どうやると見やすくなるか知りたいです！

---

#### describe / context
- - -

* テストのスコープを切り分ける機能     <!-- .element: class="fragment" -->
  * 名前が違うだけで機能としては同等
* describe は『テスト対象』     <!-- .element: class="fragment" -->
  * クラス名やメソッド名
  * reqeust spec であれば `describe "GET /users"` とか
* context は『テストケース』     <!-- .element: class="fragment" -->
  * テストケース
  * 引数が○○○の場合
* コンテキストごとにグループ化していく感じ     <!-- .element: class="fragment" -->
  * 横に長いよりも縦に長くしていくほうが context が複雑にならずみやすいかも?
  * 複雑だと context を追加する場合に悩んでしまう

>>>

```ruby
RSpec.describe Array do
  describe "#[]" do
    context "整数を渡した場合" do
      context "`0` を渡した場合" do
      end

      context "`-1` を渡した場合" do
      end
    end

    context "文字列を渡した場合" do
    end

    context "nil を渡した場合" do
    end

    context "配列が空の場合" do
      context "整数を渡した場合" do
        context "`0` を渡した場合" do
        end
      end
    end
  end
end
```

>>>

* nest を減らしてみるとこんな感じ

```ruby
RSpec.describe Array do
  describe "#[]" do
    context "`0` を渡した場合" do
    end

    context "`-1` を渡した場合" do
    end

    context "文字列を渡した場合" do
    end

    context "nil を渡した場合" do
    end

    context "空の配列に `0` を渡した場合" do
    end
  end
end
```

---

#### let / let! / before
- - -

* before は初期化処理を定義する     <!-- .element: class="fragment" -->
* let / let! は context に依存する値を定義する     <!-- .element: class="fragment" -->
  * 普通のメソッドとは違いメモ化される
  * let は参照されるまで処理は呼び出されない
  * let! は before と同じタイミングで呼び出される
* テストを実行する前に初期化しつつ、その結果を取得したい場合に let を使う     <!-- .element: class="fragment" -->

>>>

```ruby
# 参照されるまで呼び出されない
let(:mami) {  User.create(name: "mami")  }

# テスト実行前に自動的に呼び出される
let!(:homu) { User.create(name: "homu") }

# before も同様
before do
   User.create(name: "mado")
end

it do
  # このタイミングではすでに homu や mado が生成済み
  # mami は生成されていない

  # mami を呼び出した時に初めて let のブロックが呼び出される
  mami

  # mami はメモ化されているので 1回しか呼ばれない
  mami
end
```

---

#### let の使用例
- - -

```ruby
describe "#[]" do
  let(:array) { [1, 2, 3] }
  subject { array[index] }

  context "`0` を渡した場合" do
    let(:index) { 0 }
    it { expect(subject).to eq 1 }
  end
  context "`-1` を渡した場合" do
    let(:index) { -1 }
    it { expect(subject).to eq 3 }
  end
  context "文字列を渡した場合" do
    let(:index) { "" }
    it { expect { subject }.to raise_error TypeError }
  end
  context "空の配列に `0` を渡した場合" do
    let(:array) { [] }
    let(:index) { 0 }
    it { expect(subject).to be_nil }
  end
end
```

---

#### before の使用例
- - -

```ruby
describe "バリデーションのテスト" do
  subject { User.new(name: name) }
  before do
    User.create(name: "homu")
  end

  context "既存のユーザと同名で生成した場合" do
    let(:name) { "homu" }
    is { expect(subject).to be_invalid }
  end

  context "既存のユーザと別名で生成した場合" do
    let(:name) { "mami" }
    is { expect(subject).to be_valid }
  end
end
```

---


#### let! の使用例
- - -

* なにか初期データを生成しつつ、その値を参照したい場合に利用する
* let! や before の処理順は定義順なので注意

```ruby
describe "Tes" do
  let(:user) { User.create(name: "homu") }

  # これだと動かない
  # let(:comment) { Comment.create(text: "text", user_id: user.id) }

  # テスト前に Comment を生成しておく必要がる

  let!(:comment) { Comment.create(text: "text", user_id: user.id) }
  it do
    expect(user.comments.first).to eq comment
  end
end
```

---

#### let! つらいポイント
- - -

```ruby
let!(:user) { User.create(name: "homu") }


# ...ここにめっちゃ多くのテストがある


context "複雑なユーザを定義する" do
  let(:user) { @user }

  before do
    # 実際はめっちゃ複雑な初期化処理を書いていた
    @user = User.create(name: "mami")
  end

  it do
    # ここで user を参照すると let!(:user) で定義した user が帰ってくる
    pp user
  end
end
```

---

#### before つらいポイント
- - -

```ruby
before do
  # 上位の context の before で get を行う
  get user_url :homu
end

context "ユーザが存在しない場合" do
  it "ユーザ情報が表示されないこと" do
    expect(response.body).not_to include "homu"
  end
end
context "ユーザが存在する場合" do
  before do
    # get を呼び出す前にリソースを生成したい
    # しかし、ここが呼び出されるよりも前に上の
    # before で get user_url が呼び出されており意図しないテストになる
    User.create(name: "homu")
  end
  it "ユーザ情報が表示されていること" do
    expect(response.body).not_to include "homu"
  end
end
```

>>>

```ruby
subject { get "/users"  }

context "ユーザが存在しない場合" do
  it "ユーザ情報が表示されないこと" do
    # どう書くべきか？？？
    # it { expect { subject }.to change { response.body }.to include "homu" }
  end
end
context "ユーザが存在する場合" do
  before do
    User.create(name: "homu")
  end
  it "ユーザ情報が表示されていること" do
    expect { subject }.to change { response.body }.to include "homu"
  end
end
```

---

#### まとめ
- - -

* let は context に依存するようなものを定義する          <!-- .element: class="fragment" -->
* before は共通の初期化処理を行いたい場合に使用する          <!-- .element: class="fragment" -->
  * ただし、 before は必ず実行されるのであまり広い context で使用せずに狭い範囲で使用する方が変な副作用がなくてよい
* let! も before と同様に必ず実行されるので気をつけて使用する          <!-- .element: class="fragment" -->

---

# 宣伝

---

### Ruby Hack Challenge Holiday
- - -

* [Ruby Hack Challenge Holiday](https://rhc.connpass.com/event/145802/) というイベントをやっています
* Ruby 本体を自分でビルドしたりハックしたりするイベント           <!-- .element: class="fragment" -->
* Ruby のコミッタの方が主催しているので Ruby について聞けるチャンス       <!-- .element: class="fragment" -->
* Ruby 本体の開発に興味がある人はぜひ！！       <!-- .element: class="fragment" -->
* 次回は 10/06(日) 開催予定       <!-- .element: class="fragment" -->

---

## 最後にわたしからの質問

---

## Refinements を
## 使ってる人はいますか？

---

## ご清聴
## ありがとうございました
