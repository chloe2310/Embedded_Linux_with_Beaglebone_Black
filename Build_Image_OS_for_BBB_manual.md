
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



















