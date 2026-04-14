# [우리FISA 6기] OOO 과정 N팀 (프로젝트 Submit.md 작성 예시)

## 1\. 프로젝트 개요
  * **주제**: 대규모 트래픽 처리가 가능한 한정판 신발 커머스
  * **프로젝트 기획 배경**: 선착순 구매 시 발생하는 트래픽 병목이 발생함으로 인한 불편함을 해결하기 위해 기획된 프로젝트입니다.
  * **기술 스택**
      * Java 17, Spring Boot 3.2, MySQL 8.0, Redis, Kafka
   
        
## 2\. 시스템 아키텍쳐

### 시스템 아키텍쳐
  <img width="2816" height="1536" alt="Gemini_Generated_Image_14i9jf14i9jf14i9" src="https://github.com/user-attachments/assets/8df3192e-ede4-439c-aa6e-48b399c96beb" />

### 설명
본 프로젝트는 대규모 사용자의 요청을 안정적으로 처리하고, 데이터 응답 속도를 최적화하기 위해 다음과 같이 설계되었습니다.

  #### **1) Load Balancer (부하 분산)**
  * 사용자의 HTTPS 요청이 단일 서버로 몰리지 않도록 3대의 **Web Application Server(WAS)**로 균등하게 분산하여 가용성을 높이고 서버 다운타임을 최소화했습니다.
  
  #### **2) Web Application Servers (비즈니스 로직)**
  * 각 서버는 동일한 소스 코드로 동작하는 **Stateless(무상태)** 구조로 설계되었습니다. 이를 통해 서버 증설이 용이한 **확장성(Scalability)**을 확보했습니다.
  
  #### **3) In-Memory Cache (Redis 활용)**
  * 반복적으로 발생하는 동일한 데이터 조회 요청은 **Redis**에서 즉시 반환(**Cache Hit**)하도록 처리하여, 메인 데이터베이스의 부하를 약 **60% 이상 절감**했습니다.
  
  #### **4) Relational Database (MySQL 8.0)**
  * 서비스의 핵심 데이터(유저, 주문, 상품)는 **RDB(MySQL)**에 저장하여 데이터의 무결성을 유지하고, ACID 원칙에 기반한 트랜잭션의 안전성을 보장합니다.

---

### 🔄 Data Flow (데이터 흐름)

* **Read (조회)**
    * `User` → `LB` → `App Server` → `Redis(확인)` → (Cache Miss 시) `MySQL 조회`
* **Write (등록/수정)**
    * `User` → `LB` → `App Server` → `MySQL 저장` → `Redis 동기화/삭제`
    
## 3\. 기능 소개

### [기능 1] Redis 분산 락을 이용한 재고 감소 로직

  * **설명**: 동일한 상품에 1,000명이 동시에 결제를 시도할 때, 재고가 마이너스가 되지 않도록 Redisson 라이브러리를 활용해 원자성을 보장했습니다.
  * **핵심 코드**:

<!-- end list -->

```java
public void decreaseStock(Long productId, Long quantity) {
    RLock lock = redissonClient.getLock("stock_lock:" + productId);
    try {
        if (lock.tryLock(5, 1, TimeUnit.SECONDS)) { // 5초 대기, 1초 점유
            Product product = productRepository.findById(productId).orElseThrow();
            product.removeStock(quantity);
        }
    } catch (InterruptedException e) {
        throw new BusinessException("락 획득 실패");
    } finally {
        lock.unlock();
    }
}
```

  * **코드 링크**: OrderService.java

### [기능 2] ...
