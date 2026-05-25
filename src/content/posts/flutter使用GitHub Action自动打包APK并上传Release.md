---
title: "flutter使用GitHub Action自动打包APK并上传Release"
description: "最近写了一个查看河农大充电桩使用情况的APP,用的flutter,写了一个Github action记录并分享一下"
published: 2025-02-17
tags:
  - github-actions
  - github
  - flutter
draft: false
toc: true
lang: zh
---
# flutter使用github action自动打包APK/IPA(未签名)并上传release

最近写了一个查看河农大充电桩使用情况的APP,用的flutter,写了一个Github action记录并分享一下.

如何手动打包APK请先跟随网上教程跟一遍,将该写的配置写完.

## 正式发布

~~~yml
name: build
on:
  push:
    tags:
      - 'v*'
jobs:
  release:
    name: Create release
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Extract release notes
        id: extract-release-notes
        uses: ffurrer2/extract-release-notes@v2
        env:
          GITHUB_TOKEN: ${{ secrets.RELASE_TOKEN }}
      - name: Create release
        uses: ncipollo/release-action@v1
        with:
          allowUpdates: true
          body: '${{ steps.extract-release-notes.outputs.release_notes }}'
  build_android:
    name: build_android
    runs-on: ubuntu-latest
    needs: [ release ]
    permissions:
      contents: write
    steps:
      - name: Clone repository
        uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: 'zulu'
          java-version: '17'
      - name: Set up Flutter
        uses: subosito/flutter-action@v2
        with:
          channel: stable
          flutter-version-file: pubspec.yaml # path to pubspec.yaml
          cache: true
      - run: flutter --version
      - name: Setup keystore
        run: |
          echo '${{ secrets.KEYSTORE }}' | base64 --decode > android/app/Aprdec.keystore
          echo '${{ secrets.KEY_PROPERTIES }}' > android/key.properties
      - name: build
        run: |
          flutter pub get
          flutter build apk --target-platform android-arm,android-arm64,android-x64 --split-per-abi
      - name: Pack Android apk
        run: |
          pushd build/app/outputs/flutter-apk/
          mv app-arm64-v8a-release.apk HNDcharge-arm64_v8a.apk
          mv app-armeabi-v7a-release.apk HNDcharge_armeabi_v7a.apk
          popd
      - name: Release Android artifacts
        uses: ncipollo/release-action@v1
        with:
          allowUpdates: true
          omitBody: true
          omitBodyDuringUpdate: true
          artifacts: 'build/app/outputs/flutter-apk/HNDcharge-arm64_v8a.apk,build/app/outputs/flutter-apk/HNDcharge_armeabi_v7a.apk'
  build-macos-and-ios:
    name: Build iOS
    needs: [ release ]
    runs-on: macos-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: ${{env.CI_FLUTTER_VERSION}}
          cache: true
      - name: Precompile
        run: |
          git submodule update --init --recursive --force
          flutter pub get
      - name: Build iOS ipa
        run: |
          flutter build ios --release --no-codesign
      - name: Packing
        run: |
          mkdir Payload
          mv build/ios/iphoneos/Runner.app Payload
          zip -r9 HNDcharge.ipa Payload
      - name: Release iOS artifacts
        uses: ncipollo/release-action@v1
        with:
          allowUpdates: true
          omitBody: true
          omitBodyDuringUpdate: true
          omitPrereleaseDuringUpdate: true
          artifacts: 'HNDcharge.ipa'
~~~

### Step:Setup keystore

使用base64 Aprdec.keystore编码并且保存到仓库变量中,在构建时生成文件.如果你是.jks文件修改文件后缀即可

## 测试action

少了创建发布release,多了将产物上传artifacts

```yml
name: build_test
on:
  workflow_dispatch:
    inputs:
      build_android:
        description: Build Android platform artifacts.
        required: true
        type: boolean
        default: true
      build_ios:
        description: Build iOS platform artifacts.
        required: true
        type: boolean
        default: true
      dry_run:
        description: Dry run, do NOT upload artifacts.
        required: true
        type: boolean
        default: true
jobs:
  build_android:
    name: build_android
    if: ${{ github.event_name == 'push' || inputs.build_android }}
    runs-on: ubuntu-latest
    steps:
      - name: Clone repository
        uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: 'zulu'
          java-version: '17'
      - name: Set up Flutter
        uses: subosito/flutter-action@v2
        with:
          channel: stable
          flutter-version-file: pubspec.yaml # path to pubspec.yaml
          cache: true
      - run: flutter --version
      - name: Setup keystore
        run: |
          echo '${{ secrets.KEYSTORE }}' | base64 --decode > android/app/Aprdec.keystore
          echo '${{ secrets.KEY_PROPERTIES }}' > android/key.properties
      - name: build
        run: |
          flutter pub get
          flutter build apk --target-platform android-arm,android-arm64,android-x64 --split-per-abi
      - name: Pack Android apk
        run: |
          pushd build/app/outputs/flutter-apk/
          mv app-arm64-v8a-release.apk HNDcharge-arm64_v8a.apk
          mv app-armeabi-v7a-release.apk HNDcharge_armeabi_v7a.apk
          popd
      - name: Upload Android artifacts
        if: ${{ github.event_name != 'push' && inputs.build_android && !inputs.dry_run }}
        uses: actions/upload-artifact@v4
        with:
          name: HNDcharge-arm64_v8a
          path: |
            build/app/outputs/flutter-apk/HNDcharge-arm64_v8a.apk

  build-macos-and-ios:
    name: Build MacOS and iOS
    if: ${{ github.event_name == 'push'|| inputs.build_ios }}
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: ${{env.CI_FLUTTER_VERSION}}
          cache: true
      - name: Precompile
        run: |
          git submodule update --init --recursive --force
          flutter pub get
      - name: Build iOS ipa
        if: ${{ github.event_name == 'push' || inputs.build_ios }}
        run: |
          flutter build ios --release --no-codesign
      - name: Packing
        if: ${{ github.event_name == 'push' || inputs.build_ios }}
        run: |
          mkdir Payload
          mv build/ios/iphoneos/Runner.app Payload
          zip -r9 HNDcharge.ipa Payload
      - name: Upload iOS artifacts
        if: ${{ github.event_name != 'push' && inputs.build_ios && !inputs.dry_run }}
        uses: actions/upload-artifact@v4
        with:
          name: HNDcharge-ios-tarball
          path: HNDcharge.ipa
```

## 参考

[github action搭建flutter流水线 | Distant Vicinity](https://kzs.moe/blog/015-flutter-github-workflow)

[Flutter项目打包成安卓apk详解来了（解决安装没网络问题）_flutter 生成apk-CSDN博客](https://blog.csdn.net/qq_41614928/article/details/105183419)

