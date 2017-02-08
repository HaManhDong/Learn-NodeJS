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
