# VietNix - Training

## **SSL certification là gì?**

- Secure Sockets Layer (SSL), đây là một tiêu chuẩn của công nghệ bảo mật, truyền thông mã hóa giữa trình duyệt và máy chủ webserver. SSL đảm bảo dữ liệu được truyền tải giữa server và browser toàn vẹn, riêng tư và bảo mật.

SSL là tiền thân của mã hóa TLS

SSL sẽ mã hóa dữ liệu được truyền trên web

- SSL nguyên lý hoạt động:

![Untitled](Task1/VietNix - Training/Untitled 1.png)

- SSL handshake
    1. Browser gửi “Hello”, gồm các thông tin liên quan để có thể giao tiếp (version SSL của client, dữ liệu theo phiên cụ thể và thông tin khác mà webserver cần giao tiếp với client bằng SSL).
    2. Webserver phản hồi “Hello” với các thông tin tương tự (version SSL của webserver, dữ liệu theo phiên cụ thể,  bản sao của SSL certification có public key và thông tin khác mà client cần để giao tiếp với server qua SSL).
    3. Client xác minh SSL của server CA
    4. Client tạo ra session key bằng cách encrypt public key, và gửi đến server.
    5. Server decrypt session key bằng private key và ACK về lại client.

![Untitled](VietNix%20-%20Training%20f39e3cc759774d00872cd0ec16695752/Untitled%201.png)

- Data Transfer
    1. Client và server sử dụng session key để encrypt/decrypt data và gửi/nhận chúng.

![Untitled](VietNix%20-%20Training%20f39e3cc759774d00872cd0ec16695752/Untitled%202.png)

- Các thành phần của SSL
• **CSR (Certificate Signing Request)**: Đây là 1 đoạn text chứa thông tin của chủ sở hữu (tên, địa chỉ, quốc gia…); domain được mã hóa; thông tin public key. Thông tin này được gửi đến nhà cung cấp dịch vụ SSL để xác nhận.
• **CRT (Certificate)**: Là thành phần sau khi CSR được xác nhận và trả về cho người đăng ký. Nếu CSR được tạo ra với mục đích làm cho nhà cung cấp SSL tin cậy website với thông tin được mã hoá, thì CRT là phần báo lại để trình duyệt web tin tưởng.
• **Private key:** Là file mã hoá được sinh ra cùng lúc khi tạo CSR. Để đơn giản, hãy hình dung CRT là phần mã hoá công khai được trình duyệt sử dụng để truy cập đến website của bạn. Vậy khi dữ liệu đến được website sẽ cần chìa khoá để mở khoá thông tin được mã hoá ở CRT.
• **CA (Certification Authority)**: Là cơ quan hay tổ chức cung cấp thông tin SSL certification.

## Sử dụng OpenSSL để gen file CSR sau đó request SSL cho domain tech.training.vietnix.tech

Truy cập [https://csrgenerator.com/](https://csrgenerator.com/)

![Untitled](VietNix%20-%20Training%20f39e3cc759774d00872cd0ec16695752/Untitled%203.png)

## CSR file dùng làm gì trong quá trình tạo SSL

To get an SSL certificate, you must create a Certificate Signing Request (CSR).

1. **Tạo private key và public key:**
Đầu tiên tạo ra private key và public key, có thể sử dụng các công cụ như OpenSSL …
2. **Tạo CSR file từ private key:**
    
    Tạo CSR file sau khi đã có private key, trong quá trình này cần cung cấp những thông tin về website, service, organization, domain name… để đưa vào CSR file.
    CSR file sẽ chứa public key và các thông tin về website/ứng dụng.
    
3. **Gửi CSR file đến CA:**
Gửi CSR file đến Certificate Authority (CA), CA sẽ xác thực thông tin trong CSR, sau đó cấp SSL certification tương ứng.
4. **Cài đặt SSL certification:**
Cuối cùng, website sẽ cài đặt SSL certification vừa được cấp, kết hợp với private key để hoàn tất quá trình.

## Pem file là gì?

Pem (Privacy Enhanced Mail) file là 1 định dạng container file phổ biến để lưu trữ SSL certification với private key.

Pem file được mã hóa bằng Base64 thường có dạng như

———— BEGIN CERTIFICATE ———— 

———— END CERTIFICATE ———— 

## Private key ssl là gì

Như nguyên lý hoạt động được nói ở trên, private key đóng vai trò là chìa khóa để webserver có thể decrypt được thông tin từ client để xác thực và bắt đầu một phiên kết nối SSL.

## PFX file là gì? cách chuyển file từ crt sang PFX file

PFX (Personal Information Exchange) file là một loại file tập hợp nhiều thông tin bảo mật quan trọng, bao gồm: private key, digital certificate, intermediate certificate.

Tất cả những thông tin này được lưu trữ trong cùng một file, giúp tăng cường bảo mật cho các ứng dụng sử dụng nó.

SSL certification dùng file .pfx là vì chúng có thể chứa confidential certification và private key trong cùng 1 file. Ngoài ra, file .pfx có thể chứa certification ở các định dạng khác như .cert, .crt và .pem. Điều này giúp tăng cường tính bảo mật của những thông tin quan trọng này.

## Cách chuyển file từ crt sang PFX file

```jsx
openssl pkcs12 -export -certpbe PBE-SHA1-3DES -keypbe PBE-SHA1-3DES -nomac -out domain.pfx -inkey domain.key -in domain_com.crt -CAfile  My_CA_Bundle.ca-bundle
```

1. `openssl pkcs12`: Lệnh này được sử dụng để tạo ra một file PKCS#12
2. `certpbe PBE-SHA1-3DES`: Chỉ định mã hóa sử dụng cho các chứng chỉ trong file PKCS#12, trong trường hợp này là PBE-SHA1-3DES.
3. `keypbe PBE-SHA1-3DES`: Chỉ định mã hóa sử dụng cho khóa riêng trong file PKCS#12, cũng là PBE-SHA1-3DES.
4. `nomac`: Loại bỏ tính năng kiểm tra tính toàn vẹn (MAC) trong file PKCS#12.
5. `out domain.pfx`: Chỉ định tên file đầu ra là "domain.pfx".
6. `inkey domain.key`: Chỉ định file chứa khóa riêng là "domain.key".
7. `in domain_com.crt`: Chỉ định file chứa chứng chỉ X.509 là "domain_com.crt".
8. `CAfile My_CA_Bundle.ca-bundle`: Chỉ định file chứa chuỗi chứng chỉ CA (Certificate Authority) là "My_CA_Bundle.ca-bundle".

## Domain là gì ?

Domain là 1 địa chỉ website hoạt động trên internet, địa chỉ này được mapping (ánh xạ) từ tên miền thân thiện với người dùng sang địa chỉ ip.

## Các trạng thái của domain

1. Active (Kích hoạt):
Domain đang được sử dụng và truy cập được.
Trang web liên kết với domain đang hoạt động.
2. Expired (Hết hạn):
Domain đã hết thời gian đăng ký, chủ sở hữu phải gia hạn để tiếp tục sử dụng.
Trang web liên kết với domain có thể không hoạt động.
3. Pending Delete (Chờ xóa):
Domain đang trong giai đoạn chờ xóa sau khi hết hạn.
Trong giai đoạn này, domain vẫn có thể được gia hạn.
4. Locked (Khóa):
Domain bị khóa do lý do an ninh, pháp lý hoặc yêu cầu của chủ sở hữu.
Không thể thực hiện các thay đổi như chuyển nhượng, gia hạn hoặc xóa.
5. Suspended (Tạm ngừng):
Domain bị tạm ngừng sử dụng do vi phạm các quy định.
Trang web liên kết với domain không thể truy cập được.

## Subdomain là gì?

- Subdomain là một phần của main domain, nằm tách biệt khỏi tên miền chính bằng dấu (.) và được sử dụng để tạo ra các địa chỉ web phụ.
Ví dụ: “mail.example.com" là một subdomain của "example.com".

- Mục đích để phân chia và tổ chức các phần khác nhau của 1 website, giúp tạo ra các địa chỉ web riêng biệt cho mỗi mục đích đó.

- Subdomain mang lại sự linh hoạt và tổ chức cho website, cho phép phân chia nội dung và dịch vụ một cách hiệu quả.

## Virtual Hosts là gì?

- Virtual Host là một dạng lưu trữ mà cho phép bạn nhúng nhiều domain khác nhau trên một địa chỉ IP trong một Sever. Server sẽ tự động hiểu domain nào đang vận hành bên trong vị trí lưu trữ Server tùy theo cách cài đặt của bạn.

![Untitled](VietNix%20-%20Training%20f39e3cc759774d00872cd0ec16695752/Untitled%204.png)

- Cách thức vận hành của Virtual Host

1. IP Based
    
    Một IP sử dụng cho 1 Website. Webserver sẽ chịu trách nhiệm mapping IP được yêu cầu có đế đến đúng website mong muốn. Vì thế, mỗi trang web sẽ được định nghĩa bởi 1 IP duy nhất . Tuy nhiên IP-Based (dùng trên 1 máy chủ) cần thiết lập Virtual Interface trên 1 máy chủ để có thể sử dụng được nhiều IP.
    
    ![Untitled](VietNix%20-%20Training%20f39e3cc759774d00872cd0ec16695752/Untitled%205.png)
    
    1. Port Based
        
        Tương đương với IP-Based, sự khác biệt ở phương thức này là có thể quản lý nhiều trang web dựa theo số Port được định nghĩa cùng với IP hoặc tên miền. Ngoài ra, Port sử dụng tránh lặp lại với Port được mặc định của ứng dụng khác khi đang hoạt động.
        
        ![Untitled](VietNix%20-%20Training%20f39e3cc759774d00872cd0ec16695752/Untitled%206.png)
        
    2. Name Based
        
        Nhiều website sử dụng chung 1 IP. Server sẽ đối chiếu http header từ client yêu cầu để ánh xạ đến đúng website được chỉ định theo Domain.
        
        ![Untitled](VietNix%20-%20Training%20f39e3cc759774d00872cd0ec16695752/Untitled%207.png)
        

## 

## Mail Server

- Là một hệ thống máy chủ được cấu hình riêng dựa trên tên miền của doanh nghiệp hoặc tổ chức để thực hiện việc gửi và nhận thư điện tử.

- Mail Server chia sẻ một số thông số kỹ thuật giống như các loại Server thông thường khác, bao gồm RAM, CPU và Storage. Bản chất của Mail Server giống như Dedicated Server hoặc Cloud Server.

- Cách thức hoạt động của Mail Server dựa vào 2 giao thức sau

1. Outgoing Mail Server: sử dụng giao thức SMTP (Simple Mail Transfer Protocol). SMTP là một giao thức dễ sử dụng cho việc dịch chuyển mail để giao tiếp với các Server từ xa. Đặc biệt, người dùng có khả năng gửi nhiều mail cùng một lúc đến các Server khác nhau khi sử dụng SMTP
2. Ingoing Mail Server: sử dụng POP3 hoặc IMAP
    
    POP3 Mang email đến lưu tại máy tính có Mail Client, hầu hết là nội bộ máy tính của người dùng bằng những ứng dụng email phổ biến như Outlook, Mac Mail hoặc Windows Mail,…
    
    IMAP cho phép nhiều client cùng lúc kết nối tới một Mailbox. Tại Mailbox, email sẽ được sao chép đến máy client, bản gốc của email luôn được giữ lại ở Mail Server.
    

- Phân loại mail server

1. Mail server Microsoft và Google
    
    Mail server Microsoft và Google có nền tảng xây dựng quy mô và hệ thống bảo mật chặt chẽ. Đồng thời có khả năng quản lý hiệu quả dữ liệu hiện có. Giá của dịch vụ Mail server Microsoft và Google khá cao bởi nó hỗ trợ nhiều tiện ích khác nhau.
    
2. Mail server độc lập
    
    Là hệ thống Mail được thiết kế riêng cho các tổ chức hay ISP có khối lượng lớn Email cần xử lý, yêu cầu kiểm soát và linh hoạt hơn đối với các dịch vụ linh hoạt. Mail server độc lập bổ sung các tính năng như hợp tác, đồng bộ hóa Outlook, Webmail, quản trị từ xa, quản trị web nâng cao, kết nối cơ sở dữ liệu, cung cấp sức mạnh và khả năng kiểm soát các hoạt động quy mô lớn
    

## MX Record

- MX (Mail Exchanger) Record là 1 record của DNS, xác định Mail Server nào đang chịu trách nhiệm xử lý email cho domain. Hay xác định Server cho 1 domain.

- MX đi kèm với record A trỏ về Email Server và giá trị priority (độ ưu tiên) càng nhỏ thì độ ưu tiên càng cao.

- VD MX record & A record

A record

- Domain: vietnix.vn
- Host name: mail
- IP-address 11.22.33.44
- Email Server: mail.vietnix.vn

MX record

- Domain: vietnix.vn
- Mail Exchanger: mail.vietnix.vn
- Priority: 10
    
    vietnix.vn in MX 10 mail.vietnix.vn
    
    Tất cả các mail gửi đến …@vietnix.vn, sẽ được gửi đến mail server mail.vietnix.vn có IP là 11.22.33.44
    

- Cơ chế hoạt động

Giả sử 1 user gửi mail đến email phuongduy@vietnix.vn

1. Sending Mail Server gửi request MX của vietnix.vn bằng port DNS/53 đến DNS Server
    
    DNS Server respond lại “mail.vietnix.vn”
    
2. Trường hợp Sending Mail Server chưa biết IP của Email Server: mail.vietnix.vn; nó sẽ tiếp tục hỏi DNS Server về thông tin đó bằng port DNS/53.
    
    DNS Server respond về IP 11.22.33.44
    
3. Sau khi có đủ thông tin, Sending Mail Server sẽ gửi mail đến Received Mail Server bằng port SMTP/25 với địa chỉ 11.22.33.44

## PTR Record

- PTR (Pointer Reverse) Record là 1 record DNS, phân giải IP sang tên (hostname). Bản phân giải ngược của A.

- Công dụng dùng để chống spam email và xác thực thông tin.

-  Cơ chế hoạt động của PTR

Tiếp tục với phần MX record trên.

1. Received Mail Server gửi request verify lại IP 11.22.33.44 là của hostname nào bằng port DNS/53 đến DNS server.
    
    DNS Server respond lại hostname với IP 11.22.33.44
    
2. Nếu IP và hostname match với IP và hostname được nhận từ Sending Mail Server, Received Mail Server chấp nhận nhận mail.
    
    Nếu không match, Received Mail Server Drop/Reject mail đó.
    

## SPF Record

- SPF(Sender Policy Frameworker) Record là record TXT của DNS, giúp xác thực domain email người gửi.

- Công dụng chống giả mạo email (spoofing domain), hệ thống email nhận sẽ kiểm tra email người gửi có nằm trong danh sách IP server được phép mang tên source domain để gửi.

## DKIM Record

- DKIM (DomainKeys Identified Mail) là 1 kỹ thuật xác thực email cho phép người nhận kiểm tra xem email có thực sự được gửi và được ủy quyền bởi chủ sở hữu miền đó hay không, nhằm chống email spoofing.

- Điều này được thực hiện bằng cách cung cấp cho email 1 chữ ký điện tử. Chữ ký DKIM này là tiêu đề được thêm vào header và được bảo mật mã hóa. DKIM cung cấp chữ ký số và xác minh.

-  Cơ chế hoạt động của DKIM

1. Sending Mail Server tạo ra private key và public key cho DKIM, trong đó private dùng để encrypt và tạo DKIM Signature gắn vào header trên mail. Public key được đăng ký trên DNS server thông qua DKIM Record.
    
    Trong DKIM Signature có nhiều thông số, trong đó gồm 2 phần quan trọng là “b” và “bh”.
    
    b là final signed (chữ ký sau khi được encrypt bằng private key)
    
    bh là signed trước khi encrypt.
    
2. Sending Mail Server gửi Mail đến Receive Mail Server.
    
    Receive Mail Server mở header ra, lấy thông tin b gửi đến DNS Server để verify lại.
    
    DNS Server dùng public key decrypt thông tin b thành bh’. Sau đó trả về cho Receive Mail Server.
    
    Nếu bh match bh’ DKIM pass và ngược lại.
    

## DNS là gì ?

DNS là hệ thống phân giải tên miền, giúp mapping giữa domain và IP với nhau.

## Các loại record DNS

Các loại record của DNS

- A : chứa IPv4 và tên miền tương ứng
- AAAA: chứa IPv6 và tên miền tương ứng
- CNAME: chứa subdomain và main domain tương ứng
- MX: chứa Mail Exchanger record
- NS: chứa Nameserver record
- PTR: chứa Pointer record
- SOA: chứa Start of Authority record
- SRV: chứa Service Location record
- TXT: chứa Text Record

## Nguyên tắc làm việc của DNS

- Nguyên tắc hoạt động cơ bản của DNS là ánh xạ domain thành địa chỉ IP tương ứng để thiết lập kết nối trên internet.
- Cache và tối ưu hóa
    
    DNS server lưu trữ thông tin tạm thời trong cache để tối ưu quá trình truy vấn, nếu thông tn cần truy vấn nằm trong cache DNS server có thể respond ngay lập tức.
    
- Phân chia hệ thống domain
    
    DNS server sử dụng hệ thống phân cấp cho phép máy chủ DNS đảm nhận trách nhiệm cho một phạm vi domain cụ thể. Mục đích nhằm đảm bảo tính nhất quán và chính xác.
    

## Cách phân giải địa chỉ DNS

- Các loại DNS Server

- Root Name Server
    
    Quản lý tất cả domain Top-level
    
- TLD Name Server
    
    Top-level Domain là máy chủ domain cao nhất, lưu trữ thông tin về domain có phần mở rộng chung (gTDL), chẳng hạn như .com .org .net
    
- Local Name Server
    
    DNS ISP, thường được duy trì và phát triển bởi các doanh nghiệp hoặc các nhà cung cấp dịch vụ Internet.
    
- DNS Recursor
    
    Là Server DNS có nhiệm vụ chuyển đổi tên miền thành địa chỉ IP.
    
- Authoritative Name Server
    
    Authoritative Name Server lưu trữ thông tin về tên miền và địa chỉ IP tương ứng. Là điểm cuối của quá trình truy vấn và phân giải địa chỉ IP cần thiết cho DNS Recursor.
    

![Untitled](VietNix%20-%20Training%20f39e3cc759774d00872cd0ec16695752/a727fd5b-c2bc-4fa8-b048-d5570822d2c0.png)

- Cách thức hoạt động

1. Khi client cần phân giải domain, client sẽ gửi yêu cầu đến Local DNS Server.
2. Local DNS Server sẽ kiểm tra xem domain đó có trong bộ nhớ cache của mình không. Nếu có, nó sẽ trả về địa chỉ IP tương ứng luôn.
3. Nếu tên miền không có trong bộ nhớ cache của Local DNS Server, thì nó sẽ giao tiếp với một DNS Recursor.
4. DNS Recursor sẽ là người thực hiện toàn bộ quá trình phân giải tên miền, bằng cách:
    - Gửi truy vấn đến Root DNS Server để tìm TLD DNS Server tương ứng.
    - Tiếp tục gửi truy vấn đến TLD DNS Server để tìm Authoritative DNS Server.
    - Cuối cùng gửi truy vấn đến Authoritative DNS Server để nhận về địa chỉ IP.
5. Sau khi tìm ra địa chỉ IP, DNS Recursor sẽ gửi kết quả trở lại cho Local DNS Server.
6. Local DNS Server sẽ lưu kết quả vào bộ nhớ cache của mình, và trả về địa chỉ IP cho máy tính yêu cầu.
