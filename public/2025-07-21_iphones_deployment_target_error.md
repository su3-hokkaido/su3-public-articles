---
title: >-
  Flutter iOS Simulator を起動しようとしたら Simulator deployment target
  'IPHONEOS_DEPLOYMENT_TARGET' is set to 10.0, but the range of supported
  deployment target versions is 12.0 to 18.5.99 というエラーになった
tags:
  - iOS
  - Dart
  - Simulator
  - Flutter
  - モバイルアプリ
private: false
updated_at: '2025-07-21T20:35:51+09:00'
id: 607e51627ba9bf9ff50a
organization_url_name: null
slide: false
ignorePublish: false
---
# これなに

約7ヶ月振りに Flutter アプリを iOS Simulator で起動しようとしたら以下のエラーが大量に出た。
その時の解決方法をメモします。
なお、この記事は自分が解決した時のメモであり、同様の事象が発生した方々全てで解決できるかどうかは保証していません。

```zsh
    flutter run --debug --device-id 7266AD1A-89DD-471F-886D-CF3C643D33B2
    ...
    /myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 10.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target
    'gRPC-C++-gRPCCertificates-Cpp' from project 'Pods')
```

# 原因

- Pod に記載があるデプロイメントターゲットとプロジェクトデプロイメントターゲットが異なっていることが原因のようでした。
- しかし、修正箇所はかなり多く、一つひとつを手作業で修正するのは時間がかかる模様です。

# 解決方法

### 修正概要

[こちらの Stackflow の記事](https://stackoverflow.com/questions/54704207/the-ios-simulator-deployment-targets-is-set-to-7-0-but-the-range-of-supported-d)を参考にしましたが、要するに以下のことを実施すれば良いとのこと


- 指定の iOS バージョンを強制指定するようにコードを書き換える（上記エラーで言うと iOS 12 を強制指定する
- cocoapods を再インストール（こちらは念のための作業）

### コード修正内容

- 上記 Stackflow にも記載がありましたが、 `ios/Podfile` で `config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '12.0'` を直接指定することで修正できました
- また、デプロイ先の同一IDのデバイスが複数あるという警告メッセージがあったので、それを除外するように `config.build_settings['EXCLUDED_ARCHS[sdk=iphonesimulator*]'] = 'i386'` を加えています
- その他にも、gRPCコンパイルエラーがあったのでそちらも修正を行っています
- 詳細のエラーログは本記事の付録セクションに記載しました

```zsh
post_install do |installer|
  installer.pods_project.targets.each do |target|
    ############################################################
    # ▼▼▼ 修正開始
    ############################################################
    target.build_configurations.each do |config|
      # Only exclude i386 for iOS Simulator (keep arm64 for Apple Silicon Macs)
      config.build_settings['EXCLUDED_ARCHS[sdk=iphonesimulator*]'] = 'i386'
      config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '12.0'

      # Global C++ settings to fix gRPC compilation issues
      config.build_settings['CLANG_CXX_LANGUAGE_STANDARD'] = 'c++17'
      config.build_settings['CLANG_CXX_LIBRARY'] = 'libc++'
      config.build_settings['WARNING_CFLAGS'] = [
        '-Wno-missing-template-arg-list-after-template-kw',
        '-Wno-deprecated-declarations'
      ].join(' ')
      config.build_settings['OTHER_CPLUSPLUSFLAGS'] = [
        '$(OTHER_CFLAGS)',
        '-fcxx-exceptions',
        '-frtti',
        '-Wno-missing-template-arg-list-after-template-kw'
      ].join(' ')
    end
    ############################################################
    # ▲▲▲ 修正終了
    ############################################################

    flutter_additional_ios_build_settings(target)

```

# さいごに

新しい MacBook に端末を買い替えてからの初めてのデプロイをしたら上記のようなエラーになってしまいました…
実際には元々の端末で何かしら config ファイルがあって正常に動作していたのかもしれないのですが、一旦こちらで解決できたようなのでよかったです。

# 付録：ログ全文

上述の内容は必要な内容だけを抜粋したものです。
参考までに実際に遭遇したエラーログを全部掲載します。

```zsh
su3-hokkaido@Sus-MacBook-Air myapp % flutter run --debug -d 7266AD1A-89DD-471F-886D-CF3C643D33B2
Launching lib/main.dart on iPhone 16 Plus in debug mode...
Updating minimum iOS deployment target to 12.0.
Upgrading Podfile
Running pod install...                                             22.9s
Running Xcode build...                                                  
Xcode build done.                                           21.6s
Failed to build iOS app
Error output from Xcode build:
↳
    --- xcodebuild: WARNING: Using the first of multiple matching destinations:
    { platform:iOS Simulator, arch:arm64, id:7266AD1A-89DD-471F-886D-CF3C643D33B2, OS:18.5,
    name:iPhone 16 Plus }
    { platform:iOS Simulator, arch:x86_64, id:7266AD1A-89DD-471F-886D-CF3C643D33B2, OS:18.5,
    name:iPhone 16 Plus }
    ** BUILD FAILED **


Xcode's output:
↳
    Writing result bundle at path:
        /var/folders/r2/vgh07ztd4rl2bp93nh_fl4zh0000gn/T/flutter_tools.8Mlb1B/flutter_ios_build_t
        emp_dirdtWdGD/temporary_xcresult_bundle

    In file included from
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/gRPC-Core/src/core/load_balancing/xds/xds
    _wrr_locality.cc:49:
    In file included from
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/gRPC-Core/src/core/load_balancing/delegat
    ing_helper.h:37:
    In file included from
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/gRPC-Core/src/core/lib/security/credentia
    ls/credentials.h:45:
    In file included from
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/gRPC-Core/src/core/lib/transport/transpor
    t.h:56:
    In file included from
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/gRPC-Core/src/core/lib/promise/pipe.h:41:
    In file included from
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/gRPC-Core/src/core/lib/promise/seq.h:25:
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/gRPC-Core/src/core/lib/promise/detail/bas
    ic_seq.h:103:38: error: a template argument list is expected after a name prefixed by the
    template keyword [-Wmissing-template-arg-list-after-template-kw]
      103 |                     Traits::template CallSeqFactory(f_, *cur_, std::move(arg)));
          |                                      ^
    1 error generated.
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 11.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target
    'leveldb-library-leveldb_Privacy' from project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 9.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target
    'nanopb-nanopb_Privacy' from project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 9.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target 'abseil-xcprivacy'
    from project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 9.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target
    'app_links-app_links_ios_privacy' from project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 10.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target
    'GTMSessionFetcher-GTMSessionFetcher_Core_Privacy' from project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 9.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target
    'geolocator_apple-geolocator_apple_privacy' from project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 9.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target
    'geocoding_ios-geocoding_ios_privacy' from project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 9.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target
    'flutter_native_splash-flutter_native_splash_privacy' from project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 9.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target
    'flutter_email_sender-flutter_email_sender' from project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 11.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target 'RecaptchaInterop'
    from project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 9.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target
    'PromisesObjC-FBLPromises_Privacy' from project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 11.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target 'FirebaseSharedSwift'
    from project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 10.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target
    'FirebaseAppCheckInterop' from project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 10.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target
    'BoringSSL-GRPC-openssl_grpc' from project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 9.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target
    'AppAuth-AppAuthCore_Privacy' from project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 9.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target 'PromisesObjC' from
    project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 10.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target
    'GTMSessionFetcher-GTMSessionFetcher_Full_Privacy' from project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 10.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target 'BoringSSL-GRPC' from
    project 'Pods')
    warning: Run script build phase 'Create Symlinks to Header Folders' will be run during
    every build because it does not specify any outputs. To address this issue, either add
    output dependencies to the script phase, or configure it to run in every build by
    unchecking "Based on dependency analysis" in the script phase. (in target 'BoringSSL-GRPC'
    from project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 9.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target 'abseil' from project
    'Pods')
    warning: Run script build phase 'Create Symlinks to Header Folders' will be run during
    every build because it does not specify any outputs. To address this issue, either add
    output dependencies to the script phase, or configure it to run in every build by
    unchecking "Based on dependency analysis" in the script phase. (in target 'abseil' from
    project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 9.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target 'nanopb' from project
    'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 11.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target 'leveldb-library' from
    project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 9.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target 'AppAuth' from project
    'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 9.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target
    'GoogleUtilities-GoogleUtilities_Privacy' from project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 10.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target 'GTMSessionFetcher'
    from project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 10.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target 'gRPC-Core-grpc' from
    project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 9.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target 'GoogleUtilities' from
    project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 10.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target
    'GTMAppAuth-GTMAppAuth_Privacy' from project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 10.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target 'gRPC-Core' from
    project 'Pods')
    warning: Run script build phase 'Create Symlinks to Header Folders' will be run during
    every build because it does not specify any outputs. To address this issue, either add
    output dependencies to the script phase, or configure it to run in every build by
    unchecking "Based on dependency analysis" in the script phase. (in target 'gRPC-Core' from
    project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 10.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target
    'GoogleSignIn-GoogleSignIn' from project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 10.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target 'FirebaseCore' from
    project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 10.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target 'GoogleSignIn' from
    project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 11.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target 'Firebase' from
    project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 10.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target
    'FirebaseCore-FirebaseCore_Privacy' from project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 10.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target
    'FirebaseCoreExtension-FirebaseCoreExtension_Privacy' from project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 11.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target 'FirebaseAuth' from
    project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 11.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target
    'FirebaseAuth-FirebaseAuth_Privacy' from project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 10.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target
    'FirebaseCoreExtension' from project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 10.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target 'GTMAppAuth' from
    project 'Pods')
    note: Run script build phase 'Run Script' will be run during every build because the
    option to run the script phase "Based on dependency analysis" is unchecked. (in target
    'Runner' from project 'Runner')
    note: Run script build phase 'Thin Binary' will be run during every build because the
    option to run the script phase "Based on dependency analysis" is unchecked. (in target
    'Runner' from project 'Runner')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 10.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target 'gRPC-C++' from
    project 'Pods')
    warning: Run script build phase 'Create Symlinks to Header Folders' will be run during
    every build because it does not specify any outputs. To address this issue, either add
    output dependencies to the script phase, or configure it to run in every build by
    unchecking "Based on dependency analysis" in the script phase. (in target 'gRPC-C++' from
    project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 11.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target
    'FirebaseFirestore-FirebaseFirestore_Privacy' from project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 10.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target 'FirebaseCoreInternal'
    from project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 11.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target
    'FirebaseFirestoreInternal-FirebaseFirestoreInternal_Privacy' from project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 10.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target
    'FirebaseCoreInternal-FirebaseCoreInternal_Privacy' from project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 11.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target 'FirebaseFirestore'
    from project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 11.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target
    'FirebaseFirestoreInternal' from project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 10.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target 'gRPC-C++-grpcpp' from
    project 'Pods')
    /Users/su3-hokkaido/Github-su3/myapp/ios/Pods/Pods.xcodeproj: warning: The iOS
    Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 10.0, but the range of
    supported deployment target versions is 12.0 to 18.5.99. (in target
    'gRPC-C++-gRPCCertificates-Cpp' from project 'Pods')

Could not build the application for the simulator.
Error launching application on iPhone 16 Plus.
su3-hokkaido@Sus-MacBook-Air myapp % 
```
