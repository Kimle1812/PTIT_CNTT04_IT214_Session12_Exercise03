# **Phần 1 – Báo cáo phân tích**

Circuit Breaker **không mở** dù cả 5 request đều lỗi vì Resilience4j mặc định yêu cầu phải có **một số lượng request tối thiểu** trước khi tính failureRate.

Cụ thể:

* slidingWindowSize \= 20: theo dõi tối đa 20 request gần nhất.  
* failureRateThreshold \= 50: nếu tỷ lệ lỗi đạt từ 50% trở lên thì có thể chuyển sang OPEN.  
* Nhưng nếu không cấu hình minimumNumberOfCalls, Resilience4j mặc định sử dụng **100 calls**.  
* Đêm đó chỉ có **5 request**, nên mới có 5/100 request tối thiểu.  
* Mặc dù tỷ lệ lỗi thực tế của 5 request là **100%**, Circuit Breaker **chưa đủ số lượng calls tối thiểu để tính failure rate**, nên vẫn giữ trạng thái CLOSED.

  ### **Điểm cần nhớ**

slidingWindowSize **không đồng nghĩa** với minimumNumberOfCalls.

Ví dụ:

slidingWindowSize: 20

minimumNumberOfCalls: 3

có nghĩa:

> Theo dõi tối đa 20 request gần nhất, nhưng chỉ cần có 3 request là bắt đầu đánh giá tỷ lệ lỗi.

Vì vậy với 5 request đều lỗi:

5 request

5 lỗi

0 thành công

Failure Rate \= 5 / 5 × 100 \= 100%

100% \> 50% → Circuit Breaker chuyển sang OPEN.

# **Phần 2 – application.yml hoàn chỉnh**

Để Circuit Breaker hoạt động nhạy bén với bài toán chỉ có ít request, có thể đặt:

server:

  port: 8080

spring:

  application:

    name: payment-service

resilience4j:

  circuitbreaker:

    instances:

      bankClient:

        slidingWindowType: COUNT\_BASED

        slidingWindowSize: 20

        minimumNumberOfCalls: 3

        failureRateThreshold: 50

        waitDurationInOpenState: 30s

        permittedNumberOfCallsInHalfOpenState: 3

        automaticTransitionFromOpenToHalfOpenEnabled: true

### **Giải thích từng cấu hình**

| Cấu hình | Giá trị | Ý nghĩa |
| ----- | ----- | ----- |
| slidingWindowType | COUNT\_BASED | Đánh giá dựa trên số lượng request |
| slidingWindowSize | 20 | Theo dõi 20 request gần nhất |
| minimumNumberOfCalls | 3 | Có ít nhất 3 request mới bắt đầu tính tỷ lệ lỗi |
| failureRateThreshold | 50 | Tỷ lệ lỗi ≥ 50% thì mở Circuit Breaker |
| waitDurationInOpenState | 30s | Mở mạch trong 30 giây |
| permittedNumberOfCallsInHalfOpenState | 3 | HALF\_OPEN chỉ cho 3 request thử nghiệm |
| automaticTransitionFromOpenToHalfOpenEnabled | true | Sau thời gian OPEN, tự chuyển sang HALF\_OPEN |

