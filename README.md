# offline-ClickHouse-Keeper-installation-on-RHEL-8.10-using-Ansible
 Offline Installation of ClickHouse 24.8.2.3 + ClickHouse Keeper on RHEL 8.10 using Ansible


# 🚀 Offline Installation of ClickHouse 24.8.2.3 + ClickHouse Keeper on RHEL 8.10 using Ansible

This guide shows how to install **ClickHouse 24.8.2.3** and optionally **ClickHouse Keeper**  
on **RHEL 8.10 (or Rocky/Alma)** servers **without internet access**, using **Ansible**.

---

## 🧩 Prerequisites

- RHEL / Rocky / AlmaLinux 8.10 servers (offline)
- Ansible control node (can be the same or different machine)
- ClickHouse RPM files downloaded manually on a machine with internet access

---

## 📦 Step 1: Prepare ClickHouse Packages

Download the following RPMs from [https://packages.clickhouse.com/rpm/stable/](https://packages.clickhouse.com/rpm/stable/):

```bash
mkdir -p /data/ansible/clickhouse24.8.2.3
cd /data/ansible/clickhouse24.8.2.3

wget https://packages.clickhouse.com/rpm/stable/clickhouse-common-static-24.8.2.3.x86_64.rpm
wget https://packages.clickhouse.com/rpm/stable/clickhouse-server-24.8.2.3.x86_64.rpm
wget https://packages.clickhouse.com/rpm/stable/clickhouse-client-24.8.2.3.x86_64.rpm
# Optional standalone binary
#wget https://packages.clickhouse.com/rpm/stable/clickhouse-keeper-24.8.2.3.x86_64.rpm

Then transfer this directory to your offline Ansible control node:
/data/ansible/clickhouse24.8.2.3/

⚙️ Step 2: Create the Ansible Inventory
/data/ansible/inventory.ini

[clickhouse]
ch-node1 ansible_host=10.0.0.101
# ch-node2 ansible_host=10.0.0.102
# ch-node3 ansible_host=10.0.0.103

🧰 Step 3: Create the Playbook
/data/ansible/install_clickhouse.yml

---
- name: Offline install ClickHouse + Keeper
  hosts: clickhouse
  become: yes
  vars:
    clickhouse_pkg_dir: /data/ansible/clickhouse24.8.2.3
    clickhouse_pkgs:
      - clickhouse-common-static-24.8.2.3.x86_64.rpm
      - clickhouse-server-24.8.2.3.x86_64.rpm
      - clickhouse-client-24.8.2.3.x86_64.rpm

  tasks:
    - name: Copy ClickHouse RPMs to remote
      copy:
        src: "{{ clickhouse_pkg_dir }}/{{ item }}"
        dest: "/tmp/{{ item }}"
        mode: '0644'
      loop: "{{ clickhouse_pkgs }}"

    - name: Install ClickHouse RPMs
      yum:
        name: "/tmp/{{ item }}"
        state: present
        disable_gpg_check: yes
      loop: "{{ clickhouse_pkgs }}"

    - name: Enable and start ClickHouse service
      systemd:
        name: clickhouse-server
        enabled: yes
        state: started

    - name: Verify ClickHouse installation
      command: clickhouse-client --query "SELECT version();"
      register: ch_ver
      changed_when: false

    - debug:
        msg: "ClickHouse version: {{ ch_ver.stdout }}"


🧱 Step 4: (Optional) Configure ClickHouse Keeper
Create /data/ansible/templates/keeper.xml.j2:

<clickhouse>
  <keeper_server>
    <tcp_port>9181</tcp_port>
    <path>/var/lib/clickhouse/keeper</path>
    <log_storage_path>/var/lib/clickhouse/keeper/logs</log_storage_path>
    <snapshot_storage_path>/var/lib/clickhouse/keeper/snapshots</snapshot_storage_path>
    <server_id>{{ keeper_id | default(1) }}</server_id>
    <raft_configuration>
      {% for host in groups['clickhouse'] %}
      <server>
        <id>{{ loop.index }}</id>
        <hostname>{{ hostvars[host]['ansible_host'] }}</hostname>
        <port>9444</port>
      </server>
      {% endfor %}
    </raft_configuration>
  </keeper_server>
</clickhouse>


Add this task after installation:

    - name: Deploy ClickHouse Keeper config
      template:
        src: templates/keeper.xml.j2
        dest: /etc/clickhouse-server/config.d/keeper.xml
        owner: clickhouse
        group: clickhouse
        mode: '0644'

    - name: Restart ClickHouse to apply Keeper config
      systemd:
        name: clickhouse-server
        state: restarted

▶️ Step 5: Run the Playbook

cd /data/ansible
ansible-playbook -i inventory.ini install_clickhouse.yml

🧪 Step 6: Verify Installation
systemctl status clickhouse-server

Connect to ClickHouse:
clickhouse-client --query "SELECT version();"

غ
If Keeper is enabled:
clickhouse-keeper-client --host localhost --port 9181


🧠 Notes

Works 100% offline (no internet required on target nodes)

Supports single-node or clustered ClickHouse Keeper setup

Compatible with RHEL / Rocky / AlmaLinux 8.x

Uses only native yum and systemd tools

🏁 Example Output

TASK [Verify ClickHouse installation] ******************************************
ok: [ch-node1]
TASK [debug] *******************************************************************
ok: [ch-node1] => {
    "msg": "ClickHouse version: 24.8.2.3"
}


💬 Author

Created by {{ your name }}
Inspired by real-world offline deployment in air-gapped environments.


