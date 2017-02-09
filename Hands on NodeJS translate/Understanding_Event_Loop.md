###**V - Understanding**

####**Understanding the Node event loop**
Node làm cho việc lập trình với event I/O trở nên đơn giản và dễ tiếp cận hơn, và giúp lập trình viên bình thường tiếp cận nhanh hơn về tốc độ và khả năng mở rộng.

Nhưng event loop cũng có cái giá của nó nếu như bạn không hiểu được event loop hoạt động như nào (do việc dựa dẫm vào Node đã thực hiện tốt công việc quản lý các event loop). Do đó bạn nên tìm hiểu cách mà event loop hoạt động. 

**An event-queue processing loop** 
**(Một vòng lặp xử lý hàng đợi event)**
Bạn nên nghĩ rằng event loop giống như 1 vòng lặp để xử lý 1 hàng đợi các event (event queue). Khi 1 event xảy ra thì nó sẽ đi vào 1 hàng đợi để chờ cho đến lượt được xử lý. Sau đó tại hàng đợi này có 1 event loop để kiểm tra lấy ra (popping out) các event, theo từng event 1 (one by one) và gọi đến callback function của event đó. Khi hàm callback đã thực hiện xong thì event loop sẽ lấy ra event tiếp theo để gọi đến callback function của event đó để thực thi. Khi hàng đợi event rỗng thì event loop sẽ đợi có event mới nếu như có 1 số cuộc gọi đang xử lý hoặc server đang lắng nghe, còn không thì event loop sẽ quit nếu không có gì.

Bây giờ ta cùng xem xét 1 ví dụ đầu tiên về callback function trong Node. Ta sẽ tạo 1 file có tên là hello.js với nội dung như sau:
```sh
	setTimeout(function() {
		console.log('World!');
	}, 2000);
	console.log('Hello ');
```
Chạy file trên với Node command:
```sh
	$ node hello.js
```
Bạn sẽ nhận được từ 'Hello ' ngay sau khi chạy, và sau đó 2s thì sẽ có thêm từ 'World!' xuất hiện.  Bây giờ ta cùng đi phân tích tại sao thứ tự các từ được in ra màn hình lại là như vậy. 

Trong ví dụ trên ta đã định nghĩa 1 function có chức năng là in ra từ 'World!'. Nhưng function này chưa được thực thi mà ta đã truyền nó vào là tham số thứ nhất của hàm setTimeout(). Hàm setTimeout này sẽ tạo ra 1 khoảng thời gian trể là 2s (ứng với 2000 ms trong tham số thứ 2). Sau đó, ở câu lệnh cuối cùng ta in ra màn hình từ 'Hello '.

Vì hàm setTimeout đã tạo ra khoảng thời gian trể 2s nên trong khoảng thời gian này, hệ thống sẽ vẫn tiếp tục thực hiện các câu lệnh bên dưới, vì vậy nó sẽ in ra màn hình từ 'Hello ', sau đó khi hết 2s thì nó gọi lại function mà ta đã truyền vào là tham số thứ nhất trong hàm setTimeout và thực thi hàm đó, in ra từ 'World!'. 

Do đó, trong ví dụ trên thì function mà ta đã truyền vào là đối số thứ nhất trong hàm setTimeout được gọi là callback function. 

**Callbacks that will generate events**
**(Các callback sẽ tạo ra các event)**
Từ ví dụ trên, bây giờ ta chỉnh sửa để có 1 hàm phức tạp hơn, đó là làm cho Node bận rộn và tiếp tục gọi lại callback như sau:
```sh
	( function schedule(){
		setTimeout(function() {
			console.log('Hello World!');
			schedule();
		}, 1000);
	})();
```
Trong ví dụ trên, ta đã bọc toàn bộ hàm setTimeout trong 1 function có tên là ***schedule*** và ta thực thi hàm này ngay sau khi đã khai báo xong (bằng việc có thêm cặp dấu ngoặc đơn ở cuối cùng). Hàm **schedule** này sẽ lên lịch cho hàm callback được gọi lại để thực thi sau 1s. Hàm callback này khi được gọi đến sẽ in ra dòng chữ 'Hello World!' sau đó lại gọi đệ quy lại hàm **schedule** . Việc gọi đệ qui này làm cho Node không bao giờ ngừng và thoát được, mà cứ sau 1s sẽ lại in ra dòng 'Hello World!'. 

**Don't block**

**(Không sử dụng block event)**

Việc Node sử dụng event loop chính là để tạo ra khả năng mở rộng dễ dàng cho các máy chủ (tính scalable servers). Khi một event loop chỉ chạy trong 1 thread thì nó sẽ chỉ xử lý sự kiện tiếp theo khi hàm callback của sự kiện trước đã thực thi xong. Nếu bạn có thể thấy các call stack trên 1 ứng dụng Node lớn, bạn sẽ thấy nó bận rộn như thế nào, quá trình up - down diễn ra rất nhanh, gọi đến các callback và chọn đến các event tiếp theo trong hàng đợi. Nhưng để làm việc này tốt 1 cách thủ công (không sử dụng Node) thì bạn luôn phải clear even loop 1 cách nhanh chóng để tránh tình trạng bị trễ.

Có 2 cách làm event loop bị block: đồng bộ I/O và các vòng lặp lớn (big loops).

Không phải tất cả các Node API đều là không đồng bộ, mà vẫn có 1 số phần đồng bộ, ví dụ như một số thao tác trên tập tin. Nhưng những API đồng bộ trong Node đều được đặt tên với kết thúc là ***'Sync'*** ở cuối để ta dễ nhận biết, ví dụ như - ***fs.readFileSync*** - và những API này không nên được sử dụng, hoặc chỉ dùng khi khởi tạo. Trên 1 máy chủ đang chạy, bạn không bao giờ được sử dụng một blocking I/O bên trong 1 callback, vì khi đó bạn sẽ làm blocking event loop và cản trở các callback từ các client đang chờ để được phục vụ. Từ đó gây lên tăng độ trễ và giảm khả năng phục vụ trên ứng dụng server của bạn.

Có một dạng API khác nữa trong node, được gọi là ***'require'*** function, đây là những API đồng bộ nhưng lại không có tên kết thúc là ***'Sync'***. Những API này chỉ nên được sử dụng khi khởi tạo hoặc trong 1 module. Chú ý: ta cũng không nên đặt 1 require statement bên trong 1 callback, cũng vì lý do nó là các function đồng bộ nên sẽ làm chậm event loop. 

Trường hợp thứ 2 gây ra hiện tượng block event loop đó là các vòng lặp lớn (big loop). Đó là những vòng lặp mất rất nhiều thời gian thực hiện, ví dụ như lặp qua hàng nghìn đối tượng hoặc thực hiện 1 tác vụ nặng, phức tạp trên CPU, các hoạt động tốn thời gian bộ nhớ. Có một số kĩ thuật để xử lý các big loop (sẽ đề cập sau).

Dưới đây là 1 ví dụ gây ra block event loop:
```sh
	1  var open = false;
	2 
	3  setTimeout(function() {
	4	 open = true;
	5  }, 1000);
	6
	7  while(!open) {
	8	 //wait
	9  }
	10
	11 console.log('opened!');
```
Trong ví dụ trên ta đã thiết lập 1 hàm setTimeout để thiết lập 1 khoảng thời gian chờ là 1s, và gọi đến 1 callback function có chức năng đơn giản là đặt lại biến ***open*** về giá trị true.  Ở dòng 7, ta tạo ra 1 vòng lặp While để kiểm tra biến **open**. Vòng lặp này sẽ ngừng khi biến open được đặt lại về giá trị true.

Theo lý thuyết thì khi hàm setTimeout được thực thi, nó tạo ra 1 khoảng thời gian trễ là 1s. Trong khoảng thời gian trễ này thì chương trình vẫn tiếp tục chạy các câu lệnh bên dưới, vì vậy nó sẽ đi vào vòng lặp while, và sau khi hết thời gian 1s, event loop sẽ gọi tới callback function của hàm setTimeout để thực thi, đặt lại biến open bằng true và thoát khỏi vòng lặp while, thực hiện tiếp câu lệnh 11.

Nhưng thực tế không phải như vậy, trong chương trình trên, Node sẽ không bao giờ gọi lại được callback function của hàm setTimeout bởi vì event loop đã bị tắc trong vòng lặp vô hạn while. Do là vòng lặp vô hạn nên event loop sẽ bị block ở đây, không thể có 1 'cơ hội' để xử lý các event khác.
 
####**Understanding Modules**
 
Các client-side JavaScript cũng có một nhược điểm, đó là do các namespace thông thường chia sẻ tất cả các script, điều này có thể dẫn tới các xung đột và các lỗ hổng bảo mật. 

Node thực thi các module theo chuẩn CommonJS module, trong đó mỗi module được tách riêng rẽ ra khỏi các module khác, có 1 namespace riêng biệt để play, và ... (exporting only the desired properties.)

Để import 1 module có sẵn, bạn có thể sử dụng function ***require*** như sau:
```sh
	var module = require('module_name');
```
Câu lệnh trên sẽ fetch module đã được cài đặt bằng npm. 

Nếu bạn muốn import module của bạn thì bạn chỉ cần thay module_name bằng đường dẫn dẫn tới module đó:
```sh
	var module = require("./path/to/module_name");
```
Câu lệnh trên sẽ fetch (lấy về) module từ đường dẫn trên đến file hiện tại của banjd dể thực thi. Chúng ta sẽ cover đến việc tạo ra 1 module ở các chương sau.

Các module sẽ được load bởi duy nhất 1 process, đó là khi bạn có nhiều các ***require*** đến cùng 1 module, Node sẽ caches lại module đã được require trước đó nếu nó xử lý lại module đó. Ta sẽ tìm hiểu điều này ở chương sau.

***How Node resolves a module path***

Làm cách nào để node giải quyết lời gọi đến 1 module thông qua ***'require(module_path)'***, dưới đây là cách làm:

***Core modules***

Có một danh sách các core modules trong Node được biểu diễn dưới dạng nhị phân. Nếu bạn import 1 trong số các module đó thì Node sẽ đơn giản là trả về module đó và kết thúc require().

***Modules with complete or relative path***

Nếu đường dẫn module bắt đầu với ***'./'*** hoặc ***'/'***, Node sẽ load các module như 1 file. Nếu như không thành công, nó sẽ load module như 1 thư mục.

Với trường hợp load như 1 file: nếu file tồn tại thì Node sẽ load file đó một cách bình thường (load Javascript text). Nhưng nếu không tồn tại thì Node sẽ cố gắng load như bình thường bằng cách thêm đuôi ***".js"*** vào trong đường dẫn, nếu vẫn không load được thì Node sẽ thêm vào ***".node"*** và load như 1 binary add-on.

Với trường hợp load như 1 thư mục: đầu tiên node sẽ thử thêm đuôi ***'.json'*** và và kiểm tra xem có file nào có tên như vậy không, nếu có thì nó sẽ tìm đến section ***'main'*** trong file đó và load như 1 file. Nếu vẫn không load được thì nó sẽ thêm đuôi ***/index*** vào để load.

Với trường hợp là 1 module đã cài đặt: nếu đường dẫn không bắt đầu bằng ***"."*** hoặc ***"/"*** hoặc thử load với đường dẫn tương đối và tuyệt đối đều không được thì Node sẽ thử load module đó giống như 1 module đã được cài đặt trước đó bằng cách thêm đuôi ***'/node_modules*** vào thư mục hiện tại và thử load module từ thư mục /node_modules. Nếu không thành công, Node sẽ thêm đuôi ***'/node_modules'*** vào thư mục cha của thư mục hiện tại và thử load các module trong đó. Nếu vẫn không tìm được thì Node tiếp tục tìm trên các thư mục cha tiếp theo (các thư mục tổ tiên của thư mục hiện tại trong đường dẫn), cho tới khi tìm thấy module hoặc đến thư mục gốc của cây đường dẫn. 

Điều này có ý nghĩa đó là ta có thể đặt các module vào trong 1 thư mục của ứng dụng và Node sẽ tìm ra các module đó. Chúng ta sẽ tìm hiểu về cách sử dụng tính năng này cùng NPM để ta có thể bundle và 'freeze' các package dependency trong ứng dụng.

Ngoài ra ta cũng có thể có 1 thư mục ***'node_modules'*** trong thư mục Home của mỗi user. Node sẽ load các module từ thu mục Home này, bắt đầu từ đầu với đường dẫn gần nhất (starting first with the one that is closest up the path).
