# Ansible 環境設定ファイル ansible.cfg について

## 適応優先順位(1←高い　低い→4)

1. 環境変数の ANSIBLE_CONFIG で指定された path
2. ./ansible.cfg
3. ~/.ansible.cfg
4. /etc/ansible/ansible.cfg

* ansibleを実行するカレントディレクトリに配置されていればそれを見る
