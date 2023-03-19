## Pod - static pod

- api에게 요청을 보내지 않음 -> kubelet에게 요청을 보냄
- yaml파일을 넣어놓은 dir을 watch하고 있음
- kubelet이 pod를 생성하고 관리함
- /etc/kubernetes/manifests/에 yaml파일을 넣어놓으면 kubelet이 자동으로 pod를 생성해줌(항상 고정되어있는 dir은 아니다 -> node의 /var/lib/kubelet/config.yaml의 staticPodPath에 설정되어있음)  
- yaml파일이 생성되면 pod생성되고 삭제되면 pod도 삭제됨
```yaml
rotateCertificates: true
runtimeRequestTimeout: 15m0s
shutdownGracePeriod: 0s
shutdownGracePeriodCriticalPods: 0s
staticPodPath: /etc/kubernetes/manifests <- 이 부분
streamingConnectionIdleTimeout: 0s
syncFrequency: 0s
volumeStatsAggPeriod: 0s
```


<br>

/etc/kubernetes/manifests/가 아닌 다른 곳에 넣고 싶다면?  
-> config.yaml에 staticPodPath를 설정하고 kubelet을 재시작 하면 된다.

<br>

---

<br>

## Pod - Resource 할당(cpu, memory)
- 다른 pod의 동작을 보장하기 위하여 resource를 할당해야함
- resource를 할당하지 않으면 다른 pod에 영향을 줄 수 있음
- 요구조건을 적어놓으면 scheduler가 pod를 배치할 때 resource를 고려하여 배치함

### resource requests
- pod가 실행되기 위해 필요한 최소한의 resource를 요구하는 것
- node의 resource가 부족하면 pending 상태가 됨

### resource limits -> 자동으로 resource requests를 설정해줌
- pod가 사용할 수 있는 최대 resource를 제한하는 것
- memory limit을 넘어가면(넘을 수 있긴함) OOM killer가 pod를 죽임 -> 다시 스케쥴링 되어 실행됨  


메모리
> 1MB = 1000KB  
> 1MiB = 1024KiB  

cpu
> 1 core = 1000m(밀리코어)  
> 1 core => 1(코어)  
> 200m => 0.2(코어)




참조
- [자원회수](https://kubernetes.io/docs/tasks/administer-cluster/out-of-resource/#node-oom-behavior)
- [자원할당](https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/)

<br>

---

<br>

## Pod - 환경변수 설정
- spec > containers > env > name, value로 설정
```yaml
spec:
  containers:
  - name: nginx-container
    image: nginx:1.14
    ports:
    - containerPort: 80
      protocol: TCP
    env:
    - name: MY_ENV
      value: "Hello, World!"
```


<br>

---

<br>

## Pod - 구성 패턴의 종류
- Pod를 구성하고 실행하는 패턴
- multi-container pod

  
|sidecar container|adapter|ambassador|
|:---:|:---:|:---:|
|웹서버 + 로그 수집기(단독실행 불가)|프론트 웹서버 + 외부 API 연동|웹서버 + LB|
|로그를 만들어 줘야지만 실행할 수 있는 형태의 Pod|프론트에서 외부 API를 호출할 수 있도록 해주는 역할|분산된 서비스를 하나의 IP로 접근할 수 있도록 해주는 역할|