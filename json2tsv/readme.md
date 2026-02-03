json2tsv-4evergreen
JSON で記述された論文メタデータを、機関リポジトリ（QIR想定）の**一括登録用 TAB 区切りテキスト（TSV）**に変換するノートブック／変換仕様・サンプル一式です。
JPCOARスキーマ v2.0 の必須・推奨項目のうち、本プロジェクトで運用している Evergreen 向けの項目セットに対応しています。

変換ロジック本体：json2tsv-4evergreen.ipynb（Python）
仕様書：マッピング表.xlsx（JSON → TSV のマッピング・変換ルール） [output_sample_1 | Txt]
サンプル入力：json_sample_1.json（軽量版）、json_sample_2.json（重厚版） [prompt_for...generation | Txt]
サンプル出力：output_sample_1.txt（sample_1に対応）、output_sample_2.txt（sample_2に対応） [qu365-my.s...epoint.com]
生成AI用プロンプト：prompt_for_AI_code_generation.txt（要件定義と出力仕様の原本） [prompt_for...generation | Txt]


想定ユースケース

研究成果特集号など多数の論文を 一括で QIR に登録。
著者数・所属数・キーワード数が論文ごとに異なる **動的列展開（#n インクリメント）**に対応。



✨ 主な特徴

JSON → TSV の一括変換（作業ディレクトリ内の *.json を逐次処理） [prompt_for...generation | Txt]
JPCOAR2.0 ベースの列設計（Evergreen 向け最小コア＋運用固定値を同梱） [output_sample_1 | Txt]
動的な繰り返し要素に対応

/jpcoar:creator#n（著者）
/jpcoar:creator#n/jpcoar:affiliation#m（著者の所属）
/jpcoar:subject#k（キーワード）
→ JSON の要素数に応じて 列を自動増殖（#1,#2,…をインクリメント） [output_sample_1 | Txt]


言語タグ @xml:lang の簡易判定（かな漢字なら ja、アルファベットなら en） [prompt_for...generation | Txt]
発行年月（YYYY-MM）・巻/号（先頭ゼロ除去）・ページを JSON から抽出して整形 [prompt_for...generation | Txt]
ORCID 検証（形式不一致は空欄化）と URI 構築（https://orcid.org/{id}） [qu365-my.s...epoint.com]
固定値運用（権利/出版社/ISSN 等）はノートブックの設定ブロックで一元管理（将来変更に強い） [output_sample_1 | Txt]


🗂 リポジトリ構成
.
├─ json2tsv-4evergreen.ipynb      # 変換ノートブック（Python）
├─ json_sample_1.json             # サンプル（軽量） … output_sample_1.txt を生成
├─ json_sample_2.json             # サンプル（重厚） … output_sample_2.txt を生成
├─ output_sample_1.txt            # 変換結果サンプル（TSV, sample_1 に対応）
├─ output_sample_2.txt            # 変換結果サンプル（TSV, sample_2 に対応）
├─ マッピング表.xlsx             # JSON→TSV マッピング仕様書（JPCOAR2.0ベース）
├─ prompt_for_AI_code_generation.txt  # 生成AI用プロンプト（要件定義）
└─ readme.md                      # ← 本ドキュメント


参考：output_sample_1.txt のヘッダは、Creator/Affiliation/Subject の繰り返し列を含む完全な実例です。実際の列名（/dc:title#1, /jpcoar:creator#1/..., …）や固定値の配置は、このファイルを見ると理解が早いです。 [qu365-my.s...epoint.com]


🔧 セットアップと実行

環境

ローカルの Jupyter（JupyterHub など）で実行可。Python 3.9+ 推奨。


入力ファイルの配置

変換対象の *.json をノートブックと同じ作業ディレクトリに置きます。 [prompt_for...generation | Txt]


ノートブックの実行

json2tsv-4evergreen.ipynb を開き、上から順に実行します。
変換結果は output.txt に 1 論文 1 行で出力されます（ヘッダ 1 行＋データ N 行）。 [prompt_for...generation | Txt]


QIR への取り込み

output.txt は QIR の評価環境で 一括登録として取り込み確認済み（サンプルでの実証）。 [qu365-my.s...epoint.com]




📑 JSON 仕様（入力）

著者と所属（新スキーマ）

authors[*].name / authors[*].orcid / authors[*].affiliation_numbers（例：["1","2"]）
affiliations は辞書（例：{"1": "Dept. A", "2": "Dept. B"}）
これらを結合して、/jpcoar:creator#n/jpcoar:affiliation#m/jpcoar:affiliationName#1 を展開します。 [prompt_for...generation | Txt]


キーワード：keywords（配列）→ /jpcoar:subject#k 列へ展開。 [prompt_for...generation | Txt]
要旨：abstract → /datacite:description#1（@descriptionType=Abstract 固定）。 [output_sample_1 | Txt]
発行情報：publication_info.year / publication_info.month / publication_info.volume / publication_info.issue

month は英語名/略称/数値いずれも許容し MM に正規化（例：December→12）。
issue は先頭ゼロを除去（例："04"→4）。 [prompt_for...generation | Txt]


ページ情報：page_info.start_page / page_info.end_page。 [prompt_for...generation | Txt]


参考：json_sample_1.json は 3 名著者（所属 2/1/1）・キーワード 6 件の例です。 [prompt_for...generation | Txt]


🧾 TSV 仕様（出力）

行構造：ヘッダ1行＋データN行。
列構造：JPCOAR2.0 準拠のフィールド名（/dc:title#1, /jpcoar:creator#1/... など）。
繰り返し列の展開ルール（マッピング表の補足）

/jpcoar:creator#n：JSON authors の順で n=1..N を展開。
/jpcoar:creator#n/jpcoar:affiliation#m：当該著者の affiliation_numbers の順で m=1..M。
/jpcoar:subject#k：keywords の順で k=1..K。
各“ヘッダ列”（/jpcoar:creator#n や /jpcoar:creator#n/jpcoar:affiliation#m）の 値は空セル を出力します。 [output_sample_1 | Txt]


固定値の例：

/dcterms:accessRights#1=110、/dc:rights#1=Creative Commons Attribution 4.0 International（@xml:lang=en、@rdf:resource=https://creativecommons.org/licenses/by/4.0/）
出版者：/dc:publisher#1=Transdisciplinary Research and Education Center for Green Technologies, Kyushu University（@xml:lang=en）／/dc:publisher#2=九州大学グリーンテクノロジー研究教育センター（@xml:lang=ja）
PISSN=2189-0420 / eISSN=2432-5953 / sourceTitle=evergreen / language=eng / contentsType=1207000000 / version=VoR / peerReviewed=refereed
（いずれもノートブック先頭の設定ブロックで変更可能） [qu365-my.s...epoint.com], [output_sample_1 | Txt]




✅ 品質チェック（妥当性）

ORCID：0000-0000-0000-0000 形式の検証に通らない値は空欄化。URI も未付与。 [qu365-my.s...epoint.com]
言語タグ：@xml:lang は値の文字種で簡易判定（かな・漢字→ja、A-Z→en）。 [prompt_for...generation | Txt]
月名：英語名/略称/数値を MM に正規化（例：December→12）。 [prompt_for...generation | Txt]
号（issue）：先頭ゼロ除去（"04"→4）。 [prompt_for...generation | Txt]
