# viewmodel概要

****

アプリを落とす前に追加したデータを更新や削除することができない

## 解決策

わからん

調べてもでてこねえから基礎を学び直すしかなし

jetpackcomposeの状態および高度な状態と副作用を徹底的に学ぶ

## 単方向データフロー

****

### Android状態の更新
Androidアプリでは、イベントに応答して状態が更新される

イベントとはアプリ外で生成された入力であり、OnClickListener を呼び出すボタンをユーザーがタップする操作、
EditText による afterTextChanged の呼び出し、加速度計による新しい値の送信など

### UI更新ループ

****

すべてのAndroidアプリは、このUI更新ループを備えています

![](../../img/android100.png)

- イベント- ユーザーまたはプログラムの要素によってイベントが生成されます
- 状態の更新 - イベント　ハイドラがUIで使用される状態を変更
- 状態の表示 - UIが更新され、新しい状態を表示する

### composeとviewmodel

****

最終的にこんなアプリができます

![](../../img/compose12.png)

Todoscrreen.kt

```kotlin
@Composable
fun TodoScreen(
   items: List<TodoItem>,
   onAddItem: (TodoItem) -> Unit,
   onRemoveItem: (TodoItem) -> Unit
) {
   /* ... */
}
```

このコンポーザブルは、編集可能な TODO リストを表示しますが、それ自体の状態はありません。変更可能な値が状態でしたが、TodoScreen の引数はどれも変更できません。

- items 画面に表示するアイテムの不変のリスト
- onAddItem ユーザーがアイテムの追加をリクエストしたときのイベント
- onRemoveItem ユーザーがアイテムの削除をリクエストしたときのイベント

このcomposableはステートレスなので、渡されたアイテムリストが表示されるだけで、リストを直接編集不可

じゃあどうやるかって？変更をリクエストできる 2 つのイベント onRemoveItem と onAddItem が渡せばおｋ

ステートレスな場合に編集可能なリストの表示方法は*状態ホイスティング*と呼ばれる技を使え！！

状態ホイスティングとは、コンポーネントをステートレスにするために状態を移動するパターンです。ステートレスなコンポーザブルはテストが簡単で、通常はバグが少なく、再利用の機会を数多く提供

上記のパラメータの組み合わせで、呼び出し元がこのコンポーザブルから状態をホイスティングできるようになることがわかります

- イベント - ユーザーがアイテムの追加または削除をリクエストしたときに、TodoScreen が onAddItem または onRemoveItem を呼び出します。
- 状態の更新 – TodoScreen の呼び出し元で状態を更新することにより、上記のイベントに応答できます。
- 状態の表示 - 状態が更新されると、新しい items とともに TodoScreen が再度呼び出され、新しいアイテムが画面に表示されます。

### TodoActivityScreen コンポーザブルを定義する

****

TodoViewModel.kt を開き、1 つの状態変数と 2 つのイベントを定義している ViewModel を確認

Todoviewmodel

```kotlin

class TodoViewModel : ViewModel() {

    // state: todoItems
    private var _todoItems = MutableLiveData(listOf<TodoItem>())
    val todoItems: LiveData<List<TodoItem>> = _todoItems

    // event: addItem
    fun addItem(item: TodoItem) {
        /* ... */
    }

    // event: removeItem
    fun removeItem(item: TodoItem) {
        /* ... */
    }
}
```

このviewmodelを使いTodoScreenから状態ホイスティングを行う。これで次のような単方向データフロー設計を作成

![](../../img/compose13.png)

todoscreenをtodoActivityに統合するためにTodoActivity.kt を開き、新しい @Composable 関数 TodoActivityScreen(todoViewModel: TodoViewModel) を定義して、それを onCreate の中の setContent から呼び出します。

TodoActivity.kt

```kotlin

import androidx.compose.runtime.Composable

class TodoActivity : AppCompatActivity() {

   private val todoViewModel by viewModels<TodoViewModel>()

   override fun onCreate(savedInstanceState: Bundle?) {
       super.onCreate(savedInstanceState)
       setContent {
           StateCodelabTheme {
               Surface {
                   TodoActivityScreen(todoViewModel)
               }
           }
       }
   }
}

@Composable
private fun TodoActivityScreen(todoViewModel: TodoViewModel) {
   val items = listOf<TodoItem>() // in the next steps we'll complete this
   TodoScreen(
       items = items,
       onAddItem = { }, // in the next steps we'll complete this
       onRemoveItem = { } // in the next steps we'll complete this
   )
}
```

このcomposableにより、viewmodelに格納されている状態と、プロジェクトで定義済みのTodoscreenコンポーサブルの橋渡しを行います。


を直接取得するように TodoScreen を変更することは可能ですが、TodoScreen が少し再利用しにくくなります。List<TodoItem> のような単純なパラメータを選ぶことで、TodoScreen と状態がホイストされる場所とを切り離すことができます

### イベントを上に流す

****

必要なコンポーネント(ViewModel、ブリッジ コンポーザブル TodoActivityScreen、TodoScreen)が
が揃ったので、すべてを接続し、単方向データフローを使用する動的なリストを表示します

TodoActivityScreen で、ViewModel から addItem と removeItem を渡します。

TodoActivity.kt

```kotlin

@Composable
private fun TodoActivityScreen(todoViewModel: TodoViewModel) {
   val items = listOf<TodoItem>()
   TodoScreen(
       items = items,
       onAddItem = { todoViewModel.addItem(it) },
       onRemoveItem = { todoViewModel.removeItem(it) }
   )
}
```
TodoScreen に渡されるイベントでは、Kotlin のラムダ構文を使用

TodoScreen が onAddItem または onRemoveItem を呼び出すとき、ViewModel の適切なイベントへの呼び出しを渡すことができます。

### 状態を下に渡す

****

単方向データフローのイベントを接続し終わったので、次は状態を下に渡す必要があります

TodoActivityScreen を編集し、observeAsState を使用して todoItems LiveData を監視するようにします


TodoActivity

```kotlin
import androidx.compose.runtime.getValue
import androidx.compose.runtime.livedata.observeAsState

@Composable
private fun TodoActivityScreen(todoViewModel: TodoViewModel) {
   val items: List<TodoItem> by todoViewModel.todoItems.observeAsState(listOf())
   TodoScreen(
       items = items,
       onAddItem = { todoViewModel.addItem(it) },
       onRemoveItem = { todoViewModel.removeItem(it) }
   )
}
```

この行では、LiveData を監視し、現在の値を List<TodoItem> として直接使用できるようにします

解説

- val items: List<TodoItem> では、List<TodoItem> 型の変数 items を宣言しています。
- todoViewModel.todoItems は、ViewModel から取得した LiveData<List<TodoItem> です
- .observeAsState は、LiveData<T> を監視し、それを State<T> オブジェクトに変換して Compose が値の変更に反応できるようにします。
- listOf() は、LiveData の初期化前に null の結果となることを避けるための初期値です。渡されなかった場合、items は null 値許容の List<TodoItem>? になります。
- by は、Kotlin のプロパティ デリゲート構文であり、observeAsState の State<List<TodoItem>> のラッピングを自動的に解除して通常の List<TodoItem> にします。

observeAsState は LiveData を監視し、LiveData が変更されるたびに更新される State オブジェクトを返します。

コンポーザブルがコンポジションから削除されると、監視は自動的に停止されます

## Composeのメモリ

****

Compose が内部で状態を操作する仕組みを見てみましょう。

Compose がコンポーザブルを再度呼び出して画面が更新されることを確認しました。再コンポーズという処理です。再度 TodoScreen を呼び出すことで、動的リストを表示することができました。

```
ステートフルなコンポーザブルとは、時間とともに変化する可能性がある状態を所有するコンポーザブルです。
```

## ランダムデザイン

****

![](../../img/compose15.png)

このセクションでランダムデサインを作るのでまとめてみます

### コンポーザブルにランダム要素を加える

****

TodoScreen.ktを開き、TodoRowcomposableを確認

ここでtodoリストの内の１行を表す

randommTint()の新しいval iconAlphaを定義する。デザイナーの要求通り、0.3～０．９の浮動小数点。アイコンの色合いを設定

```kotlin
import androidx.compose.material.LocalContentColor

@Composable
fun TodoRow(todo: TodoItem, onItemClicked: (TodoItem) -> Unit, modifier: Modifier = Modifier) {
   Row(
       modifier = modifier
           .clickable { onItemClicked(todo) }
           .padding(horizontal = 16.dp, vertical = 8.dp),
       horizontalArrangement = Arrangement.SpaceBetween
   ) {
       Text(todo.task)
       val iconAlpha = randomTint()
       Icon(
           imageVector = todo.icon.imageVector,
           tint = LocalContentColor.current.copy(alpha = iconAlpha),
           contentDescription = stringResource(id = todo.icon.contentDescription)
       )
   }
}
```

アイコンの色合いがランダムになったお

## 再コンポーズの確認

****

先程のrandomTintは再コンポーズの処理で、リストが変更するたびに、画面上の各行に対してrandomTintが再度呼び出されていることがわかるんよね

再コンポーズ　- 新しい入力があるとコンポーザブルを再度呼び出して、コンポーズツリーを更新する処理です。

新しいリストとともにTodoScreenが再度呼び出されると、LazyColumnが画面上の全ての子を再コンポーズします、次にTodoRow が再度呼び出され、新しいランダムな色合いが作られます。←ここコピペ

今回のtodoScreenのツリー

![](../../img/compose14.png)

composeが初めてコンポジションを実行するとき、呼び出されたすべてのコンポーザブルのツリーが作成される

再コンポジション中は、呼び出された新しいコンポーザブルでツリーが更新される

TodoRowが再コンポーズされるたびにアイコンが更新されるのは、TodoRow に隠れた*副作用*があるためです。副作用とは、コンポーズ可能な関数の実行の外部に現れる変化を指します

コンポーズの再コンポーズでは副作用を発生させるべきではない

副作用はViewModel での状態の更新、Random.nextInt() の呼び出し、データベースへの書き込みは、どれも副作用です

### コンポーズ可能な関数にメモリを導入する

****

TodoRowが再コンポーズされるたびに色が変わらないようにするね

composeではなんとコンポジションツリーに値を保存できる！今回はtodoRowをアップデートしてiconAlphaをコンポジションツリーに保存できる
ようにする

それで使うのがremember。これでコンポーズ可能な関数にメモリが提供されます

```
rememberが計算した値はコンポジションツリーに保存され、rememberのキーが変更された場合にのみ再計算される

remember は、オブジェクトの private val プロパティと同じように、1 つのオブジェクト用のストレージを関数に割り当てていると考えることができます。

```

TodoScreen.kt

```kotlin
val iconAlpha: Float = remember(todo.id) { randomTint() }
Icon(
   imageVector = todo.icon.imageVector,
   tint = LocalContentColor.current.copy(alpha = iconAlpha),
   contentDescription = stringResource(id = todo.icon.contentDescription)
)
```

todoRowの新しいコンポーズツリーを見ると、iconAlphaが追加されてる

![](../../img/compose16.png)

再度アプリを実行すると、リストが変更されるたびに色合いが更新されることはなくなります

コンポーズが発生すると、rememberによって保存された以前の値がかえされるようになってる

rememberへの呼び出しを詳しく見てみると、todo.id が引数 key として渡されていることがわかります。

```kotlin
remember(todo.id) { randomTint() }
```

rememberの呼び出しは次の２つの部分で構成される

その１．キー引数-このrememberが使用する「キー」（かっこ内で渡される部分）。ここでは、キーとして todo.id を渡しています。

その２．計算 – 記憶する新しい値を計算するラムダ（後置ラムダで渡されます）。ここでは、randomTint() でランダムな値を計算します。


この

部分が初めてコンポーズされる際には、remember が常に randomTint を呼び出し、
次の再コンポジションのために結果を保存します。また、渡された todo.id も記録されます。
その後の再コンポーズの際には、randomTint の呼び出しがスキップされ、新しい todo.id が TodoRow に渡されない限り、保存された値が返されます。

```
コンポジションに保存されている値は、、呼び出し元のコンポーザブルがツリーから削除されるとすぐに削除されます。

また、呼び出し元のコンポーザブルがツリー内で移動した場合も、再度初期化されます。上位のアイテムを削除すると、LazyColumn で、この処理が発生します。
```

べき等なコンポーザブルは、同じ入力に対して常に同じ結果を生成し、再コンポーズ時に副作用を発生させません。

再コンポジションをサポートするためには、コンポーザブルをべき等にする必要があります。


まあ再婚ポーズは、べき等である必要があります。

randomTint の呼び出しを remember で囲むことで、ToDo アイテムが変更されない限り、再コンポーズ時のランダムの呼び出しがスキップされます。その結果、TodoRow には副作用がなくなり、同じ入力で再コンポーズするたびに常に同じ結果が生成されて、べき等になります。

### 保存した値を制御可能にする

****

ただし、チェックインする前に行うべきコードの変更が 1 つあります。今のところ、TodoRow の呼び出し元が色合いを指定する方法がありません。たとえば、製品担当バイス プレジデントがこの画面に気付いて、リリース直前にランダム デザインを取り除く修正が求められるなど、さまざまな理由で指定方法が必要になる可能性があります。

呼び出し元がこの値を制御できるように、remember の呼び出しを新しい iconAlpha パラメータのデフォルト引数に移動します。
```kotlin
@Composable
fun TodoRow(
   todo: TodoItem,
   onItemClicked: (TodoItem) -> Unit,
   modifier: Modifier = Modifier,
   iconAlpha: Float = remember(todo.id) { randomTint() }
) {
   Row(
       modifier = modifier
           .clickable { onItemClicked(todo) }
           .padding(horizontal = 16.dp)
           .padding(vertical = 8.dp),
       horizontalArrangement = Arrangement.SpaceBetween
   ) {
       Text(todo.task)
       Icon(
            imageVector = todo.icon.imageVector,
            tint = LocalContentColor.current.copy(alpha = iconAlpha),
            contentDescription = stringResource(id = todo.icon.contentDescription)
        )
   }
}
```

注意！！
````
remember はコンポジションに値を保存しますが、remember を呼び出したコンポーザブルが削除されると、その値は失われます。

つまり、LazyColumn など、子の追加や削除を行うコンポーザブル内に重要なものを保存する手段として、remember を使うべきではありません。

たとえば、短いアニメーションのアニメーション状態ならば、LazyColumn の子に保存しても安全ですが、ToDo タスクの完了状態は、ここに保存するとスクロール時に失われてしまいます。
````










