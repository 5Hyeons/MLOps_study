# **K8S 구성 및 개념**

- Contatiner Orchestration System.
- 컨테이너화된 워크로드와 서비스를 관리하기 위한 이식성이 있고, 확장 가능한 오픈소스 플랫폼.

## **배포 방식에 따른 비교**

![배포 방식에 따른 시스템 구성](https://d33wubrfki0l68.cloudfront.net/26a177ede4d7b032362289c6fccd448fc4a91174/eb693/images/docs/container_evolution.svg)

| Timeline    | Description           |
| ----------- | --------------------- |
| Traditional | 물리서버              |
| Virtualized | 단일 물리서버, 멀티VM |
| Container   | VM과 유사, but OS공유 |

- 장점
  - 지속적인 개발, 통합 및 배포: 안정적이고 주기적으로 컨테이너 이미지를 빌드해서 배포할 수 있고 (이미지의 불변성 덕에) 빠르고 효율적으로 롤백할 수 있다.
  - 개발과 운영의 관심사 분리(?): 배포 시점이 아닌 빌드/릴리스 시점에 애플리케이션 컨테이너 이미지를 만들기 때문에, 애플리케이션이 인프라스트럭처에서 분리된다.
  - 개발, 테스팅 및 운영 환경에 걸친 일관성: 랩탑에서도 클라우드에서와 동일하게 구동.

---

## **쿠버네티스는**

- 쿠버네티스는 소스코드를 배포하지 않으며, 애플리케이션을 빌드하지 않는다.
- **쿠버네티스는 단순한 오케스트레이션 시스템이 아니다.** 오케스트레이션의 필요성을 없앰.
  쿠버네티스는 독립적이고 조합 가능한 제어 프로세스들로 구성됨.
  > 오케스트레이션의 기술적 정의 - A를 먼저 한 다음, B를 하고, C를 하는 것과 같이 정의된 워크플로우를 수행하는 것.

---

## **구성 요소**

![K8S구성요소](https://d33wubrfki0l68.cloudfront.net/2475489eaf20163ec0f54ddc1d92aa8d4c87c96b/e7c81/images/docs/components-of-kubernetes.svg)

### **Control Plane**

- 클러스터에 관한 전반적인 결정을 수행.
- 이벤트를 감지하고 반응.

#### **kube-apiserver**

- 쿠버네티스 API의 주요 구현.
- 쿠버네티스 API를 노출.
- Control Plane의 프론트엔드.

#### **etcd**

- 모든 데이터를 담는 일관성, 고가용성 키-값 저장소

#### **kube-controller-manager**

- 컨트롤러 프로세스를 실행.

#### **cloud-controller-manager**

- 클라우드 공급자의 API에 연결

### **노드**

- 동작 중인 파드를 유지
- 런타임 환경을 제공

#### **kubelet**

- 각 노드에서 실행되는 에이전트

#### **kube-proxy**

- 각 노드에서 실행되는 네트워크 프폴시

#### **컨테이너 런타임**

- 컨테이너 실행을 담당

---

# **Kubeflow 구성 및 개념**

- Kubeflow is the machine learning toolkit for Kubernetes.

## **Architecture**

![Kubeflow Architecture](https://www.kubeflow.org/docs/images/kubeflow-overview-platform-diagram.svg)

## **Kubeflow components in the ML workflow**

![Kubeflow Components](https://www.kubeflow.org/docs/images/kubeflow-overview-workflow-diagram-2.svg)

- Central Dashboard : User interfaces
- Notebook Server : like JupyterLab
  - Pod(컨테이너)에 해당.
- Pipelines : 플랫폼 For building and deploying portable and scalable machine learning (ML) workflows by using Docker containers.
- Katib : AutoML을 위한
- Training Operators
- Multi-Tenacy

![Pipelines](https://www.kubeflow.org/docs/images/pipelines-architecture.png)
