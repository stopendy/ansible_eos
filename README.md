# ansible_eos_base

## Translations

| Language | Link                         |
| -------- | ---------------------------- |
| English  | [README.md](README.md)       |
| Japanese | [README_ja.md](README_ja.md) |

## Overview

This repository stores Ansible playbooks to configure Arista EOS.

It is created to practice automating Arista EOS initial configuration using Ansible. This repository might be deleted at any time to keep my account clean.

## Files list

| File name                       | Summary                                     |
| ------------------------------- | ------------------------------------------- |
| `hosts`                         | an inventory file                           |
| `eos_base.yml`                  | a playbook to configure EOS devices         |
| `group_vars/*`<br>`host_vars/*` | definition of inventory variables           |
| `templates/*`                   | template files referenced by `eos_base.yml` |

## Testing environment

The playbook and README.md are written based on following environment.

| Software                | Version |
| ----------------------- | ------- |
| `ansible-core`          | 2.12.5  |
| `arista.eos` collection | 5.0.0   |

## Side notes for eos_base.yml

### changed_when

`arista.eos.eos_acls` returns `OK` even if there are some changes in check mode. `changed_when` line is added to make it return `changed` if there is any in check mode.

```yml
    - name: create management ssh acls
      arista.eos.eos_acls:
      #   (omit)
      changed_when: _result.commands != []
```

### lines

`arista.eos.eos_config` usually involves `src` when a Jinja2 template file is used as a configuration source. However `lines` are called in this playbook because `match` option requires `lines` option to be called together.

Template plugin is called to use Jinja2 template file in conjunction with `lines`.

```yml
    - name: modify others (non-resource-module)
      arista.eos.eos_config:
        lines: '{{ lookup("template", "eos_base_config.j2") | split("\n") }}'
        match: strict
```

### match: strict

`arista.eos.eos_config` uses **partial match** by default to match input configurations against the existing configuration in the EOS device. For example, let's say the EOS device has a configuration below, and input configuration is `username adm`. Although it's syntactically erroneous because of absence of `secret`, Ansible returns `OK` when the playbook is run in check mode. This is a feature of default `match: line` option.

```
username admin secret sha512 $6$...
```

If `match: strict` is specified, Ansible searches EOS devices' configuration line by line in a full search manner; so Ansible won't hide errors in check mode any more.
