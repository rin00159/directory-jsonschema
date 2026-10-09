位置付け:出力形式はdirectoryの要素一覧や、directoryへのバリデーションルールを示すschema
用途:ディレクトリの説明ファイルとして利用する場合は DIRDESC.yamlファイル を利用する。

背景:
一応`tree -J`を土台にしているがかなり追加されているし、tree出力の検証用ではない。
対象の type は file / directory / link のみで、socket / fifo / block / char は対象外。
# 意味定義
プロパティが他の場所での定義と相違する場合は「より実際のfileに近い場所の定義記述」を基本的に優先する(/dir /dir/dir2 が /dir/dir2/file に対して矛盾するschemaを記す場合は /dir/dir2 を正本とするのが基本動作)
    //ただし意味定義で一番大事な部分ではない為ツールがサポートしなくても良い