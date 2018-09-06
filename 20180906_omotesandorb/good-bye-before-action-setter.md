autoscale: true
slidenumbers: true

# before action setter いる？

## 2018-09-06 表参道.rb #38

---

## 自己紹介

- @s\_osa\_
- OSA Shunsuke
- Cookpad Inc.
- 技術部 ユーザー・決済基盤グループ 💰
- Rails に対しては愛憎相半ばする複雑な気持ちを抱いている

---

## 話すこと

- before action setter って必要？
- 多くの場合、いらないのでは？

---

## before action setter

名前がないといろいろ不便なので、勝手に before action setter と呼んでいますが、要は scaffold でも生成されるこいつ。

```ruby
class HogesController < ApplicationController
  before_action :set_hoge, only: %i[show]

  private

  def set_hoge
    @hoge = Hoge.find(params[:id])
  end
end
```

---

# つらい理由

---

## 変数の初期化・代入を目的としたメソッド

`set_*` を呼ぶと、メソッド呼び出し側にインスタンス変数が増えるという重大かつ直感的でないな副作用がある。

普段コードを書くときに

```ruby
def set_message
  @message = 'Hello'
end

set_message
```

みたいなことをする人はまずいないはず。

---

## 自明でない順序依存性が発生しがち

はじめはシンプルだった before action setter も、機能追加などをしていくうちに、いつの間にか

```ruby
before_action :set_hoge
before_action :set_fuga
before_action :set_foo # set_bar より先に実行すること！！
before_action :set_bar
```

のようなことになりがち。

---

## action の関心がわかりにくい

Rails の controller におけるインスタンス変数は view で参照する変数であり、レスポンスを組み立てるために必要なものが入っている。

インスタンス変数が暗黙的に代入されると、その action の関心が何なのかが少なくともパッとはわからない。

---

# じゃあ、どう書くか

---

## シンプルなとき

そのまま書けば良い。

```ruby
class HogesController < ApplicationController
  def show
    @hoge = Hoge.find(params[:id])
  end

  def edit
    @hoge = Hoge.find(params[:id])
  end

  # ...
end
```

---

## 複雑・重要な絞り込みがあるとき

具体的には、複雑だったり重要だったりする絞り込みを行なっていて、その重複を避けたい場合。

そういうときは `set_*` じゃなく `find_*` する。

```ruby
class HogesController < ApplicationController
  def show
    @hoge = find_hoge(params[:id])
  end

  private

  # @param id [String]
  # @return [Hoge]
  def find_hoge(id)
    Hoge.very.complex.condtion.find(id)
  end
end
```

---

## 順序依存性があるとき

複数の before action setter の間に順序依存性があったケースでは依存を明確にして管理するために引数として渡す。

```ruby
class HogesController < ApplicationController
  def show
    @hoge = find_hoge(params[:id])
    @fuga = find_fuga(@hoge)
  end

  private

  # @param id [String]
  # @return [Hoge]
  def find_hoge(id)
    Hoge.very.complex.condtion.find(id)
  end

  # @param hoge [Hoge]
  # @return [Fuga]
  def find_fuga(hoge)
    hoge.fugas.very.important.condition.first
  end
end
```

---

# before action を使うべきとき

---

## 認証・検証など

- action が呼ばれる際に満たしておく事前条件のようなものが存在し、その条件を満たさないときは action を実行しないような類のもの。
  - `authenticate_*!` とか `verify_*!` みたいなメソッド名が多い。
- 別の言い方をすると、`before_action` の旧名である `before_filter` という名前がしっくり来る感じのやつ。
  - setter じゃないことも多いが、認証・検証後に set することもありがち。

---

## 横断的に使用される変数など

- 典型的には `set_current_user` のようなもの。
- そのアプリケーションで横断的に使われる変数などの初期化・代入。
  - 広く使われるものであれば、そのアプリケーションを開発する人の基礎知識とすることに妥当性が生まれる。
- 横断的に使用されるという性質上、この種の before action setter は `ApplicationController` や `Admin::BaseController` のような場所に定義される。

---

# まとめ

---

## まとめ

- before action setter って必要？
- 多くの場合、いらない。
- Rails とはいい感じに折り合いをつけて付き合っていきたい。

---

# Thank You!
