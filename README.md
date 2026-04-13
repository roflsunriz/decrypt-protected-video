# 制限されたビデオをダウンロードし復号する方法

## 注意
このドキュメントは学術目的・セキュリティ研究目的です。悪用を禁止します。

## 前提環境
- Microsoft Windows 11 25H2
- Mozilla Firefox (推奨) / Google Chrome (未検証)
```powershell
winget install Mozilla.Firefox.ja
```
- Python 3.14.2
```powershell
winget install Python.Python.3.14
```
- FFmpeg
```powershell
winget install Gyan.FFmepg
```
- Powershell 7
```powershell
winget install Microsoft.PowerShell
```
- Pathに追加するための`PathArea`ディレクトリを作成
- ワーキングディレクトリとしてC:\に`Decrypt`を作成
```powershell
New-Item -ItemType Directory -Path "C:\PathArea"
New-Item -ItemType Directory -Path "C:\Decrypt"
```
- ユーザー環境変数のPathに恒久的に追加
- 追加後はPowerShellを再起動する必要がある。
```powershell
[Environment]::SetEnvironmentVariable("Path", "C:\PathArea;$env:Path", "User")
```

## 必要ソフトウェア
- Android Studio
```powershell
winget install Google.AndroidStudio
```
- Platform Tools
```powershell
winget install Google.PlatformTools
```
- 7zip
```powershell
winget install 7zip.7zip
```
- Frida Server
```powershell
cd C:\Decrypt

$Headers = @{ "User-Agent" = "PowerShell" }
$Release = Invoke-RestMethod -Headers $Headers -Uri "https://api.github.com/repos/frida/frida/releases/latest"
$Asset = $Release.assets | Where-Object { $_.name -match '^frida-server-.*-android-x86_64\.xz$' } | Select-Object -First 1
if (-not $Asset) { throw "frida/frida の最新リリースに frida-server のアセットが見つかりません" }
Invoke-WebRequest -Uri $Asset.browser_download_url -OutFile "frida-server.xz"
7z x "frida-server.xz"
$FridaServer = Get-ChildItem -File -Filter "frida-server-*-android-x86_64" | Select-Object -First 1
Rename-Item -Path $FridaServer.FullName -NewName "frida-server"
```
- KeyDive
```powershell
pip install keydive
```
- WidevineProxy2
```powershell
cd C:\Decrypt
$Headers = @{ "User-Agent" = "PowerShell" }
$Release = Invoke-RestMethod -Headers $Headers -Uri "https://api.github.com/repos/DevLARLEY/WidevineProxy2/releases/latest"
$Asset = $Release.assets | Where-Object { $_.name -match '^WidevineProxy2_.*\.xpi$' } | Select-Object -First 1
if (-not $Asset) { throw "DevLARLEY/WidevineProxy2 の最新リリースに xpi アセットが見つかりません" }
Invoke-WebRequest -Uri $Asset.browser_download_url -OutFile "WidevineProxy2.xpi"
Start-Process "C:\Program Files\Mozilla Firefox\firefox.exe" -ArgumentList "C:\Decrypt\WidevineProxy2.xpi"

```
- n_m3u8dl-re
- `PathArea`ディレクトリにインストール 
```powershell
cd C:\PathArea
$Headers = @{ "User-Agent" = "PowerShell" }
$Release = Invoke-RestMethod -Headers $Headers -Uri "https://api.github.com/repos/nilaoda/N_m3u8DL-RE/releases/latest"
$Asset = $Release.assets | Where-Object { $_.name -match '^N_m3u8DL-RE_.*_win-x64_.*\.zip$' } | Select-Object -First 1
if (-not $Asset) { throw "nilaoda/N_m3u8DL-RE の最新リリースに Windows x64 zip アセットが見つかりません" }
Invoke-WebRequest -Uri $Asset.browser_download_url -OutFile "N_m3u8DL-RE.zip"
7z x "N_m3u8DL-RE.zip"
```
- shaka-packager
- `PathArea`ディレクトリにインストール
```powershell
cd C:\PathArea
$Headers = @{ "User-Agent" = "PowerShell" }
$Release = Invoke-RestMethod -Headers $Headers -Uri "https://api.github.com/repos/shaka-project/shaka-packager/releases/latest"
$Asset = $Release.assets | Where-Object { $_.name -match '^packager-win-x64\.exe$' } | Select-Object -First 1
if (-not $Asset) { throw "shaka-project/shaka-packager の最新リリースに packager-win-x64.exe が見つかりません" }
Invoke-WebRequest -Uri $Asset.browser_download_url -OutFile "packager-win-x64.exe"
Rename-Item -Path "packager-win-x64.exe" -NewName "shaka-packager.exe"
```
- SMPlayer (MKVファイル再生用)
```powershell
winget install SMPlayer.SMPlayer
```
- VLC (MKVファイル再生用)
```powershell
winget install VideoLAN.VLC
```

## 手順

### フェーズ1:.wvdファイルを取得する

#### Android Studioセットアップ（バーチャルデバイス用）
- 上記の「必要ソフトウェア」セクションのコマンドを参考にインストールしておく
- AndroidStudioは特に何も弄らずにインストールし、「新しいプロジェクト」->そのまま適当にプロジェクトを作成
- 新しいプロジェクトで左上の三点メニューの「Tools」->「Device Manager」->「Create Virtual Device」 -> 「Pixel 9 Pro」 -> Servicesを「Google APIs」System Imageに「Google APIs Intel x86_64 Atom System Image」を選択、Finish
- Device Managerで作られた仮想デバイスのPixel 9 Proの再生ボタンをクリックして開始

#### Frida Server & KeyDive セットアップ
- 上記の「必要ソフトウェア」セクションのコマンドを参考にインストールしておく

Pixel 9 Proが動作している状態で
```powershell
cd C:\Decrypt
adb root
adb devices
adb push "frida-server" /sdcard
adb shell
```

`adb shell` 実行後、Android シェル上で以下を入力：

```shell
mv /sdcard/frida-server /data/local/tmp
chmod +x /data/local/tmp/frida-server
/data/local/tmp/frida-server
```
このターミナルはFrida Serverが実行中なので開きっぱなしにしておく

新しいターミナルで：
```powershell
cd C:\Decrypt
keydive -kw -a player
```
- Kaltura DRM テストアプリが自動インストールされる。テストアプリを開き、「Provision Widevine」、「Refresh」、「Test DRM Playback」->「Kaltura」 or「Google」のいずれかを選択。テストプレイが開始される。
- C:\Decrypt 直下の 「device」フォルダ -> 「Google」フォルダ -> 「sdk_gphone64_x86_64」フォルダ -> 「33100」フォルダ -> 「(数字)」フォルダ -> 「client_id.bin」, 「private_key.pem」, 「google_sdk_gphone64_x86_64_19.5.0@XXXXX.wvd」 といった名前の3つのファイルが自動生成される。これで必要な.wvdファイルが取得できた。

### フェーズ2: WidevineProxy2 と n_m3u8dl-reとshaka-packagerのセットアップ

#### WidevineProxy2をインストール
- 上記の「必要ソフトウェア」セクションのコマンドを参考にFirefoxにWidevineProxy2をインストールしておく
- インストールしたら、「Enabled」をクリック（設定が有効化される）
- 「Widevine Device」が選択済みであることを確認（Remote CDMでないことを確認）
- Widevine Deviceセクション -> Choose File -> 「google_sdk_gphone64_x86_64_19.5.0@XXXXX.wvd」を選択

#### n_m3u8dl-reをC:\PathAreaディレクトリにインストール
- 上記の「必要ソフトウェア」セクションのコマンドを参考にn_m3u8dl-reをC:\PathAreaディレクトリにインストールしておく

#### shaka-packagerをC:\PathAreaディレクトリにインストール
- 上記の「必要ソフトウェア」セクションのコマンドを参考にshaka-packagerをC:\PathAreaディレクトリにインストールしておく
- ファイル名を忘れずにshaka-packager.exeにリネームする

### フェーズ3: 制限されたコンテンツを再生し復号
- DRMコンテンツを再生し、WidevineProxy2を開く。下部の"+"を展開し、"Cmd"をコピー (Ctrl+A, Ctrl+C)。或いは"Cmd"の青いリンク部分をクリックしてもコピーできる。
- カレントディレクトリをダウンロードしたい場所に移動してからPowershellにペーストして実行（右クリック又はCtrl+Vでペースト）
- 又はエクスプローラーで保存したい場所を開き、右クリックメニューの「ターミナルで開く」をクリックしてPowershellを開く（この状態ではカレントディレクトリは保存したい場所になっている）。
- ダウンロードしたい品質を選びEnter
- 完了！これでビデオは復号されて保存された。.mkvファイルは`SMPlayer`や`VLC`などのプレーヤーで再生できる。
