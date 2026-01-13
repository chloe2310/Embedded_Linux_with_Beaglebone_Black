
BUILD IMAGE OS CHO BBB (THỦ CÔNG)

1) Toolchain :
•	Toolchain = bộ công cụ biên dịch (compiler tool chain).
•	Nó gồm nhiều chương trình nhỏ, làm việc liên tiếp nhau để chuyển code C/C++ thành chương trình chạy được trên một kiến trúc CPU cụ thể.
Thành phần chính:
•	Compiler (gcc, g++) → dịch code C/C++ thành mã máy trung gian (object file .o).
•	Assembler (as) → dịch từ mã assembly thành mã máy.
•	Linker (ld) → liên kết các object file + thư viện thành file nhị phân cuối cùng (ELF hoặc executable).
•	Binutils (objcopy, objdump, nm, strip, …) → công cụ xử lý file nhị phân.
•	C library (glibc, musl, uClibc, …) → cung cấp API chuẩn C (printf, malloc, open, …) cho chương trình user-space.

Cross-toolchain và tại sao cần : 
•	Native toolchain: biên dịch code để chạy ngay trên máy hiện tại (VD: gcc trên Ubuntu x86_64 → output chạy trên Ubuntu x86_64).
•	Cross toolchain: biên dịch code trên một máy (host, x86_64 PC) nhưng output chạy trên một kiến trúc khác (target, ví dụ ARM Cortex-A8 của BBB).
=>  Lý do phải dùng cross-toolchain:
1.	BBB (ARM) yếu, không tiện để tự build kernel/U-Boot → ta build trên PC mạnh, rồi copy sang BBB.
2.	Kiến trúc CPU khác nhau (x86_64 vs ARMv7) nên gcc của PC không dịch ra binary chạy trên BBB được. Ta cần gcc được build riêng cho ARM (arm-linux-gnueabihf-gcc).

Tải toolchain Linaro : wget -c https://releases.linaro.org/components/toolchain/binaries/6.5-2018.12/arm-linux-gnueabihf/gcc-linaro-6.5.0-2018.12-x86_64_arm-linux-gnueabihf.tar.xz
Lưu ý: Bản này khá cũ (GCC 6.5, 2018). Vẫn build được U-Boot/Kernel cũ, nhưng với kernel/U-Boot mới có thể cần GCC mới hơn (10–12). Nếu gặp lỗi về compiler quá cũ, mình sẽ đổi toolchain sau.

<img width="624" height="256" alt="image" src="https://github.com/user-attachments/assets/526433e6-72d7-402a-9c42-32e088329544" />

Sau đó , dùng lệnh “ ls “ , ta sẽ thấy file nén : 
<img width="624" height="64" alt="image" src="https://github.com/user-attachments/assets/4542a083-a684-48d5-bc4a-ee5dbc093cb6" />

Giải nén bằng câu lệnh : tar xf gcc-linaro-6.5.0-2018.12-x86_64_arm-linux-gnueabihf.tar.xz

ls tiếp ta sẽ thấy file được giải nén : 
<img width="624" height="76" alt="image" src="https://github.com/user-attachments/assets/3ab7a049-1540-4f53-a3f6-96e83a3d4e4e" />

Sau đấy xóa cái file nén cũ đi cho đỡ nặng máy nhé (lệnh cho đỡ lười này ) : rm gcc-linaro-6.5.0-2018.12-x86_64_arm-linux-gnueabihf.tar.xz ( thêm -rf cũng được ) .

Sau đó bắt buộc : export CC=`pwd`/gcc-linaro-6.5.0-2018.12-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf- rồi check version : ${CC}gcc --version 

2 ,Bootloader  
Đầu tiên ,  Clone Uboot từ mainline bằng lệnh : 
git clone -b v2019.04 https://github.com/u-boot/u-boot --depth=1
<img width="624" height="370" alt="image" src="https://github.com/user-attachments/assets/e0b421fa-4f8d-4ae0-bd62-2b8beb62840f" />

Kết quả: mình sẽ có source U-Boot gốc (mainline) tại thư mục u-boot/, đúng version 2019.04.
Sau đấy nhớ cd vào thư mực u-boot để thao thác nha ( lệnh này tao không viết đâu )

Rồi giờ kéo thêm patch của BeagleBoard nữa : 
git pull --no-edit https://github.com/beagleboard/u-boot v2019.04-bbb.io-am335x

Kết quả : 
<img width="624" height="311" alt="image" src="https://github.com/user-attachments/assets/065428af-cf87-455a-9eb1-f51bd787e92e" />

Nghĩa là:
•	Ta đang lấy U-Boot gốc (DENX) version 2019.04.
•	Sau đó kéo thêm patch từ BeagleBoard để biến nó thành bản U-Boot phù hợp với BBB (AM335x).

Sao phải làm thế cho mất công : 
•  U-Boot mainline (DENX) hỗ trợ BBB, nhưng thường thiếu một số patch đặc thù (DDR timings, device overlays, eMMC boot).
•  BeagleBoard.org duy trì branch riêng (bbb.io-am335x) có đầy đủ fix để chạy ổn trên phần cứng thực tế.

Nên ta vừa giữ nền tảng gốc (mainline), vừa có patch từ vendor/community

Sau đấy , ta tiến hành build uboot bằng các lệnh sau đây :
make ARCH=arm CROSS_COMPILE=${CC} distclean (làm sạch cây mã nguồn)
make ARCH=arm CROSS_COMPILE=${CC} am335x_evm_defconfig ( chọn cấu hình cho AM335 )
make ARCH=arm CROSS_COMPILE=${CC} (build SPL + Uboot ) 

Lưu ý hộ tao : Nếu không thấy MLO: thường do chưa bật build SPL hoặc version/nhánh khác. Thử:
make mrproper
make ARCH=arm CROSS_COMPILE=${CC} am335x_evm_defconfig
make ARCH=arm CROSS_COMPILE=${CC} -j$(nproc)
và kiểm tra log có đoạn “Building spl / Creating MLO”.


3,Linux Kernel ( cái này build rất là lâu đấy , nên nhớ để máy ảo nhiều core vào)

Dùng các lệnh : 
git clone https://github.com/RobertCNelson/bb-kernel
cd bb-kernel/
git checkout origin/am33x-v5.4 -b tmp

Sau đấy ls ở thằng bb_kernel và đọc file readme.md nhé , trong đấy nó sẽ nói các lần build như nào đấy , nhưng với lần build đầu tiền thì dùng lệnh bên dưới
nhé : 
Build kernel lần đầu tiên : ./ build_kernel.sh
Lúc đang build nó ra cái màn hình như dưới : 

<img width="624" height="332" alt="image" src="https://github.com/user-attachments/assets/2de6e1dc-baed-434d-91d6-8980f5a434e9" />

Dùng mũi tên trái phải ở bàn phím ấn Save nhé , Có mấy cái bảng nhỏ cứ nhấn OK hết rồi Exit ra ngoài cho nó build tiếp nha.
Xong rồi thì cd ra ngoài để ta tiếp tục build thằng Rootfs nhé 

4, Root file system (Rootfs) :
Tải và giải nén rootfs 
Tải rootfs dạng tarball : 
wget -c https://rcn-ee.com/rootfs/eewiki/minfs/debian-10.10-minimal-armhf-2021-06-20.tar.xz

Giải nén : 
tar xvf debian-10.10-minimal-armhf-2021-06-20.tar.xz
Sau đấy nhớ xóa file nén đi nhé (file tar.xz ấy ), cho đỡ nặng máy .

Và ta đã đủ các thành phần rồi đó : 

<img width="624" height="33" alt="image" src="https://github.com/user-attachments/assets/f7e6409a-91ab-44c9-854d-d67aed6a9bc7" />

Rồi giờ tiến hành flash tất cả file xuống thẻ nhớ sd này.


BƯỚC CUỐI : FLASH XUỐNG THẺ SD 

Giờ ta cần chuẩn bị 1 cái thẻ sd , 1 cái USB đầu lọc thẻ nhé , như của mình đây  : 
<img width="355" height="302" alt="image" src="https://github.com/user-attachments/assets/fa68baa7-eb13-48e5-b5b3-6ba935ed3936" />

Sau khi cắm thẻ vào máy tính , nhớ kết nối thẻ vào máy ảo nhé : 
Dùng lệnh lsblk để kiểm tra nhé 

Sau đấy dùng cách lệnh sau đây : 
export DISK=/dev/sdb
sudo dd if=/dev/zero of=${DISK} bs=1M count=10 (Xóa dữ liệu ở thẻ đi ấy)
sudo dd if=./u-boot/MLO of=${DISK} count=1 seek=1 bs=128k
sudo dd if=./u-boot/u-boot.img of=${DISK} count=2 seek=1 bs=384k
sudo sfdisk ${DISK} <<__EOF__
4M,,L,*
__EOF__
sudo mkfs.ext4 -L rootfs -O ^metadata_csum,^64bit ${DISK}1
#write data to sdcard
sudo mkdir -p /media/rootfs/
sudo mount ${DISK}1 /media/rootfs/

# gõ lsblk để kiểm tra đã mount được chưa

sudo mkdir -p /media/rootfs/opt/backup/uboot/
sudo cp -v ./u-boot/MLO /media/rootfs/opt/backup/uboot/
sudo cp -v ./u-boot/u-boot.img /media/rootfs/opt/backup/uboot/
cd bb-kernel
cd KERNEL/
vim drivers/watchdog/omap_wdt.c ( cái này bỏ đi cũng được nha , chủ yếu để xem log trong watchdog để kiểm tra có đúng là boot đúng OS lên không thôi mà ) 
cd ..
./tools/rebuild.sh

export kernel_version=5.4.129-bone55

cd ..
sudo tar xfvp ./debian-*-*-armhf-*/armhf-rootfs-*.tar -C /media/rootfs/
sync
sudo chown root:root /media/rootfs/
sudo chmod 755 /media/rootfs/

sudo sh -c "echo 'uname_r=${kernel_version}' >> /media/rootfs/boot/uEnv.txt"
sudo cp -v ./bb-kernel/deploy/${kernel_version}.zImage /media/rootfs/boot/vmlinuz-${kernel_version}
sudo mkdir -p /media/rootfs/boot/dtbs/${kernel_version}/
sudo tar xfv ./bb-kernel/deploy/${kernel_version}-dtbs.tar.gz -C /media/rootfs/boot/dtbs/${kernel_version}/
sudo tar xfv ./bb-kernel/deploy/${kernel_version}-modules.tar.gz -C /media/rootfs/
sudo sh -c "echo '/dev/mmcblk0p1  /    auto  errors=remount-ro  0  1' >> /media/rootfs/etc/fstab"

sudo vim /media/rootfs/etc/network/interfaces

# sau đó thêm vào 4 dòng dưới:
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp
sync 
sudo umount /media/rootfs

# sau khi xong thì cắm thẻ vào bbb và giữ nút s2 trước khi cắm nguồn, để nạp vào, chứ không thì nó nạp chương trình mặc định của nó
# username: debian    pass: tempwd
dmesg để đọc log nhé 

Vậy là con board Beaglebone Black đã chạy được Linux OS (debian) rồi :
<img width="511" height="292" alt="image" src="https://github.com/user-attachments/assets/eef1d51f-6deb-4e05-8321-7c067bf1d06c" /> 

Dùng nguồn nào cũng được nhé , cục nguồn 5v-1A , hay dây cắm USB mini cũng được . 










