# **4주차 Pod**

## **Container와 Pod 비교**

### **Container**

- Container = Application

### **Pod**

- Container를 표현하는 K8S API 최소 단위
  - k8s에서는 개인 Conatiner로 직접 실행 X
- 명령의 흐름 : API -> Pod -> Container
- 하나의 Pod에는 1개 또는 그 이상의 Container가 포함 될 수 있음.

```shell
# run 실행
kubectl run [name] --image=[image_name]:[version]

# yaml file로 실행
kubectl create -f [file_name.yaml]
```

### **Multi Container Pod**

- 하나의 Pod에 여러 Container가 존재.
- 같은 Pod안의 컨테이너는 IP가 동일.

```yaml
# Multi Container 실행
apiVersion: v1
kind: Pod
metadata:
  name: multipod # Pod 이름
spec:
  containers: # container를 여러 개 기입
    # container 1
    - name: nginx-container
      image: nginx:1.14
      ports:
        - containerPort: 80
    # container 2
    - name: centos-container
      image: centos:7
      command:
        - sleep
        - "10000"
```

```shell
# Pod 안에 컨테이너 접속
kubectl exec [pod_name] -c [container_name] [options]
```

## **Pod 동작 Flow**

1. Pending
   - 문법 점검
   - Scheduler에서 Node 선택
2. ContainerCreatiing
   - Image가 있을 경우, 생성
3. Running

## **Pod: livenessProbe**

- Self-Healing 기능
  - 컨테이너가 제대로 동작되지 않을 때, Restart
  -

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod # Pod 이름
spec:
  containers: # container를 여러 개 기입
    # container 1
    - name: nginx-container
      image: nginx:1.14
      livenessProbe: # Service에 맞는 방법으로 검사(httpGet, tcpSocket, exec)
        httpGet:
        path: /
        prot: 80
```

### **Liveness Probe 매커니즘.**

- httpGet
  - 지정된 IP 주소, 포트, Path에 요청
  - 반환 값이 200인지 확인
  - 연속 3번 실패하면 Kill, 새로운 컨테이너를 다운 받아서 실행
- tcpSocket
  - 지정된 포트에 연결 시도
- exec
  - exec 명령 실행
  - 반환 값이 0인지 확인

### **Liveness Probe 매개변수**

- periodSeconds: health Check 반복 실행 시간(초) : default 10
- initialDelaySeconds: Pod 실행 후, delay 시간(초) : default 0
- timeoutSeconds: health check 후 응답 대기 시간(초) : default 1

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod # Pod 이름
spec:
  containers: # container를 여러 개 기입
    # container 1
    - name: nginx-container
      image: nginx:1.14
      livenessProbe: # Service에 맞는 방법으로 검사(httpGet, tcpSocket, exec)
        httpGet:
        path: /
        prot: 80
        initialDelaySeconds: 15
        periodSeconds: 20
        timeoutSeconds: 1
        successThreshold: 1 # 성공 횟수
        failerThreshold: 3 # 연속 실패 횟수
```
