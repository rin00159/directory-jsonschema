#
競合のJSONスキーマの検索
https://claude.ai/chat/b47cf6ce-6f1c-4a6d-871c-8c1a6cdd1896
## dirschema
dirschemaの設計は、「ディレクトリ全体を、パスの集合として扱い、各パスにルールを当てる」 というものです。ご提示の「ツリーを写して書く」方式とは対照的なので、比べると参考になる点が多いと思います。

外から valid / validMeta を充てるのが中核
### 記述例
```
# 全ての .jpg に、規約どおりのメタデータがあり、スキーマに適合すること
match: ".*\\.jpg"
validMeta: local://schemas/image_meta.schema.json
```
## eslint-plugin-project-structure
一番自分の設計に近いが、
typeを明示できず、
意味定義や説明の方法も用意されていない。

参考程度になるだけで、ベースにはとてもできない。