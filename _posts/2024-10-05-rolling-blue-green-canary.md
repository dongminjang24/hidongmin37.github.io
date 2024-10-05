---
title: 무중단 배포(Rolling, Blue-Green, Canary)
date: 2024-10-05
comments: true
categories:
  - posts
tags:
  - Rolling
  - Blue-Green
  - Canary
  - coding
---

# 무중단 배포(Zero Downtime Deployment)
개발에서는 지속적인 통합 및 지속적인 배포(CI/CD)가 필수적으로 자리잡았습니다.
무중단 배포는 중단이나 시스템 다운 없이 서비스를 마이그레이션하거나 업데이트하는 프로세스 입니다. 

무중단을 통해서 사용자 경험을 향상시키고, 서비스의 안정성을 높일 수 있습니다. 이러한 무중단 배포 방식에는 여러가지가 있습니다. 


<br>

## 1. Blue-Green Deployment
Blue-Green 배포는 두 개의 환경을 사용하여 배포하는 방식입니다.
서로 다른 두개의 동일한 환경을 만들고 그 앞에 로드 밸런서를 두는 것이 특징입니다. 로드 밸런서는 현재 프로덕션 코드 기반인 Blue환경으로 트래픽을 라우팅하고, 새로운 변경 사항이 Green 환경에 배포되고 테스트, 검증이 격리되어 수행되고 나면 로드 밸러서가 프로덕션에서 라이브 트래픽을 Green 환경으로 전환합니다.

### 장점

- 로드 밸런서가 전환되면 트래픽이 새로운 코드 베이스로 라우팅되므로 다운타임이 전혀 발생하지 않습니다.
- 개발자가 **이상적인 프로덕션 환경**에서의 테스트를 통해 예상치 못한 오류를 최소화할 수 있지만, 프로덕션 환경에서 가비지 데이터가 생성될 수 있으므로 주의해야합니다.
- 롤백이 쉽습니다. 로드 밸런서를 이전 버전으로 전환하면 됩니다.

### 단점

- 트래픽을 처리하는 한 시스템에서 다른 시스템으로 전환하면 현재 트랜잭션과 세션이 손실됩니다.(따라서 세션 지속성을 유지하기 위한 추가적인 기술적 대책이 필요합니다.)
- 이전 버전과 새 버전 모두 동일한 기본 데이터베이스가 사용되며 이전 코드 버전에서 지원하지 않는 데이터베이스 변경사항이 있을 경우, 이로 인해 N-1 데이터 호환성 문제가 발생할 수 있고, 잠재적으로 프로덕션이 중단될 수 있습니다. 


<br>

---

## 2. Canary Deployment
Canary 배포는 소수의 사용자에게 새로운 기능을 먼저 출시하고 다른 사용자에게는 점진적으로 기능을 공개하는 방식입니다. 일부 회사에서는 이 기능을 사용하여 새로운 기능에 대한 사용자의 반응을 파악하고 이를 바탕으로 전체 사용자에게 릴리스할 지 여부를 결정합니다.

### 장점

- 새로운 기능을 출시하기 전에 사용자의 반응을 확인할 수 있습니다.
- 사용자 반응을 기반으로 개선점을 찾아 조기에 문제를 해결할 수 있습니다.


### 단점

- 이 또한 N-1 데이터 호환성 문제를 일으킬 수도 있습니다.


<br>

---

## 3. Rolling Deployment
롤링 배포 전략은 현재 실행 중인 애플리케이션의 인스턴스를 최신 인스턴스로 천천히 교체하는 것으로, 어느 시점에 정확히 N+1개의 인스턴스가 실행되고 있습니다. 이전 버전의 인스턴스를 제거하고 새 버전의 인스턴스를 추가하는 방식입니다.


### 장점
- 점진적으로 롤아웃을 통해서 안정성을 확보할 수 있습니다.
- 서비스 중단 없이 새로운 버전으로의 전환을 가능하게 합니다.


### 단점
- 이 또한 N-1 데이터 호환성 문제를 일으킬 수도 있습니다.
- 점진적으로 롤아웃을 진행하면서 발생할 수 있는 문제를 감지하고 대응하는 데 시간이 걸릴 수 있습니다.


## 결론

무중단 배포 전략은 서비스의 안정성과 사용자 경험을 향상시키는데 중요한 역할을 합니다. 프로덕션 환경에서 새로운 변경 사항을 배포하는 동안 애플리케이션을 중단 없이 유지할 수 있도록 하는 것이 목표입니다. 이러한 배포 전략을 적절히 선택하여 사용자에게 최상의 서비스를 제공할 수 있도록 노력해야 합니다.




### 부록 : N-1 호환성 문제

소프트웨어의 새 버전(N)과 이전 버전(N-1) 간에 데이터 호환성 문제가 발생할 수 있습니다. 이러한 호환성 문제는 다음과 같은 상황에서 발생할 수 있습니다.

#### 1. 데이터베이스 스키마 변경:
새 버전의 소프트웨어에서 데이터베이스 스키마가 변경되었을 경우, 이전 버전의 소프트웨어가 그 변경사항을 이해하지 못하거나 잘못된 방식으로 데이터를 처리할 위험이 있습니다. 예를 들어, 새 버전에서 새로운 테이블이 추가되거나 기존 테이블의 컬럼이 수정되었을 때 이전 버전이 이를 인식하지 못하고 오류를 발생시킬 수 있습니다.

#### 2. 데이터 포맷 변경
데이터를 저장하거나 전송하는 포맷이 업데이트 되었을 때, 이전 버전이 새 포맷을 인식하지 못해 데이터를 잘못 해석하거나 처리 실패를 경험할 수 있습니다.

#### 3. 기능적 변화
새 버전에서 특정 기능이 변경되어 데이터를 다르게 처리하는 경우, 이전 버전에서는 그 변경사항을 적용할 수 없어 데이터 처리에 오류가 발생할 수 있습니다.




### 참고사이트

https://hudi.blog/zero-downtime-deployment/