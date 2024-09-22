# API testing

## API recon

Để bắt đầu thử nghiệm API, trước tiên cần tìm hiểu càng nhiều thông tin về API càng tốt để khám phá bề mặt tấn công của nó.

Để bắt đầu, nên xác định các endpoint API. Đây là các location mà API nhận được yêu cầu về một tài nguyên cụ thể trên máy chủ của nó. Ví dụ, hãy xem xét yêu cầu `GET` sau:

```
GET /api/books HTTP/1.1
Host: example.com
```

Endpoint API cho request này là `/api/books`. Điều này dẫn đến tương tác với API để truy xuất danh sách sách từ thư viện. Một endpoint API khác có thể là, ví dụ, `/api/books/mystery`, sẽ truy xuất danh sách sách bí ẩn.

Sau khi xác định được các endpoint, cần xác định cách tương tác với chúng. Điều này cho phép xây dựng các yêu cầu HTTP hợp lệ để thử nghiệm API. Ví dụ, nên tìm hiểu thông tin về những điều sau:

- Dữ liệu đầu vào mà API xử lý, bao gồm cả tham số bắt buộc và tùy chọn.

- Các loại yêu cầu mà API chấp nhận, bao gồm các phương thức HTTP và định dạng phương tiện được hỗ trợ.

- Rate limits và cơ chế xác thực.

## API documentation

API thường được ghi chép lại để các nhà phát triển biết cách sử dụng và tích hợp với chúng.

Tài liệu có thể ở cả dạng dễ đọc bằng con người và dạng dễ đọc bằng máy. Tài liệu dễ đọc bằng con người được thiết kế để các nhà phát triển hiểu cách sử dụng API. Tài liệu có thể bao gồm các giải thích chi tiết, ví dụ và tình huống sử dụng. Tài liệu dễ đọc bằng máy được thiết kế để phần mềm xử lý nhằm tự động hóa các tác vụ như tích hợp và xác thực API. Tài liệu được viết ở các định dạng có cấu trúc như JSON hoặc XML.

Tài liệu API thường được công khai, đặc biệt nếu API dành cho các nhà phát triển bên ngoài sử dụng. Nếu đúng như vậy, hãy luôn bắt đầu quá trình tìm hiểu bằng cách xem xét tài liệu.

### Discovering API documentation

Ngay cả khi tài liệu API không được công khai, vẫn có thể truy cập tài liệu này bằng cách duyệt các ứng dụng sử dụng API.

Để thực hiện việc này, có thể sử dụng Burp Scanner để thu thập API. Cũng có thể duyệt các ứng dụng theo cách thủ công bằng trình duyệt của Burp. Tìm kiếm các endpoint có thể tham chiếu đến tài liệu API, ví dụ:

- `/api`

- `/swagger/index.html`

- `/openai.json`

Nếu xác định endpoint cho một tài nguyên, hãy đảm bảo điều tra base path. Ví dụ: nếu xác định endpoint tài nguyên /api/swagger/v1/users/123, thì nên điều tra các đường dẫn sau:

- `/api/swagger/v1`

- `/api/swagger`

- `/api`

Cũng có thể sử dụng danh sách các đường dẫn phổ biến để tìm tài liệu bằng Intruder.

### Lab: Exploiting an API endpoint using documentation

Để solve lab, hãy tìm tài liệu API đã được tiết lộ và xóa `carlos`. Có thể đăng nhập vào tài khoản bằng thông tin đăng nhập sau: `wiener:peter`.

#### Solution

1. Trong trình duyệt của Burp, hãy đăng nhập vào ứng dụng bằng thông tin xác thực `wiener:peter` và cập nhật địa chỉ email.

2. Trong `Proxy > HTTP History`, nhấp chuột phải vào request `PATCH /api/user/wiener` và chọn `Send to Repeater`.

3. Đi đến tab `Repeater`. Gửi request `PATCH /api/user/wiener`. Lưu ý rằng thao tác này sẽ truy xuất thông tin xác thực cho người dùng `wiener`.

4. Xóa `/wiener` khỏi đường dẫn của yêu cầu, do đó endpoint hiện là `/api/user`, sau đó gửi request. Lưu ý rằng thao tác này sẽ trả về lỗi vì không có mã định danh người dùng.

5. Xóa `/user` khỏi đường dẫn của yêu cầu, do đó endpoint hiện là `/api`, sau đó gửi requets. Lưu ý rằng thao tác này sẽ truy xuất tài liệu API.

6. Nhấp chuột phải vào response và chọn `Show response in browser`. Sao chép URL.

7. Dán URL vào trình duyệt của Burp để truy cập tài liệu. Lưu ý rằng tài liệu này là tương tác.

8. Để xóa Carlos và solve lab, hãy click vào hàng `DELETE`, nhập `carlos`, sau đó nhấp vào `Send request`.

### Using machine-readable documentation

Có thể sử dụng một loạt các công cụ tự động để phân tích bất kỳ tài liệu API nào có thể đọc bằng máy tìm thấy.

Có thể sử dụng Burp Scanner để thu thập và kiểm tra tài liệu OpenAPI hoặc bất kỳ tài liệu nào khác ở định dạng JSON hoặc YAML. Cũng có thể phân tích tài liệu OpenAPI bằng OpenAPI Parser BApp.

Cũng có thể sử dụng một công cụ chuyên dụng để kiểm tra các endpoint được ghi lại, chẳng hạn như Postman hoặc SoapUI.

## Identifying and interacting with API endpoints

Cũng có thể thu thập nhiều thông tin bằng cách duyệt các ứng dụng sử dụng API. Điều này thường đáng làm ngay cả khi có quyền truy cập vào tài liệu API, vì đôi khi tài liệu có thể không chính xác hoặc lỗi thời.

Có thể sử dụng Burp Scanner để thu thập dữ liệu ứng dụng, sau đó điều tra thủ công bề mặt tấn công thú vị bằng trình duyệt của Burp.

Trong khi duyệt ứng dụng, hãy tìm các mẫu gợi ý endpoint API trong cấu trúc URL, chẳng hạn như `/api/`. Ngoài ra, hãy chú ý đến các tệp JavaScript. Các tệp này có thể chứa các tham chiếu đến endpoint API mà chưa kích hoạt trực tiếp qua trình duyệt web. Burp Scanner tự động trích xuất một số điểm cuối trong quá trình thu thập dữ liệu, nhưng để trích xuất dữ liệu nặng hơn, hãy sử dụng JS Link Finder BApp. Cũng có thể xem xét thủ công các tệp JavaScript trong Burp.

Sau khi xác định được các endpoint API, hãy tương tác với chúng bằng Burp Repeater và Burp Intruder. Điều này cho phép quan sát hành vi của API và khám phá thêm bề mặt tấn công. Ví dụ: có thể điều tra cách API phản hồi khi thay đổi phương thức HTTP và loại phương tiện.

Khi tương tác với các endpoint API, hãy xem xét kỹ các thông báo lỗi và các phản hồi khác. Đôi khi, chúng bao gồm thông tin có thể sử dụng để xây dựng yêu cầu HTTP hợp lệ.

### Identifying supported HTTP methods

Phương thức HTTP chỉ định hành động sẽ được thực hiện trên một tài nguyên. Ví dụ:

- `GET` - Truy xuất dữ liệu từ một tài nguyên.

- `PATCH` - Áp dụng các thay đổi một phần cho một tài nguyên.

- `OPTIONS` - Truy xuất thông tin về các loại phương thức yêu cầu có thể được sử dụng trên một tài nguyên.

Endpoint API có thể hỗ trợ các phương thức HTTP khác nhau. Do đó, điều quan trọng là phải kiểm tra tất cả các phương thức tiềm năng khi điều tra các endpoint API. Điều này có thể cho phép xác định chức năng endpoint bổ sung, mở ra nhiều bề mặt tấn công hơn.

Ví dụ: endpoint `/api/tasks` có thể hỗ trợ các phương thức sau:

- `GET /api/tasks` - Truy xuất danh sách các tác vụ.

- `POST /api/tasks` - Tạo một tác vụ mới.

- `DELETE /api/tasks/1` - Xóa một tác vụ.

Có thể sử dụng danh sách `HTTP verbs` tích hợp trong Burp Intruder để tự động tuần hoàn qua một loạt các phương thức.

```
Lưu ý: Khi thử nghiệm các phương thức HTTP khác nhau, hãy nhắm mục tiêu vào các đối tượng có mức độ ưu tiên thấp. Điều này giúp đảm bảo rằng tránh được những hậu quả không mong muốn, ví dụ như thay đổi các mục quan trọng hoặc tạo quá nhiều bản ghi.
```

### Identifying supported content types

Endpoint API thường mong đợi dữ liệu ở một định dạng cụ thể. Do đó, chúng có thể hoạt động khác nhau tùy thuộc vào loại nội dung của dữ liệu được cung cấp trong yêu cầu. Thay đổi loại nội dung có thể cho phép:

- Kích hoạt lỗi tiết lộ thông tin hữu ích.

- Bỏ qua các biện pháp phòng thủ bị lỗi.

- Tận dụng sự khác biệt trong logic xử lý. Ví dụ: API có thể an toàn khi xử lý dữ liệu JSON nhưng dễ bị tấn công tiêm nhiễm khi xử lý XML.

Để thay đổi loại nội dung, hãy sửa đổi tiêu đề `Content-Type`, sau đó định dạng lại nội dung yêu cầu cho phù hợp. Có thể sử dụng Bộ chuyển đổi loại nội dung BApp để tự động chuyển đổi dữ liệu được gửi trong các yêu cầu giữa XML và JSON.

### Lab: Finding and exploiting an unused API endpoint

Để solve lab, hãy khai thác endpoint API ẩn để mua `Lightweight l33t Leather Jacket`. Có thể đăng nhập vào tài khoản bằng thông tin đăng nhập sau: `wiener:peter`.

#### Solution

1. Trong trình duyệt của Burp, hãy truy cập lab và click vào một sản phẩm.

2. Trong `Proxy > HTTP History`, hãy lưu ý request API cho sản phẩm. Ví dụ: `/api/products/3/price`.

3. Nhấp chuột phải vào request API và chọn `Send to Repeater`.

4. Trong tab `Repeater`, hãy thay đổi phương thức HTTP cho request API từ `GET` thành `OPTIONS`, sau đó gửi request. Lưu ý rằng response chỉ định rằng các phương thức `GET` và `PATCH` được phép.

5. Thay đổi phương thức cho request API từ `GET` thành `PATCH`, sau đó gửi request. Lưu ý rằng nhận được thông báo `Unauthorized`. Điều này có thể cho biết rằng cần được xác thực để cập nhật đơn hàng.

6. Trong trình duyệt của Burp, hãy đăng nhập vào ứng dụng bằng thông tin đăng nhập `wiener:peter`.

7. Click vào sản phẩm `Lightweight "l33t" Leather Jacket`.

8. Trong `Proxy > HTTP History`, hãy nhấp chuột phải vào request `API/products/1/price` cho áo khoác da và chọn `Send to Repeater`.

9.  Trong tab `Repeater`, hãy thay đổi phương thức cho request API từ `GET` thành `PATCH`, sau đó gửi request. Lưu ý rằng điều này gây ra lỗi do `Content-Type` không chính xác. Thông báo lỗi chỉ định rằng `Content-Type` phải là `application/json`.

10. Thêm tiêu đề `Content-Type` và đặt giá trị thành `application/json`.

11. Thêm đối tượng JSON trống `{}` làm phần body request, sau đó gửi request. Lưu ý rằng điều này gây ra lỗi do phần body request thiếu tham số `price`.

12. Thêm tham số `price` có giá trị `0` vào đối tượng JSON `{"price":0}`. Gửi request.

13. Trong trình duyệt của Burp, hãy tải lại trang sản phẩm áo khoác da. Lưu ý rằng giá của áo khoác da hiện là `$0,00`.

14. Thêm áo khoác da vào giỏ hàng.

15. Vào giỏ hàng và nhấp vào `Place order` để solve lab.

### Using Intruder to find hidden endpoints

Sau khi xác định được một số endpoint API ban đầu, có thể sử dụng Intruder để khám phá các endpoint ẩn. Ví dụ, hãy xem xét một kịch bản đã xác định được endpoint API sau để cập nhật thông tin người dùng:

`PUT /api/user/update`

Để xác định các endpoint ẩn, có thể sử dụng Burp Intruder để tìm các tài nguyên khác có cùng cấu trúc. Ví dụ: có thể thêm một payload vào vị trí `/update` của đường dẫn với danh sách các hàm phổ biến khác, chẳng hạn như `delete` và `add`.

Khi tìm kiếm các endpoint ẩn, hãy sử dụng wordlist dựa trên các quy ước đặt tên API phổ biến và các thuật ngữ trong ngành. Đảm bảo rằng bao gồm các thuật ngữ có liên quan đến ứng dụng, dựa trên quá trình trinh sát ban đầu.

## Finding hidden parameters

Khi đang thực hiện recon API, có thể tìm thấy các tham số không có trong tài liệu mà API hỗ trợ. Có thể thử sử dụng các tham số này để thay đổi hành vi của ứng dụng. Burp bao gồm nhiều công cụ có thể giúp xác định các tham số ẩn:

- Burp Intruder cho phép tự động khám phá các tham số ẩn, sử dụng wordlist gồm các tên tham số phổ biến để thay thế các tham số hiện có hoặc thêm các tham số mới. Đảm bảo bao gồm các tên có liên quan đến ứng dụng, dựa trên recon ban đầu.

- Param miner BApp cho phép tự động đoán tới 65.536 tên tham số cho mỗi request. Param miner tự động đoán các tên có liên quan đến ứng dụng, dựa trên thông tin lấy từ phạm vi.

- Công cụ Content discovery cho phép khám phá nội dung không được liên kết từ nội dung hiển thị có thể duyệt đến, bao gồm các tham số.

## Mass assignment vulnerabilities

Việc gán hàng loạt (còn được gọi là tự động liên kết) có thể vô tình tạo ra các tham số ẩn. Nó xảy ra khi các framework tự động liên kết các tham số request với các trường trên một đối tượng nội bộ. Do đó, việc gán hàng loạt có thể dẫn đến việc ứng dụng hỗ trợ các tham số mà nhà phát triển không bao giờ có ý định xử lý.

### Identifying hidden parameters

Vì việc gán hàng loạt tạo ra các tham số từ các trường đối tượng, thường có thể xác định các tham số ẩn này bằng cách kiểm tra thủ công các đối tượng được API trả về.

Ví dụ, hãy xem xét request `PATCH /api/users/`, cho phép người dùng cập nhật tên người dùng và email của họ và bao gồm JSON sau:

```
{
    "username": "wiener",
    "email": "wiener@example.com",
}
```

Request `GET /api/users/123` đồng thời trả về JSON sau:

```
{
    "id": 123,
    "name": "John Doe",
    "email": "john@example.com",
    "isAdmin": "false"
}
```

Điều này có thể chỉ ra rằng các tham số `id` và `isAdmin` ẩn được liên kết với đối tượng người dùng nội bộ, cùng với các tham số tên người dùng và email đã cập nhật.

### Testing mass assignment vulnerabilities

Để kiểm tra xem có thể sửa đổi giá trị tham số `isAdmin` được liệt kê hay không, hãy thêm nó vào yêu cầu `PATCH`:

```
{
    "username": "wiener",
    "email": "wiener@example.com",
    "isAdmin": false,
}
```

Ngoài ra, hãy gửi request `PATCH` với giá trị tham số `isAdmin` không hợp lệ:

```
{
    "username": "wiener",
    "email": "wiener@example.com",
    "isAdmin": "foo",
}
```

Nếu ứng dụng hoạt động khác, điều này có thể cho thấy giá trị không hợp lệ ảnh hưởng đến logic truy vấn, nhưng giá trị hợp lệ thì không. Điều này có thể chỉ ra rằng người dùng có thể cập nhật tham số thành công.

Sau đó, có thể gửi yêu cầu `PATCH` với giá trị tham số `isAdmin` được đặt thành `true` để thử khai thác lỗ hổng:

```
{
    "username": "wiener",
    "email": "wiener@example.com",
    "isAdmin": true,
}
```

Nếu giá trị `isAdmin` trong request được liên kết với đối tượng người dùng mà không có xác thực và vệ sinh đầy đủ, người dùng `wiener` có thể được cấp quyền quản trị không đúng cách. Để xác định xem đây có phải là trường hợp hay không, hãy duyệt ứng dụng dưới dạng `wiener` để xem có thể truy cập chức năng quản trị hay không.

### Lab: Exploiting a mass assignment vulnerability

Để solve lab, tìm và khai thác lỗ hổng mass assignment để mua `Lightweight l33t Leather Jacket`. Có thể đăng nhập vào tài khoản bằng thông tin đăng nhập sau: `wiener:peter`.

#### Solution

1. Trong trình duyệt của Burp, đăng nhập vào ứng dụng bằng thông tin đăng nhập `wiener:peter`.

2. Click vào sản phẩm `Lightweight "l33t" Leather Jacket` và thêm vào giỏ hàng.

3. Vào giỏ hàng và click vào `Place order`. Lưu ý rằng không có đủ credit để mua hàng.

4. Trong `Proxy > HTTP History`, hãy lưu ý cả request API `GET` và `POST` cho `/api/checkout`.

5. Lưu ý rằng response cho request `GET` chứa cùng cấu trúc JSON với request `POST`. Lưu ý rằng cấu trúc JSON trong response `GET` bao gồm tham số `chosen_discount`, không có trong request `POST`.

6. Nhấp chuột phải vào request `POST /api/checkout` và chọn `Send to Repeater`.

7. Trong Repeater, thêm tham số `chosen_discount` vào request. JSON sẽ trông như sau:

    ```
    {
        "chosen_discount":{
            "percentage":0
        },
        "chosen_products":[
            {
                "product_id":"1",
                "quantity":1
            }
        ]
    }
    ```

8. Gửi request. Lưu ý rằng việc thêm tham số `chosen_discount` không gây ra lỗi.

9. Thay đổi giá trị `chosen_discount` thành chuỗi `"x"`, sau đó gửi request. Lưu ý rằng điều này dẫn đến thông báo lỗi vì giá trị tham số không phải là số. Điều này có thể chỉ ra rằng đầu vào của người dùng đang được xử lý.

10. Thay đổi phần trăm `chosen_discount` thành `100`, sau đó gửi request để solve lab.

## Preventing vulnerabilities in APIs

Khi thiết kế API, hãy đảm bảo rằng bảo mật được cân nhắc ngay từ đầu. Cụ thể, hãy đảm bảo rằng:

- Bảo mật tài liệu nếu không muốn API có thể truy cập công khai.

- Đảm bảo tài liệu được cập nhật để những người kiểm tra hợp pháp có thể nhìn thấy toàn bộ bề mặt tấn công của API.

- Áp dụng danh sách cho phép các phương thức HTTP được phép.

- Xác thực rằng loại nội dung được mong đợi cho mỗi request hoặc response.

- Sử dụng thông báo lỗi chung để tránh tiết lộ thông tin có thể hữu ích cho kẻ tấn công.

- Sử dụng các biện pháp bảo vệ trên tất cả các phiên bản API, không chỉ phiên bản product hiện tại.

Để ngăn chặn các lỗ hổng mass assignment, hãy cho phép các thuộc tính có thể được người dùng cập nhật và chặn các thuộc tính nhạy cảm không nên được người dùng cập nhật.

## Server-side parameter pollution

Một số hệ thống chứa API nội bộ không thể truy cập trực tiếp từ internet. Parameter pollution phía máy chủ xảy ra khi một trang web nhúng dữ liệu đầu vào của người dùng vào yêu cầu phía máy chủ tới API nội bộ mà không có mã hóa đầy đủ. Điều này có nghĩa là kẻ tấn công có thể thao túng hoặc chèn các tham số, ví dụ như:

- Ghi đè các tham số hiện có.

- Sửa đổi hành vi của ứng dụng.

- Truy cập dữ liệu trái phép.

Có thể kiểm tra bất kỳ dữ liệu đầu vào nào của người dùng để tìm bất kỳ loại parameter pollution nào. Ví dụ: tham số truy vấn, trường form, tiêu đề và tham số đường dẫn URL đều có thể bị tấn công.

![alt text](image.png)

```
Lưu ý: Lỗ hổng này đôi khi được gọi là HTTP parameter pollution. Tuy nhiên, thuật ngữ này cũng được sử dụng để chỉ kỹ thuật bỏ qua tường lửa ứng dụng web (WAF). Để tránh nhầm lẫn, trong chủ đề này sẽ chỉ đề cập đến parameter pollution phía máy chủ.

Ngoài ra, mặc dù có tên tương tự, lớp lỗ hổng này có rất ít điểm chung với prototype pollution phía máy chủ.
```

## Testing for server-side parameter pollution in the query string

Để kiểm tra parameter pollution phía máy chủ trong chuỗi truy vấn, hãy đặt các ký tự cú pháp truy vấn như `#`, `&` và `=` vào input và quan sát cách ứng dụng phản hồi.

Xem xét một ứng dụng dễ bị tấn công cho phép tìm kiếm người dùng khác dựa trên tên người dùng của họ. Khi tìm kiếm người dùng, trình duyệt sẽ đưa ra request sau:

```
GET /userSearch?name=peter&back=/home
```

Để lấy thông tin người dùng, máy chủ sẽ truy vấn API nội bộ với yêu cầu sau:

```
GET /users/search?name=peter&publicProfile=true
```

### Truncating query strings

Có thể sử dụng ký tự `#` được mã hóa theo URL để cố gắng cắt bớt request phía máy chủ. Để giúp diễn giải response, cũng có thể thêm một chuỗi sau ký tự `#`.

Ví dụ: có thể sửa đổi chuỗi truy vấn thành như sau:

```
GET /userSearch?name=peter%23foo&back=/home
```

Giao diện người dùng sẽ cố gắng truy cập vào URL sau:

```
GET /users/search?name=peter#foo&publicProfile=true
```

```
Lưu ý: Điều cần thiết là phải mã hóa URL ký tự #. Nếu không, ứng dụng front-end sẽ hiểu ký tự này là một mã định danh phân đoạn và sẽ không được chuyển đến API nội bộ.
```

Xem lại response để tìm manh mối về việc truy vấn có bị cắt bớt hay không. Ví dụ: nếu response trả về người dùng `peter`, truy vấn phía máy chủ có thể đã bị cắt bớt. Nếu thông báo lỗi `Invalid name` được trả về, ứng dụng có thể đã coi `foo` là một phần của tên người dùng. Điều này cho thấy yêu cầu phía máy chủ có thể không bị cắt bớt.

Nếu có thể cắt bớt yêu cầu phía máy chủ, điều này sẽ xóa yêu cầu trường `publicProfile` phải được đặt thành true. Có thể khai thác điều này để trả về hồ sơ người dùng không công khai.

### Injecting invalid parameters

Có thể sử dụng ký tự `&` được mã hóa theo URL để thử thêm tham số thứ hai vào request phía máy chủ.

Ví dụ: có thể sửa đổi chuỗi truy vấn thành như sau:

```
GET /userSearch?name=peter%26foo=xyz&back=/home
```

Điều này dẫn đến request phía máy chủ sau đây tới API nội bộ:

```
GET /users/search?name=peter&foo=xyz&publicProfile=true
```

Xem lại response để tìm manh mối về cách phân tích cú pháp tham số bổ sung. Ví dụ: nếu response không thay đổi, điều này có thể chỉ ra rằng tham số đã được đưa vào thành công nhưng bị ứng dụng bỏ qua.

Để xây dựng bức tranh hoàn chỉnh hơn, cần phải thử nghiệm thêm.

Nếu có thể sửa đổi chuỗi truy vấn, sau đó có thể thử thêm tham số hợp lệ thứ hai vào request phía máy chủ.

Ví dụ, nếu đã xác định được tham số email, có thể thêm tham số đó vào chuỗi truy vấn như sau:

```
GET /userSearch?name=peter%26email=foo&back=/home
```

Điều này dẫn đến request phía máy chủ sau đây tới API nội bộ:

```
GET /users/search?name=peter&email=foo&publicProfile=true
```

Xem lại response để tìm manh mối về cách phân tích tham số bổ sung.

### Overriding existing parameters

Để xác nhận xem ứng dụng có dễ bị parameter pollution phía máy chủ hay không, có thể thử ghi đè tham số gốc. Thực hiện việc này bằng cách chèn tham số thứ hai có cùng tên.

Ví dụ: có thể sửa đổi chuỗi truy vấn thành như sau:

```
GET /userSearch?name=peter%26name=carlos&back=/home
```

Điều này dẫn đến yêu cầu phía máy chủ sau đây tới API nội bộ:

```
GET /users/search?name=peter&name=carlos&publicProfile=true
```

API nội bộ diễn giải hai tham số `name`. Tác động của điều này phụ thuộc vào cách ứng dụng xử lý tham số thứ hai. Điều này thay đổi tùy theo các công nghệ web khác nhau. Ví dụ:

- PHP chỉ phân tích cú pháp tham số cuối cùng. Điều này sẽ dẫn đến việc người dùng tìm kiếm `carlos`.

- ASP.NET kết hợp cả hai tham số. Điều này sẽ dẫn đến việc người dùng tìm kiếm `peter,carlos`, có thể dẫn đến thông báo lỗi `Invalid username`.

- Node.js / express chỉ phân tích cú pháp tham số đầu tiên. Điều này sẽ dẫn đến việc người dùng tìm kiếm `peter`, đưa ra kết quả không thay đổi.

Nếu có thể ghi đè tham số gốc, có thể thực hiện khai thác. Ví dụ, có thể thêm `name=administrator` vào request. Điều này có thể cho phép đăng nhập với tư cách là người dùng quản trị viên.

### Lab: Exploiting server-side parameter pollution in a query string

Để solve lab, đăng nhập với `administrator` và xóa `carlos`.

#### Solution

1. Trong trình duyệt Burp, kích hoạt đặt lại mật khẩu cho `administrator`.

2. Trong `Proxy > HTTP history`, lưu ý request `POST /forgot-password` và file JavaScript `/static/js/forgotPassword.js` liên quan.

3. Nhấp chuột phải vào request `POST /forgot-password` và chọn `Sent to Repeater`.

4. Trong tab `Repeater`, gửi lại request để xác nhận rằng response là nhất quán.

5. Thay đổi giá trị của tham số `username` từ `administrator` thành tên người dùng không hợp lệ, chẳng hạn như `administratorx`. Gửi request. Lưu ý rằng điều này dẫn đến thông báo lỗi `Invalid username`.

6. Thử thêm cặp tham số-giá trị thứ hai vào yêu cầu phía máy chủ bằng ký tự `&` được mã hóa URL. Ví dụ: thêm `&x=y` được mã hóa URL:

    ```
    username=administrator%26x=y
    ```

Gửi request. Lưu ý rằng điều này trả về thông báo lỗi `Parameter is not supported`. Điều này cho thấy API nội bộ có thể đã diễn giải `&x=y` là một tham số riêng biệt, thay vì là một phần của tên người dùng.

7. Cố gắng cắt bớt chuỗi truy vấn phía máy chủ bằng ký tự # được mã hóa theo URL:

    ```
    username=administrator%23
    ```

Gửi request. Lưu ý rằng điều này trả về thông báo lỗi `Field not specified`. Điều này cho thấy rằng truy vấn phía máy chủ có thể bao gồm một tham số bổ sung được gọi là `field`, đã bị xóa bởi ký tự `#`.

8. Thêm tham số `field` có giá trị không hợp lệ vào request. Cắt bớt chuỗi truy vấn sau cặp tham số-giá trị đã thêm. Ví dụ: thêm `&field=x#` được mã hóa theo URL:

    ```
    username=administrator%26field=x%23
    ```

Gửi request. Lưu ý rằng điều này dẫn đến thông báo lỗi `Invalid field`. Điều này cho thấy rằng ứng dụng phía máy chủ có thể nhận ra tham số trường đã đưa vào.

9. Brute-force giá trị của tham số trường:

    1. Nhấp chuột phải vào yêu cầu `POST /forgot-password` và chọn `Send to Intruder`.

    2. Trong tab `Intruder`, thêm vị trí payload vào giá trị của tham số `field` như sau:

        ```
        username=administrator%26field=§x§%23
        ```

    3. Trong `Intruder > Payloads`, nhấp vào `Add from list`. Chọn danh sách payload `Server-side variable names` được tích hợp sẵn, sau đó bắt đầu tấn công.

    4. Xem lại kết quả. Lưu ý rằng các request có payload username và email đều trả về phản hồi `200`.

10. Thay đổi giá trị của tham số `field` từ `x#` thành `email`:

    ```
    username=administrator%26field=email%23
    ```

Gửi request. Lưu ý rằng điều này trả về request ban đầu. Điều này cho thấy `email` là một kiểu trường hợp lệ.

11. Trong `Proxy > HTTP history`, xem lại file JavaScript `/static/js/forgotPassword.js`. Lưu ý endpoint đặt lại mật khẩu, tham chiếu đến tham số `reset_token`:

    ```
    /forgot-password?reset_token=${resetToken}
    ```

12. Trong tab `Repeater`, thay đổi giá trị của tham số `field` từ `email` thành `reset_token`:

    ```
    username=administrator%26field=reset_token%23
    ```

Gửi request. Lưu ý rằng điều này trả về mã thông báo đặt lại mật khẩu. Ghi nhớ điều này.

13. Trong trình duyệt của Burp, hãy nhập endpoint đặt lại mật khẩu vào thanh địa chỉ. Thêm reset token mật khẩu làm giá trị của tham số `reset_token`. Ví dụ:

    ```
    /forgot-password?reset_token=123456789
    ```

14. Đặt mật khẩu mới.

15. Đăng nhập với tư cách là `administrator`.

16. Vào `Admin panel` và xóa `carlos` để solve lab.

## Testing for server-side parameter pollution in REST paths

API RESTful có thể đặt tên và giá trị tham số trong đường dẫn URL, thay vì chuỗi truy vấn. Ví dụ, xem xét đường dẫn sau:

```
/api/users/123
```

Đường dẫn URL có thể được chia nhỏ như sau:

- `/api` là endpoint API gốc.

- `/users` biểu thị một tài nguyên, trong trường hợp này là `users`.

- `/123` biểu thị một tham số, ở đây là một mã định danh cho người dùng cụ thể.

Xem xét một ứng dụng cho phép chỉnh sửa hồ sơ người dùng dựa trên tên người dùng của họ. Các request được gửi đến endpoint sau:

```
GET /edit_profile.php?name=peter
```

Điều này dẫn đến request phía máy chủ sau:

```
GET /api/private/users/peter
```

Kẻ tấn công có thể thao túng các tham số đường dẫn URL phía máy chủ để khai thác API. Để kiểm tra lỗ hổng này, hãy thêm các chuỗi duyệt đường dẫn để sửa đổi các tham số và quan sát cách ứng dụng phản hồi.

Bạn có thể gửi `peter/../admin` được mã hóa URL làm giá trị của tham số `name`:

```
GET /edit_profile.php?name=peter%2f..%2fadmin
```

Điều này có thể dẫn đến request phía máy chủ sau:

```
GET /api/private/users/peter/../admin
```

Nếu máy khách phía máy chủ hoặc API phía sau chuẩn hóa đường dẫn này, đường dẫn này có thể được resolved thành `/api/private/users/admin`.

## Testing for server-side parameter pollution in structured data formats

Kẻ tấn công có thể thao túng các tham số để khai thác lỗ hổng trong quá trình xử lý các định dạng dữ liệu có cấu trúc khác của máy chủ, chẳng hạn như JSON hoặc XML. Để kiểm tra điều này, hãy đưa dữ liệu có cấu trúc không mong muốn vào dữ liệu đầu vào của người dùng và xem máy chủ phản hồi như thế nào.

Xem xét một ứng dụng cho phép người dùng chỉnh sửa hồ sơ của họ, sau đó áp dụng các thay đổi của họ bằng yêu cầu tới API phía máy chủ. Khi chỉnh sửa tên, trình duyệt sẽ thực hiện request sau:

```
POST /myaccount
name=peter
```

Điều này dẫn đến request phía máy chủ sau:

```
PATCH /users/7312/update
{"name":"peter"}
```

Có thể thử thêm tham số `access_level` vào request như sau:

```
POST /myaccount
name=peter","access_level":"administrator
```

Nếu dữ liệu đầu vào của người dùng được thêm vào dữ liệu JSON phía máy chủ mà không có xác thực hoặc khử trùng đầy đủ, điều này sẽ dẫn đến yêu cầu phía máy chủ sau:

```
PATCH /users/7312/update
{name="peter","access_level":"administrator"}
```

Điều này có thể dẫn đến việc người dùng `peter` được cấp quyền quản trị viên.

Xem xét một ví dụ tương tự, nhưng dữ liệu đầu vào của người dùng phía máy khách là dữ liệu JSON. Khi chỉnh sửa tên, trình duyệt thực hiện request sau:

```
POST /myaccount
{"name": "peter"}
```

Điều này dẫn đến request phía máy chủ sau:

```
PATCH /users/7312/update
{"name":"peter"}
```

Có thể thử thêm tham số `access_level` vào request như sau:

```
POST /myaccount
{"name": "peter\",\"access_level\":\"administrator"}
```

Nếu dữ liệu đầu vào của người dùng được giải mã, sau đó thêm vào dữ liệu JSON phía máy chủ mà không có mã hóa đầy đủ, điều này sẽ dẫn đến yêu cầu phía máy chủ sau:

```
PATCH /users/7312/update
{"name":"peter","access_level":"administrator"}
```

Một lần nữa, điều này có thể dẫn đến việc người dùng `peter` được cấp quyền quản trị viên.

Injection format có cấu trúc cũng có thể xảy ra trong response. Ví dụ, điều này có thể xảy ra nếu dữ liệu đầu vào của người dùng được lưu trữ an toàn trong cơ sở dữ liệu, sau đó được nhúng vào phản hồi JSON từ API phụ trợ mà không có mã hóa đầy đủ. Có thể phát hiện và khai thác injection format có cấu trúc trong response theo cùng cách có thể làm trong request.

```
Lưu ý: Ví dụ bên dưới là JSON, nhưng parameter pollution phía máy chủ có thể xảy ra ở bất kỳ định dạng dữ liệu có cấu trúc nào.
```

## Testing with automated tools

Burp bao gồm các công cụ tự động có thể giúp phát hiện các lỗ hổng parameter pollution phía máy chủ.

Burp Scanner tự động phát hiện các chuyển đổi đầu vào đáng ngờ khi thực hiện kiểm tra. Những chuyển đổi này xảy ra khi ứng dụng nhận được đầu vào của người dùng, chuyển đổi theo một cách nào đó, sau đó thực hiện xử lý thêm trên kết quả. Hành vi này không nhất thiết cấu thành lỗ hổng, vì vậy cần phải kiểm tra thêm bằng các kỹ thuật thủ công được nêu ở trên. 

Cũng có thể sử dụng Backslash Powered Scanner BApp để xác định các lỗ hổng injection phía máy chủ. Máy quét phân loại các đầu vào thành boring, interesting hoặc vulnerable. Cần phải điều tra các đầu vào interesting bằng các kỹ thuật thủ công được nêu ở trên.

## Preventing server-side parameter pollution

Để ngăn ngừa parameter pollution phía máy chủ, hãy sử dụng danh sách cho phép để xác định các ký tự không cần mã hóa và đảm bảo tất cả dữ liệu đầu vào khác của người dùng được mã hóa trước khi đưa vào yêu cầu phía máy chủ. Nên đảm bảo rằng tất cả dữ liệu đầu vào đều tuân thủ định dạng và cấu trúc mong đợi.