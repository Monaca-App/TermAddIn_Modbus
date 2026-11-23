# Modbus通信制御用 マクロスクリプト自動生成 AI指示書 (Modbus_ASCII.csv 生成用)

> **生成AI**: Antigravity (Gemini 4 Carbon (Medium))

あなたは、通信制御システム（Term / TermAddIn）向けマクロスクリプト（CSV形式）を設計・出力する専門のエンジニアリングAIです。  
以下の「参照ドキュメント」および「システム絶対制約」「Modbus ASCII 機能別パラメータ設計仕様」を厳格に遵守し、温調計・インバータ・電力計・PLC 等の各種 Modbus スレーブ機器と **Modbus ASCII モード（RS-485 / RS-232C 16進テキスト通信・`:` 開始＋LRC＋CRLF 自動付加）** で通信・制御するための Term マクロスクリプト（`Modbus_ASCII.csv`）を作成してください。

---

## 📚 1. 参照ドキュメントおよび準拠規格

1. **システム文法・構文規約（最優先遵守）**
   * `Manual/TermAddIn/マクロスクリプト取扱説明書.Md`（Term マクロスクリプト取扱説明書 兼 AI自動生成指示書）
2. **通信・プロトコル個別仕様（`Modbus` 専用コマンドセット）**
   * `Manual/TermAddIn/Modbus/Modbus_Mode.Md`（伝送モード `ASCII`・UART 通信条件・フロー制御設定）
   * `Manual/TermAddIn/Modbus/Modbus_Open.Md` / `Manual/TermAddIn/Modbus/Modbus_Close.Md`（Modbus 回線接続・切断）
   * `Manual/TermAddIn/Modbus/Modbus_Read.Md`（レジスタ・ビット読出：FC01 / FC02 / FC03 / FC04）
   * `Manual/TermAddIn/Modbus/Modbus_Write.Md`（レジスタ・コイル書込：FC05 / FC06 / FC15 / FC16）
   * `Manual/Core/Param/Exit.Md`（シーケンス終了制御）
3. **対象通信仕様（実機エンジン `TermAnalysis.Modbus` ランタイム引数規約）**
   * 制御対象: Modbus ASCII スレーブ対応機器全般（温調計、デジタル指示調節計、インバータ、PLC 等）
   * 通信条件設定・接続:
     * `Modbus_Mode, "ASCII", "$BAUD$,$PARITY$,$DATA_BITS$,$STOP_BITS$", "$FLOW_CTRL$"`（Modbus ASCII 標準の 7bit データ長を初期値とし、8bit も選択可能。コロン `:` 開始・LRC 計算・`CRLF` 終端フレーミングは内部エンジンが自動処理）
     * `Modbus_Open`（PC 環境設定で選択された物理 COM ポートまたは `c2c` 仮想 COM ポートを Modbus 排他セッションでオープン）
   * アクセス規約:
     * `Modbus_Read, "$SLAVE_ID$", "$FUNC_CODE$", "$START_ADDR$", "$POINTS$", "RESULT_DATA"`（第5引数の格納先パラメータキーは `$` を付けずに `"RESULT_DATA"` と指定）
     * `Modbus_Write, "$SLAVE_ID$", "$FUNC_CODE$", "$START_ADDR$", "$WRITE_VALS$"`（複数点書き込み時はカンマ区切り文字列 `"100,200"` 等を指定可能）

---

## 🚨 2. システム絶対制約（構文エラー・実行時クラッシュ防止の鉄則）

1. **表記言語・文字コード・AI生成元の明記**
   * **表記言語**: スクリプト内のコメント、パラメータ表示名等は、特に指定がない限り **日本語をデフォルトとして記述・設定** してください。
   * **文字コード**: 生成する CSV ファイル（`Modbus_ASCII.csv`）は、日本語 Excel での文字化け防止のため **Shift-JIS (CP932)** で保存してください。
   * **AI生成元の明記（他者再現性の保証）**: CSV ファイルの最先頭ヘッダコメント行（`;`）に、対象システム情報とともに **実行環境（エージェント／IDE名）および自身のAIエンジン（モデル名・エディション／ティア）を正確に判別できる名称（例: `; 生成AI: Antigravity (Gemini 4 Carbon (Medium))`）** を必ず記述してください。
   * **パス表記**: ファイル内に開発環境固有のローカル絶対パスや個人名を含めることは厳禁です。

2. **専用コマンドセット自己完結原則と `List` 選択化（最重要仕様）**
   * **コマンドセット排他原則**: `TermAnalysis.Modbus` 用マクロスクリプトでは **`Modbus` コマンドセット（`Modbus_Mode`, `Modbus_Open`, `Modbus_Read`, `Modbus_Write`, `Modbus_Close`）および `Core` 共通基盤コマンドのみ** を使用してください。`Com_Mode` / `Com_Open` / `Com_Close` など他ドメインのコマンドを混在させることは厳禁です。
   * **機能別パラメータ分離と `List` 化**: 通信条件やファンクションコードなど取り得る文字列が決まっている項目は必ず **`List` 型** で定義し、UI および Excel ドロップダウンから選択できるようにしてください。
     * **ボーレート (`BAUD`)**: `List`, 初期値 `"9600"`, 追加選択肢 `"19200"`, `"38400"`, `"57600"`, `"115200"`
     * **パリティ (`PARITY`)**: `List`, 初期値 `"E"`, 追加選択肢 `"O"`, `"N"`
     * **データ長 (`DATA_BITS`)**: `List`, 初期値 `"7"`, 追加選択肢 `"8"`（ASCII テキスト通信は 7bit 標準、8bit 機器にも対応）
     * **ストップビット (`STOP_BITS`)**: `List`, 初期値 `"1"`, 追加選択肢 `"2"`
     * **フロー制御 (`FLOW_CTRL`)**: `List`, 初期値 `"NONE"`, 追加選択肢 `"RTSCTS"`, `"XONXOFF"`
     * **読出ファンクション (`FUNC_CODE`)**: レジスタ読出は `List`（`"FC03"`, `"FC04"`）、ビット読出は `List`（`"FC01"`, `"FC02"`）
     * **書込ファンクション (`FUNC_CODE`)**: レジスタ書込は `List`（`"FC06"`, `"FC16"`）、コイル書込は `List`（`"FC05"`, `"FC15"`）
     * ※書込データ（`WRITE_VALS`, `WRITE_BITS`）は複数点一括書込時のカンマ区切り値も自由入力できるよう `Text` 型とします。

3. **行頭判定・PARAM 第5/第6フィールド必須・行末コメント禁止**
   * 親ブロック（`TITLE`, `COMMAND`）は行頭第1フィールドから、子要素（`PARAM`, `SEQUENCE`）は必ず **行頭カンマ `,`** から開始してください。
   * すべての `PARAM` 行で第5フィールド（表示名）と第6フィールド（初期値、空なら `""`）を省略せず明記してください。
   * `COMMAND` 配下の `PARAM` / `SEQUENCE` 行では行末コメント（`;`）は完全禁止です。

---

## 📡 3. 対象システム仕様および TITLE ブロック定義

* **タイトル名**: `"Modbus通信制御 (ASCIIモード)"`
* **書式**:
  ```csv
  TITLE, "Modbus通信制御 (ASCIIモード)"
  ```

---

## 📐 4. 実装すべき COMMAND ブロック設計仕様（全 6 ブロック）

### ① 通信ポート接続：`COMMAND, "PORT_OPEN"`
* **目的**: Modbus ASCII モードおよび UART 通信条件（`9600,E,7,1`, フロー制御 `NONE`）を設定し、Modbus シリアル回線をオープンする。
* **パラメータ定義**:
  * `BAUD` (`List`, `"ボーレート"`, `"9600"`, `"19200"`, `"38400"`, `"57600"`, `"115200"`)
  * `PARITY` (`List`, `"パリティ"`, `"E"`, `"O"`, `"N"`)
  * `DATA_BITS` (`List`, `"データ長"`, `"7"`, `"8"`)
  * `STOP_BITS` (`List`, `"ストップビット"`, `"1"`, `"2"`)
  * `FLOW_CTRL` (`List`, `"フロー制御"`, `"NONE"`, `"RTSCTS"`, `"XONXOFF"`)
* **シーケンス手順**:
  1. ステップ 10: `Modbus_Mode, "ASCII", "$BAUD$,$PARITY$,$DATA_BITS$,$STOP_BITS$", "$FLOW_CTRL$"`
  2. ステップ 20: `Modbus_Open`
  3. ステップ 30: `Exit`

### ② レジスタ読出（FC03 保持 / FC04 入力）：`COMMAND, "REG_READ", "RESULT_DATA"`
* **パラメータ定義**:
  * `SLAVE_ID` (`Text`, `"スレーブ局番"`, `"1"`)
  * `FUNC_CODE` (`List`, `"ファンクション"`, `"FC03"`, `"FC04"`)
  * `START_ADDR` (`Text`, `"開始アドレス"`, `"0x0000"`)
  * `POINTS` (`Text`, `"読出ワード数"`, `"2"`)
  * `RESULT_DATA` (`Range`, `"読出レジスタ値"`, `""`)
* **シーケンス手順**:
  1. ステップ 10: `Modbus_Read, "$SLAVE_ID$", "$FUNC_CODE$", "$START_ADDR$", "$POINTS$", "RESULT_DATA"`
  2. ステップ 20: `Exit`

### ③ ビット読出（FC01 コイル / FC02 入力ステータス）：`COMMAND, "BIT_READ", "RESULT_DATA"`
* **パラメータ定義**:
  * `SLAVE_ID` (`Text`, `"スレーブ局番"`, `"1"`)
  * `FUNC_CODE` (`List`, `"ファンクション"`, `"FC01"`, `"FC02"`)
  * `START_ADDR` (`Text`, `"開始アドレス"`, `"0x0000"`)
  * `POINTS` (`Text`, `"読出ビット数"`, `"8"`)
  * `RESULT_DATA` (`Range`, `"読出ビット値"`, `""`)
* **シーケンス手順**:
  1. ステップ 10: `Modbus_Read, "$SLAVE_ID$", "$FUNC_CODE$", "$START_ADDR$", "$POINTS$", "RESULT_DATA"`
  2. ステップ 20: `Exit`

### ④ レジスタ書込（FC06 単一 / FC16 複数）：`COMMAND, "REG_WRITE"`
* **パラメータ定義**:
  * `SLAVE_ID` (`Text`, `"スレーブ局番"`, `"1"`)
  * `FUNC_CODE` (`List`, `"ファンクション"`, `"FC06"`, `"FC16"`)
  * `START_ADDR` (`Text`, `"書込開始アドレス"`, `"0x0010"`)
  * `WRITE_VALS` (`Text`, `"書込レジスタ値"`, `"1500"`)
* **シーケンス手順**:
  1. ステップ 10: `Modbus_Write, "$SLAVE_ID$", "$FUNC_CODE$", "$START_ADDR$", "$WRITE_VALS$"`
  2. ステップ 20: `Exit`

### ⑤ コイル書込（FC05 単一 / FC15 複数）：`COMMAND, "COIL_WRITE"`
* **パラメータ定義**:
  * `SLAVE_ID` (`Text`, `"スレーブ局番"`, `"1"`)
  * `FUNC_CODE` (`List`, `"ファンクション"`, `"FC05"`, `"FC15"`)
  * `START_ADDR` (`Text`, `"書込開始アドレス"`, `"0x0000"`)
  * `WRITE_BITS` (`Text`, `"書込コイル値"`, `"1"`)
* **シーケンス手順**:
  1. ステップ 10: `Modbus_Write, "$SLAVE_ID$", "$FUNC_CODE$", "$START_ADDR$", "$WRITE_BITS$"`
  2. ステップ 20: `Exit`

### ⑥ 通信ポート切断：`COMMAND, "PORT_CLOSE"`
* **シーケンス手順**:
  1. ステップ 10: `Modbus_Close`
  2. ステップ 20: `Exit`

---

## ✅ 5. 出力前セルフチェックリスト

- [ ] `Modbus` 専用コマンドセット（`Modbus_Mode`, `Modbus_Open`, `Modbus_Read`, `Modbus_Write`, `Modbus_Close`）と `Exit` のみを使用し、他ドメインのコマンドが一切混在していないか？
- [ ] 取り得る文字列が決まっている項目（`BAUD`, `PARITY`, `DATA_BITS`, `STOP_BITS`, `FLOW_CTRL`, `FUNC_CODE`）がすべて `List` 型で定義されているか？
- [ ] `Modbus_Read` 第5引数の格納先パラメータキーが `"RESULT_DATA"`（`$` なし）になっているか？
- [ ] すべての `TITLE`, `COMMAND` 行が行頭から、`PARAM`, `SEQUENCE` 行が行頭カンマ（`,`）から始まっているか？
- [ ] すべての `PARAM` 行で第5フィールド（表示名）および第6フィールド（初期値）が明記され、行末コメントが含まれていないか？
- [ ] 生成される CSV ファイルは Shift-JIS (CP932) で保存される前提となっているか？

---

## 📋 6. 出力フォーマット

余計な自然言語による前置きや解説を省き、すぐに CSV ファイル（`Modbus_ASCII.csv`）として保存できるように、 **マークダウンのコードブロック（`csv` 形式）に作成した CSV コードのみ** を出力してください。

---

## ⚙️ プロパティ設定優先およびパラメータ定義規約（必須）

* **プロパティ設定優先（既定動作）**: COM ポート・IP アドレス・ポート番号・通信条件・局番号・PC 番号・スレーブアドレス・ルーティング番号など、プロパティ設定画面で指定できる項目は **プロパティ設定を優先して使用すること** を基本とします。
* **空欄 "" デフォルトの List 形式**: シーケンス用コマンドのパラメータとして上記項目を定義する場合は、必ず **""（空欄＝プロパティ設定を使用）を先頭デフォルトとした代表値の List 形式** で記述してください（例: `,PARAM, STATION_NO, List, "局番号(空欄=プロパティ)", "", "00", "01", "02", "03"`）。
* **省略パターンの両対応**: コマンド実行時は、引数自体を記述しない場合（パターン A）と、空文字 ""・空白・Excel 空セルを渡した場合（パターン B）の双方で自動的にプロパティ設定値が適用されます。一時的に別局へアクセスしたい場合のみリストから代表値を選択または直接入力してください。
