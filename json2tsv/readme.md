# json2tsv-4evergreen

**JSON で記述された論文メタデータ**を、機関リポジトリ（QIR想定）の一括登録用TAB区切りテキスト（TSV）に変換するノートブック／変換仕様・サンプル一式です。  
JPCOARスキーマv2.0の必須・推奨項目のうち、本プロジェクトで運用している **Evergreen** 向けの項目セットに対応しています。

*   変換ロジック本体：**`json2tsv-4evergreen.ipynb`**（Python）
*   仕様書：**`マッピング表.xlsx`**（JSON → TSV のマッピング・変換ルール）
*   サンプル入力：**`json_sample_1.json`**（軽量版）、**`json_sample_2.json`**（重厚版）
*   サンプル出力：**`output_sample_1.txt`**（sample\_1に対応）、**`output_sample_2.txt`**（sample\_2に対応）
*   生成AI用プロンプト：**`prompt_for_AI_code_generation.txt`**（要件定義と出力仕様の原本）

> **想定ユースケース**
>
> *   研究成果特集号など多数の論文を **一括でQIRに登録**。
> *   著者数・所属数・キーワード数が論文ごとに異なる動的列展開（#n インクリメント）に対応。

***

## ✨ 主な特徴

*   **JSON → TSV の一括変換**（作業ディレクトリ内の `*.json` を逐次処理）
*   **JPCOAR2.0 ベースの列設計**（Evergreen向け最小コア＋運用固定値を同梱）
*   **動的な繰り返し要素**に対応
    *   `/jpcoar:creator#n`（著者）
    *   `/jpcoar:creator#n/jpcoar:affiliation#m`（著者の所属）
    *   `/jpcoar:subject#k`（キーワード）  
        → JSON の要素数に応じて **列を自動増殖**（#1,#2,…をインクリメント）
*   **言語タグ `@xml:lang` の簡易判定**（かな漢字なら `ja`、アルファベットなら `en`）
*   **発行年月（YYYY-MM）**・**巻/号（先頭ゼロ除去）**・**ページ**を JSON から抽出して整形
*   **ORCID 検証**（形式不一致は空欄化）と **URI 構築**（`https://orcid.org/{id}`）
*   **固定値運用**（権利/出版社/ISSN 等）はノートブックの設定ブロックで一元管理（将来変更に強い）

***

## 🗂 リポジトリ構成

    .
    ├─ json2tsv-4evergreen.ipynb      # 変換ノートブック（Python）
    ├─ json_sample_1.json             # サンプル（軽量） … output_sample_1.txt を生成
    ├─ json_sample_2.json             # サンプル（重厚） … output_sample_2.txt を生成
    ├─ output_sample_1.txt            # 変換結果サンプル（TSV, sample_1 に対応）
    ├─ output_sample_2.txt            # 変換結果サンプル（TSV, sample_2 に対応）
    ├─ マッピング表.xlsx             # JSON→TSV マッピング仕様書（JPCOAR2.0ベース）
    ├─ prompt_for_AI_code_generation.txt  # 生成AI用プロンプト（要件定義）
    └─ readme.md                      # ← 本ドキュメント

> 参考：`output_sample_1.txt` のヘッダは、Creator/Affiliation/Subject の繰り返し列を含む完全な実例です。実際の列名（`/dc:title#1`, `/jpcoar:creator#1/...`, …）や固定値の配置は、このファイルを見ると理解が早いです。

***

## 🔧 セットアップと実行

1.  **環境**
    *   Jupyter（JupyterHub など）で実行可。Python 3.9+ 推奨。
2.  **入力ファイルの配置**
    *   変換対象の `*.json` をノートブックと同じ作業ディレクトリに置きます。
3.  **ノートブックの実行**
    *   `json2tsv-4evergreen.ipynb` を開き、上から順に実行します。
    *   変換結果は **`output.txt`** に 1 論文 1 行で出力されます（ヘッダ 1 行＋データ N 行）。
4.  **QIR への取り込み**
    *   `output.txt` は QIR の評価環境で **一括登録**として取り込み確認済み（サンプルでの実証）。

***

## 📑 JSON 仕様（入力）

*   **著者と所属**
    *   `authors[*].name` / `authors[*].orcid` / `authors[*].affiliation_numbers`（例：`["1","2"]`）
    *   `affiliations` は辞書（例：`{"1": "Dept. A", "2": "Dept. B"}`）
    *   これらを結合して、`/jpcoar:creator#n/jpcoar:affiliation#m/jpcoar:affiliationName#1` を展開します。
*   **キーワード**：`keywords`（配列）→ `/jpcoar:subject#k` 列へ展開。
*   **抄録**：`abstract` → `/datacite:description#1`（`@descriptionType=Abstract` 固定）。
*   **発行情報**：`publication_info.year` / `publication_info.month` / `publication_info.volume` / `publication_info.issue`
    *   `month` は英語名/略称/数値いずれも許容し **MM** に正規化（例：`December`→`12`）。
    *   `issue` は先頭ゼロを除去（例：`"04"`→`4`）。
*   **ページ情報**：`page_info.start_page` / `page_info.end_page`。

> 参考：`json_sample_1.json` は 3 名著者（所属 2/1/1）・キーワード 6 件の例です。

***

## 🧾 TSV 仕様（出力）

*   行構造：**ヘッダ1行＋データN行**。
*   列構造：JPCOAR2.0 準拠のフィールド名（`/dc:title#1`, `/jpcoar:creator#1/...` など）。
*   **繰り返し列の展開ルール**（マッピング表の補足）
    *   `/jpcoar:creator#n`：JSON `authors` の順で **n=1..N** を展開。
    *   `/jpcoar:creator#n/jpcoar:affiliation#m`：当該著者の `affiliation_numbers` の順で **m=1..M**。
    *   `/jpcoar:subject#k`：`keywords` の順で **k=1..K**。
    *   各“ヘッダ列”（`/jpcoar:creator#n` や `/jpcoar:creator#n/jpcoar:affiliation#m`）の **値は空セル** を出力します。
*   固定値の例：
    *   `/dcterms:accessRights#1=110`、`/dc:rights#1=Creative Commons Attribution 4.0 International`（`@xml:lang=en`、`@rdf:resource=https://creativecommons.org/licenses/by/4.0/`）
    *   出版者：`/dc:publisher#1=Transdisciplinary Research and Education Center for Green Technologies, Kyushu University`（`@xml:lang=en`）／`/dc:publisher#2=九州大学グリーンテクノロジー研究教育センター`（`@xml:lang=ja`）
    *   `PISSN=2189-0420 / eISSN=2432-5953 / sourceTitle=evergreen / language=eng / contentsType=1207000000 / version=VoR / peerReviewed=refereed`  
        （いずれもノートブック先頭の設定ブロックで変更可能）

***

## ✅ 品質チェック（妥当性）

*   **ORCID**：`0000-0000-0000-0000` 形式の検証に通らない値は空欄化。URI も未付与。 
*   **言語タグ**：`@xml:lang` は値の文字種で簡易判定（かな・漢字→`ja`、A-Z→`en`）。 
*   **月名**：英語名/略称/数値を `MM` に正規化（例：`December`→`12`）。 
*   **号（issue）**：先頭ゼロ除去（`"04"`→`4`）。

***

## 🧪 動作確認用サンプル

*   `json_sample_1.json` → `output_sample_1.txt`
    *   著者3名・所属 2/1/1・キーワード6件の完全例。列ヘッダの増殖と `@xml:lang`、ORCID/URI を確認できます。
*   `json_sample_2.json` → `output_sample_2.txt`
    *   著者数・所属数・キーワード数がより多い重厚例（列増殖の上限確認に有用）。

***

## 🧩 よくある質問（FAQ）

**Q. 著者の所属が TSV に出ない場合は？**  
A. JSON の `affiliation_numbers` の ID が `affiliations{}` に存在しないとスキップされます。ID と辞書キーの **表記ゆれ**（文字列/数字・スペース混在など）を確認してください。

**Q. 月が文字列（例：December）でも動く？**  
A. はい。英語名/略称/数値のいずれも `MM` に正規化します。

**Q. QIR で取り込めるか不安です。**  
A. サンプル `output_sample_1.txt` は QIR の評価環境で取り込み確認済みです（同一の列構造で出力します）。

***

## 🛠 保守と拡張

*   **JSON スキーマが変わったら？**  
    → `マッピング表.xlsx` とノートブック設定ブロックを更新してください。繰り返しのルール（`#n` の増殖）は不変です。
*   **引き継ぎ運用**  
    → 固定値は設定ブロックに集約しています。将来のポリシー変更（出版社名・権利等）は **そこで一括変更**できます。
*   **生成AIの活用**  
    → `prompt_for_AI_code_generation.txt` は、要件定義としても利用でき、コードの再生成・微修正に有効です。

> **補足（離任時の備え）**  
> たとえメンテナンス者が不在になっても、サンプル JSON/TSV とマッピング表、そして本READMEがあれば、後任者や生成AI（Copilot など）が容易に復元・保守できます。

***

## 🤝 貢献

*   Issue／Pull Request は歓迎です。
*   大きな変更（スキーマ変更や列追加）は、まず Issue で方針共有をお願いします。
*   スタイル：PEP8準拠、ノートブックは実行順（上→下）で再現可能な状態でコミット。

***

## 📄 ライセンス

*   MIT License
*   変換後のメタデータ（TSV）に含まれる **論文情報は原著者・出版社の権利**に従います。

***

## 🙏 謝辞

*   Evergreen編集チームの皆さま、Kyaw Thu先生

***

### 付録：確認チェックリスト

*   [ ] `output.txt` にヘッダ行が 1 行だけ存在する
*   [ ] `/jpcoar:creator#n` ブロックが著者数分だけ展開されている
*   [ ] 各著者の所属が `affiliation#n` として展開されている
*   [ ] `/jpcoar:subject#k` がキーワード数分だけ展開されている
*   [ ] `/datacite:date#1` が `YYYY-MM` 形式、`issue` の先頭ゼロは除去
*   [ ] ORCID が正規表現に合致しない場合は空欄・URI 未付与
*   [ ] 固定値（rights, publisher, ISSN など）がポリシーどおり

***
