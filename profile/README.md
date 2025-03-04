# CloudWave 4기 2조 올리브영 AWS 인프라 구축 프로젝트

<div align=center>
  
<img src="https://github.com/user-attachments/assets/ae50e708-ec0a-44d0-81af-f8fc3f4daa9d" width="500">

<br>프로젝트 최종 결과 최우수상(1위)

</div>

<br>

## 프로젝트 개요  

- **프로젝트명** : 올리브영 팝업 예매 시스템, 반짝 예매  

- **프로젝트 기간** :  2025.02.07 ~ 2025.02.27 (3주)  

- **프로젝트 목표**    

    - **대기열을 활용한 티켓팅 시스템 구축** – 실시간 트래픽 관리 및 공정한 예매 환경 제공  
    - **대규모 트래픽(5만 명 이상) 처리** – 동시 접속자 증가에도 안정적인 서비스 유지  
    - **재난 상황에서도 지속적인 서비스 운영** – 장애 감지 및 복구 시스템 적용으로 고가용성 확보  


<br>

## 팀 구성원
|  김동윤  |  김은비  |  박현지  |  양서윤  |  조수훈  |  최우영  |
| :-----: | :-----: | :-----: | :-----: | :-----: | :-----: | 
| <img src="https://avatars.githubusercontent.com/u/77170214?v=4" width=150px alt="김동윤"> | <img src="https://avatars.githubusercontent.com/u/140461856?v=4" width=150px alt="김은비"> | <img src="https://avatars.githubusercontent.com/u/119161420?v=4" width=150px alt="박현지"> | <img src="https://avatars.githubusercontent.com/u/138513591?v=4" width=150px alt="양서윤"> | <img src="https://avatars.githubusercontent.com/u/82464990?v=4" width=150px alt="조수훈"> | <img src="https://avatars.githubusercontent.com/u/80168342?v=4" width=150px alt="최우영"> |
| [@BusanGukbap](https://github.com/BusanGukbap) | [@muuuumbi](https://github.com/muuuumbi) | [@hhyuun](https://github.com/hhyuun) | [@seoyun0311](https://github.com/seoyun0311) | [@s2hoon](https://github.com/s2hoon) | [@wyoung163](https://github.com/wyoung163) |


<br>


## 개발 환경
<img width="1155" alt="Image" src="https://github.com/user-attachments/assets/bddf699b-3133-4091-b258-b25093efdf90" />

<br><br>

## 전체 아키텍처
<img width="1000" alt="Image" src="https://github.com/user-attachments/assets/5258c1b3-2328-477f-9fe8-60f1dc1655bd" />

### 1.배포계
<img width="1000" alt="Image" src="https://github.com/user-attachments/assets/3b6234cb-bca2-4187-afc5-0f4557b3b324" />
배포계는 실제 운영 환경으로, EKS(Elastic Kubernetes Service) 기반으로 구축되었습니다. <br>
클라이언트 요청은 Route 53 → CloudFront → ALB → Kubernetes Ingress → ClusterIP Service → Pod 순으로 처리됩니다.
ElastiCache는 대기 순번 조회 및 티켓 잔여석 조회 용도로 활용하였으며, AuroraDB Global을 도입하여 멀티 리전 간 데이터 동기화를 구현하였습니다. <br>

### 2. 개발계
<img width="1000" alt="Image" src="https://github.com/user-attachments/assets/6178e0c4-9277-4b65-b466-8490f7e2a1b5" />
개발계는 개발과 QA를 위한 환경을 구축하였습니다. <br>

### 3. DR
<img width="1000" alt="Image" src="https://github.com/user-attachments/assets/6ca91fad-d116-4110-98ba-350ae077b5f6" />
DR은 Route53의 가중치 기반 라우팅을 활용하여 AWS 서울 리전의 서비스가 실패하면 싱가포르 리전의 서비스로 연결되도록 구성하였습니다. DR EKS 환경을 구축하기까지 시간이 소요되므로 ECS를 환경을 운영 환경 함께 Active-Active 하게 운영하고자 하였습니다. <br>
- 평상 시 가중치 : 운영(100), ECS DR(0) <br>
- 재난 발생 시 가중치(EKS 환경 구축 중) : 운영(0), DR EKS(50), DR ECS(50) <br>
- DR EKS 환경 구축 완료 시 가중치(DR EKS 환경의 헬스체크 성공 기준) : 운영(50), DR EKS(50), DR ECS(50) <br>

<br>

## 주요 기능

https://github.com/user-attachments/assets/40eaea19-4099-43f4-bba0-8179e3bda7af

<br>

## 인프라 구축 & CI/CD

### Terraform

<details>
<summary>인프라 구축을 위한 테라폼 도입</summary>
</details>


<details>
<summary>Terraform Infracost를 이용한 비용 예측</summary>
</details>


<details>
<summary>Tfsec</summary>
</details>

<br>

### CI/CD
#### Jenkins - CI
<img width="650" alt="Image" src="https://github.com/user-attachments/assets/9927125a-55b5-4a1f-bab2-aba80662818a" />

<details>
  <summary>Github webhook 연동</summary>
  <ul>
    <li>보안을 위해서 API Gateway 사용</li>
  </ul>
</details>

<details>
  <summary>Sonarqube 연동</summary>
</details>

<details>
  <summary>ECR registry 활용</summary>
</details>

<br>

#### ArgoCD - CD
<img width="600" alt="Image" src="https://github.com/user-attachments/assets/68c1f23b-f292-4d2e-a3b7-1146b68052a9" />

<details>
  <summary>Image digest 업데이트 방식</summary>
</details>

<details>
  <summary>Canary 무중단 배포 전략</summary>
</details>

<details>
  <summary>helm chart를 활용한 배포</summary>
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
      
  3. Webhook -> API Gateway -> Lambda -> Jenkins -> Terraform
    
</details>

<br>

### Jmeter를 활용한 부하 테스트
대규모 트래픽(5만) 처리를 위한 고려 사항
<img width="1000" alt="Image" src="https://github.com/user-attachments/assets/e48566eb-8cc1-47f0-8e8a-7364803f4e11" />

- 500rps를 처리하기 위한 CPU 및 메모리 스펙을 통한 50000rps를 위한 스펙 예측
- K8s pod 및 node 오토 스케일링


<details>
  <summary>8000rps 테스트 결과</summary>
  
  https://github.com/user-attachments/assets/1dca5c49-374c-4c0d-9d80-6f4b8361d13c
  
  - kafka 관련 파드: 5개 → 15 ~ 20개
  - c6.xlarge 노드: 3개 → 4개
    
  <img width="500" alt="Image" src="https://github.com/user-attachments/assets/c0f381c0-0ad1-47e6-8b5b-93cf7f255637" />
  <img width="500" alt="Image" src="https://github.com/user-attachments/assets/6137f4ab-c551-4c89-bff3-c848a7523539" />

</details>



