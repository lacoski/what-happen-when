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
- Browser kiểm tra domain trong cache trình duyệt (to see the DNS Cache in Chrome, go to chrome://net-internals/#dns).
- Nếu ko tìm thấy, browser sẽ gọi gethostbyname library function (tùy theo OS) để tìm kiếm
- Gethostbyname kiếm tra host name có thể phân giải bằng local host trước khi phân giải thông qua DNS
- Nếu gethostbyname không được cache cũng và không tìm thầy trong host local, nó sẽ request tới DNS server được cấu hình trong network stack. Thông thương sẽ là __ISP caching DNS server__
- Nếu DNS server = subnet trong network library => sẽ sử dụng tiến trình ARP cho DNS server
- Nếu DNS server khác subnet, network library sẽ sử dụng ARP process cho default gateway IP

## Tiền trình ARP
- Để gửi ARP broadcast network stack lib, yêu cầu địa chỉ Ip xác định cho hoạt động tìm kiếm. Nó cũng cần biết MAC address của interface để gửi ra ARP broadcast.
- ARP cache đầu tiên kiểm tra ARP entry các IP xác định. Nếu nó được cache, lib func trả lại target IP = mac
- Nếu entry không nằm trong ARP cache:
 - Route table sẽ tìm kiếm, kiếm tra target ip address có nằm trong subnet trên local router. Nếu có, lib sử dụng interface liên kết vơi subnet đó, nếu ko lib sử dụng interface subnet của default gateway
 - Mac address lựa chọn network interface cho hoạt động tìm kếm
 - net lib gửi tới layer 2 (data link layer trong OSI model) arp request

```python
Sender MAC: interface:mac:address:here
Sender IP: interface.ip.goes.here
Target MAC: FF:FF:FF:FF:FF:FF (Broadcast)
Target IP: target.ip.goes.here
```
Dựa trên loại hardware giữa pc và router
Kết nối trực tiếp:
- Nếu pc trực tiếp kết nối router, router trả lại ARP reply

HUB:
- Nếu pc kết nối hub, hub broadcast arp request tới tất cả port khác. Nếu router được kết nối cung dây, nó sẽ trả loại ARP Reply

Switch:
- Nếu PC kết nối switch, switch sẽ kiểm tra local CAM/MAC table, kiếm tra port có MAC đang tìm. Nếu switch ko tìm thấy mục chứa MAC Addr, nó sẽ rebroadcast ARP request tới các port khác
- switch tìm thấy mục trong MAC/CAM table sẽ gửi ARP request to port có địa chứa MAC address cần tìm
- Nếu router cung dây, nó sẽ đáp trả ARP reply

ARP Reply
```python
Sender MAC: target:mac:address:here
Sender IP: target.ip.goes.here
Target MAC: interface:mac:address:here
Target IP: interface.ip.goes.here
```
Bây h net lib có IP của our DNS server hoặc gw mặc định, nó có thể tiếp tục xử lý DNS:
- Port 53 được mở gửi UDP request tới DNS sr (nếu repon size quá lớn, TCP sử dụng thay thế)
- Nếu Local/ISP DNS sr ko có, sẽ tiếp tục đệ quy tới các DNS theo list cho đên khi đặt tới SOA. Khi phát hiện, kết quả sẽ trả lại

## Mở socket
Khi broser nhận IP add của server đích, nó sẽ lấy thống tin, sử dụng port từ URL (mặc đinh http 80, https 443), sử dụng lời gọi trong thư viện hàm socket và yêu câu TCP socket stream - AF_INET/AF_INET6 và SOCK_STREAM
+ Request đầu tiên tơi Transport Layer, nơi TCP segment đc tạo. Port đích được add vào header, port nguồn được chọn từ kernel's dynamic port range (ip_local_port_range in Linux).
+ Segment được gửi tới Net layer, đóng gói với IP header được thêm vào. Ip add của des server và IP máy hiện tại được thêm vào, tạo thành gói tin
+ Tiếp theo, Packet tới Link Layer. Frame header được thêm, bao gồm MAC address máy gửi (machine's NIC) cũng như MAC address của gateway (local router). Trước đó, nếu kernel không biết MAC add của Gateway, nó sẽ quảng bá = ARP query để tìm kiếm.

Packet khi sẵn sàng, có thể gửi qua:
+ Ethernet
+ Wifi
+ Celluler data network

Hầu như kêt nối internet của hộ gia đình hoặc doanh nghiệp nhỏ sẽ truyền packet từ your PC, tới local network, sau đó thông qua modem (MOdulator/DEModulator) convert tín hiệu 1 0 thành tín hiệu số có thể truyền qua telephone, cable, hoặc kết nối không dây. Tại đầu cuối tín hiệu, sẽ có model khác sẽ chuyển tín hiệu số thành digital data để xử lý bởi network node tiếp theo (nơi địa chỉ tới, network node sẽ được xử lý thêm).

Hầu như doanh nghiệp lớn và 1 số khu dân cứ lớn sẽ có đường truyền fiber hoặc direct ethenet connection trong trường hợp gửi dữ liệu số trực tiếp tới network node tiếp theo để xử lý

Cuối cùng gói tin tới router quản lý mạng nội bộ. Từ đây, nó sẽ tiếp tục tới các router biên autonomous system's (AS), Ases khác, cuối cùng tới server đích. Mỗi router trên tuyến đường sẽ trích xuất địa chỉ đích từ IP header và định tuyến nó tới hop thích hợp. Trường __time to live (TTL)__ trong IP header sẽ giảm dần khi đi qua mỗi router. Packet sẽ bị drop nếu TTL field giảm xuống 0 hoặc router không còn không gian trong queue (có thể do tắc nghẽn mạng)

Quá trình gửi và nhận xảy ra nhiều lần theo luồng kết nối TCP:
- Client chọn khởi tại số thứ tự (ISN) và gửi packet tới server với SYN bit set để làm rõ nó đang muốn thiết lập ISN
- Server nhận SYN và sẽ xảy ra hoạt động sau:
  + Server chọn số thứ tự riêng
  + Server set SYN cho biết nó chọn ISN của chính nó
  + Server sao chép (client ISN + 1) tới trường ACK và thêm ACK flag xác định nó đã nhận gói tin đầu tiên.
- Client nhận được bằng gửi gói tin
  + Tăng giá trị tuần tự
  + Tăng receiver acknowledgment number
  + Thiết lập ACK field
- Data được truyền theo:
  + 1 bên gửi N data bytes, nó tăng giá trị SEQ của nó = số đó
  + Khi bên nhận cho biết đã nhận được gói tin (hoặc string of packet), nó gửi ACK packet với giá trị ACK = giá trị ACK nhận được cuối cùng từ bên khác
- Để đóng kết nối:
  + closer gửi FIN packet
  + The other sides ACKs Fin packet và gửi giá trị FIN riêng
  + The closer acknowledges the other side's FIN with an ACK

## TLS handshake
- client computer gửi __ClientHello message__ tới server với phiên bản Transport Layer Security (TLS) của nó – Dánh sách thuật toán mã hóa và phương pháp nén có sẵn
- Ser trả lời = ServerHello message tới client với TLS version, lựa chọn loại mã hóa, chọn pp nén, lấy server's public certificate signed by a CA (Certificate Authority). Chứng chỉ bao gồm public key được sử dụng bởi client cho encrypts byte dựa trên thuật toán đối xứng.
- Client xác nhận server digital certificate dựa vào list CA tin cậy. Nếu tin cậy trên CA, khác hàng tạo chuỗi byte giả ngẫu nhiên và mã hóa = khóa công khai server. Các byte ngẫu nhiên có thể xác mình = khóa đối xứng.
- Server encrypt 1 vài byte = sử dụng private key của nó, sử dụng byte sinh ra để tạo bản copy của khóa đối xưng.
- Client gửi tin nhắn Finished, encrypting hash của tranmission up tới ví trí khóa đối xứng.
- Server sinh ra giá trị hash riêng, sau đó descrypt hash client gửi để xác minh nó phù hợp. Nếu đúng, nó gửi own Finished message tới client, được mã hóa = khóa đối xứng.
- Từ thời điểm, phiên TLS sẽ truyền application (HTTP) data với mã bằng bộ khóa đối xứng.

## HTTP protocol
- Nếu web broser được viết bởi gg, thay vì gửi HTTP request để nhận page, nó sẽ gửi request thử yêu cầu với máy chủ nâng cấp giao thức HTTP thành giao thức SPDY.
- Nếu client sử dụng gt http và không hỗ trợ SPDY, nó sẽ gửi request tới server với form:
```python
GET / HTTP/1.1
Host: google.com
Connection: close
[other headers]
```
- [other headers] = chuỗi cặp giá trị khóa được phân cách = dấu gạch, định dạng theo cấu trúc HTTP và phân tách = dòng mới (giả định web broser không xảy ra bất kỳ lỗi, vi phạm thông số HTTP. Sẽ có sự khác biệt giữa các phiên bản HTTP/1.1, 1.0, 0.9)
- HTTP/1.1 định nghĩa tùy chọn "close" connection khi người gửi báo hiệu connection sẽ đóng sau khi hoàn thành yêu cầu hiện tại
```python
Connection: close
```
- HTTP/1.1 app không yêu cầu các kết nối liên tiếp phải bao gồm close connection option trên mỗi tin nhắn.
- Sau khi gửi request và header, web browser gửi 1 dòng trống tơi server cho biết nội dung request đã hoàn thành
- Server responds với response code, biểu thị trạng thái yêu cầu, đáp ứng theo định dạng:
```python
200 OK
[response headers]
```
- Tiếp theo là dòng chống, gửi trọng tải HTML content của "google.com". Server sau đó có thể đóng kết nôi, hoặc mở nếu header gửi bởi client yêu cầu giữ kết nối mở cho các kết nối tiếp theo.
- Nếu HTTP header gửi bởi web browser bao gồm thông tin đây đủ để webserver xác đinh phiên bản "file cached" tại web browser hiện có sau lần cuối truy cập (ie. if the web browser included an ETagheader), nó có thể respond theo form:
```python
304 Not Modified
[response headers]
```
- và không phải tải lại nội dung, web browser thay vì request, sẽ sử dụng lại thông tin được cache.
- Sau khi phân tích cú pháp HTML, web browser (và server) sẽ lặp lại tiến trình cho mọi tài nguyên (image, CSS, favicon.ico, etc) được tham chiếu bởi HTML page, thay vì "GET / HTTP/1.1", request sẽ là "GET /$(URL relative to www.google.com) HTTP/1.1"
- Nếu HTML tham khảo tài nguyên trên tên miền khác "www.google.com", web browser quay lại bước phần giải tên miền, và làm lần lượt các bước tiếp theo.
>The Host header in the request will be set to the appropriate server name instead of google.com

## Xử lý request tại HTTP Server (HTTP Server Request Handle)
HTTPD (HTTP Daemon) server là máy chủ xử lý các request/response phía server. Các HTTPD server thông dụng là Apache, Nginx trên Linux và IIS cho Windows.
- HTTPD (HTTP Daemon) nhận các request
- Server phân loại request theo các thông số:
  + HTTP Request method (GET, HEAD, POST, PUT, DELETE, CONNECT, OPTIONS, or TRACE). Trong trường hợp nhấp tại URL sẽ là GET
  + Domain, trong trường hợp – google.com
  + Yêu cầu path/page – trong trường hợp là "/" không chỉ ra cụ thể path, path mặc định sẽ được sử dụng.
- Server xác minh Virtual Host được cấu hình trên server, được liên kết với google.com.
- Server xác minh google.com có thể accept GET requests
- Server xác minh client được phép xử dụng phương pháp (by IP, authentication, v.v).
- Nếu server thiết lập rewrite module (like mod_rewrite for Apache or URL Rewrite for IIS), nó sẽ thử ảnh xạ các request theo rule được cấu hình. Nếu rule match được tìm thấy, server sử dụng rule để rewrite request.
- Server thực hiện pull content liên quan đến request. Trong trường hợp hiện tại, nó sẽ trả lại Index File. "/" thể hiện main file (1 số trường hợp có thể override, nhưng đây là phương thức thông dụng
- Server phân tích tệp theo trình xử lý. Nếu gg chạy trên PHP, server sử dụng PHP để thông dịch index file, streams output tới client.


## Bên dưới trình duyệt (Behind the scenes of the Browser)
- Khi server cung cấp các tài nguyên (HTML, CSS, JS, images, etc.) cho browser, nó sẽ thực hiện các quá trình:
  + Parsing - HTML, CSS, JS
  + Rendering - Construct DOM Tree → Render Tree → Layout of Render Tree → Painting the render tree

## Browser
Chức năng của browser là thể hiện web resource được chọn = request tới server và hiện thị tại browser window theo tài nguyên trả lại. Tài nguyên thường là HTML docs, đôi khi có thể PDF, image, 1 số trình soạn thảo. Vị trí tài nguyên chỉ định = user sử dụng URI (Uniform Resource Identifier)

Cách browser hiện thị file phụ thuộc vào HTML, CSS nhận được. Phương pháp chung được mô tả, duy trì bởi tổ chức W3C (World Wide Web Consortium) - Tổ chức tiêu chuẩn cho web.

Browser interface có nhiều điểm chung, các phần có thể:
-	Thanh địa chỉ, cho phép nhập URL
-	Phím quay lại và tiến lên
-	Tùy chọn Bookmark
-	Làm mới và dừng lại việc load, xử lý tài nguyên hiện tại
-	Phím home để trở lại trang đầu tiên

## Cấu trúc năng cao trình duyệt - Browser High Level Structure
- Các thành phần browser:
  + User interface: giao diện bao gồm (address bar, back/forward button, bookmarking menu, etc). Tất cả sẽ được hiện thị đầy đủ trừ khi đang ở trang bắt đầu.
  + Browser engine: chịu trách nhiệm sắp xếp hành động giữa UI và rendering engine.
  + Rendering engine: chịu trách nhiệm hiện thị requested content. VD, nếu request content là HTML, rendering engine sẽ phân tích cú pháp HTML, CSS, hiện thị kết quả phân tích lên màn hình.
  + Networking: xử lý network calls như HTTP request, sử dụng thực thi khác nhau trên những nền tảng khác nhau.
  + UI backend: được sử dụng tạo widget như hộp kết hợp và cửa sổ. Sử dụng phương pháp cung cấp từ OS.
  + JavaScript engine: Sử dụng phấp tích cú pháp thực hiện Javascript code.
  + Data storage: persistence layer. Browser đôi khi cần lưu trữ các dữ liệu nội bộ như cookie. Broser cũng hỗ trợ các pp lưu như localStorage, IndexedDB, WebSQL and FileSystem.

## Phân tích cú pháp HTML (HTML parsing)
- Rendering engine xử lý khi nhận content docs sau request.
- Công việc chính của trình phân tích HTML là phân tích HTML markup thành parse tree.
- output tree (the "parse tree") là tree các thành phần DOM và các thuộc tính. DOM viết tắt Document Object Model. Nó là obj đại diện cho HTML doc và the interface of HTML elements to the outside world like JavaScript.
- Root của tree là "Document" object. Prior of any manipulation via scripting, the DOM has an almost one-to-one relation to the markup.

## Thuật toán phân tích (The parsing algorithm)
- HTML không thể phân tích theo pp đơn giản (top-down, bottom-up)
- Lý do:
  + Không thân thiện với ngôn ngữ tự nhiên
  + Các trình duyệt có khả năng chịu lỗi khi cú pháp HTML không đúng
  + The parsing process is reentrant. For other languages, the source doesn't change during parsing, but in HTML, dynamic code (such as script elements containing document.write() calls) can add extra tokens, so the parsing process actually modifies the input.
- Browser sử dụng trình phân tích cho phép tùy chỉnh cú pháp HTML. Thuật toán mô tả trong phiên bản HTML5.
- Thuật toán bao gồm 2 phần: tokenization and tree construction.

## Hoạt động sau quá trình phân tích hoàn thành (Actions when the parsing is finished)
- Trình duyệt hiện thị giao diện dựa trên (CSS, images, JavaScript files, etc.)
- Trình duyệt sẽ tự động fix lỗi cú pháp HTML để luôn hiện thị giao diện.

## Biên dịch CSS (CSS interpretation)
- Phân tích CSS files, <style> tag contents, và style attribute values using "CSS lexical and syntax grammar"
- Mỗi CSS file được phân tích thành StyleSheet object, nơi mỗi object chứa các CSS rule với bộ chọn đối tượng, đối tượng có cú pháp tương ứng.
- Phân tích CSS có thể diễn ra từ trên xuống dưới hoặc ngược lại tùy thuộc các trình tạo cú pháp.

## Tái tạo Page (Page Rendering)
- Tạo 'Frame Tree' hoặc 'Render Tree' = các phân tích các DOM node, và tính toán giá CSS style cho mỗi node.
- TÍnh toán chiều rộng mong muốn mỗi node trong 'Frame Tree' từ dưới lên = tính tổng chiều rộng cần trên mỗi node con, các lề nằm ngang, viền, và padding.
- Tính toán chiều rộng thực tế mỗi node từ trên xuống = cấp phát chiều rộng mỗi node con có sẵn.
- Tính toán chiều các các node từ dưới lên = các áp dụng gói văn bản, tổng hợp chiều cao các node con, lề, viền, padding
- Tính toán tọa độ mỗi node sử dụng dựa trên thông tin tính được từ các bước trên.
>More complicated steps are taken when elements are floated, positioned absolutely or relatively, or other complex features are used. See http://dev.w3.org/csswg/css2/ and http://www.w3.org/Style/CSS/current-work for more details.

- Tạo layer mô tả mỗi phần của trang có thể được hoạt hóa như 1 nhóm. Mỗi đối tượng frame/render được gán cho 1 layer.
- Textures cấp phát cho mỗi layer của page.
- Đối tượng frame/render của mỗi layer được truyền và các lệnh vẽ tương ứng cho mỗi trường hợp.
> This may be rasterized by the CPU or drawn on the GPU directly using D2D/SkiaGL.

-	Tất cả các bước trên có thể tái sử dụng các giá trị có được trước đó khi hiện tị web, giảm bớt công việc tăng hiệu năng
- Page layer được gửi tới các quá trình tổng hợp, nơi chúng được kết hợp vào các layer khác có thể nhìn như browser chrome, iframes và addon panels.
- Vị trí layer cuối cùng được tính toán và các lệnh ghép được thực hiện thông qua Direct3D/OpenGL. GPU command buffer đổ vào GPU để hiện thị không đồng bộ, frame được gửi tới window server.

## Xử lý thông qua GPU (GPU Rendering)
- Trong quá trình tái tạo, tính toán xây dựng có thể sử dụng CPU hoặc GPU để tính toán, xử lý.
- Khi sử dụng GPU để tính toán, xây dựng đồ họa trên các soft có thể tận dụng được ưu thế tính toán song song của GPU để thực hiện nhanh hơn.

## Window Server
## Post-rendering and user-induced execution
- Sau khi quá trình render hoàn tất, broser thực hiện js script
- Các plugin như Flash hoặc Java có thể được thực hiện, mặc dù không phải vào thời điểm tại Google homepage. Các script có thể thực hiện tác network request bổ sung, cũng như thay đổi page, bố cục, causing another round of page rendering and painting.
