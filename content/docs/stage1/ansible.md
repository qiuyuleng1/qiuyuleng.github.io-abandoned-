---
title: "Ansible"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

Inventory file
alias(别名)

ansible_host=IP/FQDN(fully qualified domain name, 完整域名)

ansible_connection=ssh/winrm/localhost

ansible_port=22(default for ssh)

ansible_user=root/administrator

ansible_ssh_pass(ansible_password for win)=password



[group_name1]

Groupmember_alias1 or host



[group_name2]

Groupmember_alias2 or host



[group_name3:children] 继承

group_name1

group_name2

YMAL
Key :(colon) value

冒号后面一定要加空格！