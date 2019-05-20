# Ghi chép lại các bước cài đặt CEPH phiên bản nautilus

### Mục lục

[1. Mô hình triển khai](#mohinh)<br>
[2. IP Planning](#planning)<br>
[3. Thiết lập ban đầu](#thietlap)<br>
[4. Cài đặt](#caidat)<br>

<a name="mohinh"></a>
## 1. Mô hình triển khai

Mô hình triển khai gồm 3 node CEPH mỗi node có 3 OSDs.

![](../images/install-ceph-nautilus/topo.png)

**OS** : CentOS7 - 64 bit<br>
**Disk**: 04 HDD, trong đó 01sử dụng để cài OS, 03 sử dụng làm OSD (nơi chứa dữ liệu của client) <br>
**NICs**:
	ens160: dùng để ssh và tải gói cài đặt
	ens192: dùng để các trao đổi thông tin giữa các node Ceph, cũng là đường Client kết nối vào
	ens224: dùng để đồng bộ dữ liệu giữa các OSD
**Phiên bản cài đặt**: Ceph nautilus


<a name="planning"></a>
## 2. IP Planning

![](../images/install-ceph-nautilus/Screenshot_1538.png)

<a name="thietlap"></a>
## 3. Thiết lập ban đầu

**Update**

```
yum install epel-release -y
yum update -y
```

**Cấu hình IP**

Thực hiện trên 3 node với IP đã được quy hoạch cho các node ở mục 2.

```
nmcli c modify ens160 ipv4.addresses 10.10.10.152/24
nmcli c modify ens160 ipv4.gateway 10.10.10.1
nmcli c modify ens160 ipv4.dns 8.8.8.8
nmcli c modify ens160 ipv4.method manual
nmcli con mod ens160 connection.autoconnect yes

nmcli c modify ens192 ipv4.addresses 10.10.13.152/24
nmcli c modify ens192 ipv4.method manual
nmcli con mod ens192 connection.autoconnect yes

nmcli c modify ens224 ipv4.addresses 10.10.14.152/24
nmcli c modify ens224 ipv4.method manual
nmcli con mod ens224 connection.autoconnect yes

sudo systemctl disable firewalld
sudo systemctl stop firewalld
sudo systemctl disable NetworkManager
sudo systemctl stop NetworkManager
sudo systemctl enable network
sudo systemctl start network
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

![](../images/install-ceph-nautilus/Screenshot_1539.png)

**Kiểm tra đủ disk trên các node CEPH**

![](../images/install-ceph-nautilus/Screenshot_1540.png)

**Bổ sung file hosts**

Thực hiện trên 3 node CEPH.

```
cat << EOF >> /etc/hosts
10.10.13.152 ceph01
10.10.13.153 ceph02
10.10.13.154 ceph03
EOF
```


**Cài đặt NTPD**

```
yum install chrony -y 
```

```
systemctl start chronyd 
systemctl enable chronyd
systemctl restart chronyd 
```
 - Kiểm tra đồng bộ thời gian
 
```
chronyc sources -v
```

![](../images/install-ceph-nautilus/Screenshot_1541.png)

**Kiểm tra kết nối**

Thực hiện trên cả 3 node CEPH.

```
ping -c 10 ceph01
```

![](../images/install-ceph-nautilus/Screenshot_1542.png)

<a name="caidat"></a>
## 4. Cài đặt CEPH



