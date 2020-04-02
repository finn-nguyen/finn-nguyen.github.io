---
layout: post
title: React Hook - Infinite Scroll Hight Order Component
---

**High Order Component** là một component nhận vào một component và trả về một component mới. Bằng cách sử dụng **High Order Component** chúng ta có thể reuse code ở những chỗ có xử lý logic giống nhau, nhưng khác nhau về mặt trình bày. Thông qua một ví dụ về infinite scroll, mình sẽ code một đoạn để demo về **High Order Component**, kết hợp với **React Hook**, cụ thể là **Custom Hook**.

## Setup

Chúng ta sẽ bắt đầu từ việc đơn giản nhất là render một list có thể scroll được. Bao gồm 2 component **ListCard** và **Card** như sau:

```javascript
import React from "react";
import Card from "./Card";

const ListCard = ({ limit }) => {
  const cards = [...Array(limit).keys()];

  return (
    <div className="list-container">
      {cards.map(element => (
        <Card key={element} />
      ))}
    </div>
  );
};

export default ListCard;
```

Tham số **limit** là số phần tử mà component này sẽ render. Chẳng hạn **limit** là 20 thì ta sẽ có 20 card trong list này.
**ListCard** sẽ render một list **Card** như bên dưới:

```javascript
import React from "react";

const Card = () => {
  return (
    <div className="card-container">
      Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod
      tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim
      veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea
      commodo consequat. Duis aute irure dolor in reprehenderit in voluptate
      velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat
      cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id
      est laborum.
    </div>
  );
};

export default Card;
```

## Infinite Scroll Handler

Để biết được khi nào có event scroll, chúng ta phải thêm listener cho event scroll trên một element. Cụ thể ở đây mình sẽ thêm event listener cho component **ListCard**. Bây giờ **ListCard** sẽ nhận vào 1 tham số là **containerElement**, biến này sẽ reference đến **div** ngoài cùng của **ListCard**

```javascript
const ListCard = ({ limit, containerElement }) => {
  const cards = [...Array(limit).keys()];

  return (
    <div ref={containerElement} className="list-container">
      {cards.map(element => (
        <Card key={element} />
      ))}
    </div>
  );
};
```

Công việc tiếp theo là phải handle cái event scroll này. Chúng ta sẽ thêm một callback, hàm này sẽ được trigger mỗi khi có event scroll.

```javascript
const scrollHandler = e => {
  const el = e.target;
  const { clientHeight, scrollHeight, scrollTop } = el;

  // End of content
  if (scrollTop + clientHeight === scrollHeight) {
    // Handle load more here
  }
};
```

Load more sẽ chỉ được gọi khi chúng ta đã scroll đến cuối list. Ở đây mình dùng điều kiện `scrollTop + clientHeight = scrollHeight` để xác định xem là đã đến cuối list hay chưa.

## Custom Hook

Để quản lý loading state và data của component, chúng ta có thể sử dụng **useState** cho từng state. Tuy nhiên, khi các state này có quan hệ với nhau, khiến cho việc code trở nên khó khăn hơn. Chúng ta có thể tách ra và đóng gói phần này để dễ dàng quản lý hơn. Đó gọi là **Custom Hook**

```javascript
const useInfinityScroll = () => {
  const size = 20;
  const [limit, setLimit] = useState(size);
  const [loading, setLoading] = useState(null);

  const loadMore = () => {
    if (!loading) {
      setLoading(true);
      setTimeout(() => {
        setLoading(false);
        setLimit(limit + size);
      }, 2000);
    }
  };

  return { loading, limit, loadMore };
};
```

## High Order Component

Sau khi đã có hook để handle state. Chúng ta sẽ sử dụng nó trong **High Order Component**

```javascript
const WithInfinityScroll = wrappedComponent => {
  const WrappedComponent = wrappedComponent;
  const { limit, loadMore, loading } = useInfinityScroll();
  const containerElement = useRef(null);

  const scrollHandler = e => {
    const el = e.target;
    const { clientHeight, scrollHeight, scrollTop } = el;

    if (scrollTop + clientHeight === scrollHeight) {
      loadMore();
    }
  };

  useEffect(() => {
    const element = containerElement.current;
    element.addEventListener("scroll", scrollHandler);

    return () => {
      element.removeEventListener("scroll", scrollHandler);
    };
  });

  return (
    <WrappedComponent
      containerElement={containerElement}
      limit={limit}
      loading={loading}
    />
  );
};
```

```javascript
export default () => WithInfinityScroll(ListCard);
```

**WithInfinityScroll** sẽ có nhiệm vụ handle scroll, load more và truyền các giá trị cần thiết vào cho **WrappedComponent**.

## Wrap up

Bằng việc sử dụng **High Order Component**, có thể thấy chúng ta đã tách rời logic và render. Cụ thể, logic hoàn toàn nằm trong **WithInfinityScroll**, còn **ListCard** sẽ nhận vào data và thực hiện render, không cần phải quan tâm đến xử lý scroll, load more...
Hơn thế nữa, chúng ta có thể sử dụng **WithInfinityScroll** với những component khác có chức năng tương tự như **ListCard**. Qua đó giảm thiểu việc duplicate code, làm cho codebase dễ dàng quản lý hơn.

[Source code!](https://github.com/finn-nguyen/react-infinite-scroll)
