# [우리FISA 6기] AI 엔지니어링 과정 N팀 (프로젝트 Submit.md 작성 예시)

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

### 2-2. AI 에이전트 워크플로우
<img width="1280" height="720" alt="슬라이드2" src="https://github.com/user-attachments/assets/cc070aee-d944-44bd-8656-f33021b34330" />


### 설명

LLM 기반으로 보고서 작성 계획을 수립한 후, 웹 검색 도구와 연계하여 각 섹션을 병렬로 생성하는 구조입니다.
조사원과 섹션 작성기가 협력하여 검색·분석된 내용을 바탕으로 섹션을 작성하고, 이를 형식화하여 일관된 구조로 정리합니다.
최종적으로 각 섹션을 병합하여 서론부터 결론까지 완성된 보고서를 생성하는 자동화된 보고서 작성 시스템입니다.

## 3\. 주요 기능 소개

### 3-1. 핵심 기술 구성
<img width="1280" height="720" alt="슬라이드4" src="https://github.com/user-attachments/assets/6f65ae06-91dc-441f-8439-6857ef98e04a" />

### 3-2. 통합 워크플로우 다이어그램
<img width="984" height="720" alt="슬라이드1" src="https://github.com/user-attachments/assets/3979ce05-2a67-4e11-b595-eee8a0b11ac4" />

### 3-3. 세부 기능 소개

#### [기능 1] Redis 분산 락을 이용한 재고 감소 로직

  * **기능 설명**: 동일한 상품에 1,000명이 동시에 결제를 시도할 때, 재고가 마이너스가 되지 않도록 Redisson 라이브러리를 활용해 원자성을 보장했습니다.
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
### [기능 3] ...
### [기능 4] ...
### [기능 5] ...
