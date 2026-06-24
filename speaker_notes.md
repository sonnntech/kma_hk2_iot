# Speaker Notes — Lỗ hổng bảo mật và mã độc trên nền tảng Raspberry Pi

> Bài trình bày bảo vệ — Học viện Kỹ thuật Mật mã, Khoa An toàn Thông tin
> Thời lượng mục tiêu: **~40 phút** (phần chính) + Q&A dùng phụ lục phản biện B1–B9
> Tốc độ nói tham chiếu: ~150 từ/phút tiếng Việt

---

## Slide 1 — HỌC VIỆN KỸ THUẬT MẬT MÃ  ·  KHOA AN TOÀN THÔNG TIN

Kính thưa thầy và các Bạn. Nhóm em gồm Nguyễn Vĩnh Hiển và bạn Nguyễn Ngọc Sơn, xin trình bày báo cáo đề tài "Lỗ hổng bảo mật và mã độc trên nền tảng Raspberry Pi"

## Slide 2 — Nội dung báo cáo

Báo cáo gồm năm phần, theo logic từ nền tảng đến ứng dụng. Phần một xây dựng hiểu biết về kiến trúc phần cứng và hệ điều hành Raspberry Pi, vì mọi lỗ hổng về sau đều bắt nguồn từ các quyết định thiết kế ở tầng này. Phần hai phân tích lỗ hổng bảo mật phân tầng, kèm tám CVE đã xác minh trên NVD. Phần ba trình bày các họ mã độc nhắm vào ARM/Linux — Mirai, Darlloz, Tsunami, Mozi — và chiến lược phòng thủ theo chiều sâu. Phần bốn là thực nghiệm: nhóm thử nghiệm chuỗi tấn công và phòng thủ trong lab cách ly. Phần năm là kết luận, hạn chế và hướng phát triển. Để các bạn lần đầu tiếp cận dễ hình dung: Raspberry Pi là một máy tính tí hon, chỉ to bằng chiếc thẻ tín dụng, nhưng chạy được hệ điều hành Linux đầy đủ. Nhờ giá rẻ và tiết kiệm điện, nó được dùng rộng rãi trong IoT, thiết bị nhúng và tự động hóa công nghiệp — và chính sự phổ biến đó khiến nó trở thành mục tiêu hấp dẫn của tấn công mạng.

## Slide 3 — Vì sao Raspberry Pi là đối tượng nghiên cứu bảo mật quan trọng?

Trước khi đi vào kỹ thuật, em muốn trả lời câu hỏi: vì sao Raspberry Pi là một đối tượng nghiên cứu bảo mật quan trọng? Câu trả lời nằm ở quy mô triển khai. Theo báo cáo tài chính của Raspberry Pi Holdings, đến nay đã có hơn bảy mươi ba triệu thiết bị bán ra trên tám mươi quốc gia. Slide thể hiện năm nhóm ứng dụng tiêu biểu: IoT gateway — nơi tập trung dữ liệu trước khi lên cloud; bảng thông tin số, ví dụ hơn một nghìn màn hình chuyến bay tại sân bay Heathrow; smart home; tự động hóa công nghiệp; và bán lẻ điểm bán hàng với chi phí chỉ bằng khoảng một phần mười hệ thống truyền thống. Điểm em muốn nhấn mạnh là: chính vì được dùng rộng rãi trong những lĩnh vực nhạy cảm như vậy, một lỗ hổng trên Raspberry Pi không còn là vấn đề của một thiết bị, mà là rủi ro ở quy mô hệ thống. Đó là lý do nhóm em chọn đề tài này.

## Slide 4 — MỞ ĐẦU

Đến năm 2023 đã có hơn 50 triệu thiết bị Raspberry pi đã được triển khai trên toàn cầu và gần một nửa trong số đó phục vụ các ứng dụng công nghiệp. Tuy nhiên, chính đặc điểm chi phí thấp và dễ sử dụng khiến nhiều thiết bị được triển khai mà không có cấu hình bảo mật đầy đủ. Điều này tạo ra một bề mặt tấn công lớn và nổi bật là sự kiện botnet Mirai (2016), trong đó hàng trăm nghìn thiết bị IoT chạy Linux trên kiến trúc ARM bị chiếm quyền điều khiển để phát động cuộc tấn công DDoS với băng thông đỉnh lên tới 620 Gbps.

## Slide 5 — Phạm vi và phương pháp nghiên cứu

Nghiên cứu tổ chức quanh ba trục bổ sung cho nhau: kiến trúc, lỗ hổng phân tầng, và mã độc kèm thực nghiệm. Về thực nghiệm, nhóm dùng Raspberry Pi 3B+ và 4B chạy Pi OS Bookworm Lite 64-bit — bản headless tiệm cận môi trường IoT thực tế nhất. Máy tấn công là Kali Linux 2024 trong VMware, kết nối với Pi qua một mạng LAN nội bộ cách ly hoàn toàn khỏi Internet — vừa tái hiện kịch bản thực, vừa loại trừ rủi ro pháp lý. Nhóm đi theo hướng Blue Team: mỗi kịch bản theo chuỗi tấn công có kiểm soát rồi đến phát hiện và phòng thủ, chứ không dừng ở trình diễn khai thác. Mẫu Mirai dùng để phân tích đã được vô hiệu hóa khả năng kết nối máy chủ điều khiển và không bao giờ được thực thi. Em cũng nêu rõ giới hạn ngay từ đầu để bảo đảm trung thực: lab đơn giản hơn thực tế vì thiếu NAT và tường lửa doanh nghiệp; phân tích tĩnh không giải mã hết phần mã hóa XOR; và phạm vi tập trung vào Pi 3B+ với 4B. Với nền tảng đó, em vào Phần một.

## Slide 6 — 1

Sang phần một, em đi vào kiến trúc của Raspberry Pi: từ phần cứng, trình tự khởi động cho đến hệ điều hành. Mục tiêu của phần này là chỉ ra rằng nhiều rủi ro bảo mật bắt nguồn ngay từ các lựa chọn thiết kế nền tảng, chứ không chỉ là lỗi phần mềm phát sinh về sau.

## Slide 7 — PHẦN 1 · KIẾN TRÚC

Slide trình bày các thế hệ Raspberry Pi từ 2012 đến 2023 cùng các vấn đề bảo mật. Thứ nhất, kiến trúc được nâng cấp từ ARMv6 trên Pi 1 lên ARMv8.2-A trên Pi 5, bổ sung các tính năng bảo mật phần cứng như TrustZone — nhưng chúng chỉ phát huy tác dụng khi được kích hoạt và cấu hình đúng, mà mặc định thường không bật. Thứ hai, từ Pi 3 năm 2016, Wi-Fi và Bluetooth được tích hợp sẵn và bật mặc định kể cả khi không dùng, làm tăng bề mặt tấn công không dây. Thứ ba, mãi đến Pi 5 mới có Secure Boot — cơ chế chỉ cho phép khởi động phần mềm đã được ký xác thực — nhưng vẫn là tùy chọn và mặc định tắt. Hệ quả quan trọng nhất là: phần lớn thiết bị đang vận hành ngoài thực tế là Pi 3 và Pi 4, hoàn toàn không có cơ chế bảo vệ chuỗi khởi động ở cấp phần cứng. Nếu hội đồng muốn, em có thể giải thích sâu hơn về vùng nhớ OTP và cách Secure Boot hoạt động ở Pi 5.

## Slide 8 — PHẦN 1 · KIẾN TRÚC

Tất cả các thế hệ Raspberry Pi đều dựng trên chip hệ thống - SoC - của Broadcom, tích hợp GPU VideoCore và CPU ARM trên cùng một đế. Tuy nhiên, GPU VideoCore là bộ xử lý được kích hoạt đầu tiên khi cấp nguồn, sau đó mới đến CPU . GPU chịu trách nhiệm khởi tạo phần cứng, đọc cấu hình và nạp firmware, rồi mới trao quyền cho CPU ARM. Điều này đảo ngược hoàn toàn mô hình quen thuộc trên máy tính x86, nơi CPU cùng BIOS hoặc UEFI là nơi bảo đảm sự an toàn cho hệ thống. Trên Pi, việc này phụ thuộc vào firmware GPU - là mã nguồn đóng của Broadcom, khó kiểm toán độc lập. Và trình tự khởi động của Pi tạo ra một điểm yếu cấu trúc.

## Slide 9 — PHẦN 1 · KIẾN TRÚC

Trên các dòng Pi 1 đến Pi 4 - chiếm phần lớn thiết bị đang vận hành - các file firmware GPU như start.elf và fixup.dat không được ký số và không qua bất kỳ bước xác thực toàn vẹn nào. Chúng nằm trên phân vùng boot định dạng FAT32, vốn không hỗ trợ phân quyền kiểu Unix, nên bất kỳ tiến trình nào mount được phân vùng này đều có thể đọc và ghi firmware. Sơ đồ bên phải mô tả năm bước khởi động; mũi tên đỏ chỉ vào giai đoạn không xác thực chữ ký. Hệ quả trực tiếp là một cuộc tấn công vật lý rất đơn giản: tháo thẻ SD, gắn sang máy khác, thay kernel hoặc start.elf bằng phiên bản độc hại rồi lắp lại. Thiết bị sẽ thực thi mã độc ở giai đoạn sớm nhất, trước khi nhân Linux và mọi cơ chế bảo mật cấp OS được nạp, và không để lại dấu vết trong hệ điều hành. Trường hợp kẻ tấn công đã có quyền sudo, họ còn có thể ghi đè phân vùng boot để cài cắm mã độc tồn tại lâu dài ngay cả sau khi cài lại hệ điều hành. điểm yếu kiến trúc này không thể vá hoàn toàn bằng phần mềm vì vậy việc bảo mật vật lý cho Pi là bắt buộc phải được thực hiện.

## Slide 10 — PHẦN 1 · KIẾN TRÚC

Raspberry Pi OS được xây dựng trên nền Debian Linux là một bản Linux ổn định và bảo mật. Tuy nhiên, mục tiêu thiết kế của Raspberry Pi là dễ sử dụng hơn là bảo mật. Vì vậy nhiều giao diện và dịch vụ được kích hoạt sẵn để hỗ trợ người dùng mới. Điều này vô tình làm tăng bề mặt tấn công của hệ thống. Ví dụ, giao diện UART cho phép truy cập trực tiếp qua cổng serial nếu kẻ tấn công tiếp cận được thiết bị. Các cơ chế quảng bá dịch vụ mạng giúp thuận tiện cho quản trị nhưng cũng hỗ trợ quá trình trinh sát cho attacker. Wi-Fi và Bluetooth tích hợp sẵn mở thêm các điểm tấn công không dây. Ngoài ra, việc không tự động cài đặt bản vá bảo mật khiến nhiều thiết bị tồn tại lỗ hổng trong thời gian dài. Mặc dù Raspberry Pi Foundation đã loại bỏ tài khoản mặc định “pi” từ năm 2022, các rủi ro liên quan đến mật khẩu yếu và cấu hình SSH chưa an toàn vẫn còn phổ biến.

## Slide 11 — PHẦN 1 · KIẾN TRÚC

Bề mặt tấn công của Raspberry Pi có thể được chia thành bốn lớp, mỗi lớp đều có thể trở thành điểm khởi đầu cho một chuỗi tấn công hoàn chỉnh. Lớp firmware nằm gần phần cứng nhất, ít khi bị khai thác hơn nhưng nếu bị xâm phạm sẽ cho phép attacker kiểm soát ở mức sâu nhất. Lớp hệ điều hành là nơi xuất hiện nhiều lỗ hổng phần mềm và sai sót cấu hình nhất. Lớp giao thức mạng liên quan đến các giao tiếp có dây và không dây, nơi các cuộc tấn công từ xa thường được thực hiện. Cuối cùng là lớp giao tiếp ngoại vi như GPIO, UART và USB. Đây là lớp thường bị bỏ qua trong các đánh giá bảo mật truyền thống nhưng lại rất quan trọng đối với thiết bị IoT triển khai trong thực tế.

## Slide 12 — 2

Phần hai là trọng tâm kỹ thuật của báo cáo: phân tích lỗ hổng bảo mật. Em sẽ trình bày một khung phân loại gồm bốn lớp kỹ thuật, kết hợp mô hình tác nhân theo vị trí truy cập và mô hình STRIDE, sau đó đi qua tám lỗ hổng CVE tiêu biểu mà nhóm em đã xác minh độc lập trên cơ sở dữ liệu NVD.

## Slide 13 — PHẦN 2 · LỖ HỔNG

Tiếp theo là phần lỗ hổng, có thể chia theo một khung phân loại ba chiều. Chiều thứ nhất là mô hình bốn lớp kỹ thuật minh họa ở sơ đồ bên trái. Chiều thứ hai là mô hình tác nhân tấn công phân theo vị trí truy cập: tác nhân vật lý tiếp cận trực tiếp bo mạch, đe dọa chủ yếu lớp firmware và ngoại vi; tác nhân trong tầm phủ sóng không dây hoặc cùng phân đoạn mạng, đe dọa lớp giao thức; và tác nhân từ xa khai thác dịch vụ lắng nghe qua Internet, đe dọa lớp hệ điều hành. Chiều thứ ba là đối chiếu với mô hình STRIDE, cho thấy các lỗ hổng trong báo cáo phân bố ở năm nhóm: giả mạo trên bus I2C/SPI, can thiệp ghi đè firmware GPU, lộ thông tin qua nghe lén Telnet và mDNS, leo thang đặc quyền qua một loạt lỗ hổng kernel, và từ chối dịch vụ qua botnet. Việc này giúp hỗ trợ xây dựng các biện pháp phòng thủ phù hợp cho từng lớp của Raspberry Pi.

## Slide 14 — PHẦN 2 · LỖ HỔNG

Firmware là lớp đầu tiên được thực thi trong quá trình khởi động và cũng là lớp có mức đặc quyền cao nhất. Trên Raspberry Pi 1 đến Pi 4, firmware không được xác thực bằng chữ ký số trước khi thực thi, tạo ra một điểm yếu mang tính kiến trúc. Nghiên cứu xác định hai vector tấn công chính. Thứ nhất là tấn công vật lý thông qua việc thay đổi nội dung trên thẻ SD. Thứ hai là tấn công sau khi hệ thống đã bị xâm nhập, trong đó kẻ tấn công sử dụng quyền quản trị để sửa đổi firmware nhằm duy trì hiện diện lâu dài. Điểm chung của hai vector này là đều cho phép kiểm soát hệ thống ở mức thấp hơn hệ điều hành. Mặc dù Raspberry Pi 5 đã hỗ trợ Secure Boot, phần lớn thiết bị đang vận hành trong thực tế vẫn là Pi 3 và Pi 4, khiến rủi ro tại lớp firmware tiếp tục là một vấn đề đáng quan tâm trong các hệ thống IoT hiện nay.

## Slide 15 — PHẦN 2 · LỖ HỔNG

CVE-2021-38759 là ví dụ điển hình cho việc một quyết định thiết kế có thể tạo ra rủi ro bảo mật nghiêm trọng. Điểm đặc biệt là đây không phải lỗi bộ nhớ hay lỗi lập trình truyền thống, mà xuất phát từ việc Raspberry Pi OS được phát hành với tài khoản mặc định “pi” và mật khẩu mặc định “raspberry”. Nếu người dùng không thay đổi cấu hình này, thiết bị có thể bị truy cập từ xa thông qua SSH. Ngoài ra, tài khoản “pi” còn được cấp quyền sudo với cơ chế NOPASSWD, làm giảm đáng kể rào cản giữa quyền người dùng và quyền quản trị. Từ góc độ quản trị rủi ro, đây là ví dụ cho thấy một cấu hình mặc định không an toàn có thể tạo ra mức độ ảnh hưởng tương đương nhiều lỗ hổng kỹ thuật nghiêm trọng khác. Trường hợp này cũng minh họa rõ sự đánh đổi giữa tính thuận tiện khi triển khai và yêu cầu bảo mật trong các hệ thống IoT quy mô lớn.

## Slide 16 — PHẦN 2 · LỖ HỔNG

Trong lớp hệ điều hành, bốn lỗ hổng leo thang đặc quyền nổi bật gồm Dirty COW, PwnKit, Baron Samedit và Dirty Pipe. Đây đều là các lỗ hổng cho phép chuyển từ quyền người dùng thông thường lên quyền root. các CVE này có đặc điểm chung là kẻ tấn công phải có được một điểm truy cập ban đầu vào hệ thống trước khi khai thác được lỗ hổng. Ví dụ, một cấu hình SSH yếu có thể cung cấp quyền truy cập ban đầu, sau đó tận dụng lỗ hổng leo thang đặc quyền được sử dụng để giành quyền root.

## Slide 17 — PHẦN 2 · LỖ HỔNG

Tiếp theo về giao thức và cấu hình không an toàn. SSH bản thân là một giao thức được thiết kế với mức độ bảo mật cao và hiện vẫn là chuẩn quản trị từ xa phổ biến nhất trên Linux. Tuy nhiên, nếu sử dụng mật khẩu yếu hoặc mật khẩu mặc định thì toàn bộ lợi ích của cơ chế mã hóa gần như bị vô hiệu hóa. Đây là lý do nhiều botnet IoT, bao gồm Mirai, không cần khai thác lỗ hổng trong SSH mà chỉ cần khai thác sai sót trong cấu hình xác thực. Vì vậy trên Raspberry nên thay đổi phương thức xác thực sang khóa công khai, vô hiệu hóa đăng nhập root và triển khai fail2ban sẽ mang lại hiệu quả bảo mật cao hơn nhiều so với việc chỉ tập trung tìm kiếm các CVE mới.

## Slide 18 — PHẦN 2 · LỖ HỔNG

Trong nhóm giao thức không dây, Bluetooth làm phát sinh  hai lỗ hổng tiêu biểu. BlueBorne, định danh CVE-2017-1000251, là lỗi tràn bộ đệm stack khi xử lý gói cấu hình L2CAP trong BlueZ - stack Bluetooth tích hợp trong nhân Linux, ảnh hưởng kernel từ 2.6.32 đến 4.13.1, bao gồm cả các phiên bản Pi OS trên Pi 3 và 3+ trước bản vá tháng chín năm 2017. Điểm nguy hiểm đặc biệt là khai thác không cần ghép đôi: bất kỳ thiết bị nào bật Bluetooth trong tầm sóng đều có thể bị tấn công mà người dùng không thao tác gì. KNOB, định danh CVE-2019-9506, ở mức 8.1 HIGH, là lỗ hổng giao thức cho phép kẻ tấn công ở vị trí trung gian buộc hai thiết bị thương lượng khóa mã hóa entropy cực thấp, chỉ một byte, rồi brute-force để giải mã toàn bộ lưu lượng. Cả hai tấn công này đều có thể sử dụng phương pháp phòng thủ đơn giản là: tắt Bluetooth nếu không thực sự cần.

## Slide 19 — PHẦN 2 · LỖ HỔNG

Tiếp theo là lớp giao thức không dây với Wi-Fi và dịch vụ khám phá mạng. KRACK, mã CVE-2017-13077, tấn công vào cơ chế bắt tay của WPA2: kẻ tấn công ở vị trí trung gian buộc thiết bị cài lại khóa đã dùng, từ đó giải mã hoặc giả mạo lưu lượng mà không cần biết mật khẩu Wi-Fi. Trên Pi, lỗ hổng tác động qua thư viện xử lý kết nối phía client. Về dịch vụ Avahi — bật mặc định để hỗ trợ truy cập kiểu raspberrypi.local — nó tiện nhưng quảng bá danh sách dịch vụ đang chạy ra toàn mạng LAN; chỉ một lệnh, kẻ tấn công liệt kê được mọi Pi cùng các dịch vụ SSH, VNC, Samba. Điểm chung: cả KRACK lẫn KNOB đều đòi hỏi kẻ tấn công ở vị trí trung gian hoặc lân cận. Do đó rủi ro lớp này phụ thuộc rất nhiều vào môi trường: Pi đặt trong mạng nội bộ phân đoạn tốt sẽ an toàn hơn nhiều so với môi trường mở hoặc công cộng.

## Slide 20 — PHẦN 2 · LỖ HỔNG

Lớp thứ tư là giao tiếp ngoại vi - nhóm vector vật lý thường bị các đánh giá bảo mật truyền thống bỏ qua vì chúng tập trung vào mối đe dọa mạng. Trong môi trường IoT thực tế, thiết bị thường lắp ở nơi có thể tiếp cận vật lý: kho thiết bị, phòng máy không khóa, hộp ngoài trời không bảo vệ - khiến các thiết bị dễ dàng bị tấn công. UART, tức serial console, là nguy hiểm nhất. Trên mọi model Pi, chỉ với một bộ chuyển đổi UART–USB giá rất thấp, kẻ tấn công có thể tương tác với quá trình khởi động của thiết bị. Ngoài ra, các giao thức I2C và SPI được thiết kế dựa trên giả định các thiết bị trên bus đều đáng tin cậy, nên không tích hợp cơ chế xác thực hay bảo vệ tính toàn vẹn dữ liệu. Trong môi trường IoT hiện đại, Một thiết bị bị chiếm quyền hoặc một thiết bị giả mạo có thể tác động đến toàn bộ hệ thống thông qua bus dùng chung. Vì vậy, khi đánh giá bảo mật Raspberry Pi, cần mở rộng phạm vi từ các mối đe dọa mạng sang cả các mối đe dọa vật lý và phần cứng.

## Slide 21 — PHẦN 2 · LỖ HỔNG

Tiếp theo về các lỗ hổng CVE trên Raspberry Pi, đa số các lỗ hổng được phân tích đều có điểm CVSS khá cao, nhưng nguyên nhân không chỉ đến từ bản thân lỗ hổng mà còn đến từ môi trường triển khai. Trong thực tế, các thiết bị IoT thường ít được cập nhật, ít được giám sát và có tuổi thọ vận hành dài hơn nhiều so với máy chủ hoặc máy trạm thông thường. Vì vậy, một lỗ hổng đã có bản vá vẫn có thể tiếp tục tồn tại trên số lượng lớn thiết bị. Phần lớn rủi ro xuất phát từ khoảng cách giữa khả năng vá lỗi về mặt kỹ thuật và việc triển khai vá lỗi trong thực tế.

## Slide 22 — 3

Phần ba là về mã độc trên nền tảng ARM/Linux, hệ điều hành cho thiết bị Raspberry Pi.

## Slide 23 — PHẦN 3 · MÃ ĐỘC

Phần lớn mã độc trên Linux dùng định dạng thực thi ELF. Mọi file ELF đều bắt đầu bằng bốn byte nhận dạng tương ứng chữ "ELF", và một trường trong phần tiêu đề cho biết kiến trúc bộ xử lý mục tiêu — ví dụ ARM 32-bit hay ARM 64-bit. Nhận diện các trường này giúp phân tích tĩnh xác định nhanh mã độc được thiết kế cho loại thiết bị nào. So với mã độc Windows, mã độc Linux/ARM có vài đặc điểm riêng: thường được biên dịch chéo ra nhiều kiến trúc để mở rộng phạm vi lây; kích thước tối ưu rất nhỏ cho thiết bị nhúng; ít dùng kỹ thuật né tránh phức tạp vì thiết bị đích thường thiếu phần mềm chống mã độc; duy trì hiện diện bằng các thành phần sẵn có của Linux như cron hay systemd; và hay dùng công cụ đóng gói như UPX để giảm kích thước và gây khó cho phân tích. Tóm lại, mã độc Linux/ARM đã phát triển thành một hệ sinh thái riêng tối ưu cho IoT — nổi tiếng nhất là họ Mirai.

## Slide 24 — PHẦN 3 · MÃ ĐỘC

Mirai là họ mã độc botnet IoT có tác động lớn nhất từ trước đến nay. Được phát hiện tháng tám 2016, nó trở nên khét tiếng khi mã nguồn bị công bố công khai tháng mười cùng năm — dẫn tới hàng trăm biến thể về sau. Cơ chế lây nhiễm rất đơn giản: Mirai quét Internet tìm thiết bị mở dịch vụ Telnet, rồi thử đăng nhập bằng một danh sách 62 cặp tài khoản/mật khẩu mặc định phổ biến — trong đó có cặp "pi/raspberry" từng dùng mặc định trên Raspberry Pi OS. Đăng nhập thành công thì thiết bị tải về và chạy phiên bản Mirai phù hợp kiến trúc của nó. Về tổ chức, Mirai có ba thành phần: máy chủ điều khiển C2 ra lệnh cho cả mạng botnet; Loader đẩy mã độc lên thiết bị mới; và Bot là các thiết bị IoT đã bị chiếm quyền, chờ lệnh để thực hiện tấn công từ chối dịch vụ phân tán DDoS. Điểm cốt lõi cho người mới: Mirai không khai thác lỗ hổng tinh vi nào — nó chỉ tận dụng mật khẩu mặc định mà người dùng quên đổi.

## Slide 25 — PHẦN 3 · MÃ ĐỘC

Ngoài Mirai, slide nhắc hai họ mã độc khác để thấy sự tiến hóa. Darlloz, công bố năm 2013, được xem là một trong những worm IoT đa kiến trúc đầu tiên — hỗ trợ ARM, MIPS, PowerPC, x86. Khác Mirai, nó không dò mật khẩu mặc định mà khai thác trực tiếp một lỗ hổng thực thi mã từ xa trong PHP, nhắm vào router, đầu thu và camera IP. Đặc điểm thú vị là Darlloz còn tự vá lỗ hổng trên thiết bị đã nhiễm để ngăn mã độc cạnh tranh — tức chiếm độc quyền tài nguyên thiết bị. Họ thứ hai là Tsunami, còn gọi là Kaiten, tiêu biểu cho thế hệ botnet điều khiển tập trung qua kênh chat IRC. Hạn chế lớn của nó là toàn bộ botnet phụ thuộc vào một vài máy chủ điều khiển trung tâm, nên chỉ cần triệt phá hạ tầng đó là vô hiệu hóa được phần lớn mạng. Chính hạn chế này thúc đẩy thế hệ tiếp theo phi tập trung, tiêu biểu là Mozi — slide sau.

## Slide 26 — PHẦN 3 · MÃ ĐỘC

Mozi, phát hiện năm 2019, đại diện cho giai đoạn hoàn thiện nhất của botnet IoT hiện đại. Khác các thế hệ trước phụ thuộc máy chủ điều khiển trung tâm, Mozi dùng mô hình mạng ngang hàng P2P: mỗi thiết bị bị nhiễm vừa là nạn nhân vừa là một nút duy trì hạ tầng điều khiển. Vì vậy, loại bỏ một vài nút không làm sập được botnet — độ bền và khả năng phục hồi tăng lên đáng kể, khiến cách phòng thủ truyền thống "đánh sập máy chủ C2" gần như vô hiệu. Mozi còn xác thực lệnh điều khiển bằng chữ ký số để chỉ chủ nhân thật mới ra được lệnh, và chủ động chiếm các dịch vụ như Telnet để ngăn mã độc cạnh tranh. Theo IBM X-Force, lưu lượng liên quan đến Mozi từng chiếm tới 90% hoạt động botnet IoT quan sát được trong năm 2020 — con số cho thấy mức độ thống trị của mô hình P2P.

## Slide 27 — PHẦN 3 · MÃ ĐỘC

Slide tổng hợp về 4 loại botnet IoT Kaiten, Darlloz, Mirai và Mozi. Phân tích cho thấy chiến thuật phát triển mã độc không nằm ở kỹ thuật khai thác lỗ hổng mà nằm ở kiến trúc điều khiển. Tsunami phụ thuộc vào IRC trung tâm nên dễ bị triệt phá. Darlloz tập trung vào khả năng tự lan truyền. Mirai mở rộng quy mô lây nhiễm và tạo ra những botnet cực lớn. Mozi tiếp tục giải quyết bài toán còn lại bằng cách loại bỏ hoàn toàn điểm điều khiển trung tâm thông qua kiến trúc P2P. Kết quả là các biện pháp phòng thủ truyền thống dựa trên việc loại bỏ máy chủ điều khiển ngày càng kém hiệu quả. Từ đó chiến lược phòng thủ hiệu quả là: thay vì chỉ bảo vệ hạ tầng mạng, cần tập trung nhiều hơn vào việc cấu hình chặt chẽ và giám sát ngay trên thiết bị IoT. Nếu thiết bị không bị chiếm quyền từ đầu thì botnet sẽ không thể hình thành, bất kể kiến trúc điều khiển hay botnet nguy hiểm ra sao.

## Slide 28 — PHẦN 3 · MÃ ĐỘC

Từ kết quả phân tích, nhóm đề xuất bảo vệ Raspberry Pi theo nguyên tắc phòng thủ theo chiều sâu — nhiều lớp kiểm soát kết hợp, để một điểm thất bại đơn lẻ không làm sập toàn hệ thống. Nguyên tắc ưu tiên là làm trước những biện pháp chi phí thấp mà hiệu quả cao, phù hợp với các hệ thống IoT eo hẹp nguồn lực quản trị. Ví dụ: bật cập nhật bảo mật tự động giúp đóng nhanh các lỗ hổng đã có bản vá như Dirty COW, PwnKit hay Dirty Pipe; chuyển SSH sang xác thực bằng khóa công khai thay cho mật khẩu giải quyết trực tiếp rủi ro mật khẩu yếu và tấn công dò quét. Tuy nhiên, em nêu rõ giới hạn: ngay cả khi làm đầy đủ, vẫn còn rủi ro tồn dư ở lớp firmware trên Pi 1 đến Pi 4 do thiếu Secure Boot phần cứng, và rủi ro tấn công chuỗi cung ứng phần mềm — những thứ nằm ngoài tầm của hardening tại thiết bị.

## Slide 29 — 4

Phần bốn hiện thực hóa toàn bộ lý thuyết phía trên bằng thực nghiệm trong một lab cách ly hoàn toàn khỏi Internet. Nhóm em dựng một chuỗi tấn công và phòng thủ gồm ba kịch bản nối tiếp nhau: thu thập thông tin, khai thác SSH, và phân tích tĩnh một mẫu mã độc thật.

## Slide 30 — PHẦN 4 · THỰC NGHIỆM

Trước khi đi vào ba kịch bản, em xin mô tả nhanh mô hình thực nghiệm để hội đồng — và các bạn lần đầu tiếp cận — hình dung được bố cục. Sơ đồ bên trái có ba thành phần: máy tấn công chạy Kali Linux 2024 trong VMware, một switch mạng LAN biệt lập ở giữa, và thiết bị nạn nhân là Raspberry Pi 4B chạy Pi OS Bookworm Lite 64-bit. Điểm quan trọng nhất, em ghi bằng chữ đỏ ở dưới, là toàn bộ mạng này hoàn toàn không kết nối Internet. Đây là quyết định có chủ đích: vừa tái hiện được kịch bản tấn công thực tế, vừa loại trừ rủi ro pháp lý và an ninh khi thao tác với công cụ tấn công và mẫu mã độc thật. Để hội đồng tiện theo dõi, em xin nói rõ vai trò: trong các slide tiếp theo, "kẻ tấn công" luôn là máy Kali, còn "nạn nhân" là Pi 4B đã được cố tình đặt mật khẩu yếu để mô phỏng lỗ hổng CVE-2021-38759. Bốn công cụ chính sẽ dùng là Nmap để dò quét, Hydra để bẻ mật khẩu, Wireshark để bắt và phân tích gói tin, và binutils để phân tích mã độc tĩnh. Với mô hình đó, em bắt đầu kịch bản một: thu thập thông tin.

## Slide 31 — PHẦN 4 · THỰC NGHIỆM

Giai đoạn đầu của chuỗi tấn công là thu thập thông tin — bước mà kẻ tấn công xác định mục tiêu và bề mặt khai thác. Với các bạn mới làm quen an toàn thông tin, đây giống như việc kẻ trộm đi một vòng quan sát ngôi nhà trước khi hành động. Lệnh trên cùng là Nmap thực thi từ máy Kali, với tùy chọn phát hiện phiên bản dịch vụ và nhận dạng hệ điều hành trên năm cổng trọng yếu. Kết quả cho thấy ba thông tin quan trọng: cổng 22 SSH đang mở với phiên bản OpenSSH 9.2p1 trên Debian; cổng 5353 mDNS mở do dịch vụ Avahi; và dấu vân hệ điều hành xác định đây là thiết bị kiến trúc ARM chạy Linux. Bấy nhiêu là đủ để kẻ tấn công biết thiết bị đang chạy SSH với xác thực mật khẩu, và Avahi đang quảng bá dịch vụ ra mạng. Điểm em muốn nhấn mạnh là tốc độ: Nmap hoàn thành quét trong chưa đầy mười giây. Con số này cho thấy việc lập bản đồ một thiết bị IoT mất rất ít thời gian và công sức. Sau khi đã biết SSH mở, kịch bản hai tiến hành khai thác chính cổng đó.

## Slide 32 — PHẦN 4 · THỰC NGHIỆM

Kịch bản hai có hai phần: bẻ mật khẩu và phân tích lưu lượng. Slide này tập trung vào phần phân tích lưu lượng bằng Wireshark, là phần em cho là trực quan nhất cho người mới. Sau khi có quyền truy cập, nhóm bắt gói tin của hai phiên kết nối để so sánh. Với phiên SSH, Wireshark cho thấy toàn bộ payload sau giai đoạn bắt tay đều được mã hóa, nên kẻ nghe lén thụ động không đọc được nội dung. Để đối chiếu, nhóm mở thêm một phiên Telnet song song — và đây là phần em đánh dấu đỏ — Wireshark hiển thị rõ toàn bộ nội dung phiên Telnet, gồm cả tên đăng nhập, mật khẩu và mọi lệnh, ở dạng văn bản thuần hoàn toàn. Với người mới, đây là minh chứng mạnh nhất cho một nguyên tắc cốt lõi: giao thức cũ như Telnet truyền mọi thứ "trần trụi" trên đường dây, còn SSH thì mã hóa. Đó là lý do trong môi trường IoT có thể bị nghe lén, dùng SSH thay Telnet là bắt buộc. Tiếp theo, em trình bày phần bẻ mật khẩu bằng Hydra.

## Slide 33 — PHẦN 4 · THỰC NGHIỆM

Đây là phần bẻ mật khẩu của kịch bản hai. Trong thực nghiệm, Pi được cố tình đặt mật khẩu "raspberry" để mô phỏng lỗ hổng cấu hình mặc định CVE-2021-38759, và chưa cài fail2ban. Lệnh trên cùng chạy công cụ Hydra với bộ từ điển rockyou — một danh sách mật khẩu phổ biến rất nổi tiếng trong giới bảo mật. Ảnh chụp màn hình thật từ máy Kali cho thấy kết quả: dòng đỏ phía dưới ghi rõ host 172.31.98.252, login "pi", password "raspberry", và thông báo bẻ thành công. Vì mật khẩu này nằm gần đầu wordlist nên Hydra tìm ra trong chưa đầy hai phút. Sau khi có thông tin đăng nhập, kẻ tấn công vào được SSH và chỉ cần gõ lệnh sudo để lên quyền root ngay lập tức — đúng như dự đoán từ cấu hình NOPASSWD, tức quyền quản trị không cần nhập lại mật khẩu, mà nhóm đã phân tích ở Phần hai. Đây là điểm em muốn người mới ghi nhớ: một mật khẩu mặc định yếu có thể vô hiệu hóa toàn bộ lớp bảo mật phía sau. Slide tiếp theo cho thấy biện pháp phòng thủ đơn giản chặn được chính cuộc tấn công này.

## Slide 34 — PHẦN 4 · THỰC NGHIỆM

Slide này chuyển sang góc nhìn phòng thủ — đúng tinh thần Blue Team của báo cáo. Sau khi đã thấy Hydra bẻ mật khẩu dễ dàng, câu hỏi đặt ra là: làm sao chặn? Nhóm cài fail2ban, một công cụ miễn phí có sẵn, rồi chạy lại đúng cuộc tấn công Hydra. Ảnh chụp cho thấy kết quả rõ rệt: fail2ban theo dõi nhật ký đăng nhập, phát hiện hàng loạt lần thử sai liên tiếp từ cùng một địa chỉ IP, và tự động chặn IP đó — dòng "Banned IP list" hiển thị địa chỉ 192.168.88.11 của máy Kali đã bị cấm. Nói cách khác, cuộc tấn công bị chặn đứng mà không cần thiết bị giám sát đắt tiền nào. Thông điệp cho người mới: phòng thủ hiệu quả không nhất thiết phải phức tạp; một biện pháp công sức thấp như fail2ban đã nâng đáng kể rào cản trước các cuộc dò quét mật khẩu. Tuy nhiên, fail2ban chỉ chặn được tấn công qua mạng. Slide tiếp theo cho thấy một vector hoàn toàn khác mà fail2ban không thể đụng tới: tấn công vật lý.

## Slide 35 — PHẦN 4 · THỰC NGHIỆM

Slide này là ảnh chụp thật thiết bị Raspberry Pi của nhóm trong lab, với thẻ nhớ microSD được rút ra. Em đưa ảnh này để minh họa một vector tấn công mà nhiều người mới thường bỏ qua: tấn công vật lý. Như đã phân tích ở Phần một, trên Pi 1 đến Pi 4 phần khởi động nằm trên thẻ SD định dạng FAT32, không được ký số và không có xác thực toàn vẹn. Hệ quả rất đơn giản: chỉ cần tiếp cận được thiết bị, kẻ tấn công có thể rút thẻ SD ra, cắm sang máy khác, đọc hoặc sửa toàn bộ nội dung, rồi lắp lại. Đây là lý do em muốn nhấn mạnh với các bạn mới: bảo mật không chỉ là chuyện phần mềm và mạng. Một thiết bị IoT đặt ở nơi ai cũng chạm tới được — kho, hộp ngoài trời, phòng máy không khóa — thì mọi biện pháp như SSH key hay fail2ban đều có thể bị vượt qua bằng cách rút thẻ. fail2ban ở slide trước không giúp được gì ở đây. Vì vậy, bảo mật vật lý cho Pi là yêu cầu bắt buộc, không phải tùy chọn.

## Slide 36 — PHẦN 4 · THỰC NGHIỆM

Đây là ảnh chụp màn hình thật của thiết bị Pi trong lab, ở thời điểm một phiên đăng nhập đã thành công. Em đưa ảnh này để hội đồng thấy môi trường thực nghiệm là thật chứ không phải mô phỏng trên giấy. Trên màn hình có thể thấy hệ thống chạy Debian GNU/Linux 12 trên nhân Raspberry Pi 64-bit, địa chỉ IP của thiết bị, và dòng "Last login" cho biết phiên gần nhất đăng nhập từ địa chỉ 192.168.88.11 — chính là máy Kali tấn công. Chi tiết nhỏ này thực ra là một dấu vết điều tra số quan trọng: nhật ký đăng nhập ghi lại nguồn truy cập, và đây là manh mối đầu tiên một người phòng thủ sẽ dùng để truy ra kẻ tấn công. Với các bạn mới, em muốn liên hệ: mỗi hành động trên hệ thống Linux đều để lại dấu vết trong log, và biết đọc log là kỹ năng nền tảng của cả tấn công lẫn phòng thủ. Sau khi đã đi qua trọn vẹn kịch bản khai thác SSH, em chuyển sang kịch bản cuối: phân tích một mẫu mã độc thật.


---

# PHỤ LỤC PHẢN BIỆN (slide ẩn — dùng khi Q&A)

## Slide 37 — PHẦN 4 · THỰC NGHIỆM *(slide ẩn)*

Kịch bản cuối minh họa quy trình nhận diện mã độc ARM trên Linux bằng phân tích tĩnh — nghĩa là phân tích mà không bao giờ chạy file, để tuyệt đối an toàn. Toàn bộ thực hiện trong môi trường cách ly, và mẫu Mirai dùng ở đây đã được vô hiệu hóa từ trước. Quy trình gồm ba bước, dùng các lệnh có sẵn trên mọi máy Linux. Bước một, lệnh file xác nhận đây là một chương trình ELF 32-bit cho ARM, liên kết tĩnh nên không cần thư viện ngoài, và đã bị stripped — tức gỡ bỏ thông tin gỡ lỗi để cản trở phân tích. Bước hai, lệnh strings kết hợp grep trích ra các chuỗi đặc trưng như bin busybox, chữ MIRAI, và dev watchdog; tuy nhiên địa chỉ máy chủ điều khiển không xuất hiện ở dạng đọc được vì đã bị mã hóa XOR — đây cũng là một hạn chế của phân tích tĩnh mà nhóm nêu rõ. Bước ba, lệnh readelf đọc phần tiêu đề của file, xác nhận chữ ký nhận dạng 7f 45 4c 46 và kiến trúc ARM. Với người mới, ý chính cần nhớ là: chỉ bằng ba lệnh miễn phí, ta đã phân loại được một mẫu mã độc và rút ra các chỉ số nhận diện, mà không cần phòng lab sandbox phức tạp. Tiếp theo là đánh giá tổng hợp kết quả ba kịch bản.

## Slide 38 — PHẦN 4 · THỰC NGHIỆM *(slide ẩn)*

Slide này tổng hợp và đánh giá kết quả ba kịch bản. Bảng trên tóm tắt: kịch bản một phát hiện SSH mở và nhận dạng ARM Linux trong dưới mười giây; kịch bản hai crack mật khẩu raspberry trong dưới hai phút rồi leo root, đồng thời đối chiếu trực quan SSH mã hóa với Telnet cleartext; kịch bản ba nhận diện thành công ELF ARM và phân tích header. Từ đó, hai luận điểm chính của báo cáo được xác nhận. Thứ nhất, toàn bộ chuỗi tấn công từ thu thập thông tin đến quyền root có thể hoàn thành trong vài phút với công cụ miễn phí và kỹ năng cơ bản, trên thiết bị chưa hardening — phản ánh đúng thực tế hàng triệu thiết bị IoT triển khai với cấu hình mặc định. Thứ hai, phân tích tĩnh ELF là kỹ thuật nhận diện mã độc IoT khả thi và hiệu quả ngay cả khi không có sandbox phức tạp, đủ để phân loại mẫu và xây dựng chỉ số nhận diện. Đồng thời, để giữ tính trung thực học thuật, nhóm nêu rõ ba hạn chế ở cột phải: môi trường lab đơn giản hơn thực tế vì thiếu NAT và tường lửa doanh nghiệp; phân tích tĩnh không giải mã hết nội dung XOR mà cần phân tích động bổ sung; và mẫu Mirai đã sanitized nên không đánh giá được hành vi kết nối C2 thật. Chính những hạn chế này định hướng cho hướng nghiên cứu tiếp theo mà em trình bày ở phần kết luận.

## Slide 39 — 5

Phần cuối, em tổng kết lại các kết luận học thuật chính của báo cáo, nêu thẳng những hạn chế còn tồn tại, và đề xuất các hướng phát triển tiếp theo cho đề tài.

## Slide 40 — PHẦN 5 · KẾT LUẬN

phần lớn rủi ro trên Raspberry Pi không đến từ các kỹ thuật tấn công quá phức tạp mà đến từ cách thiết bị được thiết kế, cấu hình và vận hành trong thực tế, các cấu hình mặc định không an toàn có thể tạo ra tác động tương đương hoặc lớn hơn nhiều lỗ hổng kỹ thuật phức tạp. Đồng thời, kiến trúc khởi động của Pi 1 đến Pi 4 tạo ra một rủi ro nền tảng mà không thể giải quyết hoàn toàn bằng phần mềm. Về phía tác nhân tấn công, các botnet IoT đang phát triển theo hướng ngày càng phân tán và khó triệt phá hơn, khiến phòng thủ tại thiết bị đầu cuối trở thành yêu cầu bắt buộc. và việc khai thác nhiều điểm yếu phổ biến không cần sử dụng các kỹ năng phức tạp, vì vậy việc bảo mật Raspberry Pi cần được thực hiện từ việc quản trị vòng đời thiết bị, từ triển khai, vận hành đến cập nhật và giám sát liên tục.

## Slide 41 — PHẦN 5 · KẾT LUẬN *(slide ẩn)*

Một báo cáo nghiêm túc phải nêu rõ hạn chế của chính nó, và đây là phần em chủ động trình bày thay vì để hội đồng phải chỉ ra. Thứ nhất, môi trường thực nghiệm đơn giản hơn triển khai IoT thực tế vì thiếu NAT, tường lửa doanh nghiệp và các lớp kiểm soát mạng — nghĩa là kết quả phản ánh kịch bản cơ sở, không phải mọi điều kiện thực địa. Thứ hai, phân tích tĩnh chưa đủ chiều sâu: nó không giải mã hết phần nội dung mã hóa XOR, đặc biệt là địa chỉ C2, nên cần bổ sung phân tích động. Thứ ba, mẫu mã độc đã được sanitized nên nhóm không thể đánh giá hành vi kết nối máy chủ điều khiển thực tế. Thứ tư, phạm vi phần cứng giới hạn ở Pi 3B+ và 4B, chưa bao phủ toàn dải sản phẩm từ Pi 1 đến Pi 5. Cuối cùng, và em đánh dấu đỏ vì đây là hạn chế mang tính nguyên tắc: không biện pháp nào loại bỏ hết rủi ro: vẫn còn rủi ro tồn dư ở lớp firmware đối với Pi 1 đến 4 do thiếu Secure Boot phần cứng, và ở khả năng tấn công chuỗi cung ứng phần mềm. Việc thừa nhận các hạn chế này không làm yếu báo cáo mà ngược lại định hướng trực tiếp cho các hướng nghiên cứu tiếp theo.

## Slide 42 — PHẦN 5 · KẾT LUẬN *(slide ẩn)*

Từ các hạn chế vừa nêu, nhóm đề xuất bốn hướng phát triển. Thứ nhất, mở rộng thực nghiệm sang phân tích động trong môi trường QEMU sandbox để giải mã được địa chỉ C2 mã hóa XOR và quan sát hành vi runtime của mẫu ARM — đây là phần bổ khuyết trực tiếp cho hạn chế của phân tích tĩnh. Thứ hai, nghiên cứu hiệu quả của các giải pháp phát hiện bất thường dựa trên học máy nhẹ chạy ngay trên Raspberry Pi; hướng này đặc biệt thú vị vì nó đặt vấn đề liệu một thiết bị tài nguyên hạn chế có thể tự giám sát hay không. Thứ ba, đánh giá tác động của Secure Boot trên Pi 5 trong việc ngăn chặn các tấn công firmware — kiểm chứng xem cơ chế phần cứng mới có thực sự đóng được điểm yếu cấu trúc đã phân tích hay không. Thứ tư, phân tích so sánh mức độ bảo mật của các bản phân phối Linux thay thế như Ubuntu Server hay Kali Linux ARM so với Pi OS chính thức. Em xin nói thêm rằng hướng thứ hai về máy học là điểm em cá nhân quan tâm và thấy phù hợp để mở rộng liên ngành cho luận văn, vì nó kết nối an toàn thông tin với học máy. Cuối cùng, em xin điểm qua các tài liệu tham khảo chính.

## Slide 43 — PHẦN 5 · KẾT LUẬN *(slide ẩn)*

Slide cuối của phần nội dung liệt kê tám tài liệu tham khảo cốt lõi, chọn từ hai mươi tư tài liệu trong báo cáo đầy đủ và trình bày theo chuẩn IEEE. Em muốn nhấn mạnh tính đa dạng và thẩm quyền của nguồn: có nguồn báo chí điều tra như Krebs về Mirai; có cơ sở dữ liệu lỗ hổng chính thức là NVD cho CVE-2021-38759; có báo cáo của hãng bảo mật như Qualys cho PwnKit và Baron Samedit, IBM X-Force cho Mozi; có công trình học thuật bình duyệt tại các hội nghị hàng đầu như USENIX Security cho KNOB và ACM CCS cho KRACK; và có tài liệu chuẩn của NIST là Special Publication 800-213. Em xin khẳng định lại nguyên tắc liêm chính mà nhóm tuân thủ xuyên suốt: toàn bộ các CVE được nêu đều đã được xác minh độc lập trên NVD, và những lỗ hổng chưa có điểm chính thức thì nhóm để trạng thái Deferred thay vì gán một con số không kiểm chứng được. Đến đây em đã trình bày xong toàn bộ năm phần của báo cáo, và xin chuyển sang phần hỏi đáp.

## Slide 44 — Xin cảm ơn Thầy và Các Bạn

phần trình bày của nhóm xin hết, xin cảm ơn Thầy và các Bạn.

## Slide 45 — PHỤ LỤC PHẢN BIỆN · B1 *(slide ẩn)*

Slide B1 phục vụ câu hỏi về một CVE cụ thể ngoài danh sách chính. Em bổ sung vector CVSS đầy đủ và trạng thái bản vá cho từng lỗ hổng. Đáng chú ý, vector của CVE-2021-38759 cho thấy mọi điều kiện đều thuận lợi cho kẻ tấn công. Với Dirty Pipe, lỗ hổng xuất hiện ở kernel 5.10.92 và được vá tại 5.10.102. Hai dòng Deferred em vẫn giữ nguyên trạng thái và chỉ ghi điểm tham chiếu, đúng nguyên tắc đã trình bày.

## Slide 46 — PHỤ LỤC PHẢN BIỆN · B2 *(slide ẩn)*

Slide B2 dành cho câu hỏi về cách thực hiện, công cụ và tham số cụ thể của từng kịch bản. Em liệt kê đầy đủ lệnh và kết quả quan sát tương ứng, gồm cả các lệnh phân tích tĩnh ELF: file để xác định kiến trúc, strings kèm grep để lọc chuỗi đặc trưng, và readelf để đọc header. Nếu hội đồng muốn, em có thể giải thích chi tiết từng tham số, ví dụ tùy chọn -t 4 trong Hydra giới hạn bốn luồng song song.

## Slide 47 — PHỤ LỤC PHẢN BIỆN · B3 *(slide ẩn)*

Slide B3 trả lời câu hỏi về điểm khác biệt so với tài liệu hiện có. Em chỉ ra rằng các hướng nghiên cứu trước thường phân tán theo từng khía cạnh: nhóm về botnet IoT tập trung cơ chế lây và hạ tầng C2; nhóm về giao thức không dây đi sâu từng lỗ hổng đơn lẻ; còn tài liệu chính thức của Raspberry Pi Foundation mô tả ở góc vận hành chứ không nhằm phản biện bảo mật. Khoảng trống mà báo cáo lấp đầy, và cũng là đóng góp nhóm tự nhận, là một khung phân tích thống nhất ánh xạ đồng thời ba trục vào cùng mô hình lớp kỹ thuật, rồi liên kết trực tiếp mỗi lớp với chiến lược phòng thủ. Em nhấn mạnh đây là đóng góp tổng hợp, xây trên các nguồn đã dẫn chứ không sao chép một khung có sẵn.

## Slide 48 — PHỤ LỤC PHẢN BIỆN · B4 *(slide ẩn)*

Slide B4 dành cho câu hỏi tại sao nhóm chọn cách tiếp cận này mà không phải cách khác. Em giải thích năm quyết định phương pháp luận then chốt. Hướng Blue Team được chọn vì phù hợp mục tiêu học thuật và không đòi hỏi demo tấn công thực ra ngoài môi trường kiểm soát. Lab cách ly khỏi Internet nhằm loại trừ rủi ro pháp lý và an ninh trong khi vẫn tái hiện được kịch bản thực. Mẫu mã độc sanitized và không thực thi để bảo đảm an toàn tuyệt đối, trong khi phân tích tĩnh vẫn đủ để phân loại và dựng chỉ số nhận diện. Pi OS Bookworm Lite 64-bit được chọn vì bản headless tiệm cận môi trường IoT thực tế nhất và tận dụng đầy đủ kiến trúc ARMv8 của Pi 4B. Cuối cùng, mô hình bốn lớp kết hợp STRIDE giúp tránh liệt kê rời rạc và cho phép đánh giá có hệ thống điều kiện khai thác lẫn tác động.

## Slide 49 — PHỤ LỤC PHẢN BIỆN · B5 *(slide ẩn)*

Slide B5 phục vụ câu hỏi về thông số cụ thể của hai thiết bị thực nghiệm. Điểm em muốn nhấn mạnh là dòng cuối: cả Pi 3B+ và Pi 4B đều không hỗ trợ Secure Boot phần cứng, nên kết luận về điểm yếu firmware ở Phần một áp dụng trực tiếp cho cả hai thiết bị nhóm dùng. Khác biệt chính giữa hai dòng nằm ở hiệu năng CPU, dung lượng RAM và băng thông kết nối, đặc biệt Pi 4B có USB 3.0 và Gigabit Ethernet thật thay vì đi qua USB 2.0 như 3B+.

## Slide 50 — PHỤ LỤC PHẢN BIỆN · B6 *(slide ẩn)*

Slide B6 đào sâu hơn slide hạn chế ở phần kết luận, dành cho câu hỏi phản biện về phạm vi và tính tổng quát. Hai rủi ro tồn dư em đánh dấu đỏ vì chúng mang tính nguyên tắc. Thứ nhất, điểm yếu firmware Pi 1 đến 4 không thể vá hết bằng phần mềm vì thiếu ký số và Secure Boot, nên kịch bản tấn công vật lý qua thẻ SD vẫn khả thi dù đã hardening đầy đủ phần mềm. Thứ hai, tấn công chuỗi cung ứng phần mềm — ví dụ bản vá hoặc kho phần mềm bị nhiễm — nằm ngoài tầm kiểm soát của hardening tại thiết bị. Ngoài ra, kết quả thực nghiệm chỉ phản ánh môi trường cơ sở chứ chưa mô phỏng hệ thống doanh nghiệp đa lớp, và rủi ro của các lỗ hổng cần vị trí trung gian như KRACK hay KNOB biến thiên theo môi trường triển khai. Em nêu rõ những điều này để cho thấy nhóm hiểu giới hạn của chính kết luận.

## Slide 51 — PHỤ LỤC PHẢN BIỆN · B7 *(slide ẩn)*

Slide B7 là hardening checklist đầy đủ mười mục từ Bảng 3.2 trong báo cáo, kèm lệnh hoặc cấu hình cụ thể, dành cho câu hỏi về biện pháp triển khai thực tế. Checklist được sắp theo tỷ lệ công sức trên hiệu quả, nên mục một đến ba — về mật khẩu và SSH — là ưu tiên cao nhất vì dễ làm mà chặn được vector tấn công phổ biến nhất. Các mục sau mở rộng sang firewall, cập nhật tự động, mã hóa phân vùng bằng LUKS và giám sát log. Nếu hội đồng hỏi về một mục cụ thể, em có thể giải thích lệnh tương ứng.

## Slide 52 — PHỤ LỤC PHẢN BIỆN · B8 *(slide ẩn)*

Slide B8 cuối cùng dành cho câu hỏi về tính hiệu quả so sánh giữa các biện pháp phòng thủ. Em xếp các biện pháp theo hai trục: công sức triển khai và mức giảm rủi ro. Nhóm biện pháp công sức thấp nhưng hiệu quả rất cao gồm chuyển sang SSH key, fail2ban và cập nhật tự động — đây là nơi nên đầu tư trước. Mã hóa LUKS tốn công hơn nhưng là biện pháp duy nhất chống được tấn công vật lý trích xuất thẻ SD. Thông điệp xuyên suốt, ghi ở dòng dưới, là không biện pháp đơn lẻ nào đủ; chính vì vậy báo cáo đề xuất kết hợp nhiều lớp theo mô hình phòng thủ theo chiều sâu. Đây cũng là slide khép lại toàn bộ phụ lục phản biện.

## Slide 53 — PHỤ LỤC PHẢN BIỆN · B9 *(slide ẩn)*

Slide B9 phản ánh nội dung mới bổ sung ở mục 3.7 báo cáo v2: phát hiện mã độc dựa trên hành vi và chỉ số xâm phạm. Slide dùng khi hội đồng hỏi: ngoài chữ ký tĩnh, còn cách nào phát hiện mã độc IoT? Bảng tổng hợp IoC tiêu biểu của bốn họ mã độc cùng phương pháp phát hiện khả thi ngay trên Raspberry Pi bằng công cụ sẵn có như tcpdump, netstat, strings và fail2ban — không cần thiết bị giám sát chuyên dụng. Em nhấn mạnh hai ý: thứ nhất, các IoC này xây trực tiếp từ đặc điểm kỹ thuật đã phân tích ở Phần ba, ví dụ cổng TCP/48101 và khóa XOR 0xDEADBEEF của Mirai; thứ hai, IoC tĩnh dễ lỗi thời nên hướng phát triển là phát hiện bất thường bằng học máy nhẹ chạy trên thiết bị, đúng như hướng nghiên cứu đã nêu ở phần kết luận.
