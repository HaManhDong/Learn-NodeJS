
##**Hands-on NodeJS**

###**I - Introduce**

Tại European JSConf 2009, một lập trình viên trẻ tuổi tên là Ryan Dahl đã giới thiệu 1 project mà anh ấy đã thực hiện. Project này là 1 platfrom kết hợp V8 JavaScript engine của Google và một event loop. Project này điều khiển các tiến trình khác với các server-side JavaScript platfrom khác: tất cả các I/O nguyên thủy đều là event-driven, và không có gì khác ngoài nó. Lợi dụng các khả năng và tính đơn giản của JavaScript đã làm cho việc tạp ra các tác vụ trong các ứng dụng không đồng bộ trở nên dễ dàng hơn. Project của Dahl đã nhận được sự ủng hộ của mọi người, và đã phát triển, trở nên phổ biến hơn.
Project có tên là Node.js, bây giờ được biết với cái tên đơn giản hơn là ‘Node’. Node cung cấp hoàn toàn là các event trên cấu trúc non-blocking để xây dựng nên các phần mềm hiện nay.
Node giúp ta dễ dàng xây dựng 1 mạng lưới các service một cách nhanh chóng và có tính mở rộng cao. 


**Mục tiêu quyển sách:**
1. Tại sao NodeJS lại phổ biến hơn các server-side solution khác
2. Tại sao nên sử dụng NodeJS
3. Bắt đầu NodeJS như thế nào

**Những điểm mà sách không đề cập đến:**
**1.** Không chú trọng vào việc hoàn thành Node API mà sẽ cover đến những     gì tác giả của quyển sách (Pedro Teixeira) cho rằng là cần thiết để xây dựng nên 1 ứng dụng dựa trên Node
**2.** Không cover bất kì 1 Node framework nào. Vì có rất nhiều framework thông dụng và phổ biến được xây dựng trên NodeJS có sẵn như: cluster management, inter-process communication, web frameworks, network  traffic collection tools, game engines ... Nhưng trước khi nhảy vào tìm hiểu 1 trong các framework trên thì ta nên làm quen và tìm hiểu với kiến trúc của Node và những gì mà Node cung cấp 

**Điều kiện tiên quyết:**
Sách này hướng đến ngay cả những người chưa có bất kì kiến thức nào về Node, nhưng sẽ có các đoạn code ví dụ  được viết bằng Javascript, nên bạn đọc nên tìm hiểu trước về ngôn ngữ Javascript.

**Những gì cần hiểu được sau khi đọc:**
Hiểu được Node API để có thể tìm hiểu các adaptors, frameworks và cá modules được xây dựng trên các Node API.
 
###***II - Chapter Overview***
**Why?**

Chương này đề cập đến vấn đề tại sao Node sử dụng event-driven programing và JavaScript với nhau? Tại sao JavaScript lại là ngôn ngữ được lựa chọn?

**Starting up?**

Cách để cài đặt Node và tải các Node modules sử dụng NPM

**Understanding Basics**

Cơ chế Node load các modules, tìm hiểu về Event Loop functions và … (chưa dịch) (How Node loads modules, how the Event Loop functions and what to look out for in order not to block it.)

**API Quick Tour**

Tìm hiểu sơ qua về các module chính bên trong Node

**Utilities**

Sử dụng Node một cách hữu ích

**Buffers**

Học cách tạo, chỉnh sửa và truy cập buffer data, đây là 1 phần cần thiết của Node

**Event Emitter**

Cách mà Event Emitter pattern được sử dụng trong Node và cách sử dụng Evetn Emitter pattern 1 cách mềm dẻo trong code của bạn

**Timer**

Tìm hiểu về timer API của Node

**Low-level File System**

Tìm hiểu về việc Node mở, đọc và viết các file

**HTTP**

Cách implementation HTTP server và client của Node

**Streams**

Nói về các lớp abstration trong node

**TCP Server**

Setup 1 TCP server tối thiểu (bare TCP server)

**UNIX Sockets**

Cách sử dụng UNIX sockets và sử dụng chúng để pass các file descriptors around

**Datagrams (UDP)**

Khả năng về datagram trong Node

**Child Processes**

Launching, watching, piping and killing other processes

**Streaming HTTP Chunked Responses**

Http trong Node là các luồng

**TLS / SSL**

Cách để cung cấp và hủy các luồng 1 cách an toàn(secure streams)

**HTTPS**

Cách xây dựng 1 HTTPS server hoặc client an toàn

**Making Modules**

Cách module hóa ứng dụng

**Debugging**

Cách debug Node app

**Automated Unit Testing**

Cách test từng phần trong các module

**Callback Flow**

Cách quản lý các callback phức tạp một cách chuẩn
 
###**III - WHY?**

####**Why the event loop?**

Event Loop là 1 mô hình phần mềm (software pattern) sử dụng non-blocking I/O (sự giao tiếp của mạng, file hoặc các process bên trong). Các chương trình traditional blocking dùng I/O giống như các hàm được gọi 1 cách thường lệ (regular functio calls), các tiến trình khác sẽ không được tiếp tục cho đến khi tiến trình của I/O thực hiện xong. Dưới đây là 1 đoạn giả mã về blocking I/O:
```sh
	var post = db.querry(‘SELECT * FROM posts where id = 1’);
	// tiến trình xử lý các câu lệnh bên dưới sẽ không được thực thi cho 
	// đến khi tiến trình xử lý câu lệnh truy vấn bên trên hoàn thành
	doSomethingWithPost(post);
	doSomethingElse();
```
Vấn đề xảy ra trong đoạn mã code trên đó là trong khi truy vấn vào database đang được thực thi thì toàn bộ các process/thread không làm gì mà đợi cho đến khi có kết quả truy vấn trả về. Đây gọi là blocking. Kết quả cho câu lệnh truy vấn phải mất hàng nghìn chu trình của CPU, rendering ra toàn bộ process không được sử dụng trong suốt khoảng thời gian này. Các process nên được phục vụ ở các client request khác thay vì phải chờ đợi.

Blocking I/O khiến bạn không thể thực hiện các luồng I/O song song như các truy vấn hoặc giao tiếp với database khác khi thực hiện request từ 1 web service từ xa. Khi đó các tiến trình trong hàng đợi để được cấp CPU bị đóng băng, chờ cho đến khi có kết quả trả về từ database server.

Có 2 cách để xử lý vấn đề blocking này, nghĩa là làm cho các tiến trình khác vẫn hoạt động bình thường trong khi chờ, đó là:  tạo nhiều call stacks hoặc sử dụng event callbacks.

**Cách 1: tạo nhiều các call stack**

Để chương trình của bạn có thể xử lý nhiều các tiến trình I/O đồng thời, bạn phải có nhiều các call stack. Để tạo ra nhiều call stack, bạn có thể sử dụng các thread hoặc 1 số loại của cooperative multi-threading scheme như co-routines, fibers, continuations…

Mô hình đa luồng này rất khó để định dạng, hiểu và debug, lý do chính là vì sự phức tạp về đồng bộ trong khi đang truy cập và thay đổi trạng thái của các tiến trình. Bạn sẽ không thể nào biết được khi nào thì thread mà bạn đang chạy sẽ bị tạm dừng để nhường CPU, điều này có thể dẫn tới các lỗi kỳ quặc và khó để giải quyết.

Có 1 cách xử lý khác trong trường hợp này là sử dụng cooperative multi-threading. Cooperative multi-threading là 1 “trick” để bạn có thể có nhiều hơn 1 stack, và mỗi thread sẽ có 1 bộ như là bảng thời gian để có thể thông báo thời gian của thread đó tới các thread khác. Điều này có thể tránh được vấn đề bất đồng bộ nhưng lại dẫn tới sự phức tạp và error-prone, khi thread tính toán sai thời gian xử lý của nó và gửi đi cho các thread khác. 

**Cách 2: sử dụng event-driven I/O**

Event-driven I/O là 1 scheme để ta có thể ‘đăng kí’ các callback function, các callback function này sẽ được gọi lại khi có 1 I/O event. 
Một event callback là 1 function được gọi lại khi có một tín hiệu nào đó xảy ra như: kết quả trả về từ câu lệnh truy vấn database đã có…
Để sử dụng event callback trong ví dụ trước, ta có thể thay đổi lại đoạn giả mã như sau:

```sh
	callback = function(post) {
		doSomethingWithPost(post); 
		// hàm này sẽ chỉ được thực thi khi db.query trả về kết quả
	};
	db.query(‘SELECT * FROM posts where id=1’, callback);
	doSomethingElese(); // hàm này sẽ được thực thi mà 
	// không phụ thuộc vào trạng thái trả về từ câu lệnh db.query
```

Trong đoạn code trên, ta đã định nghĩa 1 function để gọi lại khi quá trình truy vấn db hoàn thành, để thực hiện được gọi lại function này thì ta truyền function này vào là 1 tham số trong câu lệnh truy vấn. Khi đó quá trình truy vấn db sẽ chịu trách nhiện về việc thực thi hàm callback khi kết quả trả về đã sẵn sàng.
Bạn có thể sử dụng một cách viết khác để tạo ra 1 function callback như sau:

```sh
	db.query(‘SELECT * FROM posts where id=1’,function(post) {
		doSomethingWithPost(post); // hàm này sẽ được thực thi 
		// khi có kết quả truy vấn trả về.
	      }
	);
	doSomethingElse(); //hàm nãy sẽ được thực thi mà không 
	// phụ thuộc vào trạng thái trả về của lệnh db.query
```

Trong khi db.query() đang thực thi, tiến trình sẽ tiếp tục chạy lệnh doSomethingElse(), và thậm chí có thể phục vụ được 1 request mới từ client.

####**Why JavaScript?**

Ryan Dahl bắt đầu project với việc xây dựng 1 C platform, nhưng việc duy trì context giữa các callback là quá phức tạp và có thể làm cho cấu trúc code tồi tệ, vì vậy Ryan Dahl đã quay lại dùng Lua. Lua đã có 1 vài các thư viện blocking I/O và mix giữa blocking và non-blocking làm đa số các lập trình viên lẫn lộn và làm cản trở nhiều lập trình viên trong việc xây dựng các ứng dụng có khả năng mở rộng, vì vậy Lua cũng không phải là giải pháp. Sau đó Dahl đã nghĩ tới JavaScript. JavaScript có các closures and first-class functions, làm cho JavaScript thực sự mạnh mẽ khi kết hợp với evented I/O programming.

Closures là các function kế thừa các biến từ môi trường kèm theo chúng (Closures are functions that inherit the variables from their enclosing environment ). Khi 1 function callback thực thi, nó sẽ tự lưu lại context mà nó đã được công bố, cùng với tất cả các biến sẵn có trong context đó và bất kỳ các parent context nào khác. Tính năng hữu ích này đã tạo nên sự thành công của Node giữa ‘cộng đồng’ các chương trình (programming communities). Cùng xem qua một số ví dụ nhỏ để chứng minh sự thành công này của Node:

Trong các trình duyệt, nếu bạn muốn bắt một sự kiện, ví dụ như click vào 1 button, bạn sẽ phải làm gì đó tương tự như sau:

```sh
	var clickCount  = 0;
	document.getElementById(‘mybutton’).onclick = function() {
		clickCount ++ ;
		alert(‘Clicked ‘ + clickCount + ‘ times.’);
	};
```
Hoặc sử dụng Jquery:
```sh
	var clickCount = 0;
	$(‘button#mybutton’).click(function() {
		clickCount ++;
		alert(‘Clicked ‘ + clickCount + ‘ times.’);
	});
```
Trong cả 2 ví dụ, ta đã gán hoặc đã truyền vào 1 function với vai trò là 1 tham số sẽ được thực thi sau đó (khi có sự kiện click vào button). Function xử lý sự kiện click truy cập vào mọi biến trong phạm vi mà nó được khai báo (declared), như trong ví dụ trên, sự kiện click đã truy cập vào biến clickCount, biến này đã được khai báo trong parent closure.

Ở trong ví dụ trên, chúng ta đang sử dụng ‘clickCount’ là biến toàn cục, để chúng ta có thể lưu số lần mà người dùng đã click vào 1 button. Chúng ta cũng có thể tránh việc có 1 biến toàn cục sẽ làm ảnh hưởng tới sự nghỉ ngơi của hệ thống (the rest of the system), bằng cách bao biến này bên trong 1 closure khác, làm cho biến ‘clickCount’ chỉ được truy cập bên trong closure mà chúng ta đã tạo ra:

```sh
	(function() {
		var clickCount  = 0;
		$(‘button#mybutton’).click(function() {
			clickCount ++;
			alert(‘Clicked ‘ + clickCount + ‘ times.’);
		});
	})();
```

Ở dòng cuối cùng trong đoạn code trên, chúng ta đã thêm cặp dấu ngoặc () để làm cho function sẽ được thực thi ngay sau khi đã định nghĩa xong. Nếu điều này là lạ lẫm đối với bạn thì đừng lo lắng, chúng ta sẽ cover đến parttern này sau.

####**How I Learned to Stop Fearing and Love JavaScript?**
**(Làm cách nào để tôi hết sợ và yêu JavaScript?)**

JavaScript đều có chỗ tốt và chỗ chưa tốt. JavaScript được tạo ra trong năm 1995 bởi Brendan Eich của Nestcape, trong sự vội vàng đưa ra trình duyệt version mới nhất của Nestcape. Do sự vội vàng này đã tạo ra những phần tốt, thậm chí là tuyệt vời trong JavaScript, nhưng đồng thời cũng tạo ra một số phần xấu. 

Cuốn sách này sẽ không cover đến sự khác nhau giữa các phần tốt và xấu trong JavaScript. (Các ví dụ trong sách này sẽ chỉ sử dụng những phần tốt ).

Bạn có thể tìm hiểu thêm về chủ đề này trong cuốn sách có tên ***JavaScript, The Good Parts***, được edited bởi ***O’Reilly***. 

Dù có những nhược điểm nhưng JavaScript vẫn nhanh chóng đã trở thành de-facto language (ngôn ngữ tiêu chuẩn) cho các trình duyệt. Sau đó, JavaScript đã được sử dụng chủ yếu để kiểm tra và xử lý các văn bản HTML, cho phép tạo ra các ứng dụng web động đầu tiên phía client. 

####**Function Declaration Styles (các cách để khai báo 1 function)**
Một function có thể được khai báo bằng nhiều cách trong JavaScript. Các đơn giản nhất là khai báo nó 1 cách ẩn danh, nghĩa là không có tên hàm, như sau:

```sh
	function() {
		console.log(‘Hello’);
	}
```
Trong ví dụ trên, chúng ta đã khai báo 1 function nhưng nó không được sử dụng nhiều, bởi vì ta không gọi đến function đó. Hơn nữa, ta cũng không có cách nào để gọi nó khi nó được khai báo không có tên.

Nhưng ta có thể gọi đến 1 chương trình vô danh để thực thi nó ngay khi nó được định nghĩa xong như sau:
```sh
	(function() {
		console.log(‘Hello’);
	})();
```
Ở đây, chúng a đã thực thi function ngay lập tức sau khi khai báo nó. Lưu ý rằng chúng ta bao toàn bộ các khai báo của function trong cặp dấu ngoặc.
Ta có thể thêm tên cho 1 function như sau:
```sh
	function myFunction() {
		console.log(‘Hello’);
	}
```
Ở đây ta đã khai báo tên cho function là ***myFunction***, và ta có thể dùng tên này để gọi đến function bất cứ chỗ nào trong cùng scope mà nó được khai báo, ví dụ:
```sh
	myFunction();
```
Hoặc ta cũng có thể gọi đến nó trong các inner scope:
```sh
	function myFunction () {
		console.log(‘Hello’);
	}
	(function() {
		myFuncion();
	})();
```
Kết quả trả về của các function trong JavaScript giống như các *[first-class object](http://www.computerhope.com/jargon/f/firstclass-object.htm)*, do đó ta có thể gán một function cho 1 biến:
```sh
	var myFunc = function() {
		console.log(‘Hello’);
	}
```
Ta cũng có thể gán giá trị của biến myFunc cho một biến khác:
```sh
	var myFunc2 = myFunc;
```
Để gọi lại 2 function đã được gán cho 2 biến trên, ta chỉ cần làm đơn giản như sau:
```sh
	myFunc();
	myFunc2();
```
Ta có thể kết hợp cả 2 kĩ thuật bên trên, đó là gán 1 function đã có tên cho 1 biến:
```sh
	var myFunc2 = function myFunc() {
		console.log(‘Hello’);
	}
	myFunc(); // wrong because myFunc was not defined.
	myFunc2(); // true
```
(Chú ý: với cách khai báo như trong ví dụ trên, ta không thể truy câp đến function myFunc bên ngoài scope của chính nó)

Ta có thể sử dụng 1 biến hoặc tên của 1 function để truyền vào trong các function khác như sau:
```sh
	var myFunc = function() {
		console.log(‘Hello’);
	}
	console.log(myFunc);
```
hoặc khai báo đơn giản bằng cách sau nếu ta không cần dùng function đó trong những chỗ khác:
```sh
	console.log(function() {
		console.log(‘Hello’);
	}
```

####**Function are first-class objects**
