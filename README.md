# Kịch bản thuyết trình

## Đề tài
Lỗ hổng bảo mật và mã độc trên nền tảng Raspberry Pi

## Cách dùng tài liệu này
- Mỗi slide có 3 phần: `Mục tiêu`, `Cách nói`, `Thuật ngữ cần giải thích`.
- Ưu tiên nói chậm, câu ngắn, tránh đọc nguyên văn.
- Khi gặp thuật ngữ tiếng Anh, nên nói theo mẫu: "thuật ngữ này có nghĩa là..."

## Slide 1. Lỗ hổng bảo mật và mã độc trên nền tảng Raspberry Pi
**Mục tiêu:** Giới thiệu đề tài và định hướng Blue Team.

**Cách nói:**  
Kính thưa thầy và các bạn, nhóm em xin trình bày đề tài "Lỗ hổng bảo mật và mã độc trên nền tảng Raspberry Pi". Raspberry Pi vốn được thiết kế để dễ học, dễ tiếp cận và chi phí thấp, nhưng khi được đưa vào các hệ thống IoT thực tế thì chính sự tiện lợi đó có thể tạo ra rủi ro bảo mật. Báo cáo của nhóm đi theo hướng Blue Team, tức là tập trung phân tích rủi ro và đề xuất phòng thủ, không nhằm trình diễn tấn công ngoài thực tế.

**Thuật ngữ cần giải thích:**  
- `Raspberry Pi`: máy tính mini, thường dùng trong học tập, nhúng và IoT.  
- `IoT`: Internet of Things, tức là các thiết bị vật lý có kết nối mạng.  
- `Blue Team`: phía phòng thủ, chuyên phát hiện, giám sát và giảm thiểu tấn công.  
- `Lỗ hổng bảo mật`: điểm yếu có thể bị khai thác.  
- `Mã độc`: phần mềm độc hại như worm, botnet, backdoor.

## Slide 2. Nội dung báo cáo
**Mục tiêu:** Cho người nghe thấy cấu trúc bài.

**Cách nói:**  
Bài trình bày gồm năm phần. Phần một nói về kiến trúc Raspberry Pi vì muốn hiểu rủi ro thì trước hết phải hiểu nền tảng hoạt động thế nào. Phần hai phân tích lỗ hổng theo bốn lớp kỹ thuật. Phần ba nói về các họ mã độc trên ARM/Linux. Phần bốn là thực nghiệm trong lab cách ly. Phần năm là kết luận, hạn chế và hướng phát triển.

**Thuật ngữ cần giải thích:**  
- `Kiến trúc`: cách phần cứng và phần mềm được tổ chức.  
- `ARM/Linux`: thiết bị dùng CPU ARM và chạy hệ điều hành Linux.  
- `CVE`: mã định danh chuẩn cho lỗ hổng bảo mật.

## Slide 3. Vì sao Raspberry Pi là đối tượng nghiên cứu bảo mật quan trọng?
**Mục tiêu:** Trả lời câu hỏi "vì sao chọn Pi".

**Cách nói:**  
Raspberry Pi không còn chỉ là board học tập, mà đã được triển khai trong gateway IoT, nhà thông minh, màn hình thông tin, công nghiệp và bán lẻ. Khi một thiết bị như vậy có lỗ hổng, rủi ro không dừng ở một chiếc board mà có thể lan sang cả hệ thống. Vì thế Raspberry Pi là đối tượng nghiên cứu bảo mật có giá trị thực tế cao.

**Thuật ngữ cần giải thích:**  
- `Gateway IoT`: thiết bị trung gian thu thập dữ liệu từ cảm biến rồi gửi lên hệ thống lớn hơn.  
- `Smart home`: nhà thông minh.  
- `Digital signage`: màn hình điện tử hiển thị thông tin hoặc quảng cáo.

## Slide 4. Phạm vi và phương pháp nghiên cứu
**Mục tiêu:** Giải thích bài làm trong phạm vi nào và an toàn ra sao.

**Cách nói:**  
Nhóm chia nghiên cứu thành ba trục là kiến trúc, lỗ hổng phân tầng và mã độc ARM/Linux. Phần thực nghiệm dùng Raspberry Pi 3B+ và 4B, cài Raspberry Pi OS Bookworm Lite 64-bit, còn máy tấn công là Kali Linux trong môi trường VMware. Toàn bộ lab nằm trong mạng nội bộ biệt lập, không kết nối Internet, nhằm bảo đảm an toàn và tuân thủ định hướng học thuật.

**Thuật ngữ cần giải thích:**  
- `Bookworm Lite`: bản Linux tối giản, không có giao diện đồ họa.  
- `Headless`: chạy không cần màn hình hoặc desktop.  
- `Kali Linux`: hệ điều hành thường dùng trong kiểm thử bảo mật.  
- `VMware`: phần mềm máy ảo.  
- `Sanitized malware sample`: mẫu mã độc đã được làm vô hại một phần để phục vụ phân tích.  
- `C2`: Command and Control, máy chủ điều khiển mã độc.  
- `XOR`: phép toán dùng để che giấu dữ liệu ở mức đơn giản.

## Slide 5. Phần 1 - Kiến trúc Raspberry Pi
**Mục tiêu:** Chuyển sang phần nền tảng.

**Cách nói:**  
Phần một tập trung vào phần cứng, trình tự khởi động và hệ điều hành của Raspberry Pi. Mục tiêu là chỉ ra rằng nhiều rủi ro bảo mật không xuất hiện ngẫu nhiên, mà bắt nguồn từ chính cách nền tảng được thiết kế.

**Thuật ngữ cần giải thích:**  
- `Boot`: quá trình khởi động thiết bị.  
- `Hệ điều hành`: phần mềm nền điều phối toàn bộ tài nguyên của máy.

## Slide 6. Tiến hóa phần cứng: Pi 1 đến Pi 5
**Mục tiêu:** Giải thích quá trình phát triển phần cứng và tác động bảo mật.

**Cách nói:**  
Qua các thế hệ từ Pi 1 đến Pi 5, Raspberry Pi tăng mạnh về hiệu năng, kết nối và tính năng phần cứng. Tuy nhiên, mỗi lần bổ sung Wi-Fi, Bluetooth hoặc cơ chế boot mới cũng làm bề mặt tấn công rộng hơn. Điểm quan trọng là các tính năng bảo mật phần cứng chỉ có giá trị khi hệ điều hành thực sự bật và cấu hình đúng.

**Thuật ngữ cần giải thích:**  
- `SoC`: System on Chip, tức là nhiều thành phần như CPU, GPU, bộ điều khiển được tích hợp trên một chip.  
- `ARMv6, ARMv7, ARMv8`: các thế hệ kiến trúc CPU ARM.  
- `Bề mặt tấn công`: toàn bộ nơi mà kẻ tấn công có thể khai thác.  
- `TrustZone`: cơ chế tách vùng xử lý an toàn trên ARM.  
- `PAC`: Pointer Authentication, giúp giảm tấn công sửa con trỏ điều khiển.  
- `BTI`: Branch Target Identification, giảm tấn công chuyển hướng luồng thực thi.  
- `Secure Boot`: cơ chế chỉ cho phép khởi động phần mềm đã được tin cậy.

## Slide 7-9. Tiến hóa phần cứng: nhấn mạnh ý nghĩa bảo mật
**Mục tiêu:** Nhấn sâu ba luận điểm của slide trước.

**Cách nói:**  
Ở đây em muốn nhấn ba ý. Thứ nhất, phần cứng mới có thêm tính năng bảo vệ nhưng không phải lúc nào cũng được bật. Thứ hai, khi Wi-Fi và Bluetooth được tích hợp sẵn, thiết bị thuận tiện hơn nhưng cũng mở thêm đường tấn công không dây. Thứ ba, Secure Boot chỉ xuất hiện từ Pi 5 và vẫn mặc định tắt, nên phần lớn Pi 3 và Pi 4 hiện nay vẫn thiếu lớp bảo vệ khởi động ở cấp phần cứng.

**Thuật ngữ cần giải thích:**  
- `Mặc định tắt`: có tính năng nhưng người dùng phải tự bật.  
- `Tấn công không dây`: khai thác qua Bluetooth hoặc Wi-Fi mà không cần cắm dây.

## Slide 10-12. Kiến trúc SoC Broadcom: GPU đứng trước CPU
**Mục tiêu:** Giải thích vì sao boot của Pi khác PC.

**Cách nói:**  
Điểm khác biệt quan trọng của Raspberry Pi là GPU được cấp nguồn và chạy trước CPU. Nói đơn giản, trên máy tính PC thông thường ta thường nghĩ CPU và BIOS hoặc UEFI là trung tâm của chuỗi khởi động, còn trên Raspberry Pi thì firmware GPU lại nắm bước đầu tiên. Điều này tạo ra một hàm ý bảo mật quan trọng: gốc tin cậy ban đầu nằm ở phần firmware GPU vốn là mã đóng, rất khó kiểm toán độc lập.

**Thuật ngữ cần giải thích:**  
- `GPU`: bộ xử lý đồ họa.  
- `CPU`: bộ xử lý trung tâm.  
- `BIOS/UEFI`: firmware khởi động phổ biến trên PC.  
- `Gốc tin cậy`: thành phần đầu tiên mà hệ thống buộc phải tin tưởng để bắt đầu khởi động.  
- `Mã đóng`: phần mềm không công khai mã nguồn.

## Slide 13. Trình tự khởi động: điểm yếu cấu trúc trên Pi 1-4
**Mục tiêu:** Chỉ ra lỗ hổng mang tính kiến trúc.

**Cách nói:**  
Ở Pi 1 đến Pi 4, firmware như `start.elf` hay `fixup.dat` được đọc trực tiếp từ phân vùng boot mà không kiểm tra chữ ký số. Điều đó có nghĩa là nếu ai đó có quyền truy cập vật lý vào thẻ SD, họ có thể thay đổi thành phần khởi động trước cả khi Linux chạy. Vì vậy đây không chỉ là lỗi cấu hình mà là một điểm yếu có tính cấu trúc.

**Thuật ngữ cần giải thích:**  
- `Firmware`: phần mềm mức thấp điều khiển phần cứng.  
- `Chữ ký số`: cơ chế xác minh file có đúng từ nguồn tin cậy hay không.  
- `FAT32`: hệ thống file đơn giản, phổ biến cho phân vùng boot.  
- `Persistence`: khả năng mã độc tồn tại dai dẳng sau khi khởi động lại hoặc cài lại hệ điều hành.

## Slide 14. Raspberry Pi OS: cấu hình mặc định và dịch vụ bật sẵn
**Mục tiêu:** Chỉ ra cái gì "tiện" thường kéo theo rủi ro.

**Cách nói:**  
Raspberry Pi OS có nhiều dịch vụ thuận tiện cho học tập và quản trị như SSH, Avahi, UART, Bluetooth hay Wi-Fi. Nhưng trong góc nhìn an toàn thông tin, mỗi dịch vụ đang bật sẵn là thêm một cửa có thể bị thử khai thác. Vì vậy cấu hình mặc định an toàn chưa chắc là cấu hình mặc định tiện nhất.

**Thuật ngữ cần giải thích:**  
- `SSH`: giao thức quản trị từ xa có mã hóa.  
- `Avahi/mDNS`: cơ chế tự tìm thiết bị trong mạng nội bộ, ví dụ `raspberrypi.local`.  
- `UART`: cổng giao tiếp nối tiếp mức phần cứng.  
- `Fingerprinting`: nhận diện hệ điều hành hoặc dịch vụ của mục tiêu.  
- `unattended-upgrades`: cơ chế tự động cài bản vá bảo mật.

## Slide 15. Mô hình bề mặt tấn công bốn lớp
**Mục tiêu:** Đưa ra khung nhìn tổng quát cho phần lỗ hổng.

**Cách nói:**  
Để tránh liệt kê rời rạc, nhóm chia bề mặt tấn công của Raspberry Pi thành bốn lớp: firmware, hệ điều hành, giao thức mạng và giao tiếp ngoại vi. Cách chia này giúp ta dễ thấy mỗi loại rủi ro yêu cầu điều kiện khai thác khác nhau, ví dụ có loại cần tiếp cận vật lý, có loại chỉ cần cùng mạng LAN, và có loại có thể khai thác từ xa.

**Thuật ngữ cần giải thích:**  
- `Firmware`: lớp gần phần cứng nhất.  
- `Giao thức mạng`: luật giao tiếp trên mạng như SSH, Wi-Fi, Bluetooth.  
- `Giao tiếp ngoại vi`: kết nối với cảm biến, bus, cổng vật lý.

## Slide 16. Phần 2 - Phân tích lỗ hổng bảo mật
**Mục tiêu:** Mở phần lỗ hổng.

**Cách nói:**  
Sau khi đã hiểu kiến trúc, phần hai sẽ đi vào phân tích lỗ hổng theo một khung có hệ thống. Nhóm không chỉ nói lỗ hổng tồn tại, mà còn gắn chúng với vị trí khai thác, điều kiện tấn công và mức độ ảnh hưởng.

## Slide 17. Khung phân loại: bốn lớp, mô hình tác nhân, STRIDE
**Mục tiêu:** Giới thiệu phương pháp phân tích.

**Cách nói:**  
Nhóm dùng ba chiều để nhìn lỗ hổng. Chiều thứ nhất là bốn lớp kỹ thuật. Chiều thứ hai là tác nhân tấn công, gồm người có tiếp cận vật lý, người ở gần trong cùng vùng mạng hoặc sóng, và người ở xa qua Internet. Chiều thứ ba là STRIDE, một khung phân tích mối đe dọa giúp phân loại kiểu rủi ro như giả mạo, sửa đổi, lộ thông tin hay leo thang đặc quyền.

**Thuật ngữ cần giải thích:**  
- `STRIDE`: mô hình phân loại đe dọa của Microsoft.  
- `Spoofing`: giả mạo.  
- `Tampering`: sửa đổi trái phép.  
- `Information Disclosure`: lộ thông tin.  
- `Elevation of Privilege`: leo thang đặc quyền.  
- `DoS`: từ chối dịch vụ, làm hệ thống gián đoạn.

## Slide 18. Lớp firmware: GPU firmware và bootloader
**Mục tiêu:** Phân tích rủi ro ở tầng thấp nhất.

**Cách nói:**  
Ở lớp firmware, rủi ro lớn nhất là kẻ tấn công thay đổi thành phần khởi động để chiếm quyền trước hệ điều hành. Nếu đã có quyền `sudo`, họ cũng có thể ghi đè khu vực `/boot` để tạo persistence sâu hơn. Với Pi 1 đến Pi 4, vì không có xác thực chữ ký firmware nên đây là rủi ro khó loại bỏ triệt để bằng phần mềm.

**Thuật ngữ cần giải thích:**  
- `Bootloader`: thành phần trung gian khởi động hệ điều hành.  
- `sudo`: cơ chế cho phép người dùng thực hiện lệnh với quyền cao.  
- `/boot`: nơi chứa file phục vụ khởi động.

## Slide 19. Lớp hệ điều hành: lỗ hổng cấu hình mặc định
**Mục tiêu:** Giải thích CVE-2021-38759 theo cách dễ hiểu.

**Cách nói:**  
Đây là một điểm rất đáng chú ý vì nó không phải lỗi code phức tạp mà là lỗi do lựa chọn mặc định. Trong nhiều năm, Raspberry Pi OS dùng tài khoản `pi` với mật khẩu mặc định `raspberry`, và tài khoản này còn có `sudo NOPASSWD`, nghĩa là sau khi đăng nhập được thì có thể lên quyền cao mà không cần nhập lại mật khẩu. Vì thế chỉ một cấu hình mặc định yếu cũng đủ biến thành lỗ hổng mức rất nghiêm trọng.

**Thuật ngữ cần giải thích:**  
- `CVE-2021-38759`: mã lỗ hổng liên quan tài khoản mặc định của Raspberry Pi.  
- `CVSS 9.8`: thang điểm mức độ nghiêm trọng, 9.8 là rất cao.  
- `CWE-1188`: cấu hình khởi tạo mặc định không an toàn.  
- `NOPASSWD`: cho phép dùng `sudo` mà không hỏi lại mật khẩu.

## Slide 20. Lớp hệ điều hành: leo thang đặc quyền
**Mục tiêu:** Giải thích các lỗ hổng LPE.

**Cách nói:**  
Nhóm liệt kê bốn ví dụ điển hình là Dirty COW, PwnKit, Baron Samedit và Dirty Pipe. Điểm chung của chúng là kẻ tấn công ban đầu chỉ cần có một mức truy cập cục bộ nào đó, sau đó dùng lỗi hệ điều hành hoặc tiện ích hệ thống để leo lên quyền `root`. Điều này rất nguy hiểm vì một bước xâm nhập nhỏ có thể nhanh chóng biến thành kiểm soát toàn bộ thiết bị.

**Thuật ngữ cần giải thích:**  
- `LPE`: Local Privilege Escalation, leo thang đặc quyền cục bộ.  
- `root`: quyền cao nhất trên Linux.  
- `Kernel`: lõi của hệ điều hành.  
- `Race condition`: lỗi xảy ra khi nhiều tiến trình truy cập tài nguyên theo thời điểm không mong muốn.  
- `Heap overflow`: ghi tràn vùng nhớ heap.

## Slide 21. Lớp giao thức mạng: SSH và xác thực yếu
**Mục tiêu:** Phân biệt giao thức mạnh với cấu hình yếu.

**Cách nói:**  
SSH tự nó là giao thức an toàn vì có mã hóa, nhưng nếu người dùng đặt mật khẩu yếu hoặc dùng mặc định thì toàn bộ hệ thống vẫn dễ bị brute-force. Đây là ví dụ rất điển hình trong an toàn thông tin: công nghệ mạnh không cứu được cấu hình yếu. Vì vậy biện pháp đúng không chỉ là bật SSH, mà còn phải dùng khóa SSH, tắt đăng nhập bằng mật khẩu và giới hạn số lần thử sai.

**Thuật ngữ cần giải thích:**  
- `Brute-force`: thử đi thử lại nhiều mật khẩu cho đến khi trúng.  
- `Hydra/Medusa`: công cụ tự động thử mật khẩu.  
- `SSH key`: cặp khóa công khai và bí mật để đăng nhập an toàn hơn mật khẩu.  
- `PermitRootLogin`: tùy chọn cho phép cấm hoặc hạn chế đăng nhập trực tiếp bằng root.  
- `fail2ban`: công cụ chặn IP sau nhiều lần đăng nhập thất bại.

## Slide 22. Bluetooth: BlueBorne và KNOB
**Mục tiêu:** Giải thích rủi ro giao tiếp không dây.

**Cách nói:**  
Với Bluetooth, hai ví dụ đáng chú ý là BlueBorne và KNOB. BlueBorne cho thấy có thể tấn công qua Bluetooth mà thậm chí không cần ghép đôi thiết bị trong một số điều kiện. Còn KNOB là kiểu tấn công làm suy yếu khóa mã hóa Bluetooth xuống mức rất yếu, từ đó tăng khả năng giải mã lưu lượng.

**Thuật ngữ cần giải thích:**  
- `Ghép đôi`: pairing, quá trình hai thiết bị Bluetooth tin cậy nhau.  
- `L2CAP`: một lớp giao thức trong Bluetooth.  
- `MITM`: Man-in-the-Middle, kẻ đứng giữa để nghe lén hoặc sửa dữ liệu.  
- `Entropy thấp`: khóa quá ít ngẫu nhiên, nên dễ đoán hoặc brute-force.

## Slide 23. Wi-Fi KRACK và mDNS
**Mục tiêu:** Giải thích tấn công ở mạng không dây và khám phá dịch vụ.

**Cách nói:**  
KRACK là tấn công vào quá trình bắt tay của WPA2, làm thiết bị cài lại khóa phiên đã dùng và từ đó tăng nguy cơ giải mã hoặc giả mạo dữ liệu. Còn Avahi và mDNS thì không phải mã độc hay lỗ hổng kiểu tràn bộ nhớ, nhưng lại làm lộ thông tin dịch vụ đang chạy trong mạng nội bộ. Nói cách khác, có những thành phần không trực tiếp bị "hack", nhưng lại giúp kẻ tấn công trinh sát mục tiêu dễ hơn rất nhiều.

**Thuật ngữ cần giải thích:**  
- `WPA2`: chuẩn bảo mật Wi-Fi phổ biến.  
- `4-way handshake`: quá trình bắt tay bốn bước khi thiết bị vào mạng Wi-Fi.  
- `Nonce`: giá trị dùng một lần trong phiên mã hóa.  
- `Service discovery`: cơ chế tự phát hiện dịch vụ trong mạng LAN.

## Slide 24. Giao tiếp ngoại vi: UART, I2C, SPI
**Mục tiêu:** Nhấn mạnh rủi ro vật lý thường bị xem nhẹ.

**Cách nói:**  
Trong IoT, nhiều người chỉ nghĩ đến tấn công qua mạng mà quên mất cổng vật lý và bus ngoại vi. UART có thể cho ra shell ở mức rất nguy hiểm nếu thiết bị bị tiếp cận trực tiếp. Còn I2C và SPI vốn được thiết kế cho các linh kiện tin cậy nói chuyện với nhau, nên bản thân chúng không có xác thực hay mã hóa, dẫn đến nguy cơ thiết bị ngoại vi giả mạo dữ liệu cảm biến.

**Thuật ngữ cần giải thích:**  
- `UART`: giao tiếp nối tiếp đơn giản, thường dùng debug.  
- `I2C`, `SPI`: hai bus giao tiếp phổ biến giữa vi điều khiển và cảm biến.  
- `Shell`: giao diện dòng lệnh để điều khiển hệ thống.  
- `Rogue peripheral injection`: cắm thiết bị ngoại vi giả mạo để đưa dữ liệu sai hoặc chiếm quyền.

## Slide 25. Tổng hợp CVE tiêu biểu
**Mục tiêu:** Tóm tắt mức độ nghiêm trọng của các lỗ hổng.

**Cách nói:**  
Slide này tổng hợp tám CVE tiêu biểu mà nhóm đã xác minh trên NVD. Ý chính ở đây không phải là học thuộc từng mã, mà là thấy một bức tranh chung: nhiều lỗ hổng trên Raspberry Pi đạt mức nghiêm trọng cao, trải từ cấu hình mặc định yếu cho tới lỗi trong kernel, sudo, polkit, Bluetooth và Wi-Fi. Điều đó cho thấy rủi ro nằm ở nhiều lớp, không phải một điểm đơn lẻ.

**Thuật ngữ cần giải thích:**  
- `NVD`: National Vulnerability Database, cơ sở dữ liệu lỗ hổng chuẩn của NIST.  
- `CVSS`: thang điểm đánh giá mức nghiêm trọng.  
- `Deferred`: NVD chưa gán điểm chính thức tại thời điểm đó.

## Slide 26. Phần 3 - Mã độc trên ARM/Linux
**Mục tiêu:** Chuyển từ lỗ hổng sang mã độc.

**Cách nói:**  
Sau khi hiểu lỗ hổng, phần ba trả lời câu hỏi: nếu các điểm yếu đó bị lợi dụng thì loại mã độc nào thường nhắm vào Raspberry Pi và thiết bị ARM/Linux. Nhóm chọn bốn họ tiêu biểu để cho thấy quá trình tiến hóa của botnet IoT.

## Slide 27. Đặc điểm mã độc nhắm vào ARM/Linux
**Mục tiêu:** Giải thích malware trên Linux/ARM có gì riêng.

**Cách nói:**  
Mã độc trên ARM/Linux thường được đóng gói dưới dạng file ELF, có thể biên dịch cho nhiều kiến trúc khác nhau như ARM, MIPS hay x86 để tăng khả năng lây lan. Chúng thường nhỏ gọn, ít phụ thuộc thư viện ngoài và hay dùng các kỹ thuật che giấu cơ bản như UPX hoặc loại bỏ symbol để gây khó cho phân tích tĩnh.

**Thuật ngữ cần giải thích:**  
- `ELF`: định dạng file thực thi chuẩn trên Linux.  
- `e_machine`: trường trong ELF cho biết kiến trúc CPU đích.  
- `Cross-compile`: biên dịch trên một máy nhưng chạy ở kiến trúc khác.  
- `UPX`: công cụ nén executable.  
- `Stripped binary`: file đã bỏ symbol và thông tin gỡ lỗi.

## Slide 28. Case study 1 - Mirai
**Mục tiêu:** Giải thích vì sao Mirai nổi tiếng.

**Cách nói:**  
Mirai là một trong những botnet IoT nổi tiếng nhất vì nó khai thác đúng điểm yếu phổ biến của thiết bị nhúng: Telnet mở và tài khoản mặc định yếu. Nó quét Internet, thử một danh sách credential hardcoded, rồi nạp bot vào thiết bị. Trường hợp này rất phù hợp với Raspberry Pi vì trong quá khứ tài khoản `pi/raspberry` cũng từng là cấu hình mặc định rất nguy hiểm.

**Thuật ngữ cần giải thích:**  
- `Botnet`: mạng lưới thiết bị bị chiếm quyền và điều khiển tập trung hoặc bán tập trung.  
- `Telnet`: giao thức quản trị cũ, không mã hóa.  
- `Credential`: thông tin xác thực như username và password.  
- `Hardcoded`: ghi cố định trong mã chương trình.  
- `Loader`: thành phần đưa mã độc vào nạn nhân.

## Slide 29. Case study 2 - Darlloz và Tsunami/Kaiten
**Mục tiêu:** Cho thấy mã độc IoT không chỉ có Mirai.

**Cách nói:**  
Darlloz là một worm lây qua lỗ hổng PHP CGI, cho thấy mã độc nhắm ARM/Linux đã xuất hiện từ khá sớm. Tsunami hay Kaiten đại diện cho thế hệ botnet cũ hơn, dùng IRC làm kênh điều khiển. Ý quan trọng là dù kỹ thuật khác nhau, các mã độc này đều tận dụng một thực tế chung: thiết bị IoT thường bị giám sát kém và cập nhật vá lỗi chậm.

**Thuật ngữ cần giải thích:**  
- `Worm`: mã độc có khả năng tự lây lan.  
- `RCE`: Remote Code Execution, thực thi mã từ xa.  
- `IRC`: giao thức chat cũ nhưng từng được botnet dùng làm kênh điều khiển.  
- `Remote shell`: khả năng mở dòng lệnh từ xa trên thiết bị nạn nhân.

## Slide 30. Case study 3 - Mozi
**Mục tiêu:** Giải thích xu hướng botnet mới khó triệt phá hơn.

**Cách nói:**  
Khác với Mirai phụ thuộc rõ vào máy chủ điều khiển, Mozi chuyển sang kiến trúc P2P dựa trên DHT. Mỗi bot vừa nhận vừa chuyển tiếp thông tin, nên nếu chỉ gỡ một máy chủ thì chưa đủ để đánh sập toàn mạng. Đây là xu hướng tiến hóa quan trọng: botnet hiện đại ngày càng phân tán và bền vững hơn.

**Thuật ngữ cần giải thích:**  
- `P2P`: peer-to-peer, các nút ngang hàng giao tiếp trực tiếp.  
- `DHT`: Distributed Hash Table, cơ chế phân tán dùng để tìm và lưu thông tin trong mạng P2P.  
- `Phi tập trung`: không phụ thuộc một máy chủ trung tâm duy nhất.  
- `Lệnh ký số`: lệnh được xác thực bằng mật mã để tránh bị giả mạo.

## Slide 31. So sánh bốn họ mã độc ARM/Linux
**Mục tiêu:** Rút ra xu hướng chung.

**Cách nói:**  
Khi đặt bốn họ mã độc cạnh nhau, ta thấy một xu hướng rõ ràng: từ IRC tập trung, sang C2 tùy chỉnh, rồi đến P2P phi tập trung. Điều này có nghĩa là phòng thủ hiện nay không thể chỉ trông chờ vào việc gỡ hạ tầng điều khiển của botnet, mà phải bảo vệ trực tiếp từng thiết bị đầu cuối ngay từ đầu.

**Thuật ngữ cần giải thích:**  
- `C2 tùy chỉnh`: cơ chế điều khiển riêng của từng họ mã độc.  
- `Thiết bị đầu cuối`: endpoint, ở đây là chính Raspberry Pi hoặc thiết bị IoT chạy Linux.

## Slide 32. Phòng thủ theo chiều sâu
**Mục tiêu:** Đưa ra hướng phòng thủ thực dụng.

**Cách nói:**  
Slide này trình bày nguyên tắc defense-in-depth, tức là không dựa vào một biện pháp duy nhất. Thay vào đó, ta kết hợp nhiều lớp như mật khẩu mạnh, SSH key, fail2ban, tắt UART, tắt Bluetooth, bật firewall, cập nhật bản vá và bỏ `sudo NOPASSWD`. Với IoT, tư duy đúng là nếu một lớp bị vượt qua thì các lớp còn lại vẫn tiếp tục cản trở tấn công.

**Thuật ngữ cần giải thích:**  
- `Defense-in-Depth`: phòng thủ nhiều lớp.  
- `Firewall`: tường lửa lọc lưu lượng mạng.  
- `iptables/nftables`: công cụ cấu hình firewall trên Linux.  
- `NIST SP 800-213`: hướng dẫn bảo mật thiết bị IoT của NIST.  
- `OWASP IoT Top 10`: danh sách rủi ro IoT phổ biến.

## Slide 33. Phần 4 - Demo và thực nghiệm
**Mục tiêu:** Chuyển sang phần minh họa.

**Cách nói:**  
Sau phần lý thuyết, nhóm thực hiện ba kịch bản trong lab cách ly. Mục tiêu không phải là biểu diễn kỹ thuật tấn công phức tạp, mà là chứng minh rằng với công cụ phổ biến và kỹ năng cơ bản, một thiết bị chưa hardening có thể bị thu thập thông tin, đoán mật khẩu và bị phân tích mã độc rất nhanh.

**Thuật ngữ cần giải thích:**  
- `Reconnaissance`: giai đoạn trinh sát, thu thập thông tin mục tiêu.  
- `Hardening`: quá trình gia cố cấu hình để an toàn hơn.

## Slide 34. Mô hình mạng và công cụ thực nghiệm
**Mục tiêu:** Mô tả lab rõ ràng.

**Cách nói:**  
Trong mô hình thực nghiệm, Raspberry Pi 4B đóng vai trò nạn nhân, còn Kali Linux là máy kiểm thử. Hai bên nằm trong mạng LAN riêng và không nối Internet. Các công cụ sử dụng gồm Nmap để quét, Hydra để thử mật khẩu, Wireshark để bắt gói tin và binutils để phân tích file thực thi.

**Thuật ngữ cần giải thích:**  
- `LAN`: mạng cục bộ.  
- `Nmap`: công cụ quét cổng và nhận diện dịch vụ.  
- `Wireshark`: công cụ phân tích gói tin mạng.  
- `binutils`: bộ công cụ dòng lệnh để xem file nhị phân như `strings`, `readelf`.

## Slide 35-36. Kịch bản 1 - Reconnaissance bằng Nmap
**Mục tiêu:** Chứng minh trinh sát ban đầu rất nhanh.

**Cách nói:**  
Ở kịch bản đầu tiên, nhóm dùng Nmap để quét các cổng phổ biến. Kết quả cho thấy SSH đang mở, mDNS cũng đang hoạt động, và hệ điều hành được fingerprint là ARM Linux. Điều quan trọng ở đây là chỉ trong chưa đầy mười giây, kẻ tấn công đã có đủ thông tin cơ bản để quyết định bước tiếp theo.

**Thuật ngữ cần giải thích:**  
- `Port 22`: cổng mặc định của SSH.  
- `Port 5353`: cổng của mDNS.  
- `Open`: cổng đang lắng nghe và có thể tương tác.  
- `Fingerprint`: suy đoán loại OS hoặc dịch vụ từ phản hồi mạng.

## Slide 37-38. Kịch bản 2 - Khai thác SSH và phân tích lưu lượng
**Mục tiêu:** Minh họa tác hại của mật khẩu yếu.

**Cách nói:**  
Ở kịch bản thứ hai, nhóm dùng Hydra thử mật khẩu với tài khoản `pi`, và tìm ra mật khẩu `raspberry` trong thời gian ngắn. Sau khi đăng nhập qua SSH, việc leo lên quyền cao diễn ra rất nhanh do cấu hình `sudo NOPASSWD`. Song song đó, Wireshark cho thấy dữ liệu SSH đã được mã hóa sau handshake, còn Telnet thì lộ toàn bộ mật khẩu và lệnh ở dạng rõ, vì vậy dùng SSH thay Telnet là yêu cầu tối thiểu trong IoT.

**Thuật ngữ cần giải thích:**  
- `Wordlist`: danh sách mật khẩu dùng để thử.  
- `Handshake`: giai đoạn hai bên thiết lập kết nối an toàn.  
- `Payload`: phần dữ liệu thực sự được truyền.  
- `Cleartext`: dữ liệu dạng rõ, ai bắt được cũng đọc được.

## Slide 39. Kịch bản 3 - Phân tích tĩnh mẫu Mirai ARM
**Mục tiêu:** Giải thích cách nhận diện mã độc mà không cần chạy nó.

**Cách nói:**  
Kịch bản cuối dùng phân tích tĩnh trên một mẫu Mirai ARM đã được sanitized. Lệnh `file` cho biết đây là ELF 32-bit ARM, liên kết tĩnh và đã stripped. Lệnh `strings` trích được các chuỗi đặc trưng như `/bin/busybox`, `MIRAI`, `/dev/watchdog`, còn `readelf` xác nhận header ELF và kiến trúc đích. Từ đó ta thấy chỉ với phân tích tĩnh cũng đã đủ để phân loại mẫu và dựng chỉ số nhận diện ban đầu.

**Thuật ngữ cần giải thích:**  
- `Phân tích tĩnh`: phân tích file mà không chạy file.  
- `Statically linked`: đã gói sẵn phần phụ thuộc vào binary.  
- `IoC`: Indicator of Compromise, chỉ dấu giúp nhận diện xâm nhập.  
- `Watchdog`: cơ chế theo dõi hệ thống, malware đôi khi lợi dụng hoặc can thiệp.

## Slide 40. Đánh giá kết quả và hạn chế
**Mục tiêu:** Kết nối thực nghiệm với luận điểm nghiên cứu.

**Cách nói:**  
Ba kịch bản cho thấy hai điều. Thứ nhất, trên thiết bị chưa hardening, chuỗi từ trinh sát đến chiếm quyền có thể hoàn thành chỉ trong vài phút với công cụ miễn phí. Thứ hai, phân tích tĩnh ELF là kỹ thuật khả thi để nhận diện mã độc IoT ngay cả khi không có sandbox phức tạp. Tuy nhiên nhóm cũng nêu rõ hạn chế là lab còn đơn giản, chưa có đầy đủ NAT, firewall doanh nghiệp và phân tích động.

**Thuật ngữ cần giải thích:**  
- `Sandbox`: môi trường cách ly để quan sát hành vi mã độc.  
- `Phân tích động`: chạy mẫu trong môi trường kiểm soát để xem nó làm gì khi hoạt động.

## Slide 41. Phần 5 - Kết luận
**Mục tiêu:** Chuyển sang phần tổng kết.

**Cách nói:**  
Phần cuối sẽ tóm tắt các kết luận học thuật quan trọng nhất, đồng thời nêu rõ giới hạn của nghiên cứu và đề xuất hướng phát triển tiếp theo. Đây là phần giúp hội đồng thấy nhóm không chỉ trình bày hiện tượng mà còn rút ra thông điệp có hệ thống.

## Slide 42. Bốn kết luận học thuật chính
**Mục tiêu:** Chốt bốn luận điểm lớn.

**Cách nói:**  
Kết luận thứ nhất là nhiều rủi ro nghiêm trọng nhất đến từ thiết kế ưu tiên tiện lợi hơn bảo mật, ví dụ điển hình là tài khoản mặc định. Kết luận thứ hai là cơ chế `GPU-first boot` ở Pi 1 đến Pi 4 tạo ra điểm yếu cấu trúc khó vá hoàn toàn bằng phần mềm. Kết luận thứ ba là mã độc IoT tiến hóa từ tập trung sang phi tập trung, khiến việc triệt phá khó hơn. Kết luận cuối cùng là rủi ro trên Raspberry Pi là rủi ro thực tế, không chỉ nằm trên lý thuyết.

## Slide 43. Hạn chế của báo cáo
**Mục tiêu:** Thể hiện tính trung thực học thuật.

**Cách nói:**  
Nhóm chủ động nêu năm hạn chế chính. Môi trường thực nghiệm chưa mô phỏng đầy đủ doanh nghiệp thật. Phân tích tĩnh chưa giải mã hết phần dữ liệu bị XOR che giấu. Mẫu mã độc đã được sanitized nên không đánh giá được hành vi C2 thực tế. Phạm vi phần cứng còn tập trung vào Pi 3B+ và 4B. Cuối cùng, vẫn tồn tại rủi ro tồn dư ở firmware Pi 1 đến Pi 4 và ở chuỗi cung ứng phần mềm.

**Thuật ngữ cần giải thích:**  
- `Rủi ro tồn dư`: phần rủi ro còn lại ngay cả khi đã áp dụng biện pháp phòng thủ.  
- `Chuỗi cung ứng phần mềm`: nơi phần mềm, bản vá, thư viện hoặc repo có thể bị cài cắm độc hại trước khi đến tay người dùng.

## Slide 44. Hướng phát triển tiếp theo
**Mục tiêu:** Mở ra hướng nghiên cứu mới.

**Cách nói:**  
Nhóm đề xuất bốn hướng tiếp theo. Một là phân tích động trong QEMU sandbox để quan sát hành vi runtime và giải mã thêm phần C2. Hai là nghiên cứu phát hiện bất thường bằng mô hình máy học nhẹ chạy ngay trên Raspberry Pi. Ba là đánh giá Secure Boot trên Pi 5. Bốn là so sánh các bản phân phối Linux khác nhau trên nền ARM để xem mức độ an toàn thay đổi ra sao.

**Thuật ngữ cần giải thích:**  
- `QEMU`: công cụ mô phỏng và ảo hóa, có thể dùng để chạy môi trường ARM cách ly.  
- `Runtime`: hành vi khi chương trình đang thực thi.  
- `Machine Learning nhẹ`: mô hình đủ nhỏ để chạy được trên thiết bị tài nguyên hạn chế.

## Slide 45. Tài liệu tham khảo chính
**Mục tiêu:** Khẳng định nền tảng học thuật của bài.

**Cách nói:**  
Slide này cho thấy báo cáo dựa trên nguồn học thuật và nguồn kỹ thuật có độ tin cậy cao như NVD, nghiên cứu của Qualys, IBM X-Force, NIST và các bài báo chuyên ngành. Nhóm cũng đã xác minh độc lập các CVE quan trọng thay vì chỉ sao chép lại từ tài liệu thứ cấp.

**Thuật ngữ cần giải thích:**  
- `Nguồn sơ cấp`: nguồn gốc ban đầu như NVD, paper, advisory của hãng.  
- `IEEE`: kiểu trích dẫn tài liệu phổ biến trong kỹ thuật.

## Slide 46. Xin cảm ơn
**Mục tiêu:** Kết thúc gọn và tạo nhịp sang phần hỏi đáp.

**Cách nói:**  
Nhóm em xin cảm ơn thầy và các bạn đã lắng nghe. Nhóm rất mong nhận được góp ý và sẵn sàng trao đổi thêm ở phần phản biện.

## Phần hỏi đáp - câu chốt nhanh nên nhớ
- Nếu bị hỏi "vì sao Raspberry Pi nguy hiểm trong IoT": vì nó rẻ, phổ biến, dễ triển khai và thường bị giữ nguyên cấu hình mặc định.
- Nếu bị hỏi "điểm yếu lớn nhất là gì": tài khoản mặc định yếu và kiến trúc boot thiếu xác thực trên Pi 1 đến Pi 4.
- Nếu bị hỏi "vì sao nói ARM/Linux": vì Raspberry Pi dùng CPU ARM và chạy hệ điều hành họ Linux.
- Nếu bị hỏi "Mirai liên quan gì đến Pi": vì Mirai khai thác đúng kiểu lỗi rất phổ biến trên thiết bị nhúng, đặc biệt là credential mặc định và dịch vụ quản trị kém an toàn.
- Nếu bị hỏi "bài này thiên về tấn công hay phòng thủ": thiên về phòng thủ, vì mọi demo đều ở lab cách ly và mục tiêu là nhận diện rủi ro, không phải phát tán kỹ thuật khai thác ra thực địa.

## Mẹo trình bày cho newbie
- Mỗi khi nói một thuật ngữ mới, giải thích trong 1 câu rồi mới dùng tiếp.
- Dùng cặp đối lập để dễ nhớ: `SSH an toàn hơn Telnet`, `thiết kế tiện lợi nhưng kém an toàn`, `mã độc cũ tập trung, mã độc mới phi tập trung`.
- Với các mã CVE, không cần đọc hết, chỉ cần nói "đây là mã định danh chuẩn của lỗ hổng".
- Với slide nhiều chữ, chỉ chọn 1 ý chính và 1 ví dụ minh họa.
