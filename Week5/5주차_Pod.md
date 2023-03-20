# **5주차 Pod**

---

## **박성배**

## Init Container

- 어떤 컨테이너를 실행하기 전에, 사전에 먼저 실행하는 컨테이너
- Init Container가 모두 실행 된 후에 실행 됨.

- Ex)

  - Login Application Container
    - 사전에 DB에서 로그인 정보를 가져와야함.
    - DB에 접속해서 정보를 가져오는 컨테이너가 Init container

- Ex)
  - Model Check
  - 사전 시스템 확인 등
    - DF 실행 전, STF가 제대로 실행되었는지 확인.
    - STF 실행 전, TTS가 제대로 실행되었는지 확인.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod # 파드 이름
  labels:
    app.kubernetes.io/name: MyApp
spec:
  containers:
    - name: myapp-container
      image: busybox:1.28
      command: ["sh", "-c", "echo The app is running! && sleep 3600"]
  initContainers: # Init 컨테이너
    - name: init-myservice # Init 컨테이너 1
      image: busybox:1.28
      command: [
          "sh",
          "-c",
          "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done",
        ] # myservice가 실행될 때까지.
    - name: init-mydb # Init 컨테이너 2
      image: busybox:1.28
      command: [
          "sh",
          "-c",
          "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done",
        ] # mydb가 실행될 때까지.
```

---

## Infra Container

- Pause는 Pod를 생성 시, 자동으로 생성 됨.
- Pod 마다 생성되고, 삭제될 때, 같이 삭제됨.
- Infra(IP, Host Name)를 관리 및 생성.

---

## Static Pod

- API에게 요청을 보내지 않음.
- Kubelet Daemon에 의해 동작.
- Pod를 실행하는 Node를 직접 선택 가능.
  - static pod path : 해당 노드의 /ete/kubernetes/maifests

---

## Pod에 Resource(cpu, memory) 할당

- Request
  - 최소의 리소스를 요청.
- Limit
  - Pod 리소스 제한.
- Memory
  - 1MB == 1000KB
  - 1MiB == 1024KiB
- CPU
  - 코어수로 할당
- GPU?
  - [GPU 할당 링크](https://kubernetes.io/ko/docs/tasks/manage-gpus/scheduling-gpus/)
  - 개수로 지정.
  - GPU 번호로 지정?

[CPU 할당 공식 링크](https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/)

```yaml
# CPU 할당
apiVersion: v1
kind: Pod
metadata:
  name: cpu-demo
  namespace: cpu-example
spec:
  containers:
    # 컨테이너 별로 리소스 제한
    - name: cpu-demo-ctr
      image: vish/stress
      resources:
        limits:
          cpu: "1"
        requests:
          cpu: "0.5"
      args:
        - -cpus
        - "2"
```

[메모리 할당 공식 링크](https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource/)

```yaml
# 메모리 할당
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo
  namespace: mem-example
spec:
  containers:
    - name: memory-demo-ctr
      image: polinux/stress
      resources:
        requests:
          memory: "100Mi"
        limits:
          memory: "200Mi"
      command: ["stress"]
      args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]
```

---

## 환경 변수

- 컨테이너가 실핼될 때, 필요로하는 변수
- 컨테이너 제작 시 미리 정의
- 변경 및 추가 가능

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envar-demo
  labels:
    purpose: demonstrate-envars
spec:
  containers:
    - name: envar-demo-container
      image: gcr.io/google-samples/node-hello:1.0
      # 환경 변수 설정
      env:
        - name: DEMO_GREETING
          value: "Hello from the environment"
        - name: DEMO_FAREWELL
          value: "Such a sweet sorrow"
```

---

## Pod 실행 패턴

- Sidecar
- Adapter
- Ambassador

---
