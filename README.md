# physicalFlickKeyboardApp

## ファームウェアのビルド

右手（`futureFlickKeyboard_ino/rightHand/rightHand2/`）は BLE キーボードのライブラリ [T-vK/ESP32-BLE-Keyboard](https://github.com/T-vK/ESP32-BLE-Keyboard) を使う。このライブラリはリポジトリに含めず、下の版を別に入れる。

### 使う版

| もの | 版 | 使うスケッチ |
| --- | --- | --- |
| ESP32 ボード定義（Espressif arduino-esp32、`esp32:esp32`） | 1.0.4 | 左手・右手 |
| ESP32-BLE-Keyboard（T-vK） | 0.2.2（タグ） | 右手 |
| Adafruit NeoPixel | 1.6.0 | 左手・右手 |

- ボードは `ESP32 Dev Module`（`esp32:esp32:esp32`）を想定している。
- 右手は Wi-Fi（ESP-NOW）と BLE を両方使うためプログラムが大きく、既定のパーティションには入らない。パーティションを `Huge APP`（`PartitionScheme=huge_app`）にする。
- ESP32 ボード定義 2.x / 3.x では確かめていない。3.x では ESP-NOW の受信コールバックと BLE の API が変わっているため、そのままでは通らない見込み。
- 以前このリポジトリに同梱していた 3 ファイル（`BleKeyboard.cpp` / `BleKeyboard.h` / `KeyboardOutputCallbacks.h`）は、ESP32-BLE-Keyboard 0.2.2 の同名のファイルと BLE のデバイス名の既定値だけが違っていた。今は `rightHand2.ino` で `"FFKB"` を渡している。
- 0.2.2 のタグはコミット `928c09320e3e6e344a63640c7de21d52a20c1d47` を指す（2026-11 時点）。

### arduino-cli で入れてビルドする

```sh
arduino-cli config init   # 設定ファイルがまだ無いときだけ
arduino-cli config add board_manager.additional_urls https://espressif.github.io/arduino-esp32/package_esp32_index.json
arduino-cli core update-index
arduino-cli core install esp32:esp32@1.0.4
arduino-cli lib install "Adafruit NeoPixel@1.6.0"

# ESP32-BLE-Keyboard はライブラリマネージャに無いので Git のタグから入れる。
# --git-url を使うには library.enable_unsafe_install を true にする必要がある（入れ終わったら false に戻してよい）。
arduino-cli config set library.enable_unsafe_install true
arduino-cli lib install --git-url https://github.com/T-vK/ESP32-BLE-Keyboard.git#0.2.2

arduino-cli compile -b esp32:esp32:esp32 futureFlickKeyboard_ino/leftHand/leftHand
arduino-cli compile -b esp32:esp32:esp32:PartitionScheme=huge_app futureFlickKeyboard_ino/rightHand/rightHand2
```

### Arduino IDE で入れる

1. 環境設定の「追加のボードマネージャの URL」に `https://espressif.github.io/arduino-esp32/package_esp32_index.json` を足し、ボードマネージャで `esp32`（Espressif Systems）の 1.0.4 を入れる。
2. ライブラリマネージャで `Adafruit NeoPixel` の 1.6.0 を入れる。
3. https://github.com/T-vK/ESP32-BLE-Keyboard/releases/tag/0.2.2 から ZIP を落とし、「スケッチ → ライブラリをインクルード → .ZIP 形式のライブラリをインストール」で入れる。
4. ボードを `ESP32 Dev Module`、右手を書き込むときは「Partition Scheme」を `Huge APP` にする。

### 注意

- Windows で ESP32 ボード定義 1.0.4 を使うとき、Arduino のデータフォルダ（`arduino-cli config get directories.data`）のパスが長いと、`fatal error: algorithm: No such file or directory` でコンパイルが止まることがある（ファイルはあるのに見つからない。同梱の古いコンパイラが Windows のパス長の制限にかかっていると見ている）。データフォルダを短いパスに置く（これで通るかは未確認）か、WSL など Linux 上でビルドする。
- Linux（WSL を含む）で ESP32 ボード定義 1.0.4 を使うときは、`python` という名前のコマンドと pyserial が要る（ビルドの途中で `python` を呼ぶ）。Ubuntu なら `sudo apt install python-is-python3 python3-serial` で入る。
- ESP32-BLE-Keyboard 0.2.2 のリポジトリにはライセンスのファイルが無い（2026-11 時点）。そのためこのリポジトリには含めず、上の手順で本家から入れる。
