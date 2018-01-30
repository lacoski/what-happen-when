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
- Nếu PC kết nối switch, switch check local CAM/MAC table, kiếm tra port có MAC đang tìm. Nếu switch ko tìm thấy MAC, nó sẽ rebroadcast ARP request tới các port khác
- switch có entry trong MAC/CAM table sẽ gửi ARP request to port có địa chỉa MAC address cần tìm
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
- Nếu Local/ISP DNS sr ko có, sẽ tiếp tục đệ qua tới các DNS theo list cho đên khi đặt tới SOA, khi phát hiện sẽ trả lại

## Mở socket
Khi broser nhận IP add của đích server, nó lấy thống tin, sử dụng port trên URL (mặc đinh http 80, https 443), sử dụng lời gọi trong thư viện hàm socket và yêu câu TCP socket stream - AF_INET/AF_INET6 và SOCK_STREAM
+ Request đầu tiên tơi Transport Layer, nơi TCP segment đc tạo. Port đích được add vào header, source port được chọn từ kernel's dynamic port range (ip_local_port_range in Linux).
+ Segment gửi tới Net layer, Đóng gói với IP header được thêm vào. Ip add của des server cũng như máy hiện tại được thêm vào, tạo thành gói tin
+ Tiếp theo, Packet tới Link Layer. Frame header được thêm vào bao gồm MAC address máy gửi (machine's NIC) cũng như MAC address của gateway (local router). Trước đó, nếu kernel không biết MAC add của Gateway, nó sẽ quảng bá = ARP query để tìm kiếm.

Packet khi sẵn sàng, có thể gửi qua:
+ Ethernet
+ Wifi
+ Celluler data network

Hầu như kêt nối internet của home or small business sẽ truyền packet sẽ truyền từ your PC, tới local network, sau đó thông qua modem (MOdulator/DEModulator)  convert tín hiệu 1 0 thành tín hiệu số có thể truyền qua telephone, cable, hoặc kết nối không dây. Tại đầu cuối tín hiệu, 1 model khác sẽ chuyển tín hiệu số thành digital data để xử lý bởi network node tiếp theo (nơi địa chỉ tới, network node sẽ được xử lý thêm).

Hầu như doanh nghiệp lớn và 1 số khu dân cứ lớn sẽ có fiber hoặc direct ethenet connection trong trường hợp trường hợp dữ liệu số được trực tiếp tơi network note tiếp theo để xử lý

Cuối cùng gói tin tới router quản lý mạng nội bộ. Từ đây, nó sẽ tiếp tục tới autonomous system's (AS) border routers, other Ases, và tới server đích. Mỗi router trên đường đi sẽ trích xuất địa chỉ đích từ IP header và định tuyến nó tới hop thích hợp. Trường time to live (TTL) trong IP header sẽ giảm dần sau khi đi qua mỗi router. Packet sẽ bị drop nếu TTL field giảm xuống 0 hoặc router không còn không gian trong queue (có thể do tắc nghẽn mạng)

Quá trình gửi và nhận xảy ra nhiều lần theo luồng kết nối TCP:
- Client chọn khởi tại số thứ tự (ISN) và gởi packet tới server với SYN bit set để làm rõ nó đang muốn thiết lập ISN
- Server nhận SYN và sẽ xảy ra hoạt động sau:
  + Server chọn số thứ tự riêng
  + Server set SYN cho biết nó chọn its ISN
  + Server sao chép (client ISN + 1) tới trường ACK và thêm ACK flag đã nhận gói tin đầu tiên.
- Client nhận được kết bằng gửi gói tin
  + Tăng giá trị tuần tự
  + Tăng  receiver acknowledgment number
  + Thiết lập ACK field
- Data được truyền theo:
  + 1 bên gửi N data bytes, nó tăng giá trị SEQ của nó = số đó
  + Khi other side cho biết đã nhận được gói tin (hoặc string of packet), nó gửi ACK packet với giá trị ACK = giá trị ACK nhận được cuối cùng từ bên khác
- Để đóng kết nối:
  + closer gửi FIN packet
  + Bên khác ACKs Fin packet và gửi giá trị FIN riêng

## TLS handshake
- client computer gửi ClientHello message tới server với phiên bán Transport Layer Security (TLS) – Dánh sách thuật toán mã hóa và phương pháp nén có sẵn
- Ser đáp ứng với ServerHello message tới cliet với TLS version, lựa chọn lại mã hóa, chọn pp nén, lấy server's public certificate signed by a CA (Certificate Authority). Chứng chỉ bao gồm public key được sử dụng bởi client cho encrypts byte dựa trên thuật toán đối xướng.
- Server encrypt 1 vài byte = sử dụng it prrivate key, sử dụng byte sinh ra để tạo bản copy của khóa đối xưng.
- Client gửi tin nhắn Finished, encrypting hash của tranmission up tới ví trí khóa đối xứng.
- Server sinh ra giá trị hash riêng, sau đó descrypt hash client gửi để xác minh nó phù hợp. Nếu đúng, nó gửi own Finished message tới client, được mã hóa = khóa đối xứng.
- Từ thời điểm, phiên TLS gửi application (HTTP)  data được mã với bộ khóa đối xứng.

## HTTP protocol
- Nếu web broser được viết bởi gg, thay vì gửi HTTP request để nhận page, nó sẽ gửi request thử và đàm phán với máy chủ nâng cấp giao thức HTTP tới giao thức SPDY.
- Nếu client sử dụng gt http và không hỗ trợ SPDY, nó sẽ gửi request tới server với form:
```python
GET / HTTP/1.1
Host: google.com
Connection: close
[other headers]
```
- [other headers] = chuỗi cập giá trị khóa được phân cách = dâu gạch, định dạng theo cấu trúc HTTP và phân tách = dòng mới (giá định web broser không xảy ra bất kỳ lỗi, vi phạm thông sô HTTP. Sẽ có sự khác biệt giữa HTTP/1.1, 1.0, 0.9)
- HTTP/1.1 đinh nghĩa tùy chọn “”close” connection cho người gửi báo hiệu connection sẽ đóng sau khi hoàn thành trài lời hiện tại
```python
Connection: close
```
- HTTP/1.1 app không hỗ trợ các kết nối liên tiếp phải bao gồm  các close connection option mỗi tin nhắn.
- Sau khi gửi request và header, web browser gửi 1 dòng trống tơi server cho biết nội dụng request hoàn thành
- Server responds với response code, biểu thị trạng thái yêu cầu, đáp ứng = định dạng:
```python
200 OK
[response headers]
```
- Tiếp theo là single newline, gửi payload HTML content cho google.com. Server sau đó có thể đóng kết nôi, hoặc nếu header gửi bới client yêu cầu giữa kết nối mở cho các kết nối tiếp theo.
- Nếu HTTP header gửi bởi web browser bao gồm thông tin đây đủ cho webserver xác đinh phiên bản file cached tại web broser đã được sử sau lần truy cập cuối (ie. if the web browser included an ETagheader), nó có thể respond theo form:
```python
304 Not Modified
[response headers]
```
- and no payload, và web broser thay vì request sẽ lấy thông tin được cache.
- Sau khi phân tích cú pháp HTML, web broser (và server) sẽ lặp lại tiến trình cho mọi tài nguyên (image, CSS, favicon.ico, etc) được tham chiếu bởi HTML page, ngoại trừ trả lại “GET / HTTP/1.1”, request sẽ là “GET /$(URL relative to www.google.com) HTTP/1.1”
- Nếu HTML tham khảo tài nguyên trên tên miền khác “www.google.com”, web broser quay lại bước phần giải tên miền, và làm lần lượt các bước tiếp theo. The Host header in the request will be set to the appropriate server name instead of google.com

## HTTP Server Request Handle
- HTTPD (HTTP Daemon) server là máy chủ xử lý các request/responses phía server side. Các HTTPD server thông dụng là  Apache or nginx for Linux and IIS for Windows.
- HTTPD (HTTP Daemon) nhận request
- Server phân loại request theo các thông số:
  + HTTP Request method (GET, HEAD, POST, PUT, DELETE, CONNECT, OPTIONS, or TRACE). Trong trường hợp nhấp tại URL sẽ là GET
  + Domain, trong trường hợp – google.com
- Request path/page – trong trường hợp là “/” không chỉ ra cụ thể path, path mặc định
- Server xác minh Virtual Host được cấu hình trên server liên kết với google.com.
- Server xác minh google.com có thể accept GET requests
- Server xác minh client được phép xử dụng methods (by IP, authentication, etc.).
- Nếu server thiết lập rewrite module (like mod_rewrite for Apache or URL Rewrite for IIS), nó sẽ thử match request theo rule được cấu hình. Nếu rule match được tìm thấy, server sử dụng rule để rewrite request.
- Server thực hiện pull content liên quan đến request, trong trường hợp nó sẽ it will fall back to the index file¸ “/” thể hiện main file (1 số trường hợp có thể override, nhưng đây là phương thức thông thường
- Server phân tích tệp theo trình xử lý. Nếu gg chạy trên PHP, server sử dụng PHP để thông dịch index file, streams output tới client.


## Behind the scenes of the Browser
- Khi server cung cấp các tài nguyên (HTML, CSS, JS, images, etc.) cho broser, nó sẽ thực hiện các quá trình:
  + Parsing - HTML, CSS, JS
  + Rendering - Construct DOM Tree → Render Tree → Layout of Render Tree → Painting the render tree

## Browser
- chức năng của broser là thể hiện web resource được chọn = request từ server và hiện thị tại broser window. Tài nguyên thường là HTML docs, đôi khi có thể PDF, image, 1 số trình soạn thảo. Vị trí tài nguyên chỉ định = user sử dụng URI (Uniform Resource Identifier)
- Cách hiện thị của broser hiện thị file phụ thuộc HTML, CSS nhận được. Các mô tả được duy trì bởi tổ chức W3C (World Wide Web Consortium), tổ chức tiêu chuẩn cho web.
- Broser interface có nhiều điểm chung, các phần có thể:
  -	An address bar for inserting a URI
  -	Back and forward buttons
  -	Bookmarking options
  -	Refresh and stop buttons for refreshing or stopping the loading of current documents
  -	Home button that takes you to your home page

## Browser High Level Structure
- Các thành phần browser là:
  + User interface: giao diện bao gồm( address bar, back/forward button, bookmarking menu, etc). Tất cả sẽ được hiện thị đầy đủ trừ khi đang ở trang bắt đầu.
  + Browser engine: The browser engine marshals actions between the UI and the rendering engine.
  + Rendering engine: chịu trách nhiệm hiện thị requested content. VD, nếu request content là HTML, rendering engine sẽ phân tích cú pháp HTML, CSS, hiện thị kết quả phân tích lên màn hình.
  + Networking: xử lý network calls như HTTP request, sử dụng thực thi khác nhau trên những nền tảng khác nhau.
  + UI backend: được sử dụng tạo widget như hộp kết hợp và cửa sổ. Sử dụng phương pháp cung cấp từ OS.
  + JavaScript engine: Sử dụng phấp tích cú pháp thực hiện Javascript code.
  + Data storage: persistence layer. Broser đôi khi cần lưu trữ các dữ liệu nội bộ như cookie. Broser cũng hỗ trợ các pp lưu như localStorage, IndexedDB, WebSQL and FileSystem.

## HTML parsing
- Rendering engine bắt đi khi nhận content docs sau request.
- Công việc chính của trình phân tích HTML là phân tích HTML markup thành parse tree.
- output tree (the "parse tree") là tree các thành phần DOM và các thuộc tính. DOM viết tắt Document Object Model. Nó là obj đại diện cho HTML doc và the interface of HTML elements to the outside world like JavaScript.
- Root của tree là "Document" object. Prior of any manipulation via scripting, the DOM has an almost one-to-one relation to the markup.

## The parsing algorithm
- HTML không thể phân tích theo pp đơn giản (top-down, bottom-up)
- Lý do:
  + Không thân thiện với ngôn ngữ tự nhiên
  + Các trình duyệt có khả năng chịu lỗi khi cú pháp HTML không đúng
  + The parsing process is reentrant. For other languages, the source doesn't change during parsing, but in HTML, dynamic code (such as script elements containing document.write() calls) can add extra tokens, so the parsing process actually modifies the input.
- Broser sử dụng trình phân tích cho phép tùy chỉnh cú pháp HTML. Thuật toán mô tả trong HTML5.
- Thuật toán bao gồm 2 phần: tokenization and tree construction.

## Actions when the parsing is finished
- Trình duyệt hiện thị giao diện dựa trên (CSS, images, JavaScript files, etc.)
- Trình duyệt sẽ tự động fix lỗi cú pháp HTML để luôn hiện thị giao diện.

## CSS interpretation
- Parse CSS files, <style> tag contents, and style attribute values using "CSS lexical and syntax grammar"
- Each CSS file is parsed into a StyleSheet object, where each object contains CSS rules with selectors and objects corresponding CSS grammar.
- A CSS parser can be top-down or bottom-up when a specific parser generator is used.

## Page Rendering
- Create a 'Frame Tree' or 'Render Tree' by traversing the DOM nodes, and calculating the CSS style values for each node.
-	Calculate the preferred width of each node in the 'Frame Tree' bottom up by summing the preferred width of the child nodes and the node's horizontal margins, borders, and padding.
-	Calculate the actual width of each node top-down by allocating each node's available width to its children.
-	Calculate the height of each node bottom-up by applying text wrapping and summing the child node heights and the node's margins, borders, and padding.
-	Calculate the coordinates of each node using the information calculated above.
-	More complicated steps are taken when elements are floated, positioned absolutely or relatively, or other complex features are used. See http://dev.w3.org/csswg/css2/ and http://www.w3.org/Style/CSS/current-work for more details.
-	Create layers to describe which parts of the page can be animated as a group without being re-rasterized. Each frame/render object is assigned to a layer.
-	Textures are allocated for each layer of the page.
-	The frame/render objects for each layer are traversed and drawing commands are executed for their respective layer. This may be rasterized by the CPU or drawn on the GPU directly using D2D/SkiaGL.
-	All of the above steps may reuse calculated values from the last time the webpage was rendered, so that incremental changes require less work.
-	The page layers are sent to the compositing process where they are combined with layers for other visible content like the browser chrome, iframes and addon panels.
-	Final layer positions are computed and the composite commands are issued via Direct3D/OpenGL. The GPU command buffer(s) are flushed to the GPU for asynchronous rendering and the frame is sent to the window server.

## GPU Rendering
- Trong quá trình rendering graphical computing layers có thể sử dụng CPU hoặc GPU để tính toán, xử lý.
- Khi sử dụng GPU để tính toán, xây dựng đồ họa các soft có thể tận dụng được ưu thế tính toán song song của GPU để thực hiện nhanh hơn.

## Post-rendering and user-induced execution
- Sau khi quá trình render hoàn tất, broser thực hiện js script
- Plugins such as Flash or Java may execute as well, although not at this time on the Google homepage. Scripts can cause additional network requests to be performed, as well as modify the page or its layout, causing another round of page rendering and painting.
