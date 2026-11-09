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
> TermAddIn ではプロトコル毎に個別ライセンスを設ける体系を採用しており、本機能のご利用には **`TermAddIn_Modbus` 個別ライセンス（料金未定・準備中）** が適用されます（現在は評価期間として全機能を無償でご試用いただけます）。

---

## ✨ 主な機能・特徴

1. **RTU / ASCII / TCP の 3大伝送モードを単一エクステンションで完全網羅**
   * RS-485 / RS-232C 経由のバイナリ通信 **Modbus RTU**、テキスト形式の **Modbus ASCII**、および Ethernet LAN 経由の **Modbus TCP** に対応。`Modbus_Mode` コマンド 1 つで下位トランスポートとフレーミングエンジンを即座に動的切り替え。
2. **煩雑なチェックサム・フレーム整合性を完全自動化**
   * RTU モードの CRC-16 計算、ASCII モードの LRC 計算・コロン（`:`）開始/CRLF 終端フレーミング、および TCP モードの 7 バイト MBAP ヘッダ（トランザクション ID・プロトコル ID・長さフィールド）の生成・検証をすべて内部エンジンが自動処理。
3. **主要な読み出し・書き込みファンクションコードを標準サポート**
   * **読み出し**: FC01（コイル読出）、FC02（入力ステータス読出）、FC03（保持レジスタ読出）、FC04（入力レジスタ読出）
   * **書き込み**: FC05（単一コイル書込）、FC06（単一保持レジスタ書込）、FC15 / `0F`（複数コイル書込）、FC16 / `10`（複数保持レジスタ書込）
4. **変数・配列へのデータバインドとセル自動展開**
   * 読み出したワードデータやビット列をスカラー変数（`$VAR`）へのカンマ区切り格納、または配列（`#ARR`）への自動インデックス展開に対応。Excel シートの複数セルへ一括流し込みが可能。
5. **Excel 32bit / 64bit 完全動作マトリクス**
   * .NET Framework 標準のマネージド通信基盤およびソケット通信スタックを採用しており、外部の専用ネイティブ DLL に依存しないため、Excel 32bit 版および 64bit 版の双方で制限なく完全動作。

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
| **インターフェース** | RS-485, RS-422, RS-232C, USB 仮想 COM ポート, Ethernet (10BASE-T/100BASE-TX/1000BASE-T) |
| **対応プロトコル** | Modbus RTU / Modbus ASCII / Modbus TCP |
| **対象機器** | 温調計（理化工業・オムロン・アズビル等）、インバータ、電力計、PLC（Modbus スレーブ対応機全般） |

---

## 🔰 シーケンス記述例（Excel セル設定）

Excel シート上での典型的な Modbus 通信シーケンスの構成例です。

```csv
; 1. Modbus RTU モード（19200bps, Even, 8bit, Stop 1）を設定してオープン
,SEQUENCE, 010, Modbus_Mode, "RTU", "19200,E,8,1", "NONE"
,SEQUENCE, 020, Modbus_Open
; 2. スレーブ局番 1 の保持レジスタ（FC03）アドレス 0x0000 から 2 ワードを読出
,SEQUENCE, 030, Modbus_Read, "1", "03", "0000", "2", "$READ_VAL"
; 3. スレーブ局番 1 の設定値レジスタ（FC06）アドレス 0x0010 に 1500 を書き込み
,SEQUENCE, 040, Modbus_Write, "1", "06", "0010", "1500"
; 4. 回線をクローズして終了
,SEQUENCE, 050, Modbus_Close
,SEQUENCE, 060, Exit
```

---

## 💳 ライセンス・購入プラン（TermAddIn_Modbus）

> [!NOTE]
> **料金プランおよび正式販売について**  
> 本エクステンション（`TermAddIn_Modbus`）の販売料金およびライセンスプラン（買い切り・サブスクリプション）は **現在検討中（未定）** です。  
> 正式な価格およびお申込み受付開始時期が決定次第、本ドキュメントおよび [Monaca アプリ工房ポータル](https://github.com/Monaca-App/store) にてご案内いたします。  
> 現在は評価版として全機能をご試用いただけます。

| プラン | 価格 | お支払い・お申込み |
| :--- | :---: | :---: |
| **一括購入（買い切りライセンス）**<br>無期限で永続利用できるライセンスです。 | **未定** | 準備中 |
| **月額サブスクリプション**<br>月単位で利用できる定期プランです。中長期のプロジェクトにおすすめです。 | **未定** | 準備中 |
| **週額サブスクリプション**<br>週単位で手軽に利用できるプランです。短期評価や単発テストにおすすめです。 | **未定** | 準備中 |

* ※ **ライセンスファイル名**: `TermAddIn_Modbus.License`
* ※ **配置先**: `%USERPROFILE%\Desktop\Monaca\TermAddIn_Modbus.License`（設定ファイル `Setting.xml` と同階層）
* ※ **ライセンス形態**: 1 ユーザー / 1 PC（ノードロック方式：`PC名` および `ログインユーザー名` に紐づけ）

---

## 📄 ライセンス & サポート

* **開発元**: [Monaca アプリ工房 (Monaca App Store)](https://github.com/Monaca-App/store)
* **親プロジェクト**: [TermAddIn](https://github.com/Monaca-App/TermAddIn)
* **Copyright**: Copyright (C) Monaca App Store 2026 All rights reserved.
* **お問い合わせ**: 不具合報告や機能要望は、GitHub の Issue までお気軽にお寄せください。
