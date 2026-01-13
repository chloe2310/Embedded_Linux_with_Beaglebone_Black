CÁCH SSH TỪ MÁY ẢO VMWARE VÀO BBB BẰNG CỔNG USB

Sau khi đã cắm boot thẻ sd xuống BBB rồi nhé , và lúc boot chắc bạn cũng biết rồi ( nhấn giữ S2 và cắm nguồn vào ) 

<img width="624" height="301" alt="image" src="https://github.com/user-attachments/assets/36383663-6cd7-49b8-9dbc-efe2a2e00e40" />

(  Ảnh này là khi USB UART  kết nối với máy host là Ubuntu nhé ) 

Kết quả sau khi boot image OS xuống board BBB .

<img width="624" height="325" alt="image" src="https://github.com/user-attachments/assets/daa9f6e2-a518-4a9c-891c-7c286f609e85" />


CÁC BƯỚC SSH TỪ VM VÀO BBB QUA USB MINI : 

Bước 1 : Nạp module USB Ethernet Gadget (g_ether) trên BBB
Giao tiếp giữa VM Ubuntu (host: 192.168.7.1) và BBB (gadget: 192.168.7.2) qua cổng USB mini dạng g_ether.
Lưu ý là máy ảo VM Ubuntu thì cứ dùng trên MobaXterm cho tiện nhé , copy lệnh các thứ cho dễ .
Nhớ là trên BBB (qua UART Console) cổng COM UART 

Ta gõ lệnh : 
sudo modprobe g_ether 9 

Giao diện usb0 sẽ được tạo.
Cái mục đích ở đây là ta sẽ kích hoạt được cái driver USB gadget để con BBB nó đóng vai trò là thiết bị USB Ethernet , sau khi dùng lệnh BBB sẽ xuất hiện interface usb0 đấy.
Kiểm tra bằng cách dùng lệnh : ip a , nếu nó xuất hiện có nghĩa gadget đã hoạt động 

<img width="624" height="283" alt="image" src="https://github.com/user-attachments/assets/c89a23ac-b92d-43ab-8b94-fb19c9fe98cc" />

Bước 2 : Đổi quyền cho USB mini trên VMware :
Sau khi dùng các lệnh ở trên thì bên máy ảo VMware nó sẽ hiện thông báo với (Netchip RNDIS/Ethernet Gadget) để chọn quyền sở hữu thiết bị , nhớ tích vào ( Connect to a virtual machine) nhé các bố . Muốn kiểm tra thì xem ở đây máy ảo đã nhận chưa : 

<img width="624" height="257" alt="image" src="https://github.com/user-attachments/assets/338af4e1-d3d1-40d8-85be-8ec072a6261a" />

Bước 3 : Đặt địa chỉ tĩnh cho usb0 : 
Ta dùng 2 lệnh này thôi : 
sudo ip link set usb0 up
sudo ip addr add 192.168.7.2/30 dev usb0

<img width="624" height="233" alt="image" src="https://github.com/user-attachments/assets/bbbe564a-8f76-4e82-b4c1-029e79088ed5" />

Rồi ở bên máy ảo WMware thì ping thử tới cái địa chỉ mình set up tĩnh kia là được : 

<img width="624" height="345" alt="image" src="https://github.com/user-attachments/assets/d3a9b775-53fb-4dfe-8f7a-045f970990a3" />
Nếu có phản hồi là thành công. Nó đã thông nhau .


Bước 4 : SSH vào BBB :
Dùng lệnh :
ssh debian@192.168.7.2
nó đòi mật khẩu thì nhập mật khẩu của BBB thôi : temppwd 

<img width="624" height="149" alt="image" src="https://github.com/user-attachments/assets/e0049124-fe4f-4848-b889-2103bfec089a" />

Tạo một Terminal mới rồi dùng scp để copy file.ko sang bên BBB thông qua ssh bằng các lệnh nhé:
Ví dụ : 
scp hello_module.ko debian@192.168.7.2:/home/debian/
Sau đấy nhập lại mật khẩu thôi .

Copy xong nó sẽ hiện ra 1 file.ko và đã copy sang được máy BBB .
Ở bên terminal của BBB , sau khi ls ra : 

<img width="624" height="196" alt="image" src="https://github.com/user-attachments/assets/5c73fb94-3bb0-464e-82cf-39ed0edbb059" />




