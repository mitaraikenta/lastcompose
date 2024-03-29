# jetpack composeの基本

****

## プロジェクト作成

****

- build.gradle ファイルと app/build.gradle ファイルには、Compose に必要なオプションと依存関係が含まれています。

## コンポーズ可能な関数

****

コンポーズ可能な関数とは、@Composable アノテーションが付いている通常の関数。よって、関数は内部で他の @Composable 関数を呼び出すことができる
Greeting 関数が @Composable とマークされている例を次に示します。
この関数では UI 階層の一部が生成され、指定された入力（String）がその UI 階層で表示されます。Text は、ライブラリによって提供されるコンポーズ可能な関数です。
![](../../img/compose1.png)

## Androidアプリでのcompose

****

- Activitiesはandroidのエントリーポイント
- ユーザーがアプリを開くとMainActivityが起動する（AndroidMainfestの設定どおり）
- レイアウトはxmlではない！内部のcompose可能な関数を呼び出す!

![](../../img/compose2.png)

- BasicsCodelabTheme は、コンポーズ可能な関数のスタイルを設定する手段
- Android Studio プレビューを使用するには、※パラメータのないコンポーズ可能な関数またはデフォルト パラメータを含む関数に @Preview アノテーションを付けて、プロジェクトをビルドします。
- 同じファイルに複数のプレビューを設定し、それぞれに名前を付けることができます。

![](../../img/compose3.png)


※パラメータとは

![](../../img/parameter.png)


### UIの微調整

****

![](../../img/compose4.png)

Surface 内にネストされているコンポーネントは、その背景色の上に描画されます。


- 結果
![](../../img/HerroAndroid.png)

テキストを指定してないのに文字の色が白い。なぜか。（河田構文）

実は、定義していません。androidx.compose.material.Surface などのマテリアル コンポーネントは、
テキストに適切な色を選択するなど、一般的にアプリに望まれる内容を指定することで、ユーザー エクスペリエンスが向上するように設計されている

化け物だ

ちなマテリアルは、大半のアプリに通用する適切なデフォルト値とパターンを提供するものであるため、柔軟性がありません。

Compose のマテリアル コンポーネントは、他の基盤コンポーネント（androidx.compose.foundation にあるコンポーネント）の上に構築されており、より柔軟性が必要な場合は、アプリ コンポーネントからもアクセスできます。

今回の場合は背景が primary カラーに設定されており、
その際には背景の上にあるテキストに onPrimary カラー（これはテーマでも定義されています）
を使用すべきであることが、Surface によって認識されておるのだ

### 修飾子

****
Surface や Text といったほとんどの Compose UI 要素はね。省略可能な modifier パラメータを使って
UI 要素に対して親レイアウト内での配置、表示、動作を命令できるのだ。

よってtextにmodifierを追加し、paddingの効果発動！！
![](../../img/compose5.png)

背景色を拡大することが出来るのだ！！

![](../../img/compose6.png)

### コンポーザブルを再利用する

****

UI に追加するコンポーネントが多くなると、作成するネストのレベルが増えます
関数が非常に大規模なものになる場合、読みやすさに影響する可能性がでるんだよね。

再利用可能な小さいコンポーネントを作成すると、アプリで使用する UI 要素のライブラリを簡単に構築できます。

UI 要素は、それぞれで画面の一部を処理し、個別に編集ってことができるわけ

### 列と行の作成

****
アーニャここよく忘れる
![](../../img/compose8.png)

これらのコンポーザブルは、コンポーズ可能なコンテンツを受け取るコンポーズ可能な関数であり、
内部にアイテムを配置できます。たとえば、Column 内のそれぞれの子は縦方向に配置されます。

使用例だとこんな感じ

![](../../img/compose9.png)

![](../../img/compose10.png)

#### Compose と Kotlin

****

コンポーズ可能な関数は、Kotlin の他の関数と同様に使用できます

つまりUI の表示方法に影響を与えるステートメントを追加できるので、UI の作成が非常に容易することが可能！！

この技を使ってこんなことができる↓

![](../../img/compose11.png)

forとの合わせ技！

![](../../img/HerroAndroid2.png)

#### ボタンの追加

****

Button は、マテリアル パッケージによって提供されるコンポーザブルで、
最後の引数としてコンポーザブルを受け取ります。後置ラムダはかっこの外側に移動できるため、任意のコンテンツを子としてボタンに追加できます。たとえば、次のように Text を追加できます。

```kotlin
Button(
    onClick = { } // You'll learn about this callback later
) {
    Text("Show less")
}
```

composeにはマテリアルデザインの仕様に応じて、異なるタイプのButtonが用意されている。
(Button、OutlinedButton、TextButton)

#### composeの状態

****

ボタンを押したら変化を起こしてみる

各アイテムが展開されているかどうかを示す値（つまり、アイテムの状態）をどこかに保存する必要がある

```kotlin
@Composable
private fun Greeting(name: String) {
    var expanded = false // Don't do this!

    Surface(
        color = MaterialTheme.colors.primary,
        modifier = Modifier.padding(vertical = 4.dp, horizontal = 8.dp)
    ) {
        Row(modifier = Modifier.padding(24.dp)) {
            Column(modifier = Modifier.weight(1f)) {
                Text(text = "Hello, ")
                Text(text = name)
            }
            OutlinedButton(
                onClick = { expanded = !expanded }
            ) {
                Text(if (expanded) "Show less" else "Show more")
            }
        }
    }
}
```


onClick アクションと動的ボタンテキストも追加

しかし、これは期待どおりに機能しません。expanded 変数に異なる値が設定されても、Compose はそれを「状態変更」として検出しないため、何も起こりません

再コンポジション・・・Compose アプリは、コンポーズ可能な関数を呼び出すことにより、データを UI に変換します。データが変更されると、Compose はそれらの関数を新しいデータで再実行し、 更新された UI を作成

composeの思想いわく「コンポーズ可能な関数は任意の順序で頻繁に実行される可能性があるため、コードが実行される順序、または関数が再コンポーズされる回数に依存しないようにしてください。」だそうです

### 状態ホイスティング

##### 個人的にほしい情報

****

```kotlin
 var shouldShowOnboarding by remember { mutableStateOf(true) }
```

- shouldShowOnboarding は、= キーワードではなく by キーワードを使用しています。これは、毎回 .value を入力する手間を省くためのプロパティ デリゲートです

- ComposeではUI要素を非表示にすることはありません。。非表示にするのではなく、単に UI 要素をコンポジションに追加しないようにして、それらの要素が Compose の生成する UI ツリーに追加されないようにします。
そのために、シンプルな Kotlin の条件ロジックを使用します。
たとえば、オンボーディング画面またはあいさつ文のリストを表示するには、次のようなコードを記述します。



