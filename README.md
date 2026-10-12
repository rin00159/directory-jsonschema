開発に使ったセッション:
https://claude.ai/share/9ddc0999-005c-46ec-b6c6-0bafc69f3764

参考になる類似例:
https://gemini.google.com/app/5a40255e79b6bf0a
・特にFrictionless Data　はかなり同じだが、「フラットだったり、モダンで書きやすい」のが自分の利点。
    あとschema記述にはあまり対応してなさそう
・Ansible (ansible.builtin.file) の仕様
    Frictionless Dataはシンボリックリンク非対応だがこちらは対応している。
        プロパティの参考にすべき。
tag:Frictionless Data,Ansible

yaml対応に向けての議論:
https://gemini.google.com/app/463646ab4e0b9f64

yamlのschema制約:
https://gemini.google.com/app/72b2c95c13ddde7c

yamlのマルチドキュメントストリーム
#
yamlにおいて、
「# yaml-language-server: $schema /path/to/stream.schema.json」形式の検証schema指定は絶対pathに正規化したうえで、$schemaプロパティ指定に解釈される。

以下の様な一般的な命名のディレクティブならば自動で変換するし、併用も可能だが衝突した場合はerrorを返す。
・yaml-language-server
・$schema


jsonschemaでも絶対pathへの解決は実装する。