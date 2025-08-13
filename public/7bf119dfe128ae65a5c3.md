---
title: Flutter で share_plus を利用してシェア機能を実装する
tags:
  - Android
  - Mobile
  - iOS
  - Dart
  - Flutter
private: false
updated_at: '2024-08-25T13:02:32+09:00'
id: 7bf119dfe128ae65a5c3
organization_url_name: null
slide: false
ignorePublish: false
---
# これなに
Flutter でのシェア機能実装の備忘録


# 利用するパッケージ
### 必須

今回の実装のキーパッケージ、OSのシェア機能を利用可能にする

https://pub.dev/packages/share_plus

### 任意

具体的な実装の際に画像ファイル取得のために利用しただけなので、ファイルピッカーから取得しても良いと思う

https://pub.dev/packages/screenshot


# モジュール化したコード
- `flutter pub add share_plus` でパッケージを取得
- メソッド化した以下のコードを呼び出して共有を実装
- 画像とシェアするときのメッセージにあたる image, subject は任意に設定
- text は必須にしている
```dart
import 'package:flutter/material.dart';
import 'package:share_plus/share_plus.dart';

class ShareImageAndTextController {
  void shareImageAndText(XFile? image, String? subject, String text) async {
    try {
      if (image != null) {
        await Share.shareXFiles(
          [
            image,
          ],
          subject: subject,
          text: text,
        );
      } else {
        await Share.share(
          text,
          subject: subject,
        );
      }
    } catch (e) {
      debugPrint('*** ${DateTime.now()} | shareImageAndText: error: $e');
    }
  }
}
```


# 具体的なメソッドの実行を行ったコード
- `demo_share_project` というサンプルプロジェクトを作成しました
- このままコピペするだけだと呼び出せません
- インポートパスは任意のものに変更して利用してください

### main.dart

```dart
import 'package:flutter/material.dart';
import 'package:demo_share_project/view/my_home_page_view.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Demo app to share',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const MyHomePage(title: 'Demo app to share!'),
    );
  }
}
```

### convert_image_data_controller.dart

```dart
import 'dart:io';
import 'dart:typed_data';
import 'package:share_plus/share_plus.dart';
import 'package:path_provider/path_provider.dart';

class ConvertImageDataController {
  Future<XFile> convertImageData(Uint8List? imageData) async {
    XFile res = XFile('');

    if (imageData != null) {
      final directory = await getTemporaryDirectory();
      final imagePath = '${directory.path}/screenshot.png';
      final imageFile = File(imagePath);
      await imageFile.writeAsBytes(imageData);
      res = XFile(imagePath);
    }

    return res;
  }
}
```

### my_home_page_view.dart

```dart
import 'package:flutter/material.dart';
import 'package:screenshot/screenshot.dart';
import 'package:demo_share_project/controller/convert_image_data_controller.dart';
import 'package:demo_share_project/controller/share_image_and_text_controller.dart';

class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key, required this.title});

  final String title;

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  final ScreenshotController _screenShotController = ScreenshotController();
  final ShareImageAndTextController _shareImageAndTextController =
      ShareImageAndTextController();
  final ConvertImageDataController _convertImageDataController =
      ConvertImageDataController();

  @override
  Widget build(BuildContext context) {
    return Screenshot(
      controller: _screenShotController,
      child: Scaffold(
        appBar: AppBar(
          backgroundColor: Theme.of(context).colorScheme.inversePrimary,
          title: Text(widget.title),
        ),
        body: Center(
          child: ElevatedButton(
            child: const Text('Take screenshot and share'),
            onPressed: () async {
              final screenshot = await _screenShotController.capture(
                delay: const Duration(milliseconds: 500),
              );
              final imageXfile = await _convertImageDataController
                  .convertImageData(screenshot);

              _shareImageAndTextController.shareImageAndText(
                  imageXfile, 'Test subject!', 'Test to share!');
            },
          ),
        ),
      ),
    );
  }
}
```


# 実際のスクリーンショット

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/7e4d3a6e-b752-96ce-f417-3e67e9a32d08.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/3377c9da-7938-e5c6-f8d4-0f2183699168.png)


# サンプルリポジトリ
今回ついでに作成したので公開しておきます。
ご自由にご利用ください。

https://github.com/su3-hokkaido/demo_share_project

