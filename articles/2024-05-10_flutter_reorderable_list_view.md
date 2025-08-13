---
title: "【Flutter】 ReorderableListView を使ってちょっと戸惑った話" # 記事のタイトル
emoji: "📝" # Choose an appropriate emoji for each article
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["Dart", "TextView", "ListView", "Flutter", "ReorderableListView"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---
# これなに
Flutter で ListView 並べ替えしたいなあと思ってたところ、公式に ReorderableListView があるということで使ってみた。
しかし、実はそのままだと並べ替えた時に値がその場所から変わらなくて困ったので色々勉強したメモ。


# 遭遇した事象
なんでだよぉ、なんで入れ替えようとしているのに入れ替わらないんだよおおおおお
![変わらないよお.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/8f0f5dad-fde7-90e5-c5a3-2cc2dd5b63aa.gif)


# 完成したコード
とりあえずコードコピペしたい人用にコードだけ置いておきます。

### Sample 1: とりあえず並べ替えだけできるようにしたもの
![変わったよお.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/2cbf246e-ba08-695d-0350-77323eb6a716.gif)

```dart
// Ref:
// https://api.flutter.dev/flutter/material/ReorderableListView-class.html
// https://api.flutter.dev/flutter/material/ReorderableListView/buildDefaultDragHandles.html
// https://stackoverflow.com/questions/72371330/flutter-textfield-does-not-follow-listtiles-within-reorderablelistview
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  static const String _title = 'Flutter Sample';

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: _title,
      home: Scaffold(
        appBar: AppBar(title: const Text(_title)),
        body: ReorderableListViewWithTextField(),
      ),
    );
  }
}

class Item {
  int? item;
  String? text;

  Item(this.item, this.text);
}

class ReorderableListViewWithTextField extends StatefulWidget {
  State<ReorderableListViewWithTextField> createState() =>
      _ReorderableListViewWithTextFieldState();
}

class _ReorderableListViewWithTextFieldState
    extends State<ReorderableListViewWithTextField> {
  final List<Item> _listItems =
      List<Item>.generate(50, (int index) => Item(index, ''));

  @override
  Widget build(BuildContext context) {
    final ColorScheme colorScheme = Theme.of(context).colorScheme;
    final Color oddItemColor = colorScheme.primary.withOpacity(0.05);
    final Color evenItemColor = colorScheme.primary.withOpacity(0.15);

    return ReorderableListView(
      padding: const EdgeInsets.symmetric(horizontal: 40),
      children: <Widget>[
        for (int index = 0; index < _listItems.length; index += 1)
          ListTile(
            key: ObjectKey(_listItems[index]),
            tileColor:
                _listItems[index].item!.isOdd ? oddItemColor : evenItemColor,
            title: Text(
                'Item ${_listItems[index].item}: ${_listItems[index].text}'),
            subtitle: TextFormField(
              initialValue: _listItems[index].text,
              onChanged: (String? value) {
                setState(() {
                  _listItems[index].text = value!;
                });
              },
              decoration: const InputDecoration(
                border: OutlineInputBorder(),
              ),
            ),
          ),
      ],
      onReorder: (int oldIndex, int newIndex) {
        setState(() {
          if (oldIndex < newIndex) {
            newIndex -= 1;
          }
          final Item listItem = _listItems.removeAt(oldIndex);
          _listItems.insert(newIndex, listItem);
        });
      },
    );
  }
}

```


### Sample 2: Dragger を好きなアイコンに変えてみたもの
![カスタムしたよお.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/07fe6aff-dc6a-2f6d-4c4a-7a4b13fc23d4.gif)

```dart
// Ref:
// https://api.flutter.dev/flutter/material/ReorderableListView-class.html
// https://api.flutter.dev/flutter/material/ReorderableListView/buildDefaultDragHandles.html
// https://stackoverflow.com/questions/72371330/flutter-textfield-does-not-follow-listtiles-within-reorderablelistview
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  static const String _title = 'Flutter Sample';

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: _title,
      home: Scaffold(
        appBar: AppBar(title: const Text(_title)),
        body: ReorderableListViewWithTextFieldAndCustomDragger(),
      ),
    );
  }
}

class Item {
  int? item;
  String? text;

  Item(this.item, this.text);
}

class ReorderableListViewWithTextFieldAndCustomDragger extends StatefulWidget {
  State<ReorderableListViewWithTextFieldAndCustomDragger> createState() =>
      _ReorderableListViewWithTextFieldAndCustomDraggerState();
}

class _ReorderableListViewWithTextFieldAndCustomDraggerState
    extends State<ReorderableListViewWithTextFieldAndCustomDragger> {
  final List<Item> _listItems =
      List<Item>.generate(50, (int index) => Item(index, ''));

  @override
  Widget build(BuildContext context) {
    final ColorScheme colorScheme = Theme.of(context).colorScheme;
    final Color oddItemColor = colorScheme.primary.withOpacity(0.05);
    final Color evenItemColor = colorScheme.primary.withOpacity(0.15);

    return ReorderableListView(
      padding: const EdgeInsets.symmetric(horizontal: 40),
      buildDefaultDragHandles: false,
      children: <Widget>[
        for (int index = 0; index < _listItems.length; index += 1)
          ListTile(
            key: ObjectKey(_listItems[index]),
            tileColor:
                _listItems[index].item!.isOdd ? oddItemColor : evenItemColor,
            title: Text(
                'Item ${_listItems[index].item}: ${_listItems[index].text}'),
            subtitle: Row(children: [
              Expanded(
                  child: TextFormField(
                initialValue: _listItems[index].text,
                onChanged: (String? value) {
                  setState(() {
                    _listItems[index].text = value!;
                  });
                },
                decoration: const InputDecoration(
                  border: OutlineInputBorder(),
                ),
              )),
              Padding(padding: const EdgeInsets.all(5.0)),
              SizedBox(
                  height: 40,
                  width: 40,
                  child: ReorderableDragStartListener(
                      index: index, child: const Icon(Icons.drag_handle))),
            ]),
          ),
      ],
      onReorder: (int oldIndex, int newIndex) {
        setState(() {
          if (oldIndex < newIndex) {
            newIndex -= 1;
          }
          final Item listItem = _listItems.removeAt(oldIndex);
          _listItems.insert(newIndex, listItem);
        });
      },
    );
  }
}
```

# 何を書いていたかのメモ
### 並べ替えには ReorderableListView を使うこと
以下の公式ページで紹介されているコードをコピペで使ってもらえばとりあえずは並べ替えができる ListView が作れます。

https://api.flutter.dev/flutter/material/ReorderableListView-class.html

```dart
import 'dart:ui';

import 'package:flutter/material.dart';

/// Flutter code sample for [ReorderableListView].

void main() => runApp(const ReorderableApp());

class ReorderableApp extends StatelessWidget {
  const ReorderableApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('ReorderableListView Sample')),
        body: const ReorderableExample(),
      ),
    );
  }
}

class ReorderableExample extends StatefulWidget {
  const ReorderableExample({super.key});

  @override
  State<ReorderableExample> createState() => _ReorderableExampleState();
}

class _ReorderableExampleState extends State<ReorderableExample> {
  final List<int> _items = List<int>.generate(50, (int index) => index);

  @override
  Widget build(BuildContext context) {
    final ColorScheme colorScheme = Theme.of(context).colorScheme;
    final Color oddItemColor = colorScheme.secondary.withOpacity(0.05);
    final Color evenItemColor = colorScheme.secondary.withOpacity(0.15);
    final Color draggableItemColor = colorScheme.secondary;

    Widget proxyDecorator(
        Widget child, int index, Animation<double> animation) {
      return AnimatedBuilder(
        animation: animation,
        builder: (BuildContext context, Widget? child) {
          final double animValue = Curves.easeInOut.transform(animation.value);
          final double elevation = lerpDouble(0, 6, animValue)!;
          return Material(
            elevation: elevation,
            color: draggableItemColor,
            shadowColor: draggableItemColor,
            child: child,
          );
        },
        child: child,
      );
    }

    return ReorderableListView(
      padding: const EdgeInsets.symmetric(horizontal: 40),
      proxyDecorator: proxyDecorator,
      children: <Widget>[
        for (int index = 0; index < _items.length; index += 1)
          ListTile(
            key: Key('$index'),
            tileColor: _items[index].isOdd ? oddItemColor : evenItemColor,
            title: Text('Item ${_items[index]}'),
          ),
      ],
      onReorder: (int oldIndex, int newIndex) {
        setState(() {
          if (oldIndex < newIndex) {
            newIndex -= 1;
          }
          final int item = _items.removeAt(oldIndex);
          _items.insert(newIndex, item);
        });
      },
    );
  }
}
```


### Sample 2 の `buildDefaultDragHandles: false` と `ReorderableDragStartListener`
`buildDefaultDragHandles` に false を指定することでデフォルトのドラッグハンドルの機能を無効化しています。
代わりにアイコンを設定して、`ReorderableDragStartListener` で並べ替えできるようにしています。
`buildDefaultDragHandles` については公式ページでも紹介されています。

https://api.flutter.dev/flutter/material/ReorderableListView/buildDefaultDragHandles.html

### `onReorder` >> `setState`
ここでは並べ替えの処理を行っています。
配列になっている _items に対して動きがあった添字の列に対して .removeAt して、入れ替え先のところの添字に .insert しているというのがこのロジックのようです。
```dart
      onReorder: (int oldIndex, int newIndex) {
        setState(() {
          if (oldIndex < newIndex) {
            newIndex -= 1;
          }
          final int item = _items.removeAt(oldIndex);
          _items.insert(newIndex, item);
        });
      },
```

### `newIndex -= 1` で何が起きているか
以下のようになっています。ここ何が起きているか図で確認します。
```dart
          if (oldIndex < newIndex) {
            newIndex -= 1;
          }
```

例えば D を B と C の間に移動するだけなら、添字をいじる必要はないことは図で見るとわかりやすいかもです。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/6cddc02d-3be8-ef5a-d7d6-1b1d12e079a3.png)

しかし、たとえば D を F と G に移動すると、3の添え字を持っていた列がいなくなったことで配列の添え字の番号が1つずつ減るという形になります。
それゆえ、もし oldIndex より newIndex が大きい場合、添え字を1つ減らすという作業が発生するということになります。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/49b1d582-b365-1119-345b-ac439a2edee2.png)


### Item class
大きい Item という単位でしかステートを保持していなかったものを丸めて TextFormField の値と一緒に持っていけるようにまとめてくれている。以下のページがとても参考になった。

https://stackoverflow.com/questions/72371330/flutter-textfield-does-not-follow-listtiles-within-reorderablelistview

```dart
class Item {
  int? item;
  String? text;

  Item(this.item, this.text);
}
```

# 最後に
この ReorderableListView との戦いで多分3日間くらい使った気がする…
結構大変でした
