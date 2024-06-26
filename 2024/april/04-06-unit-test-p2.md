# Test double

- Một "Test Double" là một đối tượng có thể thay mặt cho một đối tượng thực tế trong test.
- Có thể hiểu Test Double giống như 1 interface, trong đó có nhiều kiểu test double (implementation): Dummy, Fake, Stubs, Spies, Mocks
  ![test double](images/04-06-unit-test-test-doubles.png)
- 2 khái niệm:
  - State ver

## Dummy

- Một tham số mà không cần test, nhưng vẫn cần truyền vào tham số.
- Thường dùng cho các parameter có kiểu function callback

```typescript
// code.ts
updateCart(items: Item[], fetchDiscount: (items) => {}) {
  this.cart.totalItems += items.length;
  fetchDiscount(this.cart);
}

// code.spec.ts
describe('updateCart', () => {
  const dummy = (items) => ({});

  it('should update cart total', () => {
    component.updateCart(items, dummy);
    expect(component.cart.totalItems).toEqual(4);
  })
})
```

## Stub
- Giả lập giá trị trả về cho các function external.
```typescript
// code.ts
getDeliveryDate(orderId: number) {
  let deliveryDate;

  if (this.verified && cart.totalItems > 0) {
    deliveryDate = this.myService.getDeliveryDate(cart);
  }

  deliveryDate = format(deliveryDate);

  return this.deliveryDate;
}

// code.spec.ts
describe('getDeliveryDate', () => {
  const date = Date.now();
  myService.getDeliveryDate = jest.fn().mockReturnValue(date);

  it('should get the delivery date for order', () => {
    component.verified = true;
    component.cart = cart;
    component.getDeliveryDate(cart);
    expect(component.deliveryDate).toEqual(date);
  })
})
```

## Fake

- Tương tự như Stub, nhưng thay đổi chức năng bên trong.
- Ví dụ hàm dưới dùng localStorage để xử lý dữ liệu.
- Tạo ra 1 bộ mock với đầy đủ tính năng cho storage.
- Thường dùng giả lập cả 1 database.

```typescript
// code.ts
placeOrder() {
  if (this.verified) {
    window.localStorage.setItem('cart', JSON.stringify(this.cart));
    this.myService.placeOrder(this.cart, this.deliveryDate);
  } else {
    this.cart = JSON.parse(window.localStorage.getItem('cart'));
  }
}

// code.spec.ts
// Global setup
const fake = () => {
  let storage: { [key: string]: string } => {};

  return {
    getItem: (key: string) => (key in storage ? storage[key] : null),
    setItem: (key: string, value: string) => (storage[key] = value || ''),
  }
}
Object.defineProperty(window, 'localStorage', { value: fake() });
Object.defineProperty(window, 'sessionStorage', { value: fake() });

// Test
describe('PlaceOrder', () => {
  it('should put cart in localStorage if item is verified', () => {
    component.cart = cart;
    component.verified = true;

    component.placeOrder();
    const localCart = JSON.parse(window.localStorage.getItem('cart'));
    expect(localCart).toBeDefined();
  })
});
```

## Mock
- Tương tự như Stub. Nhưng không cần check giá trị trả về, check xem các hàm con bên trong nó có được gọi không, và được gọi đúng thứ tự, đúng param hay không

```typescript
// code.ts
getDeliveryDate(orderId: number) {
  let deliveryDate;

  if (this.verified && cart.totalItems > 0) {
    deliveryDate = this.myService.getDeliveryDate(cart);
  }

  deliveryDate = format(deliveryDate);

  return this.deliveryDate;
}

// code.spec.ts
describe('PlaceOrder', () => {
  const spy = jest.spyOn(myService, 'placeOrder');

  it('should call placeOrder', () => {
    component.cart = cart;
    component.verified = true;

    component.placeOrder();
    expect(spy).toHaveBeenCalled();
  })
});
```

## Mock vs Spy
![mock and spy](images/mock-vs-spy.webp)
- Mock — Mock object replace mocked class entirely, returning recorded or default values. You can create mock out of “thin air”. This is what is mostly used during unit testing.

- Spy — When spying, you take an existing object and “replace” only some methods. This is useful when you have a huge class and only want to mock certain methods (partial mocking)

# Reference

- https://viblo.asia/p/xu-ly-phu-thuoc-va-test-double-trong-viec-viet-unit-test-phan-1-naQZRY30Kvx
- https://stackoverflow.com/a/8758060/7228412
- https://martinfowler.com/articles/mocksArentStubs.html#TheDifferenceBetweenMocksAndStubs
- https://jesusvalerareales.com/testing-with-test-doubles/#:~:text=Dummy%3A%20It%20is%20used%20as,how%20it%20will%20be%20used.
- https://www.youtube.com/watch?v=vyjUIcr6iYY

# Notes and thought

- Behaviour & state verification
  - Mock = behaviour verification
  - Stub = state verification
- Unit testing decision tree.
