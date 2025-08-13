---
title: Flutter で Firebase.initializeApp() が iOS でうまく動かなかった話
tags:
  - iOS
  - Dart
  - Swift
  - Firebase
  - Flutter
private: false
updated_at: '2024-04-22T01:15:15+09:00'
id: 5b9a99f7918bc3876dcd
organization_url_name: null
slide: false
ignorePublish: false
---
# これなに
Android で Firebase が使えるようになったので、 iOS でも試そうと思ったら Firebase 初期化処理で引っかかってしまったので解決した方法をメモした。


# 原因
- VisualStudio で `GoogleService-Info.plist` を `ios/Runner/` に貼り付けたことが原因らしい。
- Xcode で貼り付けないと `project.pbxproj` の更新がされず、正しく plist ファイルを読み込むことができなかったっぽい。


# 解決方法
- Xcode で `ios/` を開く。（FLutter プロジェクト直下ではなく iOS まで潜る）
- Xcode 上で `GoogleService-Info.plist` を `ios/Runner/` に貼り付ける。
- これをもって flutter run でビルドし直すとちゃんと動くようになった。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/a86026d7-beae-bd90-7bf1-e3c8862aa117.png)
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/18de3573-d27e-9ebf-5e6a-7c3d600ab291.png)



# 参考：解決までに遭遇したエラー
### Firebase 初期化処理が終わる前に Firebase のメソッドをコール？
なにもないけど、このエラーが出てきた。
```zsh
FirebaseException ([core/not-initialized] Firebase has not been correctly initialized.

Usually this means you've attempted to use a Firebase service before calling `Firebase.initializeApp`.

View the documentation for more information: https://firebase.flutter.dev/docs/overview#initialization
    )
```

### Firebase の再定義エラー
複数定義がある時に出るっぽいエラーではあったものの、そんなものないよ？ってなって頭にハテナマークが浮かんでいた。
```zsh
Failed to build iOS app
Error (Xcode): redefinition of module 'Firebase'
/Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Firebase/CoreOnly/Sources/module.modulemap:0:7


Error (Xcode): failed to emit precompiled header
'/Users/su/Library/Developer/Xcode/DerivedData/Runner-bhnjccbzxqqqqjawkrnzkwnxtbpx/Build/Intermediates.noindex/PrecompiledHeaders/Runner-Bridging-Header-swift_14UCLV31P57KM-clang_2OZP1VBCBKMH4.pch' for bridging header
'/Users/su3-hokkaido/Github-su3/myapp/ios/Runner/Runner-Bridging-Header.h'


Could not build the application for the simulator.
Error launching application on iPhone 15 Pro Max.
```
