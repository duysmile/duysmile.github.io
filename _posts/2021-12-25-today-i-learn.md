---
title: 'Today I Learn - 2021/12/25'
date: 2021-12-25
permalink: /posts/2021/12/til20211225/
tags:
  - til
---
Locking read trong MySQL, giải quyết việc read tuần tự

Locking in MySQL
======

Trong một số trường hợp chúng ta cần lock 1 record trong table để những process khác không thể read hay update gì record đó trong khi nó đang được sử dụng ở một nơi khác. Việc này giúp chúng ta tránh các vấn đề về race condition, đảm bảo các xử lí theo đúng tuần tự, ...

Một context cụ thể cho việc này là khi chúng ta `SELECT` và `UPDATE` record liên quan trong cùng một transaction thì `SELECT` không thể giúp cho record vừa được lấy ra tránh khỏi việc bị update từ một transaction khác. Và MySQL cung cấp những *Lock reads* sau nhằm đảm bảo việc này:
1. SELECT ... FOR SHARE
- Set share mode lock cho những row được read. Những session khác có thể đọc những row này nhưng không thể tahy đổi tới khi transaction commit. Nếu bất kì rows nào thay đổi bởi transaction khác mà chưa được commit thì query của bạn sẽ phải chờ tới khi transaction đó kết thúc và dùng giá trị mới nhất.

2. SELECT ... FOR UPDATE
- Sẽ lock rows tương ứng và index liên quan, tương tự như khi chạy lệnh `UPDATE` trên những row đó vậy.
- Những transaction khác sẽ bị block `Update`, `Select ... for share` hoặc là read data trong những level cố định.
- Consistent read bỏ qua bất kì lock nào có trong read view


Tất cả lock được set bởi FOR SHARE hay FOR UPDATE đều được release khi transactionc commit hoặc roll back

Ví dụ về locking read:
1. Giả sử bạn muốn thêm 1 row mới vào table `child`, và cần đảm bảo child row phải có một parent row ở table `parent`. Có thể làm được việc này bằng application code, đầu tiên get parent row và verify parent có tồn tại sau đó thêm child row. Một vấn đề ở đây là sau khi select parent thì parent này có thể bị xoá và bạn không thể nào biết được. Để tránh issue này, thì chúng ta sẽ dùng lệnh sau:
  
  ```sql
  SELECT * FROM parent WHERE name = 'A' FOR SHARE;
  ```
  - Sau khi query trên get đươc parent A thì bạn có thể thêm child record an toàn và commit transaction đó. Bất kì transaction nào khác yêu cầu một exclusive lock với row đó ở table parent đều phải đợi tới khi bạn finish, và dữ liệu trong các table phải consistent.

2. Một ví dụ khác, chúng ta có một field counter trong table `child_codes`, được dùng để assign một id cho mỗi child được thêm vào table `child`. Không nên dùng consistent read hay share mode read để read giá trị hiện tại của counter, bởi vì có thể có 2 user cùng thấy một giá trị của counter và lỗi duplicate-key có thể xảy ra nếu cả hai transaction cùng thêm row với cùng id vào table child.
  - Trong tình huống này thì FOR SHARE không phải là giải pháp tốt vì nếu cả hai user cùng read counter cùng lúc, thì ít nhất một trong hai có thể dính deadlock khi cố update counter.
  - Để implement việc read và increment counter, đầu tiên cần phải locking read cho counter với `FOR UPDATE`, sau đó increment counter.
  
  ```sql
  SELECT counter_field FROM child_codes FOR UPDATE;
  UPDATE child_codes SET counter_field = counter_field + 1;
  ```
  - Command `SELECT ... FOR UPDATE` sẽ read giá trị mới nhất, setting exclusive lock trên mỗi row nó read. Lock này tương tự như lock với command UPDATE.
  - Ở trên chỉ là một ví dụ cho việc `SELECT ... FOR UPDATE`, còn thực tế ví dụ trên có thể giải quyết trong MySQL bằng cách chỉ dùng single access tới table:
  ```sql
  UPDATE child_codes SET counter_field = LAST_INSERT_ID(counter_field + 1);
  SELECT LAST_INSERT_ID();
  ```
  - Lệnh SELECT trên chỉ lấy giá trị id (cụ thể cho connection hiện tại) chứ không truy cập bất kì table nào.

Tham khảo thêm ở đây: https://dev.mysql.com/doc/refman/8.0/en/innodb-locking-reads.html

    
    
