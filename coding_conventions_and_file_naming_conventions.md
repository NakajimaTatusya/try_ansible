# Ansible 規約案

## ファイル命名規則

- YAML ファイルの拡張子は、".yml"で統一する。
- ファイル名に使用できる文字は、小文字の英数字と"_"のみとする。
- Windows 向け PlayBook は、"win_"から開始する。
- CentOS 向け PlayBook は、"cen_"から開始する。
- Ubuntu 向け PlayBook は、"ubu_"から開始する。
- Debian 向け PlayBookは、"deb_"から開始する。
- PlayBook の名称は、目的を英語でつける。

## コーディング規約

- PlayBook(YAML FILE) は"---"から開始し、"..."で終了すること！
- タスクやプレイブック内で宣言した変数は**アンダースコア**から開始する変数名にすること(`_xxx`)
- `register`の変数名は`_result_xxx`とする
- Host変数は、`hv_*****`とする
- Group変数は、`gv_*****`とする
- 変数名、ロール名、playbook名、ファイル名、タスク名の命名規則はスネークケースとする(Ansibleの規約)
- 変数名の開始文字列に **`ansible_`を使用してはならない**(Ansibleの予約変数名)
- jinja2テンプレートはファイル名に`j2`を含めること(区別するため)
- 環境ごとに変数の階層分けする場合は、以下のように命名すること
  - 本番用：`prd`
  - 本番検証用：`prd_dev`
  - システム管理用(ansible、zabbix)：`sys`
  - ステージング環境の場合は、ファイル名に`prd`（本番）、`dev`（開発）
- 変数ファイルはYAMLファイルを用いる(JSONはコメントを記載できないため、不採用)
- 機密情報（パスワード、アクセストークンなど）は情報漏えいを防ぐために**ansible vault**で暗号化する
- タスク名の先頭はそのタスクが属性を表すものとする<br/>
    例）<br/>

```sh
# アプリ関連の場合は
tasks/app_xxx.yml
```

- 実行ログでどのタスクなのかがわかるように`name`を付与する
- whenで同一変数に対して複数のor条件を指定する場合は、`in list`を用いること<br/>

```sh
# NG
- debug:
  msg: "test"
  when: myvar == "test1" or myvar == "test2" or myvar == "test3"

  # OK
- debug:
  msg: "test"
  when: myvar in ["test1", "test2", "test3"]
```

- whenのand条件の場合は、以下のようにリスト形式で記載すること<br/>

```sh
  - debug:
    msg: "test"
    when:
      - myvar == "test1"
      - myvar2 == "test2"
      - myvar3 == "test3"
```

- jinjaテンプレート内のコメントは`{#- hogehoge -#}`を使用すること<br/>

```sh
- set_fact:
  _hoge: >-
  {#- hogeを作成 -#}
```

## 自作モジュール

- モジュール名の接頭語に`hoge_`とすること
- 冪等性を担保するような実装にすること（すでに状態を満たしている場合は、処理をしない）
- 適宜にコメントを残すこと（特にビジネス条件にかかわる部分）
