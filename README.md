<img src="images/Monaca_store.png" align="right" width="130" alt="Monaca Mascot" />

# TermAddIn_Modbus - Modbus (RTU / ASCII / TCP) 通信エクステンション

👉 [← TermAddIn 統合本体に戻る](https://github.com/Monaca-App/TermAddIn)  
👉 [← Monaca アプリ工房（トップ）に戻る](https://github.com/Monaca-App/store)

---

[![Platform](https://img.shields.io/badge/Platform-Windows%2010%20%2F%2011-blue.svg)](https://www.microsoft.com/)
[![Excel](https://img.shields.io/badge/Excel-2016%20%2F%202019%20%2F%202021%20%2F%20M365-green.svg)](https://www.microsoft.com/excel)
[![Framework](https://img.shields.io/badge/.NET%20Framework-4.8-purple.svg)](https://dotnet.microsoft.com/)
[![Architecture](https://img.shields.io/badge/Architecture-TermAddIn%20Extension-orange.svg)]()
[![License](https://img.shields.io/badge/License-TermAddIn__Modbus%20License-blue.svg)]()

## 📖 概要

**TermAddIn_Modbus** は、Excel 総合通信アドイン **TermAddIn** 上で動作する **Modbus (RTU / ASCII / TCP) プロトコル通信専用エクステンション（機能ドメイン単一 DLL）** です。

産業用オートメーション（FA）現場で広く普及している各種温調器、デジタル指示調節計、インバータ、電力計、PLC（プログラマブルロジックコントローラ）、および各種 Modbus スレーブ対応センサ機器とのデータ通信・レジスタ監視・パラメータ一括書込を Excel ワークシート上から完全自動化します。

VBA マクロを一切記述することなく、 **標準の `.xlsx`（マクロ無効ブック）形式のままで Modbus フレームの自動構築、CRC-16 / LRC / MBAP ヘッダ計算・検証、レジスタ値のセル自動展開** が安全に実行できます。

> [!NOTE]
> **利用前提とライセンス体系**  
> 本エクステンションは [TermAddIn 本体](https://github.com/Monaca-App/TermAddIn) 上で動作します。  
> TermAddIn ではプロトコル毎に個別ライセンスを設ける体系を採用しており、本機能のご利用には **`TermAddIn_Modbus` 個別ライセンス（一括 ¥49,000 / 月額 ¥4,400 / 週額 ¥1,500・税別）** が適用されます（ライセンス未配置・無効時は、Excel ブック再オープン時の既存ボタン無効化、および `Term.exe` 単体起動時の 3 分間期間限定版として動作します）。

---

## ✨ 主な機能・特徴

1. **RTU / ASCII / TCP の 3大伝送モードを単一エクステンションで完全網羅**
   * RS-485 / RS-232C 経由のバイナリ通信 **Modbus RTU** 、テキスト形式の **Modbus ASCII** 、および Ethernet LAN 経由の **Modbus TCP** に対応。`Modbus_Mode` コマンド 1 つで下位トランスポートとフレーミングエンジンを即座に動的切り替え。
2. **煩雑なチェックサム・フレーム整合性を完全自動化**
   * RTU モードの CRC-16 計算、ASCII モードの LRC 計算・コロン（`:`）開始/CRLF 終端フレーミング、および TCP モードの 7 バイト MBAP ヘッダ（トランザクション ID・プロトコル ID・長さフィールド）の生成・検証をすべて内部エンジンが自動処理。
3. **主要な読み出し・書き込みファンクションコードを標準サポート**
   * **読み出し**: FC01（コイル読出）、FC02（入力ステータス読出）、FC03（保持レジスタ読出）、FC04（入力レジスタ読出）
   * **書き込み**: FC05（単一コイル書込）、FC06（単一保持レジスタ書込）、FC15 / `0x0F`（複数コイル書込）、FC16 / `0x10`（複数保持レジスタ書込）
4. **変数・インデックス付きキーへのデータバインドとセル自動展開**
   * 読み出したワードデータやビット列をスカラー変数へのカンマ区切り格納、または指定ターゲット変数名に対する連番キー（`MB_DATA[0]`, `MB_DATA[1]` 等、`$KEY$` 変数体系）への自動インデックス展開に対応。Excel シートの複数セルへ一括流し込みが可能。
5. **Excel 32bit / 64bit 完全動作マトリクス**
   * .NET Framework 標準のマネージド通信基盤およびソケット通信スタックを採用しており、外部の専用ネイティブ DLL に依存しないため、Excel 32bit 版および 64bit 版の双方で制限なく完全動作。
6. **実機・ドライバ不要の Modbus 仮想スレーブ＆電文モニター連携（`TermEmulator` / `c2c` 対応）**
   * **[TermEmulator](https://github.com/Monaca-App/TermEmulator)** の Modbus 仮想スレーブ（TCP / RTU / ASCII・共有メモリ対応、本ライセンス共用可）および公式無償ツール **[c2c](https://github.com/Monaca-App/c2c)** の仮想 COM ペア・TCP プロキシと組み合わせることで、実機なしでの検証とリアルタイム HEX フレーム監視を行えます。

---

## 🔌 提供コマンド一覧（全 5 コマンド）

各コマンドの詳細な構文・パラメータ仕様・記述例は、以下の個別リファレンスをご参照ください。

| コマンド名 | 構文概要 | 主な役割・用途 | 詳細マニュアル |
| :--- | :--- | :--- | :---: |
| **`Modbus_Open`** | `Modbus_Open` | 設定された伝送モード（UART/TCP）に基づき Modbus マスター回線をオープン | [詳細説明](./Manual/Modbus/Modbus_Open.Md) |
| **`Modbus_Close`** | `Modbus_Close` | オープン中の Modbus セッションを安全に切断・リソース解放 | [詳細説明](./Manual/Modbus/Modbus_Close.Md) |
| **`Modbus_Mode`** | `Modbus_Mode, "<ProtocolMode>", "[CommProperty]", "[Handshake]"` | 伝送モード（`RTU`/`ASCII`/`TCP`）およびシリアル通信条件・フロー制御の動的変更 | [詳細説明](./Manual/Modbus/Modbus_Mode.Md) |
| **`Modbus_Read`** | `Modbus_Read, "<SlaveAddr>", "<FuncCode>", "<StartAddr>", "<Points>", "<Target>"` | スレーブ機器からレジスタ/ビットデータを読み出し変数・配列へ格納（FC01〜04） | [詳細説明](./Manual/Modbus/Modbus_Read.Md) |
| **`Modbus_Write`** | `Modbus_Write, "<SlaveAddr>", "<FuncCode>", "<StartAddr>", "<Values>"` | スレーブ機器へ単一/複数レジスタまたはビットデータを書き込み（FC05/06/15/16） | [詳細説明](./Manual/Modbus/Modbus_Write.Md) |

---

## 💻 動作環境・システム要件

| 項目 | システム要件 |
| :--- | :--- |
| **OS** | Windows 10 / Windows 11 (64bit / 32bit) |
| **対応 Excel** | Microsoft Excel 2016 / 2019 / 2021 / Microsoft 365 (デスクトップ版 32bit / 64bit) |
| **必須アドイン** | [TermAddIn](https://github.com/Monaca-App/TermAddIn) (Excel VSTO アドイン) |
| **ランタイム** | .NET Framework 4.8 / Visual Studio 2010 Tools for Office Runtime |
| **インターフェース** | RS-485, RS-422, RS-232C, USB 仮想 COM ポート, [c2c](https://github.com/Monaca-App/c2c) 仮想ヌルモデムポート, Ethernet (10BASE-T/100BASE-TX/1000BASE-T) |
| **対応プロトコル** | Modbus RTU / Modbus ASCII / Modbus TCP |
| **対象機器** | 温調計（理化工業・オムロン・アズビル等）、インバータ、電力計、PLC（Modbus スレーブ対応機全般） |

---

## 🚀 インストール手順

<a href="https://github.com/Monaca-App/TermAddIn_Modbus/raw/main/Setup/TermAddIn_Modbus_Setup.zip">
  <img src="images/Monaca_app_get.jpg" width="140" alt="プログラム（入手）" />
</a>

> 上記の **「プログラム（入手）」アイコン** をクリックすると、セットアッププログラム一式（[`TermAddIn_Modbus_Setup.zip`](https://github.com/Monaca-App/TermAddIn_Modbus/raw/main/Setup/TermAddIn_Modbus_Setup.zip)）が直接ダウンロードされます。展開後、`TermAnalysis.Modbus\setup.exe`（または `TermAnalysis.Modbus.Setup.msi`）を実行してください。

> [!IMPORTANT]
> **導入前提**  
> 本エクステンションを利用するには、事前に **[TermAddIn 本体](https://github.com/Monaca-App/TermAddIn)** （`Term.Setup` および `TermAddIn.Setup`）をインストールしておく必要があります。

### 配布アーカイブ（本リポジトリ `Setup/` 配下）

* **アーカイブ**: [`Setup/TermAddIn_Modbus_Setup.zip`](https://github.com/Monaca-App/TermAddIn_Modbus/raw/main/Setup/TermAddIn_Modbus_Setup.zip)（収録内容: `TermAnalysis.Modbus/setup.exe`, `TermAnalysis.Modbus/TermAnalysis.Modbus.Setup.msi`）

### 導入ステップ

1. **Microsoft Excel** および **Term** をすべて終了します。
2. ダウンロードした **`TermAddIn_Modbus_Setup.zip`** を展開し、 **`TermAnalysis.Modbus/`** フォルダ内の **`setup.exe`** （または **`TermAnalysis.Modbus.Setup.msi`** ）を実行します。
   * ※ 管理者権限（UAC 承認）が必要です。
   * ※ 拡張 DLL は共通インストール先（`C:\Program Files (x86)\Monacaアプリ工房\Monaca Common\`）へ配置され、サンプル資材が `C:\Program Files (x86)\Monacaアプリ工房\Monaca Sample\Modbus\` へ導入されます。
3. インストール完了後、Excel（または `Term.exe`）を起動し、メイン画面の **`[通信モード]`** メニューから **`MODBUS`** が選択できることを確認します。

### アンインストール手順
* Windows の「設定」>「アプリ」>「インストールされているアプリ」（または「コントロールパネル」の「プログラムと機能」）から **`TermAnalysis.Modbus.Setup`** を選択してアンインストールしてください。

---

## 🔰 シーケンス記述例・同梱サンプル資材（`Sample/Modbus/`）

Excel シート上での典型的な Modbus 通信シーケンスの構成例です。

```csv
; 1. Modbus RTU モード（19200bps, Even, 8bit, Stop 1）を設定してオープン
,SEQUENCE, 010, Modbus_Mode, "RTU", "19200,E,8,1", "NONE"
,SEQUENCE, 020, Modbus_Open
; 2. スレーブ局番 1 の保持レジスタ（FC03）アドレス 0x0000 から 2 ワードを読出
,SEQUENCE, 030, Modbus_Read, "1", "03", "0x0000", "2", "READ_VAL"
; 3. スレーブ局番 1 の設定値レジスタ（FC06）アドレス 0x0010 に 1500 を書き込み
,SEQUENCE, 040, Modbus_Write, "1", "06", "0x0010", "1500"
; 4. 回線をクローズして終了
,SEQUENCE, 050, Modbus_Close
,SEQUENCE, 060, Exit
```

### 同梱サンプルマクロ・AI自動生成プロンプト集（RTU / ASCII / TCP）
Modbus の全 3 伝送モードそれぞれに対応した動作確認用マクロスクリプト（CSV）および AI 自動生成プロンプトが **`Sample/Modbus/`** に収録されています（インストール時には `C:\Program Files (x86)\Monacaアプリ工房\Monaca Sample\Modbus\` へ配置され、デスクトップの `Monaca\Sample` ショートカットから参照できます）。

| 伝送モード | サンプルマクロ（Shift-JIS CSV） | AI 自動生成プロンプト |
| :--- | :--- | :--- |
| **Modbus RTU** （バイナリ＋CRC-16） | [`Sample/Modbus/Modbus_RTU.csv`](./Sample/Modbus/Modbus_RTU.csv) | [`Sample/Modbus/Modbus_RTU_prompt.md`](./Sample/Modbus/Modbus_RTU_prompt.md) |
| **Modbus ASCII** （テキスト＋LRC） | [`Sample/Modbus/Modbus_ASCII.csv`](./Sample/Modbus/Modbus_ASCII.csv) | [`Sample/Modbus/Modbus_ASCII_prompt.md`](./Sample/Modbus/Modbus_ASCII_prompt.md) |
| **Modbus TCP** （Ethernet MBAP・Port 502） | [`Sample/Modbus/Modbus_TCP.csv`](./Sample/Modbus/Modbus_TCP.csv) | [`Sample/Modbus/Modbus_TCP_prompt.md`](./Sample/Modbus/Modbus_TCP_prompt.md) |

---

## 💳 ライセンス・購入プラン（TermAddIn_Modbus）

> [!NOTE]
> **試用動作およびオンライン決済について**  
> ライセンス未配置・無効時は、Excel ブック再オープン時の既存ボタン無効化（新規追加ボタンのみ試用可能）、および `Term.exe` 単体起動時の 3 分間期間限定版として動作します。  
> 用途や利用期間に合わせて、一括購入（買い切り）またはサブスクリプション（月額・週額）からお選びいただけます。メールアドレスの誤入力によるライセンス不着を防ぐため、 **事前のメールアドレス確認** を行った上で、 **Stripe** による安全なオンライン決済へご案内いたします（表記価格はすべて税別です）。

| プラン | 価格（税別） | お支払い・お申込み |
| :--- | :---: | :---: |
| **一括購入（買い切りライセンス）**<br>無期限で永続利用できるライセンスです。 | **¥49,000** | [👉 一括購入手続きへ](https://stripe-license-generator.y-nakai-5a3.workers.dev/?target=modbus) |
| **月額サブスクリプション**<br>月単位で利用できる定期プランです。中長期のプロジェクトにおすすめです。 | **¥4,400** / 月 | [👉 月額プランに申し込む](https://stripe-license-generator.y-nakai-5a3.workers.dev/?target=modbus) |
| **週額サブスクリプション**<br>週単位で手軽に利用できるプランです。短期評価や単発テストにおすすめです。 | **¥1,500** / 週 | [👉 週額プランに申し込む](https://stripe-license-generator.y-nakai-5a3.workers.dev/?target=modbus) |

* ※ **ライセンスファイル名**: `TermAddIn_Modbus.License`
* ※ **配置先**: `%USERPROFILE%\Desktop\Monaca\TermAddIn_Modbus.License`（設定ファイル `Setting.xml` と同階層）
* ※ **ライセンス形態**: 1 ユーザー / 1 PC（ノードロック方式：`PC名` および `ログインユーザー名` に紐づけ）

---

## 📄 ライセンス & サポート

* **開発元**: [Monaca アプリ工房 (Monaca App Store)](https://github.com/Monaca-App/store)
* **親プロジェクト**: [TermAddIn](https://github.com/Monaca-App/TermAddIn)
* **関連コンパニオンツール**: [c2c - ドライバ不要の仮想ヌルモデム＆通信モニター](https://github.com/Monaca-App/c2c)
* **Copyright**: Copyright (C) Monaca App Store 2026 All rights reserved.
* **お問い合わせ**: 不具合報告や機能要望は、GitHub の Issue までお気軽にお寄せください。
