# Flutter-Github-Action-CI-Example

此示例演示了如何使用 GitHub Actions 通过 Flutter 同时构建 Windows、Linux、macOS、Android、iOS 和 Web 端的软件包。

## 依赖项

- [subosito/flutter-action](https://github.com/subosito/flutter-action)
- [actions/checkout](https://github.com/actions/checkout)
- [actions/upload-artifact](https://github.com/actions/upload-artifact)
- [actions/setup-java](https://github.com/actions/setup-java)
- [softprops/action-gh-release](https://github.com/softprops/action-gh-release)

## 前提条件

- 确保你的 Flutter 项目已正确配置。
- 在 GitHub 仓库中启用 GitHub Actions 
- 设置 `Workflow permissions` 为 `Read and write permissions`。位于`Settings`>`Code and automation`>`Action`>`General`。
- 配置必要的证书和密钥，尤其是 iOS 和 Android。

## 工作流配置

在你的 Flutter 项目的根目录下创建一个 `.github/workflows/build.yml` 文件，添加以下内容：

```yaml
name: Flutter Build

on:
  workflow_dispatch:
  push:
    tags:
      - '*'


jobs:
  web:
    runs-on: ubuntu-latest
    steps:
      - name: Set up repository
        uses: actions/checkout@v4

      - name: Set up Flutter
        uses: subosito/flutter-action@v2

      - name: Install dependencies
        run: flutter pub get

      - name: Run tests
        run: flutter test

      - name: Build web
        run: flutter build web --release

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: web-build
          path: build/web/

      - name: Create ZIP of compiled files
        run: tar -czvf web.tar.gz -C build/web .

      - name: Publish release
        uses: softprops/action-gh-release@v2
        if: startsWith(github.ref, 'refs/tags/')
        with:
          files: web.tar.gz
          tag_name: ${{ github.ref_name }}
          name: Release ${{ github.ref_name }}
          draft: false
          prerelease: false

  linux:
    runs-on: ubuntu-latest
    steps:
      - name: Set up repository
        uses: actions/checkout@v4

      - name: Set up Flutter
        uses: subosito/flutter-action@v2

      - name: Install dependencies
        run: |
          sudo apt-get update -y
          sudo apt-get install -y ninja-build libgtk-3-dev

      - name: Install dependencies
        run: flutter pub get

      - name: Run tests
        run: flutter test

      - name: Build for Linux
        run: flutter build linux --release

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: linux-build
          path: build/linux/x64/release/bundle/

      - name: Create tar.gz archive
        run: tar -czvf linux-x86_64.tar.gz -C build/linux/x64/release/bundle .

      - name: Publish release
        uses: softprops/action-gh-release@v2
        if: startsWith(github.ref, 'refs/tags/')
        with:
          files: linux-x86_64.tar.gz
          tag_name: ${{ github.ref_name }}
          name: Release ${{ github.ref_name }}
          draft: false
          prerelease: false
  android:
    runs-on: ubuntu-latest
    steps:
      - name: Set up repository
        uses: actions/checkout@v4

      - name: Set up Java
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'

      - name: Set up Flutter
        uses: subosito/flutter-action@v2

      - name: Install dependencies
        run: flutter pub get

      - name: Run tests
        run: flutter test

      - name: Build APK
        run: flutter build apk --release

      - name: Build split APKs
        run: flutter build apk --split-per-abi

      - name: Build App Bundle
        run: flutter build appbundle --release

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: android-build
          path: build/app/outputs/

      - name: Rename APKs and AABs
        run: |
          mv build/app/outputs/flutter-apk/app-release.apk build/app/outputs/flutter-apk/app-universal-release.apk
          mv build/app/outputs/bundle/release/app-release.aab build/app/outputs/bundle/release/app-universal-release.aab

      - name: Publish release
        uses: softprops/action-gh-release@v2
        if: startsWith(github.ref, 'refs/tags/')
        with:
          files: |
            build/app/outputs/flutter-apk/app-universal-release.apk
            build/app/outputs/flutter-apk/app-armeabi-v7a-release.apk
            build/app/outputs/flutter-apk/app-arm64-v8a-release.apk
            build/app/outputs/flutter-apk/app-x86_64-release.apk
            build/app/outputs/bundle/release/app-universal-release.aab
          tag_name: ${{ github.ref_name }}
          name: Release ${{ github.ref_name }}
          draft: false
          prerelease: false

  windows:
    runs-on: windows-latest
    steps:
      - name: Set up repository
        uses: actions/checkout@v4

      - name: Set up Flutter
        uses: subosito/flutter-action@v2

      - name: Install dependencies
        run: flutter pub get

      - name: Run tests
        run: flutter test

      - name: Build for Windows
        run: flutter build windows --release

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: windows-build
          path: build/windows/

      - name: Zip compiled files
        run: Compress-Archive -Path build/windows/x64/runner/Release/* -DestinationPath windows-x86_64.zip

      - name: Publish release
        uses: softprops/action-gh-release@v2
        if: startsWith(github.ref, 'refs/tags/')
        with:
          files: windows-x86_64.zip
          tag_name: ${{ github.ref_name }}
          name: Release ${{ github.ref_name }}
          draft: false
          prerelease: false
  macos:
    runs-on: macos-latest
    steps:
      - name: Set up repository
        uses: actions/checkout@v4

      - name: Set up Flutter
        uses: subosito/flutter-action@v2

      - name: Install dependencies
        run: flutter pub get

      - name: Run tests
        run: flutter test

      - name: Build for macOS
        run: flutter build macos --release

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: macos-build
          path: build/macos/

      - name: Zip compiled files
        run: zip -r macos-universal-release.zip build/macos/Build/Products/Release/app.app

      - name: Publish release
        uses: softprops/action-gh-release@v2
        if: startsWith(github.ref, 'refs/tags/')
        with:
          files: macos-universal-release.zip
          tag_name: ${{ github.ref_name }}
          name: Release ${{ github.ref_name }}
          draft: false
          prerelease: false
  ios:
    runs-on: macos-latest
    steps:
      - name: Set up repository
        uses: actions/checkout@v4

      - name: Set up Flutter
        uses: subosito/flutter-action@v2

      - name: Install dependencies
        run: flutter pub get

      - name: Run tests
        run: flutter test

      - name: Build for iOS
        run: flutter build ios --release --no-codesign

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: ios-build
          path: build/ios/

      - name: Zip compiled files
        run: zip -r ios-universal-release.zip build/ios/iphoneos/Runner.app

      - name: Publish release
        uses: softprops/action-gh-release@v2
        if: startsWith(github.ref, 'refs/tags/')
        with:
          files: ios-universal-release.zip
          tag_name: ${{ github.ref_name }}
          name: Release ${{ github.ref_name }}
          draft: false
          prerelease: false
```

## 构建步骤

1. **Checkout 代码**：从 GitHub 仓库中检出代码。
2. **设置 Flutter**：安装指定版本的 Flutter。
3. **安装依赖**：运行 `flutter pub get` 安装项目依赖。
4. **构建平台**：依次构建 Android、iOS、Web、Windows、Linux 和 macOS 平台的发布包。

## 使用说明

1. 将上述 `build.yml` 文件添加到你的 GitHub 仓库中。
2. 每当你推送带有标签的代码时，GitHub Actions 会自动触发构建过程。
3. 构建成功后，可以在 Actions 面板查看构建日志和生成的二进制文件。

![](./doc/42d6b845959177a9334d882ce474b541_MD5.jpeg)

## 贡献

欢迎任何贡献，提交问题或拉取请求！

