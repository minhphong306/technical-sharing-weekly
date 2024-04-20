# Giới thiệu
- Phong Do, @minhphong306.
- 2016: MP Telecom - Fullstack developer (Java + AngularJS).
- 2018 ~ 2022: OCG - Backend developer (Golang).
  - Technical leader, moved to Sai Gon.
- 2022: Starbots - Blockchain game on Solana (Typescript).
- 2023: OCG - QE Manager (Playwright, Typescript, Django).
- 2024: Outpost24 - Backend Engineer (Golang, Python, Java).

# Trải nghiệm
## Sập production lần 1
- Do thay đổi thứ tự gRPC
- gRPC lưu trữ theo thứ tự của protobuf
```golang
message Person {
  string name = 1;
  int32 id = 2;
}
```
- Khi đổi thứ tự cần đổi cả 2 service, nếu không sẽ fallback về giá trị default.
- Ví dụ đổi field id lên trước
```golang
message Person {
  int32 id = 1;
  string name = 2;
}
```
- Lúc này nếu service A đã đổi, mà service B không đổi thì B sẽ nhận được giá trị default của kiểu
![Protobuf](images/04-20-proto.png)
- Cụ thể là id = 0, name = ""
- Việc này dẫn tới id invalid ~> theo logic unavailable ~> trả về 404.

## Sập production lần 2: Redis keys
[[Redis] ĐỪNG dùng “keys” trên production
](https://minhphong306.wordpress.com/2021/02/04/redis-dung-dung-keys-tren-production/)

## Xoá 16 triệu sản phẩm khách hàng, cấm merge code 2 tuần

## Giới hạn 50 email đúng ngày release

## User cheating
