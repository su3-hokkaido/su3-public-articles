---
title: 'Error: error:0308010C:digital envelope routines::unsupported エラーが出た時の解決方法'
tags:
  - JavaScript
  - Node.js
  - npm
  - TypeScript
  - webpack
private: false
updated_at: '2024-12-01T16:47:24+09:00'
id: cd8b34f72e363877caf2
organization_url_name: null
slide: false
ignorePublish: false
---
# これなに

- `npm start` でサーバー起動しようとしたら `Error: error:0308010C:digital envelope routines::unsupported` というエラーが出た
- その時の解決方法をメモ

# 原因

- Node.js のバージョンと Webpack の互換性の問題
- Node.js v17 以降で OpenSSL の変更が原因で発生するようになった模様で、v16 以前は発生しなかったらしい

# 解決方法

## その１：npm_modules フォルダを削除して再インストール

- `npm_modules`: フォルダを削除する
- `npm install`: で再インストール

## その２：react-scripts を最新版にアップグレード

- `npm install react-scripts@latest`

## その３：Node v16 にダウングレード

- `nvm install 16`: node v16 インストール
- `nvm use 16`: node v16 に切り替え

## その４：OpenSSL レガシーサポートを有効にする

- `export NODE_OPTIONS=--openssl-legacy-provider`


# 補足

## StackOverflow でのディスカッション

欲しい情報はこちらに全部載っていた

https://stackoverflow.com/questions/69692842/error-message-error0308010cdigital-envelope-routinesunsupported

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2819748/10efb08e-0c31-186a-8ca3-93cb8d10a540.png)

## 実際のエラーログ

```zsh
Starting the development server...

Error: error:0308010C:digital envelope routines::unsupported
    at new Hash (node:internal/crypto/hash:80:19)
    at Object.createHash (node:crypto:139:10)
    at module.exports (/Users/su3-hokkaido/Github/react_sample/node_modules/webpack/lib/util/createHash.js:90:53)
    at NormalModule._initBuildHash (/Users/su3-hokkaido/Github/react_sample/node_modules/webpack/lib/NormalModule.js:401:16)
    at handleParseError (/Users/su3-hokkaido/Github/react_sample/node_modules/webpack/lib/NormalModule.js:449:10)
    at /Users/su3-hokkaido/Github/react_sample/node_modules/webpack/lib/NormalModule.js:481:5
    at /Users/su3-hokkaido/Github/react_sample/node_modules/webpack/lib/NormalModule.js:342:12
    at /Users/su3-hokkaido/Github/react_sample/node_modules/loader-runner/lib/LoaderRunner.js:373:3
    at iterateNormalLoaders (/Users/su3-hokkaido/Github/react_sample/node_modules/loader-runner/lib/LoaderRunner.js:214:10)
    at iterateNormalLoaders (/Users/su3-hokkaido/Github/react_sample/node_modules/loader-runner/lib/LoaderRunner.js:221:10)
    at /Users/su3-hokkaido/Github/react_sample/node_modules/loader-runner/lib/LoaderRunner.js:236:3
    at runSyncOrAsync (/Users/su3-hokkaido/Github/react_sample/node_modules/loader-runner/lib/LoaderRunner.js:130:11)
    at iterateNormalLoaders (/Users/su3-hokkaido/Github/react_sample/node_modules/loader-runner/lib/LoaderRunner.js:232:2)
    at Array.<anonymous> (/Users/su3-hokkaido/Github/react_sample/node_modules/loader-runner/lib/LoaderRunner.js:205:4)
    at Storage.finished (/Users/su3-hokkaido/Github/react_sample/node_modules/enhanced-resolve/lib/CachedInputFileSystem.js:55:16)
    at /Users/su3-hokkaido/Github/react_sample/node_modules/enhanced-resolve/lib/CachedInputFileSystem.js:91:9
=============

WARNING: You are currently running a version of TypeScript which is not officially supported by typescript-estree.

You may find that it works just fine, or you may not.

SUPPORTED TYPESCRIPT VERSIONS: >=3.2.1 <3.5.0

YOUR TYPESCRIPT VERSION: 4.9.5

Please only submit bug reports when using the officially supported version.

=============
/Users/su3-hokkaido/Github/react_sample/node_modules/react-scripts/scripts/start.js:19
  throw err;
  ^

Error: error:0308010C:digital envelope routines::unsupported
    at new Hash (node:internal/crypto/hash:80:19)
    at Object.createHash (node:crypto:139:10)
    at module.exports (/Users/su3-hokkaido/Github/react_sample/node_modules/webpack/lib/util/createHash.js:90:53)
    at NormalModule._initBuildHash (/Users/su3-hokkaido/Github/react_sample/node_modules/webpack/lib/NormalModule.js:401:16)
    at /Users/su3-hokkaido/Github/react_sample/node_modules/webpack/lib/NormalModule.js:433:10
    at /Users/su3-hokkaido/Github/react_sample/node_modules/webpack/lib/NormalModule.js:308:13
    at /Users/su3-hokkaido/Github/react_sample/node_modules/loader-runner/lib/LoaderRunner.js:367:11
    at /Users/su3-hokkaido/Github/react_sample/node_modules/loader-runner/lib/LoaderRunner.js:233:18
    at context.callback (/Users/su3-hokkaido/Github/react_sample/node_modules/loader-runner/lib/LoaderRunner.js:111:13)
    at /Users/su3-hokkaido/Github/react_sample/node_modules/babel-loader/lib/index.js:51:103
    at process.processTicksAndRejections (node:internal/process/task_queues:95:5) {
  opensslErrorStack: [
    'error:03000086:digital envelope routines::initialization error',
    'error:0308010C:digital envelope routines::unsupported'
  ],
  library: 'digital envelope routines',
  reason: 'unsupported',
  code: 'ERR_OSSL_EVP_UNSUPPORTED'
}

Node.js v20.12.1
% 
```


# あとがき

擦り切れるほどに質問がたくさんインターネットに転がっていたので今更感ありますが、僕は知らなかったのでメモしました！（迫真）
