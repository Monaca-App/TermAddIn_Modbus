# コマンドリファレンス：Modbus_Open

## 概要

**Modbus_Open** は、Term アプリケーションの UI 設定および `Modbus_Mode` で指定された伝送モード（`RTU` / `ASCII` の場合は UART シリアルポート、`TCP` の場合は TCP ソケット回線）をオープンし、Modbus マスター通信セッションを開始するコマンドです。
実行時に Term 通信制御の排他選択モードを **Modbus モード** （`TermAnalysis.Modbus.dll`）へ確定させます。

> 💡 **補足**: 使用する COM ポート番号や接続先 IP アドレスは、引数を省略した場合は PC 環境設定（UI 設定値）から取得し、引数を指定した場合は個別条件でオーバーライドして、プロトコル種別（RTU / ASCII / TCP）に応じた下位トランスポートを自動選択します。RTU / ASCII モードでは物理 COM ポートのほか、仮想シリアル中継ツール **[c2c](https://github.com/Monaca-App/c2c)** が公開中の仮想 COM ポート（例: `COM10` ⇔ `COM11`）も透過的にオープン可能です。

---

## 💡 書式・パラメータ仕様

### コマンド構文

```csv
,SEQUENCE, [ステップキー], Modbus_Open, "[Mode]", "[PortOrIp]", "[BaudOrPort: 0]", "[SlaveId: 1]"
```

※ 全引数を省略した場合は、UI プロパティ設定および `Modbus_Mode` で構成された現在の設定値を使用してオープンします。

### パラメータ（引数）一覧

| 引数位置 | 引数名 | 必須/任意 | 説明・制約 |
| :--- | :--- | :---: | :--- |
| 第 1 引数 | `Mode` | 任意 | 伝送モード指定（`"RTU"` / `"ASCII"` / `"TCP"`、省略時既定値: `""`＝現在のモード設定を使用）。 |
| 第 2 引数 | `PortOrIp` | 任意 | シリアル COM ポート名（例: `"COM3"`）または接続先 IP アドレス（例: `"192.168.1.50"`、省略時既定値: `""`）。 |
| 第 3 引数 | `BaudOrPort` | 任意 | シリアル時のボーレート［bps］または TCP 時のポート番号（例: `19200` または `502`、省略時既定値: `0`＝UI 設定値）。 |
| 第 4 引数 | `SlaveId` | 任意 | 既定ターゲットのスレーブ局番／ユニット ID（`0`〜`247`、省略時既定値: `1`）。 |

---

## 🔄 特殊置換規約・分岐規約の適用

本コマンドの引数における特殊記法および解釈ルールです。
* **動的置換（`$`）**: 引数内で `$PARAMキー$` のように記述すると、実行時に変数プール（PARAM）の値へ動的置換されます。（適用可能な引数: 全引数対応）
* **ジャンプ先指定（`#`）**: 本コマンドはジャンプ制御を行わないため適用されません。（適用可能な引数: 非適用）
* **共通コマンド指定（`@`）**: 本コマンドは外部マクロを呼び出さないため、共通コマンド呼び出し記法は適用されません。（適用可能な引数: 非適用）
* **カンマのエスケープ**: 通信条件文字列（例: `"9600,E,7,1"`）や複数データのカンマ区切り値を単一引数として渡す場合は、必ず `"..."` と二重引用符で囲んで記述します。
* **エイリアス（別名）**: なし（標準コマンド名 `Modbus_Open` のみ。大文字・小文字は区別されません）

---

## 💡 動作例（正しい構文での記述例）

### スクリプト記述例

※COMMAND ブロック配下の子要素行（PARAM, SEQUENCE）は必ず行頭カンマから開始し、スペースインデントや行末コメント（`;`）は記述しないでください。

```csv
COMMAND, "SAMPLE_MODBUS_OPEN", ""
; 1. Modbus RTUモード（38400bps, Even, 8bit, Stop 1）を設定して回線オープン
,SEQUENCE, 010, Modbus_Mode, "RTU", "38400,E,8,1", "NONE"
,SEQUENCE, 020, Modbus_Open
; 2. スレーブ局番1の保持レジスタ(FC03) 開始アドレス0から2ワード読出
,SEQUENCE, 030, Modbus_Read, "1", "03", "0000", "2", "@REG_DATA"
,SEQUENCE, 040, Modbus_Close
,SEQUENCE, 050, Exit
```

### 実行時の挙動・変数変化

* **実行前の状態**: Term 通信エンジンが初期化され、必要な通信パラメータおよび変数プール（`PARAM`）が準備されている状態。
* **実行後の状態**: `Modbus_Mode` により `RTU` フレーム形式および UART 通信条件（`38400,E,8,1`）が設定されます。 `Modbus_Open` によりシリアルポートが開かれ、Modbus RTU フレーム送受信エンジンが待機状態になります。

---

## 🔄 動作フロー（シーケンス図）

```mermaid
sequenceDiagram
    autonumber
    participant Script as "スクリプト実行エンジン (SequenceRunner)"
    participant Cmd as "Modbus_OpenCommand (TermAnalysis.Modbus)"
    participant Trans as "ModbusTransport (Serial / TcpClient)"
    participant Buf as "TermRingBufferManager (受信バッファ)"

    Script->>Cmd: Execute(context)
    Cmd->>Cmd: 通信排他ドメイン(Modbus)の整合性チェック
    alt 別ドメイン(COM/Telnet/MELSEC)がアクティブな場合
        Cmd-->>Script: InvalidOperationException (ドメイン排他違反)
    else Modbusドメイン利用可能・未接続の場合
        Cmd->>Buf: Clear() (受信バッファ初期化)
        Cmd->>Trans: OpenAsync() (RTU/ASCIIならSerialPort, TCPならTcpClientを接続)
        alt 回線オープン成功
            Trans-->>Cmd: 接続完了
            Cmd-->>Script: 正常終了 (Next Step)
        else ポート不在またはTCP接続失敗
            Trans-->>Cmd: IOException / SocketException
            Cmd-->>Script: 接続エラー送出・シーケンス停止
        end
    end
```

---

## 🚫 エラーハンドリング・安全設計（仕様制限）

1. **二重オープンの安全な扱い（Idempotent 制御）**:
   - すでに Modbus 回線がオープンされている状態で再実行された場合、エラーとせず安全に継続します。
2. **他通信ドメインとの排他保護**:
   - 同一シーケンス内で `Com_Open` や `Telnet_Open` 等と併用された場合は、排他制御エラーとして即座に検出・遮断します。

---

## 🧪 ユニットテスト仕様

本コマンドクラス（`Modbus_OpenCommand.cs`）に実装されている `RunUnitTest()` メソッドが検証するユニットテストの仕様です。

### テスト実行前提条件

* **実行トリガー**: `RunUnitTest()` メソッドの呼び出し
* **有効化条件**: システム全体の最小ログレベル（`ErrorLogger.MinimumLevel`）が `LogLevel.Test` であること（それ以外の場合は無効化ガードにより早期リターン）
* **検証結果出力**: `_log.LogTest` によるログ出力。検証失敗時は `_log.LogError`、予期せぬ例外発生時は `_log.LogCritical` で記録し、`finally` ブロックで `CloseSession(TermTransportDomain.Modbus, null)`、`ModbusCore.CurrentMode = ModbusProtocolMode.RTU`、`DefaultSlaveAddress = 1`、および UART／Socket の `IsSimulationMode` 復元を実行します。

### テストケース一覧

| No | テストカテゴリ | テスト条件（入力値） | 期待される結果（出力値） | 検証の目的 / 観点 |
| :---: | :--- | :--- | :--- | :--- |
| **1** | 正常系 | UART／Socket 両方の `IsSimulationMode = true` 環境下で引数 `"TCP"`, `"192.168.0.100"`, `"502"`, `"2"` を指定して実行 | `!param.HasError` かつ `ActiveDomain == TermTransportDomain.Modbus`、`CurrentMode == ModbusProtocolMode.TCP`、`Host == "192.168.0.100"`、`Port == 502`、`DefaultSlaveAddress == 2` となる | Modbus TCP 伝送モード・ホスト IP・ポート番号・既定スレーブ局番の反映と排他接続確立の検証 |
| **2** | 異常系 | 引数 5 個以上の過剰指定（`ValidateParamRange(0, 4)` 違反）または排他セッションオープン失敗（不在ポート／到達不能ホスト） | `_log.LogError` 出力および `SetErrorAndHalt(-1, errMsg)`（または引数検証失敗による早期リターン）により安全に停止する | 引数境界チェックおよび Modbus 回線接続失敗時のフェイルセーフ検証 |
