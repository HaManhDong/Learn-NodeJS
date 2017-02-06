
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

Trong thực tế, không có second-class objects trong JavaScript (second-class objects là các object có thể đóng vai trò là tham số trong 1 function nhưng không thể gán nó cho 1 biến hoặc đóng vai trò là giá trị trả về của 1 hàm). JavaScript là ngôn ngữ hướng đối tượng hoàn toàn, nghĩa là trong JavaScript, mọi thứ đều là đối tượng. Do đó, 1 function cũng là 1 đối tượng và ta có thể thiết lập các thuộc tính, truyền vào nó các tham số và trả về kết quả.

Ví dụ:

```sh
	1 var schedule = function(timeout, callbackfunction) {
	2	return {
	3		start: function() {
	4			setTimeout(callbackfunction, timeout);
	5		}
	6	};
	7 };

	8 (function () {
	9	 var timeout = 1000; // 1s
	10	 var count = 0;
	11	 shcedule(timeout, function doStuff(){
	12	 	 console.log( ++ count);
	13	 	 schedule(timeout, doStuff);
	14	 }).start();
	15 })();
	16
	17 // các biến timeout và count sẽ không tồn tại ở 
	18 // scope này
```
Trong ví dụ trên, ta đã tạo ra 1 function và lưu nó trong 1 biến gọi là ***schedule***. Function này đơn giản chỉ để trả về 1 object có 1 thuộc tính là ***start***. Giá trị của thuộc tính ***start*** này cũng là 1 function, mà khi được gọi, nó sẽ thiết lập 1 khoảng thời gian chờ (timeout) để gọi đến 1 function, khoảng thời gian chờ này được truyền vào như là 1 tham số giống như ***callbackfunction***. Khoảng thời gian chờ này sẽ lên lịch cho 1 callback function được gọi lại sau khoảng thời gian trễ là 1s (chính là giá trị của biến timeout).

Ở dòng code thứ 9, chúng ta khai báo 1 function để nó sẽ được thực thi ngay sau khi được định nghĩa xong (dòng 16). Đây là 1 cách bình thường để tạo ra 1 scope mới trong JavaScript. bên trong scope mới này, ta tạo 2 biến là ***timeout*** và ***count*** (dòng 10 và 11). Lưu ý rằng ta không thể truy cập vào 2 biến này từ scope bên ngoài.

Sau đó, ở dòng 12, chúng ta gọi lại function ***schedule***, và truyền vào đó tham số đầu tiên là biến ***timeout*** và tham số thứ 2 là function ***doStuff***. Khi timeout xảy ra, function này sẽ tăng biến ***count*** thêm 1 đơn vị và hiển thị nó lên, đồng thời gọi lại function ***schedule***.

Trong ví dụ trên, ta đã: truyền các function với chức năng làm đối số, tạo ra function trong scope mới, tạo function để phục vụ các callback không đồng bộ và returning 1 function. Đồng thời ta cũng đã đưa ra khái niệm về đóng gói (bằng cách ẩn các biến địa phương trong 1 scope khác đối với scope ngoài) và gọi đệ quy function.

Trong JavaScript, bạn thậm chí có thể thiết lập và truy cập vào 1 biến ngay bên trong 1 function như sau:
```sh
	var myFunction = function() {
		// do something
	};
	myFunction.someProperty = 'abc';
	console.log(myFunction.someProperty);
	// #=> "abc"
```
JavaScript thật sự là 1 ngôn ngữ mạnh mẽ, và nếu bạn chưa tìm hiểu nó thì hãy tìm hiểu nó ngay bây giờ!

####**JSHint**
JSHint không được cover trong cuốn sách này, nhưng JavaScript vẫn thực sự có những phần xấu, và ta cần phải tránh nó bằng mọi giá.

Một tool đã được ra đời để hỗ trợ JavaScript, đó là JSHint. JSHint sẽ phân tích các file JavaScript và xuất ra một loạt các error và các warning, bao gồm cả 1 số lạm dụng được biết đến trong JavaScript, chẳng hạn như sử dụng các biến ***globallyscoped*** (như khi bạn quên từ khóa var), và các giá trị bên trong vòng lặp mà được sử dụng trong các callback, và nhiều tính năng hữu ích khác.

JSHint có thể được cài đặt như sau:
```sh
	$ npm install -g jshint
```
Để run 1 file bằng JSHint, ta làm như sau:
```sh
	$ jshint myFile.js
```

####**Handing callback**
Trong Node, bạn có thể thực thi các function của bạn một cách không đồng bộ. Nghĩa là bạn có thể tự định nghĩa một function làm một chức năng nào đó, sau đó gọi lại nó qua callback function. Bạn sẽ gọi đến callback function khi luồng I/O thực hiện xong:
```sh
	1 var myAsyncFunc = function(argument1, argument2, callback){
	2		//giả sử 1 số luồng I/O đã xong
	3		setTimeout(function() {
	4			//1s sau, khi ta đã hoàn thành xong các 
	5			// luồng I/O thì ta sẽ gọi đến callback
	6			callback();
	7		}, 1000)
	8 }
```
Ở dòng số 3, ta đã định nghĩa 1 function là ***setTimeout*** để mô phỏng sự chậm trễ và không đồng bộ của 1 luồng I/O.

Function ***setTimeout*** sẽ gọi đến tham số đầu tiên (chính là function mà ta đã định nghĩa với vai trò là tham số bên trong function setTimeout - ở đây ta tạm gọi là inline function) sau khi 1s (tham số thứ 2: 1000) trôi qua. 

Inline function (function đóng vai trò là tham số thứ 1 trong function ***setTimeout***) sau đó sẽ gọi đến callback được truyền vào giống như là đối số thứ 3 của function ***myAsyncFunc***, để thông báo cho người gọi các hoạt động đã kết thúc.

Tip: để tuân thủ theo các quy ước của Node, function này sẽ nhận được thông báo lỗi (hoặc null nếu không có lỗi) là tham số đầu tiên, và sau đó là 1 số các tham số thực nếu bạn muốn thêm vào. (Xem ví dụ dưới)
```sh
	fs.open('path/to/file', function(err, fd) {
		if(err) {
			// handle err
			return;
		}
		console.log('opened file and got file descriptor ' + fd);
	})
```
Ở trong ví dụ trên, ta đã sử dụng 1 Node API function là fs.open nhận 2 tham số đầu vào là đường dẫn của file và 1 function sẽ được gọi đến với một lỗi nào đó hoặc giá trị null trong tham số thứ nhất, và tham số thứ 2 là nội dung của file.


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
