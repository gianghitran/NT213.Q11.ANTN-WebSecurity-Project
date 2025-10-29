# Automated broken object-level authorization attack detection in REST APIs through OpenAPI to colored petri nets transformation

## 1. Abstract & Introduction:
- Bài báo đề xuất phương pháp detect các cuộc tấn công Phân quyền cấp đối tượng bị hỏng (Broken Object-Level Authorization - BOLA) trong các API RESTful 
- Phương pháp phát hiện: Chuyển đổi OpenAPI specification (đặc tả OpenAPI) thành mạng Petri màu (Colored Petri Nets - CPNs)
- Phát triển công cụ Links2CPN để tự động hóa việc chuyển đổi và phân tích log của web server

## 2. Background:
### 2.1. REST APIs:
- Là dịch vụ Web được expose ra Internet sử dụng API, tuân theo kiến trúc REST (bộ các ràng buộc và quy ước để client và server giao tiếp với nhau)
- Các ràng buộc bao gồm:
    - Client-Server: Phân tách nhiệm vụ và chức năng của Client và Server
    - Staless: Xử lý thông tin mà không cần ngữ cảnh từ những lần xử lý trước
    - Cache: Có sự lưu trữ và sử dụng thông tin trên bộ nhớ đệm
    - Uniform Interface: Triển khai giao diện tách rời với phát triển services
    - Layered System: Các thành phần chỉ được tương tác với các thành phần khác cùng lớp hoặc các thành phần được kết nối tới, không được xem lớp khác
    - Code-On-Demand: Mở rộng chức năng của máy khách bằng cách cho phép tải xuống và execute code
### 2.2. OpenAPI specification (OAS)
- Là một tiêu chuẩn mở (open specification) để mô tả các RESTful API, cho phép cả người và máy có thể hiểu được các khả năng của services
- OAS documentation là một đối tượng có cấu trúc (JSON or YAML)
- Sử dụng 1 tập hợp các field cố định và có cấu trúc: `openapi`, `info, servers`, `paths, components`,`security`, `tags`, `jsonSchemaDialect`, `webhooks` và `externalDocs`. Trong đó `paths` là quan trọng nhất đối với nghiên cứu trong bài báo, chứa danh sách các endpoint của API và cách hoạt động
- Có 2 hướng thiết kế OAS:
    - Code-first: Code trước rồi mới tạo documentation. Có thể tạo từ các comments hoặc chú thích trong source code
    - Design-first: Tạo documentation trước rồi mới code. Có thể sử dụng AI để tự động việc tạo và cải thiện OAS

### 2.3. Colered Petri nets (CPNs)
#### a. Petri nets
- Là biểu diễn đồ họa dùng để đặc tả các hệ thống phân tán rời rạc, biểu thị bằng đồ thị có hướng
- Thành phần của Petri nets:
    - Place: Biểu thị bằng hình tròn, đại diện cho một trạng thái hoặc một nơi chứa tài nguyên
    - Transition: Biểu thị bằng hình chữ nhật, đại diện cho một sự kiện hoặc một hành động có thể xảy ra
    - Arc (cung): Là các mũi tên nối giữa place và transition để thấy được luồng di chuyển của công việc, có thể có số để biểu thị trọng số
    - Token: Biểu thị bằng chấm đen/số nằm trong các place, đại diện cho các đối tượng hoặc tài nguyên
- Một hành động transition chỉ có thể kích hoạt (fire) khi tất cả các kho đầu vào place của nó có đủ số lượng token yêu cầu. Khi kích hoạt, nó sẽ tiêu thụ token từ các kho đầu vào và tạo ra token mới ở các kho đầu ra. Số lượng token vào/ra tương ứng với trọng số trên cung

#### b. CPNs:
- Là bản mở rộng của Petri nets
- Mỗi token trong place sẽ có thêm điểm đặc biệt là cùng mang 1 kiểu dữ liệu (màu). Và place đó cũng được gán kiểu dữ liệu đó
---> CPNs không chỉ mô hình hóa luồng điều khiển mà còn cả luồng dữ liệu
- Điều kiện kích hoạt (Enabling condition): Một hành động transition được kích hoạt không chỉ phụ thuộc vào việc có đủ token hay không, mà còn phải thỏa mãn các điều kiện về giá trị dữ liệu (màu) của các token đó. Ví dụ, một hành động "xử lý đơn hàng" chỉ được kích hoạt nếu token đầu vào có "màu" là "đơn hàng đã thanh toán"
- Quy tắc kích hoạt (Firing rule): Khi một hành động được kích hoạt, nó sẽ tiêu thụ các token có giá trị dữ liệu cụ thể từ đầu vào và tạo ra các token mới ở đầu ra với các giá trị dữ liệu cũng được tính toán và xác định cụ thể

## 3. Running example:
- Ví dụ này sẽ sử dụng xuyên suốt bài báo
- Ví dụ này là một Web app đơn giản (client-server), user login và check giỏ hàng của họ
- Ví dụ này lấy trên OWASP Juice Shop
![image](https://hackmd.io/_uploads/ry8auT9aee.png)
- Kịch bản:
    - Người dùng đăng nhập (POST /login) 
    - Truy xuất giỏ hàng (GET /basket/{bid})
    - Lỗ hổng BOLA trong ví dụ này cho phép attacker xem giỏ hàng của người dùng khác bằng cách thay đổi giá trị bid (mã giỏ hàng) trong yêu cầu API

## 4. Methodology and the tool Links2CPN
![image](https://hackmd.io/_uploads/ryX6zpi6ll.png)

### 4.1. Model-to-model transformation: from OpenAPI to CPN
- Chuyển đổi OAS thành CPN
- Các bước chuyển đổi:
    - Bước 1: Tạo tất cả transition và place liên quan đến paths
    - Bước 2: Kết nối các transition và place bằng links
    - Bước 3: Xóa các transition và place không có kết nối tới

### 4.2. Conformance checking: detecting broken object level authorization attacks
- Sau khi có CPN, cần validate lại bằng kỹ thuật process mining (khai phá quy trình) để kiểm tra các cặp `request`-`response`
- Cơ chế: Thuật toán replay lặp qua các trace từ log của web server trên mô hình CPN
- Control flow violation (Vi phạm luồng điều khiển): Không có token nào trong input place của transition
- Data flow violation (Vi phạm luồng dữ liệu): 1 token trong log có giá trị không khớp với những gì được mong đợi bởi mô hình CPN, khiến cho một transition không thể được kích hoạt
--> Phát hiện được BOLA attack khi có sự **vi phạm luồng dữ liệu (Data flow violation)**

### 4.3. Tool support
 `Links2CPN`: https://github.com/ailton07/openapi-links-to-CPNs

## 5. Experiment
- Môi trường:
    - OWASP Juice Shop: https://github.com/ailton07/juice-shop-with-winston
    - Memos: https://github.com/ailton07/memos-with-BOLA

## 6. Demo
### Installation:
``` bash
python -m venv venv
source venv/bin/activate
pip3 install -r requirements.txt
```



```
sudo apt install graphviz
```
### Usage

```
cd src/
python3 execute_replay.py <open_api_specification_path> <event_logs_path>
```


### Example
```
$ python3 execute_replay.py examples/OWASP-Juice-Shop-experiment.yaml logs/real_logs_experiment.log
```