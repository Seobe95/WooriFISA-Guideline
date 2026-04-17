# [우리FISA 6기] 클라우드 엔지니어링 과정 N팀 (프로젝트 Submit.md 작성 예시)

## 1\. 프로젝트 개요
  * **주제**: 대규모 트래픽 처리가 가능한 한정판 신발 커머스
  * **프로젝트 기획 배경**: 선착순 구매 시 발생하는 트래픽 병목이 발생함으로 인한 불편함을 해결하기 위해 기획된 프로젝트입니다.
  * **기술 스택**
      * Java 17, Spring Boot 3.2, MySQL 8.0, Redis, Kafka
   
        
## 2\. 아키텍쳐

### 2-1. 시스템 아키텍쳐
  <img width="862" height="690" alt="슬라이드5" src="https://github.com/user-attachments/assets/67b4da1a-377b-46cb-b1bd-efe41b41dac6" />


### 설명
GitHub과 Jenkins를 연계한 CI/CD 파이프라인을 통해 코드 변경 시 자동으로 빌드 및 배포가 이루어지는 구조입니다.
Frontend(React)와 Backend(FastAPI)는 Docker 기반으로 구성되어 있으며, MySQL, S3, OpenSearch와 연동하여 데이터 저장 및 검색 기능을 수행합니다.
또한 Prometheus, Grafana, Elasticsearch, Kibana를 활용한 모니터링 및 로그 관리 환경을 통해 시스템의 안정적인 운영을 지원합니다.

### 2-2. 소프트웨어 아키텍처
<img width="1148" height="705" alt="프레젠테이션2" src="https://github.com/user-attachments/assets/c3c2f677-bd1f-4277-83cb-59dcb5897e63" />

### 설명

해당 아키텍처는 Presentation부터 Database까지 계층적으로 구성된 Layered 구조로, 각 레이어가 역할에 따라 분리되어 있습니다.
요청은 상위 레이어에서 하위 레이어로 순차적으로 전달되며, Controller–Service–Component–DBIO를 거쳐 데이터 처리 및 비즈니스 로직이 수행됩니다.
또한 Utility와 Interface 영역을 통해 외부 시스템 연동 및 공통 기능을 분리하여, 확장성과 유지보수성을 고려한 구조로 설계되었습니다.

## 3\. 주요 기능 소개

### 3-1. 핵심 기술 구성
<img width="1280" height="720" alt="슬라이드4" src="https://github.com/user-attachments/assets/6f65ae06-91dc-441f-8439-6857ef98e04a" />

### 3-2. 통합 워크플로우 다이어그램
<img width="984" height="720" alt="슬라이드1" src="https://github.com/user-attachments/assets/3979ce05-2a67-4e11-b595-eee8a0b11ac4" />

### 3-3. 세부 기능 소개

#### [기능 1] Redis 분산 락을 이용한 재고 감소 로직

  * **기능 설명**: 동일한 상품에 1,000명이 동시에 결제를 시도할 때, 재고가 마이너스가 되지 않도록 Redisson 라이브러리를 활용해 원자성을 보장했습니다.
  * **핵심 코드(스크립트)**:

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

  * **코드(스트립트) 링크**: OrderService.java

### [기능 2] ...
### [기능 3] ...
### [기능 4] ...
### [기능 5] ...
