# Multi-Node Multi-Drive MinIO S3 storage cluster in Docker Ansible role  

This Ansible role deploys a 4 node MinIO S3 storage cluster in Docker containers.

## Deployment  
### Prepare servers
1. See https://min.io/docs/minio/linux/operations/install-deploy-manage/deploy-minio-multi-node-multi-drive.html#minio-mnmd to get an idea about Multi-Node Multi-Drive MinIO deployment.
2. Make sure you have 4 servers with 4 additional disks for MinIO storage.
3. Format the 4 storage disks in XFS filesystem and label them as `DISK1, DISK2, DISK3, DISK4` according to the above MinIO document.

### TLS certificates
If you are going to use TLS, place fullchain certificate and private key files named after the DNS domain name into the `files/certs/` directory.

Example:
```
- example.org.crt
- example.org.key
```

### Ansbile inventory
Edit `ansible/inventory.ini` and specify your Nexus server addresses, username and ssh key or password, for example:
```ini
[minio]
minio-01   ansible_host=192.168.100.11   ansible_user=ubuntu   ansible_ssh_private_key_file=./ssh_key
minio-02   ansible_host=192.168.100.12   ansible_user=ubuntu   ansible_ssh_private_key_file=./ssh_key
minio-03   ansible_host=192.168.100.13   ansible_user=ubuntu   ansible_ssh_private_key_file=./ssh_key
minio-04   ansible_host=192.168.100.14   ansible_user=ubuntu   ansible_ssh_private_key_file=./ssh_key
```

### Ansible vars
Edit `ansible/roles/minio-cluster/defaults/main.yml` and specify your network addresses, DNS domain name and other parameters, for example:
```yaml
# ==== Network settings
# Docker network address on the docker0 bridge of the target hosts, ex: 172.17.0.0/16
docker_network: "172.17.0.0/16"
# LAN network address of the target hosts, ex: 10.1.1.0/24
lan_network: "10.0.0.0/24"
# DNS domain name
domain: "example.org"

# ==== MinIO
minio_version: "RELEASE.2023-06-29T05-12-28Z"
minio_root_user: "minioadmin"
minio_root_password: "minioadmin"
# Change schema to 'http' if you are not using TLS certificate
minio_cluster_url: "https://minio.{{ domain }}"
```

### Install Docker on MinIO servers
```shell
cd ./ansible
ansible-playbook -i inventory.ini docker-and-compose.yml
```

### Install MinIO
```shell
cd ./ansible
ansible-playbook -i inventory.ini minio-cluster.yml
```
