## 쿠버네티스 Pod - single/multi container

### pod란
- 컨테이너를 표현하는 K8S API의 최소 단위
- API에서 컨테이너 동작은 불가능
- Pod에는 하나 또는 여러 개의 컨테이너가 포함될 수 있음

<br>

pod 생성하기
- kubectl run 명령으로 생성
```bash
kubectl run webserver --image=nginx:1.14
```
- pod yaml 을 이용해 생성
```bash
kubectl create -f pod-nginx.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: webserver
spec:
    containers:
    - name: nginx-container
     image: nginx:1.14
     imagePullPolicy: Always
     ports:
     - containerPort: 80
        protocol: TCP
```

```yaml
# multi containers
apiVersion: v1
kind: Pod
metadata:
  name: multi-nginx
spec:
  containers:
    - name: nginx-container
      image: nginx:1.14
      ports:
        - containerPort: 80
          protocol: TCP
    - name: centos-container
      image: centos:7
      command:
      - sleep
      - "3600"
```

<br>

- pod를 자세히 보기
```bash
# 상세하게 볼 수 있다.
kubectl describe pod [pod-name]
```


NAME          |READY   |STATUS    |RESTARTS |  AGE  | IP |           NODE  |     NOMINATED NODE|   READINESS GATES
|--|--|--|--|--|--|--|--|--|
|multi-nginx |  2/2    | Running |  0  |        15s  | 10.244.0.22  | minikube |  none    |       none
|nginx-pod    | 1/1   |  Running |  0  |        58m  | 10.244.0.21  | minikube |  none    |       none
|web1         | 1/1  |   Running |  0  |        60m  | 10.244.0.20  | minikube |  none    |       none

<br>

> READY 는 container의 갯수
> 모든 container가 잘 동작되어야 Running


```bash
# pod의 container 접속
kubectl exec --[pod-name] -c [container-name] -it -- /bin/bash

# 위의 centos container에서 nginx로 curl
kubectl exec multi-nginx -c centos-container -it -- /bin/bash
curl localhost:80
# success..!
```

> <strong>multi-container의 pod에서 container들의 pod 명과 ip는 동일하다..!!</strong>

```bash
# multi pod는 container 이름까지
kubectl logs [multi-pod-name] -c [container-name]
# single pod는 pod만
kubectl logs [pod-name]
```

<br>

---

<br>

## 쿠버네티스 Pod - Pod 동작 flow

- 스케줄링 받을때 까지는 -> pending  
- 스케쥴링 받은 후 -> running  
- running 후 -> succedd / failed  


```bash
# 동작중인 pod
kubectl get pods
kubectl get pods -o wide
kubectl describe pod [podname]

# 동작중인 pod 수정
kubectl edit pod [pod-name]

# 모든 pod 삭제
kubectl delete pod --all
```



<br>

---

<br>

## 쿠버네티스 Pod - libeness probe

```livenessprobe로 helath check```

### livenessProbe 매커니즘
>- httpGet : 지정한 IP주소, port, path에 HTTP GET 요청을 보내, 해당 컨테이너가 응답하는지 확인한다. 반환코드가 200이 아닌 값이 나오면 오류. 컨테이너를 다시 시작한다.
```yaml
libenessProbe:
    httpGet:
        path: /
        port: 80
```
>- tcpSocket : 지정된 포트에 TCP연결을 시도. 연결되지 않으면 컨테이너를 다시 시도한다.(nfsd/nfs-daemon, sshd/ssh-daemon)
```yaml
libenessProbe:
    tcpSocket:
        port: 22
```
>- exec : exec 명령을 전달하고 명령의 종료 코드가 0이 아니면 컨테이너를 다시 시작한다.(특정 파일이 존재하는지?)
```yaml
libenessProbe:
    exec:
        command:
        - ls
        - /data/file
```

<br>

### livenessProbe 매개 변수 -> 안쓰면 default 값
- periodSeconds: health check 반복 실행 시간
- initialDelaySeconds : Pod 실행 후 delay 시간
- timeoutSeconds : health check 후 응답을 기다리는 시간
```yaml
libenessProbe:
    httpGet:
        path: /
        port: 80
    initialDelaySeconds: 15
    periodSeconds: 20
    timeoutSeconds: 1
    successThreshold: 1
    failureThreshold: 3
```

```bash
# liveness 조회 -> describe or yaml or json
    Liveness:       http-get http://:80/ delay=15s timeout=1s period=20s #success=1 #failure=3
# liveness yaml 조회
kubectl get pod [pod-name] -o yaml
```

```yaml
# 따로 입력하지 않아도 default 값이 존재한다.
    livenessProbe:
      failureThreshold: 3
      httpGet:
        path: /
        port: 80
        scheme: HTTP
      initialDelaySeconds: 15
      periodSeconds: 20
      successThreshold: 1
      timeoutSeconds: 1
```


<br>

---

<br>

## 쿠버네티스 Pod - init container & infra container

init container
- pod가 생성되고 container가 생성되기 전에 실행되는 container
- 앱 컨테이너가 실행되기 전에 준비해야 할 작업이 있을 때 사용
- 초기화 컨테이너가 모두 실행된 후에 앱 컨테이너가 실행된다.


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app.kubernetes.io/name: MyApp
spec:
    # 메인 컨테이너
    containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
    # 초기화 컨테이너 -> 다 실행된 후 메인컨테이너
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
```

init container를 실행시키는 yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376

apiVersion: v1
kind: Service
metadata:
  name: mydb
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9377
```

### pause container -> pod에 대한 infr(ip, hostname, dns, port 등)을 제공하는 container
- 그냥 같이 생성된다..?