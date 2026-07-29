---
title: "Using Filters in Ansible"
description: "Ansible filter stuff"
---

## [filters](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_filters.html)

**[regex_search](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/regex_search_filter.html)**
- search a string to extract the part that matches a regular expression.

Extract the database name from a string:
`{{ 'server1/database42' | regex_search('database[0-9]+') }}`

Returns: `database42`