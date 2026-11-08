# 製造業ERP 組織マスタ設計 Mermaid図（非エンジニア向け）

対象読者は人事・経理・経営管理・情報管理の担当者です。図中の値は説明用の実データ例です。

> 読み方：**組織ID**はシステム内部で1から振る意味を持たない番号、**部門コード**は業務で利用する0100・0120などのコードです。

## 1. 所属と原価集計

**この図で伝えたいこと：** 班まで所属を管理しても、会計上の原価は組立課単位にまとめられます。

```mermaid
flowchart LR
  subgraph s1["所属組織：人がどこに所属するか"]
    o1["0100<br/>製造本部"] --> o2["0110<br/>東京工場"]
    o2 --> o3["0120<br/>組立課"]
    o3 --> o4["0121<br/>組立1班"]
    o3 --> o5["0122<br/>組立2班"]
  end

  a1["会計部門 4100<br/>組立課<br/><br/>原価をまとめる単位"]

  o3 -.->|原価の集計先| a1
  o4 -.->|班別にせず同じ集計先へ| a1
  o5 -.->|班別にせず同じ集計先へ| a1

  n1["班は所属管理に使う<br/>班単位では会計原価を集計しない"]
  o4 --- n1

  classDef organization fill:#DDEEEB,stroke:#147C78,color:#20313D,stroke-width:2px
  classDef accounting fill:#DFE8F1,stroke:#315F8C,color:#20313D,stroke-width:2px
  classDef note fill:#FFF7E8,stroke:#D89B35,color:#20313D
  class o1,o2,o3,o4,o5 organization
  class a1 accounting
  class n1 note
```

## 2. 3つのマスタとテーブル間の関係

**この図で伝えたいこと：** 組織を中心に、会計の集計先と経営管理の分類を別テーブルでつなぐことで、用途ごとの変更を切り離せます。

「組織・会計部門対応」は**複数の組織を同じ原価集計先にまとめる**ための表です。「組織・グループ対応」は**一つの組織を複数のグループに登録できる**ようにするための表です。

```mermaid
flowchart LR
  t1["<b>組織マスタ</b><br/>──────────<br/>組織ID<br/>部門コード<br/>組織名<br/>上位組織ID<br/>──────────<br/>例：3｜0120｜組立課｜2<br/>例：4｜0121｜組立1班｜3<br/>例：5｜0122｜組立2班｜3"]

  t2["<b>組織・会計部門対応</b><br/>──────────<br/>組織ID<br/>会計部門ID<br/>──────────<br/>3 → 1<br/>4 → 1<br/>5 → 1"]

  t3["<b>会計部門マスタ</b><br/>──────────<br/>会計部門ID<br/>会計部門コード<br/>会計部門名<br/>──────────<br/>例：1｜4100｜組立課"]

  t4["<b>組織・グループ対応</b><br/>──────────<br/>組織ID<br/>グループID<br/>──────────<br/>3 → 1<br/>3 → 2<br/>4 → 2<br/>5 → 2"]

  t5["<b>部門グループマスタ</b><br/>──────────<br/>グループID<br/>グループコード<br/>グループ名<br/>──────────<br/>例：1｜E01｜東日本<br/>例：2｜F01｜製造機能"]

  t1 -->|組織IDでつなぐ| t2
  t2 -->|複数の組織を同じ原価集計先にまとめる| t3
  t1 -->|組織IDでつなぐ| t4
  t4 -->|一つの組織を複数のグループに登録できる| t5

  n2["会計の見方<br/>原価は組立課 4100に集計"]
  n3["経営管理の見方<br/>地域別と職能別を同時に利用"]
  t3 --- n2
  t5 --- n3

  classDef organization fill:#DDEEEB,stroke:#147C78,color:#20313D,stroke-width:2px
  classDef accounting fill:#DFE8F1,stroke:#315F8C,color:#20313D,stroke-width:2px
  classDef group fill:#F6E7DA,stroke:#D47A43,color:#20313D,stroke-width:2px
  classDef bridge fill:#F3F5F7,stroke:#7A8793,color:#20313D,stroke-width:2px
  classDef note fill:#FFF7E8,stroke:#D89B35,color:#20313D
  class t1 organization
  class t2,t4 bridge
  class t3 accounting
  class t5 group
  class n2,n3 note
```

### 図2の実データの読み方

| 見たい内容 | 実データから分かること |
|---|---|
| 組立1班・2班の原価集計先 | 組織ID 4と5は、どちらも会計部門ID 1（4100 組立課）につながる |
| 組立課の経営管理上の分類 | 組織ID 3は、東日本（E01）と製造機能（F01）の両方につながる |
| 多対多にする理由 | 一つの組織に複数分類を付けられ、同じ分類に複数組織を登録できる |

## 3. 組織変更時に変わるもの

**この図で伝えたいこと：** コードや名称の変更では同じ組織IDを使い続け、組織を分割するときだけ新しい組織IDを発行します。

```mermaid
flowchart LR
  subgraph c1["コード・名称だけを変更する場合"]
    b1["変更前<br/>組織ID 3<br/>部門コード 0120<br/>組立課"]
    keep["同じ組織として扱う<br/>組織ID 3は変えない"]
    b2["変更後<br/>組織ID 3<br/>部門コード 0130<br/>組立製造課"]
    b1 --> keep --> b2
  end

  subgraph c2["組織を分割する場合"]
    old["旧組織<br/>組織ID 3<br/>利用終了日を記録"]
    new1["新しい組織<br/>組織ID 6<br/>組立A課"]
    new2["新しい組織<br/>組織ID 7<br/>組立B課"]
    old -->|新しい組織として作成| new1
    old -->|新しい組織として作成| new2
  end

  classDef keep fill:#DDEEEB,stroke:#147C78,color:#20313D,stroke-width:2px
  classDef close fill:#F3DFDB,stroke:#B65342,color:#20313D,stroke-width:2px
  classDef create fill:#DFE8F1,stroke:#315F8C,color:#20313D,stroke-width:2px
  class b1,keep,b2 keep
  class old close
  class new1,new2 create
```

## PowerPointとの対応

| Mermaid図 | PowerPointの主な対応ページ |
|---|---|
| 図1 所属と原価集計 | 3〜6枚目 |
| 図2 3つのマスタとテーブル間の関係 | 3〜6枚目、9枚目 |
| 図3 組織変更時に変わるもの | 7〜8枚目 |

