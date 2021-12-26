---
title: 'HyperLogLog'
date: 2021-12-26
permalink: /posts/2021/12/hyper_log_log/
tags:
  - til
---
Giải quyết bài toán count distint (tính số lượng phần tử unique)

HyperLogLog là gì?
======

### Bài toán là gì?
- Giả sử bạn muốn đếm xem có bao nhiêu người đến dự tiệc, thì cách đơn giản nhất là cần ghi lại hết tên những người bạn gặp, nếu một người đã có trong danh sách bạn ghi rồi thì sẽ không được tính lại nữa. Như vậy tên có trong danh sách của bạn là duy nhất và đảm bảo chính xác 100%.
- Nhưng vấn đề là nếu phải ghi lại hết tên và lưu trữ để tìm kiếm nhanh nhất khi bạn gặp một người bất kì thì rất tốn công, tốn giấy (memory, storage).

### Giải pháp
- Thay vì lưu tên, chúng ta có thể chỉ cần lưu 5 số cuối của số điện thoại, có thể sẽ giảm độ chính xác không còn là 100% nhưng độ chính xác vẫn chấp nhận được nếu số lượng khách mời lớn, và bộ nhớ lưu trữ sẽ ít hơn so với việc lưu trữ tên.
- Tuy nhiên việc ghi lại số điện thoại vẫn còn phải ghi rất nhiều, và giấy dù đã tốn ít hơn nhưng vẫn chưa đáng kể, vậy có cách nào mà chỉ cần dùng những ngón tay mà có thể đếm được gần đúng số lượng khách mời hay không?
- Câu trả lời là có, chúng ta sẽ dùng một thuật toán gọi là Log Log, cụ thể như sau:
  - Thay vì ghi lại tất cả 5 số cuối của số điện thoại, chúng ta chỉ cần nhớ 5 số cuối có số số 0 ở đầu nhiều nhất là được. 
  - Ví dụ:
    - 12345 -> số số 0 ở đầu là 0
    - 01234 -> số số 0 ở đầu là 1
    - 00045 -> số số 0 ở đầu là 3
  - Từ đó có thể suy ra số khách mời gần đúng là khoảng `10^n` với n là số số 0 ở đầu dài nhất
  - Lí do có thể hiểu đơn giản là ở mỗi vị trí số điện thoại chúng ta có 10 trường hợp 0 -> 9, vậy xác suất để số 0 xuất hiện là 1/10, nghĩa là cứ trung bình 10 người thì có 1 người có số 0 ở đầu, từ đó có thể suy ngược lại là chúng ta đã gặp khoảng 10 người khác nhau để gặp được 1 người có số 0 ở đầu.
  - Đó chính là ý tưởng của thuật toán Log Log.
  - Con số chính xác sẽ nằm trong khoảng `10^(n-1) < con số chính xác < 10^n`
  - Trong lập trình thì thay vì tính toán với hệ số 10, chúng ta sẽ dùng binary để giảm sai số
  - Với bất kì input đầu vào nào thì chúng ta sẽ dùng một hash function để chuyển thành một chuỗi binary, khi đó con số chính xác sẽ nằm trong khoảng `[2^(n-1), 2^n]`, tất nhiên hash function phải đủ tốt để tạo ra những chuỗi ngẫu nhiên và tránh trùng lặp nhiều nhất có thể để xác xuất 0 và 1 tại mỗi vị trí của chuỗi hash là gần với 50%.
  - Nhưng với cách đơn giản này thì nếu gặp trường hợp vị khách đầu tiên chúng ta gặp đã có 4 số 0 ở đầu 😂. Và dĩ nhiên là tác giả của thuật toán này (Flajolet) cũng đã tính tới trường hợp này. Ý tưởng như sau:
    - Chúng ta sẽ lấy thêm một số nữa, 6 số thay vì 5 số cuối, và con số lấy thêm này để chia những số điện thoại vào những nhóm khác nhau (buckets) và kết quả sẽ lấy trung bình từ tất cả các nhóm. Ví dụ:
    - `0``12345` -> nhóm 0, có 0 số 0 ở đầu
    - `1``02345` -> nhóm 1, có 1 số 0 ở đầu
    - `1``12345` -> nhóm 1, có 0 số 0 ở đầu
    - `5``00045` -> nhóm 5, có 3 số 0 ở đầu
    - Kết quả sẽ là (1 + 0 + 3)/3 = 1,33333 (chỉ lấy kết quả lớn nhất ở mỗi nhóm)
    - Vậy số người có thể ước tính vào khoảng 10^1,33333
    - Như vậy ta đã giảm được sai số cụ thể còn là 1.3/√m với m là số nhóm (bucket)
  - Sau đó thì tác giả vẫn tiếp tục nghiên cứu và cho ra đời Super Log Log giúp giảm sai số xuống còn 1.05/√m, ý tưởng là chỉ tất 70% các nhóm có giá trị nhỏ, và loại bỏ 30% các nhóm có giá trị lớn.
  - Và cuối cùng là Hyper Log Log bằng việc cải thiện cách tính mean từ geometric mean sang harmonic mean giảm sai số xuống chỉ còn 1.04/√m. Cụ thể tham khảo thêm ở bài viết này: https://towardsdatascience.com/hyperloglog-a-simple-but-powerful-algorithm-for-data-scientists-aed50fe47869.

### Áp dụng thực tế
- Reddit: https://www.redditinc.com/blog/view-counting-at-reddit/
- Google Big Query

Nguồn tham khảo:
- https://odino.org/my-favorite-data-structure-hyperloglog/
- https://www.youtube.com/watch?v=2PlrMCiUN_s
- https://towardsdatascience.com/hyperloglog-a-simple-but-powerful-algorithm-for-data-scientists-aed50fe47869
