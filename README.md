# 制限されたビデオをダウンロードし復号する方法

## 注意
このドキュメントは学術目的・セキュリティ研究用です。悪用を禁止します。

## 前提環境
Microsoft Windows 11 25H2
Mozilla Firefox / Google Chrome
Python 3.14.2
FFMpeg essentialsまたはfullをpathに追加済み
Git
Powershell 7

## 手順

### フェーズ1:.wvdファイルを取得する

#### Android Studioセットアップ（バーチャルデバイス用）
```powershell
winget install Google.AndroidStudio Google.PlatformTools 7zip.7zip
```
- AndroidStudioは特に何も弄らずにインストールし、「新しいプロジェクト」->そのまま適当にプロジェクトを作成
- 新しいプロジェクトで左上の三点メニューの「Tools」->「Device Manager」->「Create Virtual Device」 -> 「Pixel 9 Pro」 -> Servicesを「Google APIs」System Imageに「Google APIs Intel x86_64 Atom System Image」を選択、Finish
- Device Managerで作られた仮想デバイスのPixel 9 Proの再生ボタンをクリックして開始

#### Frida Server & KeyDive セットアップ
任意の場所（今回はC:\Users\UserName\Documents）で展開
```powershell
cd C:\Users\UserName\Documents
Invoke-WebRequest -Uri "https://github.com/frida/frida/releases/download/17.8.0/frida-server-17.8.0-android-x86_64.xz" -OutFile .
7z x "frida-server-17.8.0-android-x86_64.xz"
pip install keydive
```

Pixel 9 Proが動作している状態で
```powershell
adb root
adb devices
adb push frida-server-17.8.0-android-x86_64 /sdcard
adb shell
mv /sdcard/frida-server-17.8.0-android-x86_64 /data/local/tmp
chmod  +x /data/local/tmp/frida-server-17.8.0-android-x86_64
/data/local/tmp/frida-server-17.8.0-android-x86_64
```
このターミナルはFrida Serverが実行中なので開きっぱなしにしておく

新しいターミナルで：
```powershell
cd C:\Users\UserName\Documents\frida-server-17.8.0-android-x86_64
keydive -kw -a player
```
- Kaltura DRM テストアプリが自動インストールされる。テストアプリを開き、"Provision Widevine" -> "Refresh" -> "Test DRM Playback" -> "Kaltura" or "Google"のいずれかを選択。テストプレイが開始される。
- "frida-server-17.8.0-android-x86_64"フォルダの"device" -> "Google" -> "sdk_gphone64_x86_64" -> "33100" -> "(数字)" -> "client_id.bin", "private_key.pem", "google_sdk_gphone64_x86_64_19.5.0@XXXXX.wvd" といった名前の3つのファイルが自動生成される。これで必要な.wvdファイルが取得できた。

### フェーズ2: WidevineProxy2 と n_m3u8dl-reとshaka-packagerのセットアップ

#### WidevineProxy2をインストール
- https://github.com/DevLARLEY/WidevineProxy2/releases でxpiファイルをダウンロードして拡張機能としてインストールする。
- インストールしたら、"Enabled"をクリック
- "Widevine Device"が選択済みであることを確認
- Widevine Deviceセクション -> Choose File -> google_sdk_gphone64_x86_64_19.5.0@XXXXX.wvdを選択

#### n_m3u8dl-reをpathが通っている箇所にインストール
ffmpegフォルダにffmpegをインストール済みでこのフォルダにpathが通っていると仮定
```powershell
cd C:\ffmpeg
Invoke-WebRequest -Uri "https://github.com/nilaoda/N_m3u8DL-RE/releases/download/v0.5.1-beta/N_m3u8DL-RE_v0.5.1-beta_win-x64_20251029.zip" -OutFile .
7z x "N_m3u8DL-RE_v0.5.1-beta_win-x64_20251029.zip"
```

#### shaka-packagerをpathが通っている箇所にインストール
shaka-packagerにリネームする
```powershell
cd C:\ffmpeg
Invoke-WebRequest -Uri "https://github.com/shaka-project/shaka-packager/releases/download/v3.6.0/packager-win-x64.exe" -OutFile .
Rename-Item -Path "packager-win-x64.exe" -NewName "shaka-packager.exe"
```

### フェーズ3: 制限されたコンテンツを再生し復号
- DRMコンテンツを再生し、WidevineProxy2を開く。下部の"+"を展開し、"CmdLet"をコピー (Ctrl+A, Ctrl+C)
- カレントディレクトリをダウンロードしたい場所に移動してからPowershellにペーストして実行
- ダウンロードしたい品質を選びEnter
- 完了！お疲れ様でした。