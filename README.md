# AWS EKS Vulnerability Assessment & NetworkPolicy

##  Project Overview
이 프로젝트는 **AWS EKS(Elastic Kubernetes Service)** 환경을 비용 효율적으로 구축하고 컨테이너 보안 취약점 진단 및 **NetworkPolicy**를 활용한 네트워크 격리를 구현하여 보안을 강화하는 과정

* **주요 목표:** 비용 최적화된 클러스터 구성, 이미지/설정 취약점 식별, 파드 간 접근 제어 구현
* **역할:** 인프라 구축, 보안 진단, 정책 수립 및 검증

##  Tech Stack & Environment
| Category | Tool / Spec | Description |
| --- | --- | --- |
| **Cloud Provider** | AWS EKS v1.30 |  Seoul Region (ap-northeast-2) |
| **Infrastructure** | eksctl |  Spot Instance를 활용한 비용 절감 구성(t3.medium)  |
| **CNI** | **Calico v3.27.0** |  NetworkPolicy 적용을 위한 CNI 플러그인 |
| **Security Ops** | Trivy | 컨테이너 이미지 취약점 스캔 (CVE) |
| **Security Ops** | kube-bench | CIS Benchmark 기반 클러스터 설정 점검 |

## Key Achievements

### 1. Cost-Effective Cluster Setup
* `eksctl`을 사용하여 Managed Node Group을 구성했습니다.
***Spot Instance** 옵션을 사용하여 온디맨드 대비 비용을 최대 90% 절감하며 실습 환경을 최적화했습니다.
*  EKS v1.30 최신 안정 버전을 적용했습니다.

### 2. Vulnerability Assessment (Trivy & kube-bench)
*  **Image Scanning:** `nginx:1.16` 구버전 이미지 배포 후 **Trivy**로 스캔하여 총 **363개**의 취약점(CRITICAL 39개 포함)을 식별하고 분석했습니다.
*  **Cluster Hardening:** **kube-bench**를 통해 CIS Kubernetes Benchmark를 수행하여 `Audit Log 미활성화`, `과도한 cluster-admin 권한` 등의 보안 설정 미흡점을 발견하고 개선안을 도출했습니다.

### 3. Network Segmentation with Calico
*  **Challenge:** EKS 기본 CNI는 NetworkPolicy를 미지원하여 파드 간 통신 제어가 불가능함.
*  **Solution:** **Calico CNI**를 설치하여 NetworkPolicy 기능을 활성화함.
* **Implementation:** `access=true` 라벨이 있는 파드만 Nginx에 접근하도록 Whitelist 정책 적용.
* **Result:**
    *  **Unauthorized (hacker-pod):** 접근 시도 시 5초 후 Timeout (차단 성공).
    *  GREEN **Authorized (client-pod):** 즉시 응답 (HTTP 200 OK).

## Security Verification (Before vs After)

| 시나리오 | NetworkPolicy 적용 전 | NetworkPolicy 적용 후 | 결과 |
| :--- | :--- | :--- | :--- |
| **비인가 파드 접근** | 모든 파드에서 자유롭게 접근 가능 | 타임아웃 발생 (차단됨) | 보안 강화 |
| **내부망 스캔** | 탈취된 컨테이너가 내부 서비스 스캔 가능 | 허용된 경로 외 통신 불가 | 확산 방지 |

## Trouble Shooting & Lessons Learned

**1. NetworkPolicy 미동작 이슈**
* **문제:** NetworkPolicy를 적용했음에도 차단이 되지 않는 현상 발생.
* **원인:** EKS의 기본 VPC CNI는 NetworkPolicy를 지원하지 않음.
*  **해결:** **Calico CNI**를 추가 설치하여 정책이 네트워크 레이어에서 패킷을 필터링하도록 조치함.

**2. Spot Instance 중단**
* **문제:** 실습 도중 노드가 회수되어 파드가 재스케줄링됨.
*  **교훈:** Spot 인스턴스는 비용 효율적이나 중단 가능성이 있으므로 운영 환경에서는 신중히 혼합 구성해야 함을 학습

## Repository Structure
```bash
.
├── manifests/
│   ├── policy.yaml        # NetworkPolicy 정의서
│   └── nginx-vuln.yaml    # 타겟 애플리케이션
├── reports/               # 상세 실습 보고서 (PDF)
└── README.md
