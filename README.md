Mino Seslax Format 0.1
===============================================================================

シェルトに関する諸々を機械で保存したり送受信する際に便利なデータ形式について考えたものをとりあえずまとめた文書です。主に棋譜形式に関する文書ですが、局面と指し手の形式についても扱っています。

以下シェルトに関する諸々を JSON（RFC 8259）で表現する方法について定義します。型の表記は TypeScript に準じています。

日時の表現
-------------------------------------------------------------------------------

型名         | 定義
-------------|------
`DateString` | RFC 3339

手番の表現
-------------------------------------------------------------------------------

型名     | 定義
---------|------
`Player` | `"arxe" \| "sorn"`

駒の表現
-------------------------------------------------------------------------------

同名の使徒が存在する駒はその三文字の略称で、テームスは `"tem"` として表現します。

型名             | 定義
-----------------|------
`Piece`          | 任意の駒名
`Piece.Lantis`   | テームスを除く駒名
`Piece.Arxe`     | アルシェの駒名
`Piece.Sorn`     | ソーンの駒名
`Piece.Turnable` | 方向転換可能な駒名（前衛とテームス）

局面の表現（`State` 型）
-------------------------------------------------------------------------------

次の JSON は、開始局面を表した例です。

```json
{
    "sast": "arxe",
    "arxe": {
        "txifol": [],
        "evol": null
    },
    "sorn": {
        "txifol": [],
        "evol": null
    },
    "ele": [
        ["rav", "nen", "pin", "mir", "ket", "lin", "len"],
        ["din", "rez", "kun", "mat", "lax", "jil", "tan"],
        [ null,  null,  null,  null,  null,  null,  null],
        [ null,  null,  null, "tem",  null,  null,  null],
        [ null,  null,  null,  null,  null,  null,  null],
        ["pal", "ful", "mik", "fav", "zan", "gil", "ruj"],
        ["dyu", "lis", "mel", "ser", "dia", "vio", "ral"]
    ],
    "korol": []
}
```

### `State` 型

プロパティ | 型                           | 意味
-----------|------------------------------|------
`sast`     | `Player`                     | 手番（次に指すプレイヤー）
`arxe`     | `CapturedPieces<Piece.Sorn>` | アルシェの持ち駒と張っている駒
`sorn`     | `CapturedPieces<Piece.Arxe>` | ソーンの持ち駒と張っている駒
`ele`      | `Board`                      | 盤面
`korol`    | `Piece.Turnable[]`           | 方向転換されている駒のリスト（重複不可）

加えて以下の制約を満たす必要があります。

- `arxe`, `sorn`, `ele` 以下の駒名全体で重複がない
- `korol` 以下の駒名はすべて `ele` 以下にも存在する

### `CapturedPieces<T>` 型

プロパティ | 型          | 意味
-----------|-------------|------
`txifol`   | `T[]`       | 持ち駒のリスト（張っている駒を除く、重複不可）
`evol`     | `T \| null` | 張っている駒

### `Board` 型

型名    | 定義
--------|------
`Board` | `Piece \| null` を値に取る 7x7 配列（`Piece` は重複不可）

指し手の表現（`Move` 型）
-------------------------------------------------------------------------------

指し手の種類は `pit` プロパティの値で判別できます。

### `Move` 型

型名   | 定義
-------|------
`Move` | `Move.Leim \| Move.Okke \| Move.Kor \| Move.Ev \| Move.Sed`

#### `Move.Leim` 型（通常の指し手）

プロパティ | 型         | 意味
-----------|------------|------
`pit`      | `"leim"`   |
`luul`     | `[FromTo]` | 駒の動き

#### `Move.Okke` 型（前衛の随伴移動）

プロパティ | 型                 | 意味
-----------|--------------------|------
`pit`      | `"okke"`           |
`luul`     | `[FromTo, FromTo]` | 各駒の動き（順不同）

#### `Move.Kor` 型（前衛の方向転換）

プロパティ | 型         | 意味
-----------|------------|------
`pit`      | `"kor"`    |
`ka`       | `Position` | 方向転換する駒の座標

#### `Move.Ev` 型（テームスへの張り）

プロパティ | 型             | 意味
-----------|----------------|------
`pit`      | `"ev"`         |
`evol`     | `Piece.Lantis` | 張る駒

#### `Move.Sed` 型（テームスからの戻し）

プロパティ | 型
-----------|----
`pit`      | `"sed"`

### `FromTo` 型

プロパティ | 型         | 意味
-----------|------------|------
`i`        | `Position` | 移動元の座標
`a`        | `Position` | 移動先の座標

### `Position` 型

プロパティ | 型                                | 意味
-----------|-----------------------------------|------
`alsia`    | `0 \| 1 \| 2 \| 3 \| 4 \| 5 \| 6` | 左端を 0 とした横方向の位置
`soom`     | `0 \| 1 \| 2 \| 3 \| 4 \| 5 \| 6` | velm を 0 とした縦方向の位置

棋譜の表現
-------------------------------------------------------------------------------

この棋譜をファイルとして保存する場合、拡張子には `.msf` をつかいます。サンプルは [examples](examples) にあります。

プロパティ | 型              | 意味
-----------|-----------------|------
`aptex`    | `GameTags`      | 棋譜のメタデータ
`kit`      | `RootNode`      | 棋譜の初めに関する情報
`fixt`     | `Result`        | 対局結果
`seslax`   | `NonrootNode[]` | 各局面に関する情報

### `GameTags` 型

`string` を値とするオブジェクトです。何らかの情報を棋譜に持たせるために使います。

統一性があったほうがよさそうな項目については以下に幾つかまとめてみました。

プロパティ  | 意味
------------|------
`est?`      | 棋譜の題名
`arxe?`     | 対局者名（アルシェ）
`sorn?`     | 対局者名（ソーン）
`ladan?`    | 作局者名
`kalte?`    | 対局場所
`evita?`    | 大会、棋戦名

### `Result` 型

型名     | 定義
---------|------
`Result` | `Result.XeltsoldesOrTeomsast \| Result.Artansoldes \| Result.Daim`

#### `Result.XeltsoldesOrTeomsast` 型（月駒が取られた，千日手）

プロパティ | 型                       | 意味
-----------|--------------------------|------
`pit`      | `"xeltsoldes" \| "teomsast"` |
`vastan`   | `Player`                 | 勝者
`im?`      | `DateString`             | 結果が決まった日時
`oprens?`  | `string`                 | コメント

#### `Result.Artansoldes` 型（すべての魔駒が取られた）

プロパティ | 型                      | 意味
-----------|-------------------------|------
`pit`      | `"artansoldes"`         |
`vastan`   | `Player`                | 勝者
`ito`      | `0 \| 1 \| 2 \| 3 \| 4` | 勝ち点
`im?`      | `DateString`            | 結果が決まった日時
`oprens?`  | `string`                | コメント

#### `Result.Daim` 型（上記以外）

プロパティ | 型           | 意味
-----------|--------------|------
`pit`      | `"daim"`     |
`im?`      | `DateString` | 結果が決まった日時
`oprens?`  | `string`     | コメント

### `RootNode` 型

プロパティ | 型                             | 意味
-----------|--------------------------------|------
`fala`     | `0 \| 1 \| 2 \| ... \| 2147483647` | ここまでに指された手の数（継続対局でなければ `0`）
`slax`     | `State`                        | 開始局面
`mit`      | `Variation[]`                  | 変化手順（次の手以降）
`im?`      | `DateString`                   | 対局開始日時
`oprens?`  | `string`                       | コメント

### `NonrootNode` 型

プロパティ | 型            | 意味
-----------|---------------|------
`ov`       | `Move`        | 指し手
`slax`     | `State`       | 手が指された直後の局面
`mit`      | `Variation[]` | 変化手順（次の手以降）
`im?`      | `DateString`  | 手が指された日時
`oprens?`  | `string`      | コメント

### `Variation` 型

プロパティ | 型              | 意味
-----------|-----------------|------
`fixt`     | `Result`        | 変化手順の結末（特に無い場合は `Result.Daim` にする）
`seslax`   | `NonrootNode[]` | 棋譜

参考リンク
-------------------------------------------------------------------------------

- [CSA標準棋譜ファイル形式](http://www2.computer-shogi.org/protocol/record_v22.html)
- [na2hiro/json-kifu-format](https://github.com/na2hiro/json-kifu-format)
- [Portable Game Notation - Wikipedia](https://ja.wikipedia.org/wiki/Portable_Game_Notation)
