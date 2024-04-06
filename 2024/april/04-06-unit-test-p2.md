# Test double

- Một "Test Double" là một đối tượng có thể thay mặt cho một đối tượng thực tế trong test.
- Có thể hiểu Test Double giống như 1 interface, trong đó có nhiều kiểu test double (implementation): Dummy, Fake, Stubs, Spies, Mocks
  ![test double](images/04-06-unit-test-test-doubles.png)

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

# VD với test thông thường

```typescript
// redis-key.ts
export const getRedisSyncWalletStatusKey = (wallet: string) => {
  return "syncing_wallet_v2:" + wallet;
};

export const getRedisWalletAddressByUserId = (userId: string) => {
  return `wallet_address_${userId}`;
};

// redis-key.spec.ts
describe("utils / redisKey", () => {
  it("should be pass", () => {
    expect(getRedisSyncWalletStatusKey("wallet")).toBe(
      "syncing_wallet_v2:wallet"
    );
    expect(getRedisWalletAddressByUserId("user_id")).toBe(
      "wallet_address_user_id"
    );
  });
});
```

# Spy

# Reference

- https://viblo.asia/p/xu-ly-phu-thuoc-va-test-double-trong-viec-viet-unit-test-phan-1-naQZRY30Kvx
- https://stackoverflow.com/a/8758060/7228412
- https://martinfowler.com/articles/mocksArentStubs.html#TheDifferenceBetweenMocksAndStubs
- https://jesusvalerareales.com/testing-with-test-doubles/#:~:text=Dummy%3A%20It%20is%20used%20as,how%20it%20will%20be%20used.

# Notes and thought

- Behaviour & state verification
  - Mock = behaviour verification
  - Stub = state verification
- Unit testing decision tree.
