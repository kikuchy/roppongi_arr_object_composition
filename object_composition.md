# ActivityをスリムにするObject Compositionパターン

@kikuchy


---

# だれだこいつ


----

![kikuchy](https://avatars1.githubusercontent.com/u/600512?v=3&s=460)

* 菊池紘 (kikuchy)
* 株式会社ミクシィ（株式会社Diverseへ出向中）
* Android アプリ専業になって半年
* もともとはWebのフロントが専門

---

## この話のターゲット

* つい Activity にべったりコード書きすぎちゃう(・ω<)
* MVC? MVP? 知ってるけど実現するの難しいよね(・ω<)
* 退職者が残していったスパゲッティコードを崩さないようにするの大変だよね(・ω<)

Note: そんな方向けです。


---

# Object Composition パターンとは

----

## こんなイメージ

![コンポーネント](http://cnet2.cbsistatic.com/hub/i/r/2014/05/21/925357d4-d858-48a8-a5df-774b4c6b907e/thumbnail/670x503/d203e2445668366d53c4e2cf14f77073/google-project-ara-small.jpg)


----

## Effective Java によると

> This design is called composition because the existing class becomes a component of the new one.
>
>（このパターンはコンポジションと呼ばれています。なぜなら、既存のクラスが新しいクラスのコンポーネントとなっているからです。）

*-- Effective Java Second Edition  Item 16: Favor composition over inheritance*


----

### オブジェクトの組み合わせで別のオブジェクトを構成するパターン


---

# 何が嬉しいの？


----

## Activity の役割がはっきりする


----

## 同じような機能を複数の画面で使いまわせる


----

* コードが綺麗になったり
* 分業しやすくなったり
* 次からの実装が楽になったり
* メンテナンスしやすくなったり

＼\ ٩( 'ω' )و /／

----

### 実際に嬉しかった例

* 「写真を撮る」「ギャラリーから選択する」のダイアログを出して、撮影or選択されたUriを返す機能を切り出し。
	* あちこちで使用可能に。
* 検索結果を複数件選択する機能を切り出し。
	* 新しく入社されたエンジニアさんにも説明なしに使ってもらえていた。


---

# やり方

----

## ポイント

1. 適当な名前のクラスをつくる
1. 適当に Activity の中でインスタンス化する
1. 適当に使う
1. 適当にメンバが保持されるようにする (Optional)
1. Activity に依存しないよう適当にフィードバックする (Optional)


----

##  ( ﾟдﾟ) < 適当


----

## 適当な名前のクラスをつくる

* Composition するクラスとは **「Activity に組み合わせて使える、単一機能を実現するクラス」**
* これがわかる名前で、チーム内で合意が取れれば良い。
	* `〜〜Composable`
	* `〜〜ActivityCOmposition`
	* `〜〜Manager`
など。

Note: もちろん、ちゃんと意味の取れる命名にしないと後で混乱の元になる。

----

## 適当に Activity の中でインスタンス化する

* `onCreate()` の中でインスタンス化してメンバに持っておく。
* Context や Activity を使う処理をさせたい場合はコンストラクタで渡す。

Note: Activity と一緒に生き死にしてくれるので管理が楽。

----

![ライフサイクル](https://qiita-image-store.s3.amazonaws.com/0/41896/f740f542-c71b-d530-796f-aac52cce5f9d.png)

Note: こういうライフサイクルで生き死にするオブジェクトにすると良い感じ。


----

## 適当に使う

* Composition するオブジェクトの担当機能を呼び出すメソッドを用意。
* Activity の中から適当に呼んで使う。


----

```java
// onCreate の中
selectionComposable.registerObserver(new DataSetObserver {
    @Override
    public void onChanged() {
        // 要素の選択数が変化したときのUI操作は Activity でハンドリング
    }
    ...
});

// Adapter の OnItemClick などの中
selectionComposable.addItem(item);


// 次の画面へ行くボタンの onClick の中

Serializable selectedItems = selectionComposable.getSelectedItems();
Intent intent = NextScreenActivity.createIntent(this, selectedItems)
```


----

## 適当にメンバが保持されるようにする

* Composition するクラスにも `onSaveInstanceState()` と `onRestoreInstanceState()` を生やしておく。
* Activity の同名メソッドで呼び出す


Note: Activity と同じライフサイクルで生き死にできるようになる。Bundle の key はクラス名をprefixにするとかしておくと、 Activity や同時に使う他の Composable との衝突の恐れが減る。


----

## Activity に依存しないよう適当にフィードバックする

Activity へ結果を返す必要があるとき。

* 絶対に Activity を import しない。
* ライブラリなど使わないなら以下のどちらかの方法が簡単。
	* コールバック interface を用意して、匿名クラスをメソッドの引数にとる。
	* interface を作って Activity に実装させる。
* EventBus, RxJava などを使ってもOK。

Note: 特定の Activity に依存するのは避けましょう。今は特定の Activity でしか使わないとしても、もし他で使うことになった場合（そういう場合は往々にして生じる）に使い回すのが難しくなってしまいます。


----


```java
pictureComposable.onActivityResult(
	requestCode,
	responseCode,
	data,
	new OnPictureChosenCallback {
        @Override
        onPictureChosen(Uri pictureUri) {
            // 好きなようにハンドリングする
        }
        ...
});
```

Note: 匿名クラスを使ったコールバックの例。

----

## Fragment は？

Activity と同様に、 Fragment からも機能を切り離すことができる。

これまでの "Activity" を "Fragment" に読み替えてください。


---

# 魔法の呪文

----

思わずベタッと Activity に書いてしまった機能。

そんなものが出てきてしまったら、自問自答してみましょう。

----

## 「これはViewに載せるべき処理なのか？」


----

答えがNOなら、

**Composition するクラスへ機能をカット＆ペースト** 。


---

![Qiita](./qiita_header.png)

Qiitaにも書いてあるのでこちらもご参照ください。


---

## ありがとうございました！

＼\ ٩( 'ω' )و /／