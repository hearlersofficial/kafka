# hearlers-kafka

이 저장소는 "Hearlers" 프로젝트를 위한 Kafka 클러스터를 Helm 차트를 사용하여 관리합니다.

## 개요

이 Helm 차트는 KRaft 모드로 동작하는 Kafka 클러스터와, 웹 UI를 통해 Kafka를 쉽게 관리할 수 있도록 도와주는 Kafka-UI를 함께 배포합니다. Zookeeper 없이 컨트롤러 quorum을 사용하여 클러스터를 구성하므로 더 간단하고 효율적인 아키텍처를 가집니다.

- **Kafka Version:** 3.6.1
- **Helm Chart:** `hearlers-kafka` (version: 0.1.0)
- **Dependencies:**
  - Bitnami Kafka (version: 26.11.4)
  - Provectus Kafka-UI (version: 0.7.6)

---

## 프로젝트 구조

```
hearlers-kafka/
├── .github/
│   └── workflows/
│       └── deploy-dev.yml  # 개발 환경 자동 배포 워크플로우
├── helm/
│   └── kafka/
│       ├── Chart.yaml          # Helm 차트 정보 및 의존성 정의
│       ├── values.yaml         # 기본 설정값
│       ├── values-development.yaml # 개발 환경용 설정값
│       └── values-production.yaml  # 운영 환경용 설정값
└── README.md
```

- **`.github/workflows/`**: `develop` 브랜치에 코드가 푸시되면 GitHub Actions를 통해 개발 서버에 자동으로 Kafka 클러스터를 배포하는 CI/CD 파이프라인이 설정되어 있습니다.
- **`helm/kafka/`**: Kafka 및 Kafka-UI 배포를 위한 Helm 차트와 환경별 설정 파일이 위치합니다.

---

## 설정 (Configuration)

Helm 차트의 설정은 `values.yaml` 파일을 통해 관리되며, 각 환경에 따라 특정 값을 덮어쓸 수 있습니다.

- **`values.yaml`**: 모든 환경에 적용되는 기본 설정 파일입니다. KRaft 모드 활성화, 기본 리소스 할당량, Kafka-UI 설정 등이 포함되어 있습니다.
- **`values-development.yaml`**: 개발 환경을 위한 설정 파일입니다.
  - Broker 및 Controller replica 수를 `2`로 설정합니다.
  - 외부 접근을 위해 `NodePort` 타입의 서비스를 사용하며, 포트는 `32092`, `32093`으로 설정됩니다.
- **`values-production.yaml`**: 운영 환경을 위한 설정 파일입니다.
  - 안정적인 운영을 위해 Broker 및 Controller replica 수를 `3`으로 유지합니다.
  - 개발 환경보다 높은 CPU 및 메모리 리소스를 할당합니다.

---

## 배포 (Deployment)

### 자동 배포 (CI/CD)

`develop` 브랜치에 변경 사항이 푸시되면, `.github/workflows/deploy-dev.yml` 워크플로우가 자동으로 실행되어 개발 환경에 최신 버전의 Helm 차트를 배포합니다.

### 수동 배포

로컬 환경이나 다른 환경에 수동으로 배포하려면 다음 명령어를 사용할 수 있습니다.

**1. Helm 의존성 업데이트**

```bash
helm dependency update ./helm/kafka
```

**2. Helm 차트 배포**

- **개발 환경 (Development)**
  ```bash
  helm upgrade --install kafka-dev ./helm/kafka \
    -f ./helm/kafka/values-development.yaml \
    --namespace development \
    --create-namespace
  ```

- **운영 환경 (Production)**
  ```bash
  helm upgrade --install kafka-prod ./helm/kafka \
    -f ./helm/kafka/values-production.yaml \
    --namespace production \
    --create-namespace
  ```

---

## 접속 정보

### Kafka Broker

- **개발 환경 (Development)**:
  - `NodePort`를 통해 외부에 노출됩니다.
  - 접속 주소: `<Node_IP>:32092`, `<Node_IP>:32093`

### Kafka UI

- **개발 환경 (Development)**:
  - `NodePort`를 통해 외부에 노출됩니다.
  - 접속 주소: `http://<Node_IP>:30000`
