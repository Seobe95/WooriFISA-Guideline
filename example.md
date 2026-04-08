# [우리FISA 6기] OOO 과정 N팀 (최종 프로젝트 README 제출 예시)

## 1\. 프로젝트 개요
  * **주제**: 03번. 대규모 트래픽 처리가 가능한 한정판 신발 커머스
  * **서비스 한 줄 소개**: 선착순 구매 시 발생하는 트래픽 병목을 해결한 고성능 커머스 플랫폼
  * **해결하려는 핵심 문제**:
      * 인기 상품 출시 시 동시 접속자 급증으로 인한 서버 다운 문제 해결
      * 동일 상품에 대한 동시 결제 시 재고 수량의 정합성(Race Condition) 보장

## 2\. 기술 스택

  * **스택**: Java 17, Spring Boot 3.2, MySQL 8.0, Redis, Kafka
  * **추가 스택(선택)**:
      * **Redis**: 대규모 결제 요청 시 데이터베이스 부하를 방지하기 위해 분산 락(Distributed Lock) 구현 용도로 사용했습니다.

## 3\. 서비스 설계

  * **전체 서비스 Flow**:
    > 
    > (심사 가이드: 위와 같이 인프라와 데이터 흐름이 포함된 아키텍처 다이어그램을 첨부하세요.)

## 4\. 핵심 기능 구현 (최대 3개)

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

  * **코드 위치**: [OrderService.java (README 파일)](https://github.com/Seobe95/WooriFISA-Guideline/blob/07b70cc4fa8a99407b820478d5b8dc76037f029a/README.md)

### [기능 2] ...

-----

## 5\. 문제 해결 및 성능 개선

  * **문제 상황**: 1만 건의 주문 데이터를 조회할 때 응답 시간이 5초 이상 소요되어 사용자 경험이 저하됨.
  * **문제 원인**: 주문 테이블과 상품 테이블을 조인(Join)하는 과정에서 인덱스가 설정되지 않아 Full Table Scan이 발생함을 실행 계획(EXPLAIN)을 통해 확인.
  * **해결 방안**:
    1.  자주 조회 조건으로 사용되는 `user_id`와 `created_at` 컬럼에 복합 인덱스(Composite Index) 생성.
    2.  불필요한 조인을 제거하고 Querydsl을 사용하여 필요한 필드만 조회하도록 최적화.
  * **결과**: 동일 데이터 기준 조회 속도가 **5.2초에서 0.15초로 약 97% 개선**됨을 확인.
