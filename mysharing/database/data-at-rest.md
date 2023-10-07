# Giới thiệu
- Một trong các yêu cầu đảm bảo thông tin là dữ liệu lưu trữ phải được mã hoá, bảo vệ để chống khả năng hacker tái sử dụng được các phần dữ liệu lưu trữ phía dưới để đánh cắp dữ liệu. LUKS - Linux Unified Key Setup được Redhat phát triển là 1 trong các giải pháp sẽ giải quyết vấn đề mã hoá dữ liệu mức block. 
- Ưu điểm của giải pháp là trong suốt với lớp dịch vụ, ứng dụng phía trên nghĩa là nó tương thích hoàn toán với các dịch vụ phía trên. Chính vì thế để đảm bảo tính năng Data at rest encryption cho MySQL, Elasticsearch, Mongodb... hoàn toàn độc lập với solution sẽ sử dụng. Ngoài ra việc vận hành, cài đặt LUKS device là tương đối đơn giản và đặc biệt hiệu năng của LUKS và device ban đầu không bị ảnh hưởng nhiều (tôi đã thử benchmark với percona 5.7 dung lượng db 20TB/40TB SSD - raid0, 256GB RAM, 112 CPU - tốc độ đọc ghi trước và sau khi enable LUKS tăng chỉ 2-5% resource CPU và IOPS).
- Nhược điểm: Mã hoá mức block nên việc 1 hacker truy cập được mức OS hoặc dịch vụ vẫn có thể lấy được data trước mã hoá của dịch vụ, db. Tóm lại, chỉ chống được physical attack không giải quyết được logical attack.
# Cấu hình
- Cấu hình lab:
  - Ổ cứng cần mã hoá đặt tại /dev/sdb 
  - Hệ điều hành: Ubuntu 22.04
  - password encryption: plz_change_me_now
  - Thực thi các câu lệnh dưới với quyền root
- Các câu lệnh thực hiện mã hoá:
```sh
wipefs --all --backup /dev/sdb
lsblk
pvcreate /dev/sdb
vgcreate data /dev/sdb
lvcreate -l +100%free -n data01 data
cryptsetup --verbose --verify-passphrase luksFormat  /dev/data/data01
cryptsetup luksOpen /dev/data/data01 data
mkdir -p /data
mkfs.xfs -f /dev/mapper/data
```
- Lệnh này chỉ chạy khi ổ cứng sdb của bạn là SSD hoặc NVMe thôi nhé. Mục đích là không cần cơ chế quản lý queue của OS [here](https://www.cloudbees.com/blog/linux-io-scheduler-tuning)
```sh
echo none > /sys/block/sdb/queue/scheduler
```
- Add vào crypttab để nó tự boot khi server khởi động lại
```sh
echo 'data /dev/mapper/data-data01 /etc/data-at-rest/data-data01 luks' >> /etc/crypttab
```
- Tạo file chứa thông tin secret để mã hoá và loại bỏ thông tin nhạy cảm trong history của bash
```sh
export HISTSIZE=0
mkdir -p /etc/data-at-rest/
mkdir -p /data
touch /etc/data-at-rest/data-data01
echo "plz_change_me_now" > /etc/data-at-rest/data-data01
chmod 400 /etc/data-at-rest/data-data01
chmod 0400 
chmod 0744 /etc/data-at-rest
export HISTSIZE=20000
```
- Add file key cho phép thông tin mount được sử dụng
```shell
cryptsetup -v luksAddKey /dev/data/data01 /etc/data-at-rest/data-data01
```
- Add thông tin mount trong fstab, 2 option noatime và nodiratime để tối ưu performance disk thôi bạn có thể tìm hiểu ý nghĩa 2 options này trên google nhé ;)
```shell
echo '/dev/mapper/data /data xfs defaults,noatime,nodiratime 0 0' >> /etc/fstab
mount /data
```
- kiểm tra kết quả
```shell
lsblk
cryptsetup luksDump /dev/data/data01
```
- *Notes*: Cơ bản ổ cứng /dev/sdb đã được tạo 1 LVM partition ở /dev/data/data01 và được mount ở folder /data. Tiếp theo bạn chỉ cần cài đặt mysql, es, mongodb, redis, blabla... ở folder /data. Folder được mã hoá ổ cứng. Khi ổ cứng máy chủ bị mất cắp hoặc mount vào 1 máy chủ khác nếu kẻ tấn công không có key giải mã, chúng sẽ không thể giải mã được ổ cứng.