# examples ディレクトリの書き方(directory-jsonschema)

`json-examples.schema.json`(rin00159/json-examples-schema)を使った `examples/` の書き方を、現行リポジトリの実物から書き起こしたもの。
`json-examples.schema.json` 自体の仕様説明は含めない。ここでは「どこに・何を・どう置くか」だけを扱う。

> 注記: 以下は `examples/0/0/` 配下の4ファイルと `schema/0.0.0/` を読み取った結果。
> 「観測」と書いた箇所は、実物から読み取った慣習であり、明文の規則が別にあるわけではない。

---

## 1. ディレクトリ構成

```
directory-jsonschema/
├── schema/
│   └── 0.0.0/
│       └── directory.schema.json        ← テスト対象のスキーマ本体
└── examples/
    └── 0/                               ← メジャーバージョン「0」
        ├── directory_defs-struct.json-examples.json   ← メジャー0を通して有効な例(1.1)
        └── 0/                           ← スキーマのバージョンに対応
            ├── directory.json-examples.json
            ├── directory_defs-struct.json-examples.json
            └── schema/
                ├── directory.json-examples.schema.json
                └── directory_defs-struct.schema.json-examples.json
```

- 1つの「テスト対象」につき、**例ファイル(`.json`)と、それ専用のラッパースキーマ(`schema/*.schema.json`)の2点セット**を置く。
- バージョン階層は `examples/0/0/` のように `schema/0.0.0/` に対応して切る(観測: `0.0.0` に対して `0/0` なので、パッチ番号は階層にしていない)。

### 1.1 メジャーバージョン全体で有効な例(`examples/<数字>/` 直下)

`examples/<数字>/` の **直下** に例ファイルを置くと、そのメジャーバージョン全体を通して有効な例を記述できる。

```
examples/
└── 0/
    ├── directory_defs-struct.json-examples.json   ← メジャー「0」のどのバージョンでも有効
    └── 0/
        └── directory_defs-struct.json-examples.json   ← このバージョン固有の例
```

| 置き場所 | 意味 |
|---|---|
| `examples/<メジャー>/<マイナー>/` | そのバージョンでだけ有効な例 |
| `examples/<メジャー>/`(直下) | そのメジャーバージョンを通して有効な例 |

- 例えば `examples/0/directory_defs-struct.json-examples.json` に置いた例は、メジャーバージョン「0」の全体で成り立つ内容として扱う。
- ファイル名・中身(`jsons` の書き方)は、バージョン階層の中に置く場合と同じ規則(2、3 を参照)。
- 未確認の点: 直下に置くファイルの `$schema` とラッパースキーマ(`schema/`)の置き場所・相対パスは、現行リポジトリに実例がないため、ここでは規定していない。階層が1つ浅くなるので、`_hook_valid` の相対パス(`../` の数)も1つ減る点には注意する。

## 2. ファイル名の規則

| 種類 | 形式 | 例 |
|---|---|---|
| 例ファイル | `<対象>.json-examples.json` | `directory.json-examples.json` |
| ラッパースキーマ | `<対象>.json-examples.schema.json` | `directory.json-examples.schema.json` |

`<対象>` の決め方(観測):

| テスト対象 | `<対象>` |
|---|---|
| `directory.schema.json` 全体 | `directory` |
| `directory.schema.json#/$defs/struct` | `directory_defs-struct` |

つまり `<スキーマ名>` に、部分を指す場合は `_` で区切って JSON Pointer を付ける。その際 `$` は除き、`/` は `-` に置き換える(`/$defs/struct` → `defs-struct`)。

例ファイルとラッパースキーマは **`<対象>` を完全に一致** させる(拡張子だけ `.json` と `.schema.json` で違う)。

## 3. 例ファイルの書き方

### 3.1 全体の形

```json
{
	"$schema": "./schema/directory.json-examples.schema.json",
	"jsons": [
		{ ... },
		{ ... }
	]
}
```

- トップレベルは `$schema` と `jsons` の2つ。
- `$schema` は **同名のラッパースキーマへの相対パス**(`./schema/<対象>.json-examples.schema.json`)にする。
- `jsons` に例を1件ずつ並べる。
- インデントはタブ。

### 3.2 1件の例

```json
{
	"group": "basic",
	"label": "ルート配列: dir + report",
	"valid": true,
	"instance": [ ... ]
}
```

| キー | 役割(慣習) |
|---|---|
| `group` | 例の分類。後述の4種を使う |
| `label` | 何を確かめる例かを短い日本語で書く |
| `valid` | `instance` がテスト対象スキーマに **通るべきなら `true`**、**通ってはいけないなら `false`** |
| `instance` | テストする実データ本体。`valid` の判定対象 |

必要に応じて次のキーを足す(観測)。

| キー | 使いどころ |
|---|---|
| `group-pointer` | その例が検証している定義の場所。`/$defs/common/type`、`/$defs/common/name`、`/$defs/file-data`、`/$defs/reference`、`/$defs/logic`、`/$defs/directory-body`、`/concrete`、`/$defs/struct` など |
| `pointer` | 個別の例が検証しているスキーマ内のより細かい場所。複数あるときは配列で書く(例: `/name`、`/type`、`["/$defs/reference/properties/$ref", "/$defs/directory-body/properties/contents"]`) |
| `name` | 関連する例をまとめる名前。使われている値は `JSONSchema-Array base`、`size`、`recursive-closure` |
| `valid@when_formatAssertion` | 検証系の `format` を厳密に評価する場合だけ結果が変わる例で、そのときの期待値を書く。通常の `valid` と食い違うときに使う(例: 不正な正規表現を `pattern` に書いた例は、通常は `valid: true`、format 評価時は `false`) |
| `expectedErrors` | `valid: false` の例で、エラーの出る場所と種類を固定したいときに書く。`[{ "instanceLocation": "/$defs/ts-tree", "keyword": "unevaluatedProperties" }]` の形 |

### 3.3 キーの並び(観測)

```
group → group-pointer → name → pointer → label → valid → valid@when_formatAssertion → instance → expectedErrors
```

必要なキーだけを、この順で書く。`label` は全150件中149件にあり(1件だけ省略)、基本的には付ける。

### 3.4 `group` の種類(観測)

| `group` | 内容 |
|---|---|
| `basic` | 全体像を示す基本形。実用的な通しの例 |
| `property:root` | ルート(`struct` / `concrete` など)に関する例 |
| `property` | 各ノードのプロパティ別の例。`group-pointer` で対象の定義を示す |
| `property:defs` | `$defs` / `$ref` / 再帰定義など、定義まわりの誤用や応用の例 |

`property` の例は `group-pointer` ごとにまとめて並べる(`type` → `name` → `file-data` → `reference` → `logic` → `directory-body` の順に出現)。

### 3.5 `label` の書き方(観測)

`valid` の向きが、ラベルの言い回しで分かるようにしている。

| 種類 | 言い回し | 例 |
|---|---|---|
| 通るべき(基本) | `正常: ...` | `正常: type と name` |
| 通るべき(許容範囲の確認) | `補足を許可: ...` | `補足を許可: size の範囲` |
| 通ってはいけない | `...は不可` | `'.' は不可`、`type 必須`、`allOf は不可(ノード)` |
| 使い方の誤り | `誤用: ...` | `誤用: $ref は文字列でなければならない` |
| 仕様の使い方の見本 | 目的と仕組みを1文で | `全階層を閉じる: どの階層でも ... (items + 自己参照)` |

- 1つの例で **確かめる事柄は1つ** にする。不可の例は、不可になる理由が1つだけになるように作る。
- 対にできるものは、通る例と通らない例を並べて置く(`'.gitignore' は可` と `'.' は不可`、など)。

### 3.6 `instance` の書き方

- テスト対象スキーマの対象が配列なら配列、オブジェクトならオブジェクトを書く。
- 例ファイルごとの対象:
  - `directory.json-examples.json` → ルートが **配列**(ディレクトリと report の並び)。
  - `directory_defs-struct.json-examples.json` → ルートが **struct オブジェクト**(`contents`、`$schema`、`description`、`report` など)。
- 最小限の中身にとどめ、確かめたい部分以外は書かない。

## 4. ラッパースキーマの書き方

`examples/0/0/schema/<対象>.json-examples.schema.json` は、例ファイルの `jsons[*].instance` を、`valid` に応じてテスト対象スキーマで検証させるためのもの。
2つの既存ファイルは **`_hook_valid` の `$ref` 以外が全く同じ** なので、新しい対象を足すときはコピーして、その1か所だけ変える。

```json
{
	"$schema": "https://json-schema.org/draft/2020-12/schema",
	"allOf": [
		{
			"$ref": "https://raw.githubusercontent.com/rin00159/json-examples-schema/refs/heads/main/schema/0.0.0/json-examples.schema.json"
		}
	],
	"$defs": {
		"_hook_valid": {
			"$ref": "../../../../schema/0.0.0/directory.schema.json"
		}
	},
	"type": "object",
	"properties": {
		"jsons": {
			"type": "array",
			"items": {
				"type": "object",
				"if": {
					"properties": {
						"valid": { "const": true }
					}
				},
				"then": {
					"properties": {
						"instance": { "$ref": "#/$defs/_hook_valid" }
					}
				},
				"else": {
					"properties": {
						"instance": {
							"not": { "$ref": "#/$defs/_hook_valid" }
						}
					}
				}
			}
		}
	}
}
```

変える場所:

| 場所 | 内容 |
|---|---|
| `$defs._hook_valid.$ref` | テスト対象スキーマへの相対パス。全体なら `../../../../schema/0.0.0/directory.schema.json`、一部なら末尾に `#/$defs/struct` のようにフラグメントを付ける |

仕組み:

```
valid: true  → instance は _hook_valid に「通る」べき
valid: false → instance は _hook_valid に「通らない」べき(not で反転)
```

`../../../../` は、`examples/0/0/schema/` からリポジトリのルートまでの4階層(`schema` → `0` → `0` → `examples` → ルート)に当たる。バージョン階層の深さを変えたら、ここも合わせる。

## 5. 新しい例を足す手順

### 既存の対象に例を足す場合

1. 対象の例ファイル(例: `directory_defs-struct.json-examples.json`)の `jsons` に1件追加する。
2. `group`、(あれば `group-pointer`)、`label`、`valid`、`instance` を、3.3 の並びで書く。
3. 近い例(同じ `group-pointer`)の隣に置く。通る例と通らない例は対にして並べる。

### 新しい対象(別スキーマ、または `$defs` の別の部分)を足す場合

1. 対象の名前を決める(2 の規則)。例: `directory.schema.json#/$defs/logic` → `directory_defs-logic`。
2. `examples/0/0/schema/directory_defs-logic.json-examples.schema.json` を、既存のラッパースキーマをコピーして作り、`_hook_valid` の `$ref` を `…/directory.schema.json#/$defs/logic` に変える。
3. `examples/0/0/directory_defs-logic.json-examples.json` を作り、`$schema` を `./schema/directory_defs-logic.json-examples.schema.json` にして、`jsons` に例を書く。

## 6. チェックリスト

- [ ] メジャー全体で有効な例は `examples/<メジャー>/` 直下、特定バージョン固有の例は `examples/<メジャー>/<マイナー>/` に置き分けている
- [ ] 例ファイルと、ラッパースキーマの名前の `<対象>` が一致している
- [ ] 例ファイルの `$schema` が `./schema/` 配下の同名ラッパースキーマを指している
- [ ] ラッパースキーマの `_hook_valid` の `$ref` が正しい対象を指している(相対パスの階層数も確認)
- [ ] 各例に `group`・`label`・`valid`・`instance` がある
- [ ] 通らない例は、通らない理由が1つに絞られている
- [ ] 実装依存で結果が変わる例には `valid@when_formatAssertion` を付けている
- [ ] エラーの種類まで固定したい誤用の例には `expectedErrors` を付けている
- [ ] インデントはタブ