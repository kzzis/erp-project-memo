# 製造業ERP 組織マスタ設計 Mermaid図

このファイルの値は説明用のデータ例です。PowerPointと同じid・コードを使用しています。

## 1. 組織階層・会計集計・部門グループ

- 実線：所属組織の階層
- 点線：会計部門または部門グループとの対応
- 班は所属組織として管理し、原価は会計部門4100の組立課で集計します。

```mermaid
flowchart LR
  subgraph ORG["組織マスタ"]
    org1["id 1<br/>0100 製造本部"]
    org2["id 2<br/>0110 東京工場"]
    org3["組織id 3<br/>0120 組立課"]
    org4["id 4<br/>0121 組立1班"]
    org5["id 5<br/>0122 組立2班"]
    org1 --> org2
    org2 --> org3
    org3 --> org4
    org3 --> org5
  end

  subgraph ACC["会計部門マスタ"]
    acc1["会計部門id 1<br/>4100 組立課"]
  end

  subgraph GRP["部門グループマスタ"]
    grp1["グループid 1<br/>地域 E01 東日本"]
    grp2["グループid 2<br/>職能 F01 製造機能"]
  end

  org3 -. 会計集計 .-> acc1
  org4 -. 会計集計 .-> acc1
  org5 -. 会計集計 .-> acc1

  org2 -. 地域所属 .-> grp1
  org3 -. 地域所属 .-> grp1
  org3 -. 職能所属 .-> grp2
  org4 -. 職能所属 .-> grp2
  org5 -. 職能所属 .-> grp2

  classDef organization fill:#DDEEEB,stroke:#147C78,color:#20313D
  classDef accounting fill:#DFE8F1,stroke:#315F8C,color:#20313D
  classDef group fill:#F6E7DA,stroke:#D47A43,color:#20313D
  class org1,org2,org3,org4,org5 organization
  class acc1 accounting
  class grp1,grp2 group
```

## 2. テーブル間の関係

idは意味を持たないサロゲートキーです。部門コード、会計部門コード、グループコードは業務上のナチュラルキーとして扱います。

```mermaid
erDiagram
  ORGANIZATION {
    int id PK
    string department_code
    string department_name
    string organization_type
    int parent_id FK
  }

  ACCOUNTING_UNIT {
    int id PK
    string accounting_code
    string accounting_name
  }

  ORG_ACCOUNTING_ASSIGNMENT {
    int id PK
    int organization_id FK
    int accounting_unit_id FK
    date valid_from
    date valid_to
  }

  DEPARTMENT_GROUP {
    int id PK
    string group_type
    string group_code
    string group_name
  }

  ORG_GROUP_MEMBERSHIP {
    int id PK
    int organization_id FK
    int group_id FK
    date valid_from
    date valid_to
  }

  LEGACY_CODE_MAPPING {
    int id PK
    string code_purpose
    string legacy_code
    string target_type
    int target_id
  }

  ORGANIZATION ||--o{ ORGANIZATION : "parent"
  ORGANIZATION ||--o{ ORG_ACCOUNTING_ASSIGNMENT : "organization_id"
  ACCOUNTING_UNIT ||--o{ ORG_ACCOUNTING_ASSIGNMENT : "accounting_unit_id"
  ORGANIZATION ||--o{ ORG_GROUP_MEMBERSHIP : "organization_id"
  DEPARTMENT_GROUP ||--o{ ORG_GROUP_MEMBERSHIP : "group_id"
  ORGANIZATION ||--o{ LEGACY_CODE_MAPPING : "target organization"
  ACCOUNTING_UNIT ||--o{ LEGACY_CODE_MAPPING : "target accounting unit"
  DEPARTMENT_GROUP ||--o{ LEGACY_CODE_MAPPING : "target group"
```

## 3. 組織変更と履歴

名称・コード変更では組織idを維持します。組織の分割・統合では新しいidを発行し、終了したidを別の組織へ再利用しません。

```mermaid
flowchart LR
  subgraph CHANGE["名称・コード変更"]
    h1["履歴id 1<br/>組織id 3<br/>0120 組立課<br/>2026/04/01 - 2027/03/31"]
    h2["履歴id 2<br/>組織id 3<br/>0130 組立製造課<br/>2027/04/01 -"]
    same["組織id 3を維持"]
    h1 --> same --> h2
  end

  subgraph SPLIT["組織の分割"]
    old["旧 組織id 3<br/>有効期間を終了"]
    new6["新 id 6<br/>組立A課"]
    new7["新 id 7<br/>組立B課"]
    old --> new6
    old --> new7
  end

  classDef keep fill:#DDEEEB,stroke:#147C78,color:#20313D
  classDef close fill:#F3DFDB,stroke:#B65342,color:#20313D
  classDef create fill:#DFE8F1,stroke:#315F8C,color:#20313D
  class h1,h2,same keep
  class old close
  class new6,new7 create
```

## PowerPointとの対応

| Mermaid図 | PowerPoint |
|---|---|
| 図1 全体関係 | 3〜6枚目 |
| 図2 テーブル間の関係 | 3〜6枚目、9枚目 |
| 図3 組織変更と履歴 | 7〜8枚目 |

