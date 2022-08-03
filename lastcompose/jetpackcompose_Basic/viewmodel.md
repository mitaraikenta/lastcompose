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

## composeとviewmodel

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








