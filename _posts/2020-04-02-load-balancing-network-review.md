---
layout: post
title: Load Balancing - Scaling System and Networking
---

## Câu hỏi triệu đô?

Load balancing là một chủ đề cực kỳ hấp dẫn và lôi cuốn. Bởi lẽ ai cũng muốn biết câu trả lời cho câu hỏi triệu đô **Làm thế nào để systems xử lý được tầm vài triệu concurrent request**. Một server thông thường handle được tầm vài chục nghìn request là đã muốn đình công cmnr nói chi đến vài triệu.

Cần nhiều yếu tố mới có thể lời cho câu hỏi trên một cách đầy đủ nhất. Từ việc đầu tiên là thiết kế database, tối ưu code, áp dụng caching để tăng performance cho đến scale system khi lượng request tăng cao. Trong bài viết này, mình sẽ chỉ nói đến một khía cạnh của vấn đề. Đó là scale system và sử dụng load balancer.

## Scaling System

Sau khi release version đầu tiên của ứng dụng. Mọi thứ đều hoạt động trơn tru, lúc này số lượng user đâu đó khoảng vài trăm người. Sau đó, công ty chạy quảng cáo và lượng user bắt đầu tăng lên, tín hiệu tích cực cho thấy tết này bánh chưng có thịt rồi anh em ạ.

Nhưng không, vào một đẹp trời, người người, nhà nhà kéo nhau vào ứng dụng, thế là con server phải thở oxy vì bắt nó OT nhiều quá. Vì một nồi bánh chưng có thịt, ae dev phải nghiên cứu, tìm cách scale system lên để handle được nhiều request hơn.
Sau một hồi tra cứu SGK **300 bài code thiếu nhi** thì ta có những cách để scale system như sau:

- **Scaling up**

![]({{ site.baseurl }}/images/scaling-up.png)

Nếu server yếu thì mình có thể buff sức mạnh cho nó. Cụ thể ở đây là thêm RAM hay là nâng cấp ổ dĩa cho em nó lên SSD, ...(lương thấp thì mình có thể tăng lương).

Tuy nhiên, sức người có hạn, không thể cứ cầm cái khiêng là thành Captain và giao lưu với Thanos được ae ạ. Đến lúc nào đó, ta không để buff cho server được nữa. Lương tăng cao cũng đến lúc đụng nóc nhà. Khi đó, phải chuyển sang scaling out.

- **Scaling out**

![]({{ site.baseurl }}/images/scaling-out.png)

Chắc ae đã từng nghe qua câu **Một cây làm chẳng nên non, ba cây chụm lại nên hòn non bộ** hay **câu chuyện bó củi**. Bây giờ một server không xử lý nổi, thì mình tăng lên 2, 3, ...n server. Một dev làm không xuể, thì mình tuyển thêm một tá dev.

Khi scaling out system, chúng ta phải có một load balancer đứng ở trước để điều hướng request về từng server. Giống như chú cảnh sát ở ngã 4 phân luồng giao thông vậy đó. Trước khi đến với Load Balancer thì phải lật cuốn bí thuật **300 bài code thiếu nhi** ra để ôn lại một tí xíu xíu về Network ae ạ.

## Network Layers

**OSI Model**

![OSI Model]({{ site.baseurl }}/images/osi-model.png)

#### Layer 1

**Physical layer** có nhiệm vụ truyền nhận data dưới dạng bit thông qua các thiết bị phần cứng.

#### Layer 2

**Link layer** sẽ encode, decode data packets sang bit và ngược lại. Dựa trên Layer 1, **Link layer** cung cấp cơ chế để truyền data từ node này sang node khác (node ở đây có thể là computer, router, ...)

#### Layer 3

**Network layer** thực hiện chức năng chuyển tiếp, định hướng đường đi của gói tin dựa vào IP.

#### Layer 4

**Transport layer** đảm bảo rằng các gói tin được gửi đi và nhận theo thứ tự, không bị mất hay là lặp lại gói tin.

#### Layer 5

**Session layer** có nhiệm vụ thiết lập một session giữa 2 ứng dụng để chúng có thể gửi và nhận dữ liệu của nhau.

#### Layer 6

**Presentation layer** sẽ format data trước khi gửi lên tầng trên và ngược lại.

#### Layer 7

Application hoặc application process sẽ hoạt động ở tầng **Application layer** này. Các application sẽ sử dụng các protocol để giao tiếp với nhau như: HTTP, FTP, SMTP, ...

Như vậy chúng ta đã có cái nhìn tổng quan về scale system và ôn lại một tí kiến thức về network. Trong bài viết tiếp theo, chúng ta sẽ cùng tìm hiểu về **Load Balancer Layer 7**.
