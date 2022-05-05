# ansible_eos_base

## 翻訳

| 言語   | リンク                       |
| ------ | ---------------------------- |
| 英語   | [README.md](README.md)       |
| 日本語 | [README_ja.md](README_ja.md) |

## 概要

Arista EOSの設定を行うAnsible playbookを格納します。

本リポジトリは、Arista EOSの初期コンフィグ投入の練習用です。  
用が済んだら、整理のためにリポジトリを削除するかもしれません。

## 動作環境

本Playbookは、以下の環境で挙動を確認しました。

| ソフトウェア            | バージョン |
| ----------------------- | ---------- |
| `ansible-core`          | 2.12.5     |
| `arista.eos` collection | 5.0.0      |

## ファイル構成

| ファイル名                      | 概要                                       |
| ------------------------------- | ------------------------------------------ |
| `hosts`                         | インベントリ                               |
| `eos_base.yml`                  | eosの設定変更を行うPlaybook               |
| `group_vars/*`<br>`host_vars/*` | インベントリ変数の定義                     |
| `templates/*`                   | `eos_base.yml`が使用するJinja2テンプレート |

## eos_base.ymlの補足

### changed_when

`arista.eos.eos_acls`は、Check modeで実行した場合に変更箇所を検知した場合も`OK`を返します。  
変更箇所がある場合は`Changed`を返してほしいので、`changed_when`を追記しました。

```yml
    - name: create management ssh acls
      arista.eos.eos_acls:
      #   (略)
      changed_when: _result.commands != []
```

### lines

`arista.eos.eos_config`は、Jinja2でコンフィグを投入する場合は`src`を使うのが一般的です。  
しかし、今回は敢えて`lines`を使いました。  
その理由は、`match`オプションを使うには`lines`オプションの指定が必要だったためです。

`lines`とJinja2を併用するために、template pluginを使用しました。

```yml
    - name: modify others (non-resource-module)
      arista.eos.eos_config:
        lines: '{{ lookup("template", "eos_base_config.j2") | split("\n") }}'
        match: strict
```

### match: strict

`arista.eos.eos_config`は、デフォルトでは投入予定のコンフィグと実機のコンフィグを行単位で**部分一致**検索を行います。  
例えば実機コンフィグが以下の場合、投入コンフィグが`username adm`のように不完全な行だった場合も、Check mode実行した時に`OK`と判定されてしまいます。  
当然、実際に実機投入したらsyntax errorになります。  
これがデフォルトの`match: line`の挙動です。

```
username admin secret sha512 $6$...
```

`match: strict`を指定すれば行単位で**完全一致**検索してくれるので、上記のような不具合はなくなります。
