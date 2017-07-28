# iSCSI
ghi chép về iSCSI


## 1. Khái niệm 

### SCSI 

SCSI (Small Computer System Interface) là một tập hợp các chuẩn kết nối để truyền dữ liệu giữa các máy tính và các thiết bị với nhau. Chuẩn SCSI định nghĩa các [lệnh](https://en.wikipedia.org/wiki/SCSI_command), giao thức, các tính chất về điện, quang. 

SCSI thường sử dụng cho HDD và đĩa từ, bên cạnh đó nó có thể sử dụng cho một loạt các thiết bị khác như Scanner, CD driver.

Parallel SCSI (SPI) là interface đầu tiên của chuẩn này, sau đó có các interface khác như SSA, Fibre Channel, SAS, ATA, SATA, iSCSI, SRP, ...

### iSCSI

 iSCSI (internet Small Computer System Interface) là giao thức chạy trên nền TCP/IP sử dụng để kết nối Storage bằng đường mạng. Giao thức này cho phép truyền các SCSI command qua đường Network giúp ta có thể truy xuất và truyền dữ liệu trong mạng LAN, WAN hoặc Internet và truy xuất độc lập dữ liệu theo vị trí.
 
 Giao thức này cho phép client (gọi là initiator) gửi SCSI command tới thiết bị lưu trữ (target) qua remote server. Nó là một giao thức SAN, cho phép tổ chức  lưu trữ tập trung vào các mảng lưu trữ khi cung cấp cho client (như database, web server).
 
 Không như Fibre Channel phải sử dụng cáp chuyên biệt, iSCSI có thể chạy trên khoảng cách xa bằng cách sử dụng hạ tầng mạng hiện có.

iSCSI cho phép 2 host trao đổi các SCSI command sử dụng mạng IP. Bằng cách này, iSCSI có một bus local storage với hiệu năng cao trên một phạm vi rộng tạo thành một mạng SAN. 

Mặc dù iSCSI có thể truyền thông giữa các thiết bị SCSI với nhau, nhưng các admin system hầu hết sử dụng để cho phép các các server truy cập disk volume trên storage array. 

iSCSI SAN thường có 1 trong 2 mục đích :

- Storage consolidation : Tài nguyên storage từ các server sẽ được tập trung lại, điều này cho phép cấp phát storage hiệu quả hơn. Trong môi trường SAN, một server có thể cấp phát một disk volume mới mà không có bất cứ thay đổi nào về phần cứng và dây cáp.
- Disaster recovery : Sao lưu storage từ một data center này tới một data center khác. Đặc biệt, iSCSI SAN cho phép disk array migrate qua WAN với sự thay đổi cấu hình tối thiểu, làm cho storage có khả năng định tuyến giống như network traffic.

## Cấu hình iSCSI Target (targetcli)

Một stprage trên network gọi là iSCSI target, một client kết nối tới iSCSI target gọi là iSCSI initiator.

```
+----------------------+          |          +----------------------+
| [   iSCSI Target   ] |10.0.0.8  | 10.0.0.9 | [ iSCSI Initiator  ] |
|     target.loc.vu     +----------+----------+     www.loc.vu    |
|                      |                     |                      |
+----------------------+                     +----------------------+
```

```
# apt-get -y install iscsitarget iscsitarget-dkms
# mkdir /var/iscsi_disks 
# dd if=/dev/zero of=/var/iscsi_disks/disk01.img count=0 bs=1 seek=10G

```

Edit file cấu hình 

```
# vi /etc/default/iscsitarget
```
```
ISCSITARGET_ENABLE=true
``` 

```
# vi /etc/iet/ietd.conf
```
```
Target iqn.2017-07.loc.vu:target00
    # provided devicce as a iSCSI target
    Lun 0 Path=/var/iscsi_disks/disk01.img,Type=fileio
    # iSCSI Initiator's IP address you allow to connect
    initiator-address 10.0.0.9
    # authentication info ( set anyone you like for "username", "password" )
    incominguser admin admin 
```	

Restart lại dịch vụ 

```
# systemctl restart iscsitarget
```

Check status
```
# ietadm --op show --tid=1 
Wthreads=8
Type=0
QueuedCommands=32
NOPInterval=0
NOPTimeout=0
``` 

## Cấu hình iSCSI Initiator (Ubuntu)

```
# apt-get -y install open-iscsi
```

Edit file config 

```
# vi /etc/iscsi/initiatorname.iscsi
```

```
InitiatorName=iqn.2017-07.vu.loc:www.loc.vu
```

```
# vi /etc/iscsi/iscsid.conf
```

Uncomment line 53

```
node.session.auth.authmethod = CHAP
```

Line 57, 58

```
node.session.auth.username = admin
node.session.auth.password = admin
```

Restart service 

```
# systemctl restart iscsid open-iscsi
```

Discover target

```
# iscsiadm -m discovery -t sendtargets -p 10.0.0.8
10.0.0.8:3260,1 iqn.2017-07.vu.loc:target00
```

Confirm status after discovery

```
# iscsiadm -m node -o show 
node.name = iqn.2017-07.vu.loc:target00
node.tpgt = 1
node.startup = manual
node.leading_login = No
...
...
node.conn[0].iscsi.IFMarker = No
node.conn[0].iscsi.OFMarker = No
# END RECORD
```

Login to the target 

```
# iscsiadm -m node --login 
Logging in to [iface: default, target: iqn.2017-07.vu.loc:target00, portal: 10.0.0.8,3260] (multiple)
Login to [iface: default, target: iqn.2017-07.vu.loc:target00, portal: 10.0.0.8,3260] successful.
```

Confirm the established session
```
# iscsiadm -m session -o show 
tcp: [1] 10.0.0.8:3260,1 iqn.2017-07.vu.loc:target00 (non-flash)
````

Confirm the partitions

```
# cat /proc/partitions 
major minor  #blocks  name

  11        0       1152 sr0
   2        0          4 fd0
   8        0   31457280 sda
   8        1   31456239 sda1
   8       16    7340032 sdb
   8       17    7338944 sdb1
   8       32   10485760 sdc
```

Ta có thể thấy một thiết bị mới được thêm vào từ server target (/sdc)

Cấu hình trên Initiator để sử dụng /sdc 

```
# parted --script /dev/sdc "mklabel msdos" 
# parted --script /dev/sdc "mkpart primary 0% 100%" 
# mkfs.ext4 /dev/sdc1 
# mount /dev/sdc1 /mnt 
# df -hT 
/dev/sdc1      ext4      9.8G   23M  9.2G   1% /mnt
```

## Cấu hình iSCSI Initiator (Windows)

Trong file `vi /etc/iet/ietd.conf` của server Targer, sửa lại tham số

```
initiator-address 10.0.0.0/24
```

Mặc định là ALL

Sau đó restart lại dịch vụ : 

```
systemctl restart iscsitarget
```

Trên Windows (Initiator) :

Vào Control Panel, chọn iSCSI Initiator

![iSCSI Initiator](https://github.com/locvx1234/iSCSI/blob/master/images/iSCSI_Initiator.png)

Sau đó chọn Yes khi có hộp thoại xuất hiện

Điền IP Target sau đó chọn Quick Connect...

![Quick Connect](https://github.com/locvx1234/iSCSI/blob/master/images/quick_connect.png)

![]()

Click Connect 

![Connect](https://github.com/locvx1234/iSCSI/blob/master/images/connect.png)

Click Advanced...  

![Advanced](https://github.com/locvx1234/iSCSI/blob/master/images/advance.png)

Check vào ô `Enable CHAP log on` sau đó điền username và password như cấu hình phía Target

![CHAP](https://github.com/locvx1234/iSCSI/blob/master/images/CHAP.png)

**Note** Password phải tối thiểu 12 ký tự 

Chạy `diskmgmt.msc`, một disk đã được attack

![disk](https://github.com/locvx1234/iSCSI/blob/master/images/disk.png)


