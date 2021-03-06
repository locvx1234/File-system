# Tìm hiểu về file system

## 1. Khái niệm
File system là hệ thống tổ chức lưu trữ các tập tin và siêu dữ liệu trên ổ đĩa.

## 2. Journaling 
Cơ chế Journaling cho phép ghi lại nhật ký khi ghi dữ liệu, giúp hệ điều hành phát hiện được lỗi trong quá trình ghi dữ liệu khi gặp sự cố.

<img src="http://i.imgur.com/PfWI3O9.jpg">

Đầu tiên, file được ghi vào Journal, sau đó Journal ghi file đó vào ổ cứng.
Nếu quá trình ghi file thành công thì dữ liệu ở journal bị xóa bỏ và hoàn tất.
Còn nếu xảy ra lỗi, file system kiểm tra và ghi lại lỗi ở journal.

**Ưu điểm : ** Phát hiện lỗi trong khi gặp sự cố.

**Nhược điểm : ** Giảm hiệu suất hệ điều hành.

## 3. Phân loại
Linux hỗ trợ nhiều loại file system 

| File system | Max File Size | Max Par Size | Journaling | Notes |
|-------------|---------------|--------------|------------|-------|
| ext2 | 2TiB | 32 TiB | Không | Kế thừa hệ thống file cũ, phù hợp với các thiết bị lưu trữ ngoài như thể nhớ , USB... |
| ext3 | 2TiB | 32 TiB | Có | Phát triển của ext2, là chuẩn file system của Linux  trong nhiều năm |
| ext4 | 16TiB | 1 EiB | Có | Giảm hiện tượng phân mảnh, hỗ trợ các file dung lượng lớn |
| xfs | 8 EiB | 8 EiB | Có | Tương đồng với ext4, phù hợp mô hình server media |
| jfs | 4 PiB | 32 PiB | Có | Không duy trì tốt |
| reiserFS | 8 TiB | 16 TiB | Có | Phù hợp với các file nhỏ, database, server email | 

Bên cạnh đó còn rất nhiều loại file system khác : isofs (iso9660), sysfs, procfs, nfs, ...

Linux cũng hỗ trợ Microsoft NTFS, vfat và một số file system khác. 

## 4. Cấu trúc phân cấp file system 

<img src="http://www.gocit.vn/wp-content/uploads/2015/09/linux-file-system.jpg">

Đứng đầu là / (root) và đứng sau là các thư mục con của nó.

- /bin - Chứa chương trình thực thi (binary)
- /boot - Chứa các file phục vụ quá trình boot 
- /dev - Chứa các file thiết bị
- /etc - Chứa các file cấu hình của các chương trình
- /home - Chứa file cá nhân của người dùng
- /lib - Những thư viện cần thiết và các module của nhân
- /media - Mount point cho các thiết bị media
- /mnt - Mount point cho file-system được mount tạm thời
- /opt - Các gói add-on của phần mềm ứng dụng
- /sbin - Chứa chương trình thực thi của admin, bảo trì hệ thống
- /tmp - Chứa các file tạm thời được tạo bởi hệ thống và người dùng.
- /proc - Thông tin về các tiến trình 
- /var - Các biến của hệ thống được lưu trong thư mục này
- /usr - Chứa các thư viện, file thực thi, tài liệu hướng dẫn và mã nguồn cho chương trình chạy ở level 2

## 5. Các lệnh làm việc với file
### du
Lệnh `du` để xem dung lượng file hoặc thư mục.

Ví dụ liệt kê kích thước các file trong thư mục `/var` :
	
	# du -ah /var 
	
Các tùy chọn:
- -a (all): tất cả các file
- -h : hiển thị đơn vị kích thước
- -c: hiển thị total

Để sắp xếp các file theo thứ tự kích thước giảm dần

	# du /var -a | sort -nr | head -10
	
<img src="http://i.imgur.com/I9dTByN.png">

### df 
Lệnh `df` dùng để xem tình trạng các phân vùng và các thiết bị được mount

	# df -h
	
Các tùy chọn:
- -h : Hiểm thị thêm kích thước các thư mục được mount
- -T : Loại file-system
- -a : Tất cả các file

<img src="http://i.imgur.com/CDQvr1d.png">

## 6. fstab 

fstab (file system table) tại đường dẫn `/etc/fstab` là một file text lưu thông tin về các thiết bị, mount point và các thiết lập của nó.

Khi khởi động, hệ thống sẽ đọc file này để tự động mount các thiết bị được chỉ ra trong file. 

Ví dụ về file `fstab` :

```
#
# /etc/fstab
# Created by anaconda on Fri Nov 17 17:06:18 2017
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=6a73e21d-0e4e-4716-bbd4-4b1bcd07f125 /boot                   xfs     defaults        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0

```

Ý nghĩa các trường : 

Cột 1 : UUID hoặc đường dẫn thiết bị. Ví dụ : `/dev/mapper/centos-root` hoặc `UUID=6a73e21d-0e4e-4716-bbd4-4b1bcd07f125`
 
Cột 2 : mount point. Ví dụ `/` , `/boot`

Cột 3 : định dạng file system. Ví dụ `ext2`, `ext3`, `xfs` 

Cột 4 : các tùy chọn, nếu có nhiều thì được phân cách bởi dấu phẩy 

- *auto/noauto* : phân vùng có được tự động mount thiết bị khi máy tính khởi động hay không 
- *exec/noexec* : phân vùng có thể thực thi các chương trình nhị phân trên đó hay không 
- *ro/rw* : `ro` là read-only, `rw` là read-write. Nếu bạn là user thường, bạn cần đặt `rw` nếu muốn ghi dữ liệu trên phân vùng đó 
- *sync/async* : các thao tác nhập xuất được thực hiện đồng bộ / không đồng bộ 
- *user/nouser* : người dùng được quyền mount và unmount. `user` bao hàm cả `noexec` nên nếu bạn muốn thực thi file nhị phân, cần phải có thêm `exec` 
- *default* : bao gồm `rw`, `suid`, `dev`, `exec`, `auto`, `nouser`, async`

Cột 5 : thiết lập backup khi hệ thống down. 1 là true, 0 là false 

Cột 6 : thiết lập kiểm tra lỗi bằng fsck. 1 là true, 0 là false 

## Tham khảo 
---
- http://quantrimang.com/tim-hieu-khai-niem-co-ban-ve-he-thong-file-trong-linux-84900
- https://help.ubuntu.com/community/LinuxFilesystemsExplained
- https://wiki.archlinux.org/index.php/file_systems
- https://www.howtogeek.com/howto/38125/htg-explains-what-is-the-linux-fstab-and-how-does-it-work/