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
