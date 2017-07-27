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

## 2.Hoạt động của iSCSI

iSCSI cho phép 2 host trao đổi các SCSI command sử dụng mạng IP. Bằng cách này, iSCSI có một bus local storage với hiệu năng cao trên một phạm vi rộng tạo thành một mạng SAN. 