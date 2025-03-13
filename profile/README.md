# CloudWave 4기 올리브영 AWS 인프라 구축 프로젝트 2조

<div align=center>
  
<img src="https://github.com/user-attachments/assets/ba1b7b45-0d63-4e80-99cb-8832f237e70b" width="500">

<br>프로젝트 최종 결과 최우수상(1위)

</div>

<br>

## 프로젝트 개요  

- **프로젝트명** : 올리브영 팝업 예매 시스템, 반짝 예매  

- **프로젝트 기간** :  2025.02.07 ~ 2025.02.27 (3주)  

- **프로젝트 목표**    

    - **대기열을 활용한 티켓팅 시스템 구축** – 실시간 트래픽 관리 및 공정한 예매 환경 제공  
    - **대규모 트래픽 처리** – 최대 50,000 RPS 안정적인 서비스 유지  
    - **재난 상황에서도 지속적인 서비스 운영** – 장애 감지 및 복구 시스템 적용으로 고가용성 확보  


<br>

## 팀 구성원
|  김동윤  |  김은비  |  박현지  |  양서윤  |  조수훈  |  최우영  |
| :-----: | :-----: | :-----: | :-----: | :-----: | :-----: | 
| <img src="https://avatars.githubusercontent.com/u/77170214?v=4" width=150px alt="김동윤"> | <img src="https://avatars.githubusercontent.com/u/140461856?v=4" width=150px alt="김은비"> | <img src="https://avatars.githubusercontent.com/u/119161420?v=4" width=150px alt="박현지"> | <img src="https://avatars.githubusercontent.com/u/138513591?v=4" width=150px alt="양서윤"> | <img src="https://avatars.githubusercontent.com/u/82464990?v=4" width=150px alt="조수훈"> | <img src="https://avatars.githubusercontent.com/u/80168342?v=4" width=150px alt="최우영"> |
| [@BusanGukbap](https://github.com/BusanGukbap) | [@muuuumbi](https://github.com/muuuumbi) | [@hhyuun](https://github.com/hhyuun) | [@seoyun0311](https://github.com/seoyun0311) | [@s2hoon](https://github.com/s2hoon) | [@wyoung163](https://github.com/wyoung163) |


<br>


## 개발 환경
<img width="950" alt="infra" src="https://github.com/user-attachments/assets/c67825a5-6eef-4b46-9a01-6b52dd34a963" />
<img width="1000" alt="tool" src="https://github.com/user-attachments/assets/23fb8aa1-7551-4dfd-a18a-3cb64f3c26b6" />


<br><br>

## 전체 아키텍처
<img width="1000" alt="Image" src="https://github.com/user-attachments/assets/5258c1b3-2328-477f-9fe8-60f1dc1655bd" />

### 1.배포계
<img width="1000" alt="Image" src="https://github.com/user-attachments/assets/3b6234cb-bca2-4187-afc5-0f4557b3b324" />

**실제 운영 환경**

- EKS(Elastic Kubernetes Service) 기반으로 구축 <br>
- 클라이언트 요청은 Route 53 → CloudFront → ALB → Kubernetes Ingress → ClusterIP Service → Pod 순으로 처리 <br>
- ElastiCache는 대기 순번 조회 및 티켓 잔여석 조회 용도 활용, AuroraDB Global을 도입하여 멀티 리전 간 데이터 동기화 구현 <br>

### 2. 개발계
<img width="1000" alt="Image" src="https://github.com/user-attachments/assets/6178e0c4-9277-4b65-b466-8490f7e2a1b5" />

**개발 및 QA 환경** <br>

- 보안: Jenkins는 Bastion 서버를 통해서만 접근 가능
- CI/CD 흐름:
    API Gateway + Lambda → Jenkins Webhook 호출 → Jenkins가 ECR에 컨테이너 이미지 푸시 → ArgoCD가 Digest 기반으로 이미지 변경 감지 후 EKS 배포

### 3. DR
<img width="1000" alt="dr 최종수정" src="https://github.com/user-attachments/assets/f2595625-35f2-4cab-ae47-70b16b5dfed6" />



**재해 복구 모델 환경** <br>

재난 발생 시 복구 시간 최소화를 위해 2가지 환경 설계 <br>
1. EKS 기반 DR 환경 : Standby 상태 <br>
2. ECS 기반 DR 환경 : Active 상태 <br>

- DR 모델은 Route53의 가중치 기반 라우팅을 활용하여 AWS 서울 리전의 서비스가 실패하면 싱가포르 리전의 서비스로 연결되도록 구성 <br>
- DR EKS 환경을 구축하기까지 시간이 20~30분 소요, DR ECS를 환경을 운영 환경 함께 Active-Active 하게 운영하여 DR EKS 환경이 구축될 때까지 이용 <br><br>

라우팅 가중치 설정
| 상태                          | 운영 (Production) | DR EKS | DR ECS |
|------------------------------|------------------|--------|--------|
| 평상시                 | 100              | 0      | 0      |
| 재난 발생 (DR EKS 구축 중)  | 0                | 50     | 50     |
| DR EKS 구축 완료 (Health Check 성공 시) | 0               | 100     | 0     |


<br>

## 주요 기능

https://github.com/user-attachments/assets/40eaea19-4099-43f4-bba0-8179e3bda7af

### 데이터 흐름

- **예매 버튼 클릭**
    - Kafka Producer에게 선택한 예매 시간 데이터를 전달
- **Kafka Producer 응답**
    - UUID 쿠키 발급 후 로딩창으로 이동
- **Kafka Consumer 처리**
    - Kafka Consumer가 메시지 수신 후, 대기열 데이터를 ElastiCache 에 저장
- **로딩창 동작** (WebSocket 활용)
    - ElastiCache 에 저장된 대기열 정보(예매창 접속 여부, 현재 대기 인원)를 WebSocket을 통해 지속 확인
    - 접속 여부가 true가 되면 즉시 예매창으로 이동
- **예매창 동작** (WebSocket 활용)
    - 각 예매 시간별 남은 인원수를 WebSocket으로 실시간 확인
    - 특정 시간의 남은 인원이 0이면 해당 시간 선택 버튼 비활성화
- **예매 완료 프로세스**
    - 예매 시간 선택 후 "예매 완료" 클릭 시 Kafka Producer에게 선택한 시간 정보를 전달
    - Kafka Consumer 가 메시지 수신후, 예매 완료 데이터 ElastiCache 에 저장

<br>



## 대규모 트래픽(5만) 처리를 위한 고려 사항
<img width="1000" alt="Image" src="https://github.com/user-attachments/assets/e48566eb-8cc1-47f0-8e8a-7364803f4e11" />

- 500rps를 처리하기 위한 CPU 및 메모리 스펙을 통한 50000rps를 위한 스펙 예측
- K8s pod 및 node 오토 스케일

<br>

## 트래픽 처리를 위한 Kafka 도입

<img width="1000" alt="Image" src="https://github.com/user-attachments/assets/a4fc9135-e99f-48b8-91cb-37541746624a" />
 
- **도입 배경** : 기존 시스템의 제한된 리소스와 처리 성능으로는 5만 이상의 트래픽 처리에 한계가 있다는 점 인식
- **해결 방안** : Kafka 도입을 통한 처리 성능 향상 및 트래픽 손실 최소화
- **메시지 처리 방식** :
  - Kafka Broker에 메시지를 순차적으로 저장하고 메시지를 여러 파티션에 복제하여 트래픽 손실 감소 
  - Consumer에서 토픽을 구독하고 메시지에 대한 비즈니스 로직 처리 


<br/>

## 인프라 구축 & CI/CD

### Terraform

<details>
<summary>인프라 구축을 위한 테라폼 도입</summary>

  - Terraform : 클라우드 및 온프레미스 환경의 인프라를 코드로 정의하고 관리할 수 있도록 지원하는 오픈 소스 소프트웨어
  - 장점
    1. 인프라를 코드로 정의 → 자동화
    2. 모듈화 기능을 활용하여 반복적으로 사용되는 구성 재사용 → 유지보수성 향상
    3. 장애 발생 시 이용 → 신속한 복구

  - 적용 범위
    1. 개발 환경
    2. DR 환경 EKS 클러스터 구성
    
</details>


<details>
<summary>Terraform Infracost를 이용한 비용 예측</summary>

![image 75](https://github.com/user-attachments/assets/1fdfadbb-e233-4cf3-9876-d98c7e1f20ac)
비용 예측 및 실제 비용 비교

- Terraform Infracost 예측: 한 달 약 $1,388 (약 200만 원)
- 실제 부하 테스트 결과: 하루 $132, 10일 기준 한 달 $2,257 예상
  
비용 절감 전략

- MSK 대신 자체 Kafka 클러스터 운영 → MSK 사용 비용 절감
- ECS 대신 EKS 활용 → Kafka 같은 데이터 집약적 서비스의 높은 스토리지 비용 절감

</details>



<details>
<summary>Tfsec</summary>
<img width="360" alt="prod problem 43 1" src="https://github.com/user-attachments/assets/7be91fd4-2cea-481e-b8a4-9c570c393e3c" />
<img width="360" alt="prod problem 10개 1" src="https://github.com/user-attachments/assets/6640b448-da51-4c5c-b666-99875f082117" />


- Tfsec : Terraform 코드 보안 검사 도구로, IaC 보안 취약점을 분석하여 클라우드 환경에서의 보안 위험을 사전에 감지하는 역할
- 보안 점검 통과율을 초기 58.67%에서 90.10%로 개선하여 +49.18% 향상

</details>

<br>

### CI/CD
#### Jenkins - CI
<img width="650" alt="수정 CI" src="https://github.com/user-attachments/assets/65e7d27d-0715-4351-a812-dcd7f90030c9" />

<details>
  <summary>Jenkins 사용 이유</summary>
  
  <ul>
    <li>멀티 플랫폼(Windows, macOS, Linux, Docker, Kubernetes) 등 어떤 환경에서도 동작 가능</li>
    <li>자체적인 분산 빌드 시스템(Multi-agent 구조) 지원 → 여러 개의 노드 사용으로 병렬 빌드 가능</li>
    <li>무료 오픈소스(비용 절감), 커스터마이징(매우 유연), 다양한 플러그인(높은 확장성)</li>
  </ul>
</details>

<details>
  <summary>Github webhook 연동</summary>
  
  <ul>
    <li>보안을 위해 Jenkins를 Public subnet에 노출하지 않고, API Gateway와 Lambda를 사용</li>
    <li>Github webhook에 secret 설정 → Lambda에서 webhook secret 검증 후 Jenkins로 전달</li>
    <li>API Gateway는 RestAPI 구축 후 lambda 프록시 통합을 통해 webhook 응답을 그대로 lambda에게 전달</li>
    <li>lambda 함수에서 Github webhook이 올바른지 판단 후 Jenkins로 webhook 전달</li>
  </ul>
</details>

<details>
  <summary>Sonarqube 연동</summary>
  
  <ul>
    <li>Sonarqube : 소스 코드 품질 및 보안 분석을 자동으로 수행하는 정적 코드 분석 도구</li>
    <li>코드 버그와 보안 취약점 탐지, 지속적으로 코드 품질 개선 목적</li>
    <li>sonarqube로 정적 코드 분석으로 코드 품질향상</li>
    <li>checkout → sonarqube → test → build → image push</li>
  </ul>
    
</details>

<details>
  <summary>ECR registry 활용</summary>
  
  <ul>
    <li>EKS에서 배포하기 위해 Jenkins CI 마지막에 Docker Image 형태로 저장 후 ECR에 Push</li>
  </ul>
</details>

<br>

#### ArgoCD - CD
<img width="600" alt="수정 CD" src="https://github.com/user-attachments/assets/1427c206-c6f7-4ffc-a384-5e33ba23d4bc" />

<details>
  <summary>ArgoCD 사용 이유</summary>
  
  <ul>
    <li>EKS에 배포 & GitOps 방식에 적합</li>
    <li>Helm을 사용하기에 적합</li>
    <li>Jenkins CD는 Kubernetes에 친화적이지 않음</li>
  </ul>
</details>

<details>
  <summary>Image digest 업데이트 방식</summary>
  
  <ul>
    <li>latest 업데이트 방식을 사용 시 이미지 태그 변경 없이 같은 태그(latest)만 업데이트 된다면, ArgoCD가 이를 감지하지 못함 → digest 업데이트 방식 채택</li>
  </ul>
</details>

<details>
  <summary>Canary 무중단 배포 전략</summary>
  
  <ul>
    <li>롤링, 블루/그린, 카나리 3가지의 무중단 배포 전략 중 카나리 선택</li>
    <li>운영 비용 절감 및 낮은 배포 리스크, 모니터링 및 분석을 위해 Canary 배포 채택</li>
  </ul>
</details>

<details>
  <summary>helm chart를 활용한 배포</summary>
  
  <ul>
    <li>GitOps 방식으로 ArgoCD를 관리하기 위해 Helm 사용</li>
    Git Repository → Helm Chart → ArgoCD → EKS 배포
    <li>helm 사용으로 개발/배포 환경을 구분하여 관리 가능</li>
  </ul>
</details>


<br><br>

## 모니터링 & 부하 테스트
모니터링 도구 **Datadog**을 이용하여 Kubernetes 기반 컨테이너 환경의 상태 및 리소스 사용량을 추적하고, Pod, Node, Cluster의 상태를 모니터링 했습니다.
<details>
  <summary>Datadog 통합 대시보드</summary>
  
  <img width="800" alt="Image" src="https://github.com/user-attachments/assets/71d4e019-9c21-491a-931b-1c85e83eb00f" />
  
</details>


### 모니터링 도구를 활용한 DR 구축

<details>
  <summary>재난 상황 발생 시 슬랙을 통한 알림과 DR 구축 시나리오</summary>
  
  ![dr 시나리오](https://github.com/user-attachments/assets/b18062bf-f79f-4222-a939-f7918e508917)
    
  1. Route 53의 Health Check 실패
  2. Datadog이 Slack으로 알림 발송
    
  <img width="400" alt="Image" src="https://github.com/user-attachments/assets/5c985023-1069-4ac4-9996-999bca50744e" />
      
  3. Webhook 트리거 발생 -> API Gateway가 Lambda 호출 -> Lambda가 Jenkins 실행 -> Jenkins가 Terraform 실행하여 인프라 프로비저닝
    
</details>

### Jmeter를 활용한 부하 테스트

<details>
  <summary>8000rps 테스트 결과</summary>
  
  https://github.com/user-attachments/assets/1dca5c49-374c-4c0d-9d80-6f4b8361d13c
  
  - kafka 관련 파드: 5개 → 15 ~ 20개
  - c6.xlarge 노드: 3개 → 4개
    
  <img width="400" alt="Image" src="https://github.com/user-attachments/assets/c0f381c0-0ad1-47e6-8b5b-93cf7f255637" />
  <img width="400" alt="Image" src="https://github.com/user-attachments/assets/6137f4ab-c551-4c89-bff3-c848a7523539" />

</details>



