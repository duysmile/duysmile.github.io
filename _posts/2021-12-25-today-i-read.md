---
title: 'Today I Read - 2021/12/25'
date: 2021-12-25
permalink: /posts/2021/12/tir20211225/
tags:
  - til
  - tir
---
Thay đổi cách build một Reusable Component

Cách build UI components
======

Chúng ta thường hay tạo ra các Resuable Component bằng các copy code sang, và sử dụng bằng cách truyền props vào. Kiểu như thế này
```javascript
<LoginFormModal
  onSubmit={handleSubmit}
  modalTitle="Modal title"
  modalLabelText="Modal label (for screen readers)"
  submitButton={<button>Submit form</button>}
  openButton={<button>Open Modal</button>}
/>
```
Cách này làm cho chúng ta cảm giác như đang abstract một Component, và làm cho nó có thể reusable, flexible.

Nhưng thực tế thì không phải vậy, nếu như muốn custom gì ở Component này, chúng ta lại phải thêm một prop và chỉnh sửa ở code trong Component. Việc này khiến cho việc sử dụng không còn linh hoạt, và việc maintain nó cũng khó hơn.

Thay vào đó, chúng ta nên tạo ra các Component dạng này để việc sử dụng được hiệu quả và linh hoạt hơn.
```javascript
<Modal>
  <ModalOpenButton>
    <button>Open Modal</button>
  </ModalOpenButton>
  <ModalContents aria-label="Modal label (for screen readers)">
    <ModalDismissButton>
      <button>Close Modal</button>
    </ModalDismissButton>
    <h3>Modal title</h3>
    <div>Some great contents of the modal</div>
  </ModalContents>
</Modal>
```
Cách tạo component như thế này gọi là Compound Component, kết hợp các component với nhau và việc custom dễ dàng hơn rất nhiều, phục vụ được nhiều use-cases mà không cần chỉnh sửa gì ở những component gốc.

\# *Tham khảo bài viết chi tiết ở đây*: https://dev.to/harshkc/stop-building-your-ui-components-like-this-19il

