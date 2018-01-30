# Sẽ thế nào khi?
---
## Giới thiệu
Bài viết này sẽ cố gắng trả lời câu hỏi phỏng vấn "Chuyện gì sẽ xả ra khi gõ google.com vào browser address và nhấn enter". Với bài viết, ta sẽ cố gắng giải thích chi tiết nhất có thể, không bỏ qua bất kỳ thành phần nào.
Vì đây là 1 quá trình rất phực tạp, vì vậy bài viết sẽ còn thiếu xót nhiều phần và mong chờ các bạn đóng góp.

## Nhấn phím "g"
Phần này sẽ giải thích tất cả về __bàn phím vật lý (physical keyboard )__ và __cơ chế ngắt OS (OS interrupt)__, tuy nhiên sẽ lược bỏ 1 số phần khó hiểu.

Khi bạn nhấn phím "g", browser sẽ nhận sự kiện (toàn bộ quá trình sẽ diễn ra tự động). Dựa trên thuật toán mỗi trình duyệt (browser), trạng thái trình duyệt (private/ẩn danh), các gợi ý có thể khác nhau và chúng sẽ hiện ra bên dưới __thanh URL (URL bar)__. Hầu hết các thuật toán sẽ ưu tiên các kết quá tìm thấy trong __History hoặc Bookmarks__. Trong quá trình gõ "google.com" rất nhiều sự kiện, code được thực thi cho đến khi bạn hoàn thành, sẽ có những gợi ý cung cấp lựa chọn. Có thể xuất hiện ngay lời nhắc "google.com" trong quá trình gõ.

# Nhấn phím "enter"
Tại thời điểm nhấn phím Enter từ bàn phím, 1 quá trình chuyển mạch xác định sẽ xảy ra. Có 1 dòng điện nhỏ chạy trong mạch logic của bàn phìm (keyboard), quét trạng thái mỗi phím. Quá trình thao tác, nhiễu điện sẽ xảy ra khi có hoạt động đóng mở liên tục, chuyển sự kiến xảy ra thành mã phím, trong trường hợp là mã 13 (enter).

__Bộ điều kiến bản phím (Keybroad controller)__ sau đó mã hóa key code rồi truyền tới hệ điều hành. Quá trình truyền diễn ra qua Universal Serial Bus (USB) hoặc kết nối bluetooth, trong quá khứ sử dụng kết nối PS/2 và ADB.

## Trong trường hợp sử dụng USB keyboard:
- Mạch điện của USB keyboard được cung cấp dòng 5v thông qua pin 1 kết nối từ computer USB host controller.
- Key code sinh ra được lưu bên trong bộ nhớ bàn phím, trong thanh ghi được gọi "endpoint"
- Host USB controller quét endpoint mỗi 10ms (giá trị nhỏ nhất định nghĩa bởi bàn phím), sau đó lấy keycode value được lưu trong đó. Giá trị sẽ tới USB SIE (Serial Interface Engine) để chuyển đối thành 1 hoặc nhiều gói tin USB theo giao thức USB mức thấp.
- Các packet được gửi tới bằng các tín hiểu điện tử khác nhau theo D+ và D- pins với tốc độ cao nhất 1.5mb/s, theo HID (Human Interface Device) device luôn khai báo mức "low speed device" (tuân thủ USB 2.0)
- Tín hiệu nối tiếp sẽ được giải mã tại computer host usb controller và giải thích = computer's Human Interface Device (HID) universal keyboard device driver. Giá trị của key được sẽ truyền vào OS hardware abstraction layer.

## Trong trường hợp sử dụng bàn phìm ảo (Thiết bị có màn hình cảm ứng)
- Khi user chạm tay vào màn hình cảm ứng, sẽ có 1 luồng điện nhỏ sẽ truyền vào ngón tay. Hoàn thành mạch thông qua các trường tĩnh điện của lớp dẫn diện, tạo điện áp tạm thời tại thời điểm chạm tay vào màn hình.
- Screen controller quét theo chu kỳ, khi phát hiện sẽ tạo ra ngắt, báo cáo tọa độ keypress.
- Mobile OS thông báo với app hiện hành event xảy ra trên GUI element (tức bàn phím ảo hiện tại).
- Bàn phím ảo từ sự kiến trên màn hình tạo ra các ngắt báo các các key được nhấn tới soft, ngắt thông báo cho ứng dụng hiên các key được nhấn.

## Tạo ngắt [NOT for USB keyboards]
Bàn phím gửi tín hiệu tới __interrupt request line (IRQ)__, nó được ánh xạ tới __interrupt vector (integer)__ quản lý bởi __interrupt controller__. CPU sử dụng __Interrupt Descriptor Table (IDT)__ để ánh sạ __interrupt vectors__ tới hàm (interrupt handlers), được cung cấp bởi kernel. Khi interrupt đơn, CPU đánh chỉ số IDT với interrupt vector và chạy xử lý thích hợp.

## (Trên Windows) WM_KEYDOWN message gửi tới app
- HID truyền __sự kiện keydown__ tới __KBDHID.sys driver__ (convert HID sử dụng trong scancode). Trong trường hợp, scan code là VK_RETURN (0x0D). KBDHID.sys driver interfaces với KBDCLASS.sys (keyboard class driver).
- Driver này sẽ chịu trách nhiệm xử lý tất cả keyboard và keypad nhập trong secure manner.  Sau đó Driver sẽ gọi Win32K.sys (sau khi có thể truyền message thông qua 3rd party keyboard filters đã được cài đặt). Tất cả xảy ra trong kernel mode.
- Win32K.sys phát hiện window nào đang active thông qua GetForegroundWindow() API. API cung cấp window xử lý hiện tại của browser là address box. Sau đó Main Windows "message pump" gửi SendMessage(hWnd, WM_KEYDOWN, VK_RETURN, lParam).
- lParam là bitmask chỉ ra các thông tin thêm về phím nhấn: repeat count (0 trong trường hợp),  actual scan code (can be OEM dependent, but generally wouldn't be for VK_RETURN), các key mở rộng (e.g. alt, shift, ctrl).
- Windown SendMessage API  tập trung tính năng add message tới queue cho nhưng window handle (hWnd) cụ thể. Sau đó main message processing function (called a WindowProc) gán tới HWnd được gọi trong order to process each message in the queue.
- Window (hWnd) là hoạt động thực sự điều khiến chỉnh sửa Windowc trong trường hợp có message handler cho WM_KEYDOWN message. This code looks within the 3rd parameter that was passed to SendMessage (wParam) and, because it is VK_RETURN knows the user has hit the ENTER key.

## (Trên GNU/Linux) Xorg server lắng nghe keycodes
Khi graphical X server được sử dụng, X sẽ sử dụng trình điều kiển sự kiện chung endev để thu được sự kiến nhấn. Ánh xạ lại các keycode tơi scancode, quá trình được thực hiện bởi X server với các qua tắc cụ thể.

## Phân tích URL
Trình duyệt quan tâm thông tin sau trong URL
> Protocol "http"

Sử dụng 'Hyper Text Transfer Protocol'
> Resource "/"

Tìm lại trang chính Index

## Sẽ coi là URL hoặc tìm kiếm thay thế?
Trong trường hợp không chỉ định giao thức hoặc nhập domain gửi cho browser xử lý (chỉ đơn gian nhập text vào address box), trình duyệt sẽ sử dụng web search engine.

## Chuyển đổi non-ASCII Unicode characters trong hostname
- Browser kiếm tra các ký tự trong hostname có nằm trong a-z, A-Z, 0-9, -, or .
- Nếu hostname là google.com thì hoạt động sẽ không diễn ra.

## Kiểm tra HSTS list
- Browser kiểm tra "preloaded HSTS (HTTP Strict Transport Security)" list – list website chỉ sử dụng https.
- Nếu web nằm trong list, browser sẽ gửi request https thay vì http (và ngược lại).
> Web có thể vẫn sử dụng HSTS policy mà không nằm trong HSTS list. Http request đâu tiên đến web sẽ nhận lại yêu cầu sử dụng https request. Tuy nhiên có lỗ hổng bảo mật trong quá trình này (downgrade attack), đó là lý do HSTS list được thêm vào trong modern web browser.

## DNS lookup
- Browser check nếu domain trong cache (to see the DNS Cache in Chrome, go to chrome://net-internals/#dns).
- Nếu ko tìm thấy, browser gọi gethostbyname library function (tùy theo OS) để tìm kiếm
- Gethostbyname kiếm tra host name có thể phân giải bằng local host trước khi phân giải thông qua DNS
- Nếu gethostbyname không được cache cũng và không tìm thầy trong host local, nó sẽ request tới DNS server được cấu hình trong network stack. Thông thương sẽ là __ISP caching DNS server__
- Nếu DNS server = subnet trong network library => sẽ sử dụng tiến trình ARP cho DNS server
- Nếu DNS server khác subnet, network library sẽ sử dụng ARP process cho default gateway IP

## Tiền trình ARP
- Để gửi ARP broadcast network stack lib, yêu cầu target IP address to lookup. Nó cần biết MAC address của interface sẽ gửi ARP broadcast.
- ARP cache đầu tiên check ARP entry cho target IP. Nếu nó được cache, lib func trả lại target IP = mac
- Nếu entry không nằm trong ARP cache:
 - Route table tìm kiếm, kiếm tra target ip address có nằm trong subnet trên local router. Nếu có, lib sử dụng interface liên kết vơi subnet đó, nếu ko lib sử dụng interface có subnet của default gateway
 - Mac address lựa chọn network interface cho hoạt động tìm kếm
 - net lib gửi tới layer 2 (data link layer trong OSI model) arp request

>Sender MAC: interface:mac:address:here
>Sender IP: interface.ip.goes.here
>Target MAC: FF:FF:FF:FF:FF:FF (Broadcast)
>Target IP: target.ip.goes.here
