###**IV - Starting up**

####**Install Node**
Nếu máy tính của bạn chưa có Node hoặc bạn muốn cập nhật phiên bản mới nhất của Node thì bạn có thể truy cập vào trang  http://nodejs.org và nhấp vào chỗ Install để cài đặt phiên bản mới nhất của Node. 

Tùy thuộc vào nền tảng hệ điều hành mà bạn đang dùng để bạn chọn package cài đặt phù hợp. 

Sau khi cài đặt xong, bạn nên kiểm tra lại version của Node đã được cài bằng dòng lệnh:
```sh
	$ node -v
	v0.8.12
```
Bạn có thể có 2 cách để dùng Node thực thi, đó là thông qua CLI (command line interface) hoặc qua file.

Nếu sử dụng CLI, bạn chỉ cần gõ:
```sh
	$ node
```
và bạn sẽ nhận được 1 JavaScript command line prompt để bạn gõ các câu lệnh JavaScript và thực thi các câu lệnh đó. 

Bạn cũng có thể dùng Node để chạy 1 file JavaScript. Đầu tiên Node sẽ parse và evaluate các câu lệnh JavaScript trong file đó, sau khi xong thì nó sẽ bước vào event loop. Một khi vào trong event loop, Node sẽ thoát nếu không có gì để làm hoặc sẽ chờ đợi và lắng nghe các events.

Bạn có thể dùng Node để run file như sau:
```sh
	$ node myfile.js
```
hoặc nếu muốn, bạn cũng có thể làm cho file của bạn thực thi 1 cách trực tiếp bằng cách thêm quyền vào file đó như sau:
```sh
	$ chmod o+x myfile.js
```
sau đó chèn vào dòng đầu tiên của file dòng sau:
```sh
	#!/usr/bin/env node
```
Bây giờ bạn có thể thực thi file của bạn 1 cách trực tiếp:
```sh
	 $ ./myfile.js
```

####**NPM - Node Package Manager**
NPM đã trở thành tiêu chuẩn để quản lý các package Node trong suốt thời gian qua, và sự hợp tác chặt chẽ giữa Isaac Schlueter - người tạo ra ra NPM và Ryan Dahl - tác giả và cũng là người duy trì Node  - đã thắt chặt mỗi quan hệ này hơn tới mức từ version 0.4.0, Node hỗ trợ file có định dạng package.json để định dạng các package và các dependences khởi đầu của ứng dụng bên trong file packafe.json. NPM cũng sẽ được cài đặt cùng khi bạn cài đặt Node.

#####**Global vs Local**
NPM có 2 cách khác nhau cơ bản để làm việc: local và global.

Trong chế độ global, tất cả các package được cài đặt bên trong 1 thư mục chia sẻ và bạn chỉ có thể giữ được 1 version cho mỗi package (để tránh xung đột?)

Trong chế độ local, bạn có thể có các package cài đặt khác nhau trong mỗi thư mục. Trong chế độ local, NPM sẽ lưu 1 thư mục local có tên là ***node_modules*** để chứa các module được cài đặt.  Bằng cách chuyển qua lại giữa các thư mục local khác nhau, bạn có thể chuyển tới các thư mục có context local khác nhau, nghĩa là chứa các module được cài đặt trong mỗi thư mục. 

Mặc định, NPM làm việc ở chế độ local. Để chuyển sang chế độ global, bạn phải thêm cờ ***'-g'*** vào trong bất kì câu lệnh nào ở phần dưới.

#####**NPM commands**
NPM có thể được sử dụng thông qua command line. Câu lệnh cơ bản trong NPM là:
```sh
	$ npm ls [filter]
```
Sử dụng câu lệnh trên để xem danh sách tất cả các package và version của chúng (npm ls mà không có filter) hoặc filter theo 1 tag (npm filter tag). Ví dụ:

Liệt kê tất cả các package đã cài đặt:
```sh
	$ npm ls installed
```
Liệt kê tất cả các stable package:
```sh
	$ npm ls stable
```
Ta cũng có thể kết hợp cả 2 cách filter trên:
```sh
	$ npm ls installed stable 
```
Ta cũng có thể dùng npm để tìm kiếm theo tên của package:
```sh
	$ npm ls fug
```
Câu lệnh trên sẽ trả về tất cả các package mà có ***'fug'*** trong tên package.

Bạn cũng có thể tìm các package theo version, bắt đầu bằng kí tự ***'@'***:
```sh
	$ npm ls @1.0
```
Để cài đặt 1 package và tất cả các package dependency của nó, bạn có thể dùng lệnh:
```sh
	$ npm install package_name
```
Câu lệnh trên sẽ cài đặt phiên bản mới nhất của package.
Ví dụ:
```sh
	$ npm install express
```
Trong trường hợp bạn không muốn cài đặt phiên bản mới nhất mà muốn tải 1 package với version nào đó, bạn có thể dùng lệnh:
```sh
	$ npm install package_name@version
```
Ví dụ:
```sh
	$ npm install experss@2.0.0beta
```
Để cài đặt phiên bản cao nhất trong giới hạn version mà bạn xác định, bạn có thể thực hiện như sau:
```sh
	$ npm install express@">=0.1.0
```
Tất cả các câu lệnh trong các ví dụ trên đều cài đặt các package và trong thư mục local. Để cài đặt 1 package vào trong global, 	bạn sử dụng thêm cờ ***'-g'*** vào trong câu lệnh cài đặt như trong ví dụ sau:
```sh
	$ npm install -g express
```
Để xóa 1 hoặc nhiều package trong node, bạn sử dụng lệnh:
```sh
	$ npm rm package_name[@version] [package_name[@version]...]
```
Nếu không có version thì tất cả các version được tìm thấy với package_name tương ứng sẽ bị xóa.
Ví dụ: 
```sh
	$ npm rm sax
```
Để xóa 1 package đã được cài đặt trong global, bạn phải thêm ***'-g'*** vào trong câu lệnh xóa:
```sh
	$ npm rm -g express
```
Để xem tất cả các thông tin của 1 hoặc nhiều package, sử dụng lệnh:
```sh
	$ npm view package_name[@version] [package_name[@version]...]
```
Mặc định sẽ hiển thị thông tin của version mới nhất nếu không có version được chỉ định.

Ví dụ:
```sh
	$ npm view connect
	$ npm view connect@1.0.3
```
Trên đây chỉ là các câu lệnh cơ bản về NPM. Bạn nên tìm hiểu thêm về NPM để biết cách bundle và 'freeze' (bó và đóng băng) các dependencies package của 1 ứng dụng.
