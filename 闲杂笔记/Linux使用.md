# XXX 不在sudoers文件中，此事将被报告

su进入root，vim /etc/sudoers，在#Allow root to run any commands anywhere下添加 用户名 ALL=(ALL) NOPASSWD:ALL