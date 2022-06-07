# 쿠버네티스 완벽 가이드
     
![Untitled](https://books.google.co.kr/books/publisher/content?id=0708EAAAQBAJ&printsec=frontcover&img=1&zoom=5&edge=curl&imgtk=AFLRE70k0J08Sy4yFKLFTK4IH03Hbp4LgoegcnS6k9dp2UR7mc786VkUm_r7ThfWPE-VIpFwd7lQCmV21acZnW28MNXM1DioF_E-QwG0EywNkxsaGUNU5OgcBXrEKD_pFKp0Y_CXg9Jn).  
   
   
- - -
```Index```
 
* [CH1. 도커 복습과 hello,k8s](#CH1)
* [CH2. 왜 k8s가 필요할까?](#CH2)

- - - 

   

# CH1
### 도커 복습과 hello,k8s

**DOCKER?**

- 도커 컨테이너  : 도커 이미지를 기반으로 실행되는 프로세스. 환경의 영향을 받지 않고 다양한 환경에서 컨테이너를 기동시킬 수 있어 이식성이 높다.
- 컨셉
    
     java : WORA(Write Once, Run Anywhere) feat.jvm
    
    docker : BORA(Build Once, Run Anywhere) feat.namespae,cgroups
    
- 장점 : 가상 머신에 비해 가벼우며, 시작/중지가 빠르다
- 호스트 머신의 커널을 이용하며 네임스페이스 분리와 cgroups 를 이용한 제어를 통해 독립적 os 같은 환경을 만들 수 있으므로 게스트 os 기동을 기다릴 필요가 없어 프로세스 빠르게 시작/중지 가능.
- 컨테이너 생성 시 주의할 점
    - 1 컨테이너 1 프로세스
        - 주변 에코시스템도 이 사상을 바탕으로 만들어졌으니 꼭 지키도록
    - 변경 불가능한 인프라
        - 한번 만들어진 환경은 절대 변경되지 않게 한다.
        - 실행 바이너리와 관련 리소스를 가능한 포함시킨다.
    - 경량 이미지
    - 실행 계정은 root 제외
        - 프로세스 실행 계정 권한을 최소화 하여 보안을 철저히 한다
    


**Dockerfile**

```docker
#alpine3.11 버전을 사용하는 golang 1.14.1 이미지를 베이스 이미지로 사용
FROM golang:1.14.1-alpine3.11

#호스트 머신(로컬)에 있는 main.go 를 컨테이너 home으로(./) 복사
COPY ./main.go ./

#빌드 할 때 컨테이너에서 커맨드 실행
RUN go build -o ./go-app ./main.go

#실행 계정을 nobody로 지정
USER nobody

#컨테이너가 기동할 때(docker run할 때) 실행할 명령어 정의
ENTRYPOINT ["./go-app"]
```

- ENTRYPOINT , CMD → 명령어/인자 == $ENRYPOINT $CMD

**이미지 빌드하기**

![Untitled](https://user-images.githubusercontent.com/18088806/171381732-c761ff41-0a65-4a67-8da7-d674d6e21247.png)
![Untitled 1](https://user-images.githubusercontent.com/18088806/171381534-06a72aa9-0cb9-4c0d-9c37-3d1ea741ae7a.png)

**경량화 빌드(1)**

멀티 스테이지 빌드 

레이어를 줄여 결과 이미지의 크기를 줄이기 위해 사용

위에서 사용한 golang 이미지는 컴파일만을 위해 필요하기 때문에 이전 스텝에서 컴파일한 결과물 바이너리를 이용하여 실행용 이미지만 복사하여 사용

여러 단계 이상의 구성도 가능

소스 코드 컴파일에 필요한 도구를 실제 어플리케이션을 기동시키는 컨테이너에 포함하지 않아도 됨.

```docker
################stage 1################

FROM golang:1.14.1-alpine3.11 as builder
COPY ./main.go ./
RUN go build -o /go-app ./main.go

#######################################

###############stage 2#################

FROM alpine:3.11

COPY --from=builder /go-app .
ENTRYPOINT ["./go-app"]

#######################################
```

![Untitled 2](https://user-images.githubusercontent.com/18088806/171381811-1e4df0b3-98ce-4bdb-86d0-3fd708d7d895.png)


크기가 훨~~씬 줄어든 이미지 확인 가능
![Untitled 3](https://user-images.githubusercontent.com/18088806/171382068-80ce5a64-b2fe-4eb8-af55-c9ddcf3822b0.png)


**경량화 빌드(2)**

—squash 옵션 사용 : 최종 파일 상태를 가진 한개의 레이어에 합쳐짐

(experimental feature)

Experimental 을 true 로 해줘도 엔진이 true로 안 바껴서 빌드 안됨... 왜?!

![Untitled 4](https://user-images.githubusercontent.com/18088806/171382104-4d5080e0-e63d-4366-bba2-c64301192b79.png)

![Untitled 5](https://user-images.githubusercontent.com/18088806/171382124-fd0260d0-82d9-476d-819d-fe8a51744fde.png)


+**Docker** **inspect**
![Untitled 6](https://user-images.githubusercontent.com/18088806/171382178-48d21c44-a015-49c7-941c-e99e7531d078.png)

사용된 레이어 확인 가능

---

# CH2
## 왜 k8s가 필요할까?

### 도커

- 언어, 프레임워크에 상관없이 어플리케이션을 실행할 수 있음
- 컨테이너 이미지를 활용한 배포와 롤백이 간단

### 도커 사용이 활성화 되며 나타난 불편 사항들

- 모든 도커 서버에 직접 접속해 docker run,stop 을 해줘야함
- 하나하나 도메인 관리하기가 어려움
- 컨테이너 실행 시 어떤 서버가 여유로운지 확인하기 어렵
- 버전업이 될때 모든 컨테이너 업데이트(롤아웃) 일일히 하기가 힘듦→ 롤백 시에도 하나하나 버전 내리기도 힘듦
- 컨테이너가 문제가 생기면 각각 로그를 확인하여 직접 실행하는 작업이 필요함
- 컨테이너가 매우 많아질 경우 서비스 접속 경로검색 및 노출 시 관리하기가 힘듦
    
    로드 밸런서를 두어도 도메인 제공을 자동으로 하기 어렵
    
    프록시 서버의 경우 재시작을 통해 새 설정을 반영해야함
    

### 2.1 쿠버네티스란 ?

- 컨테이너화된 여러 어플리케이션의 배포,확장 등을 관리하는 것을 자동화하기 위한 플랫폼(컨테이너 오케스트레이션 엔진)
- 도커는 도커가 설치된 호스트를 동시에 여러대 동작시키거나, 중앙에서 통합/관리할 수 없음
- 도커 자체로 여러 호스트로 구성되거나 일정 규모 이상의 서비스 환경에서 사용할 수 있는 시스템 구축이 어렵

### 2.2 쿠버네티스의 역사

- 구글이 사용하던 컨테이너 클러스터 관리 도구(Borg)에서 아이디어를 얻어 만들어진 오픈소스 sw → CNCF 로 이관

### 2.3 쿠버네티스를 사용하면 무엇을 할 수 있을까?

1. 선언적 코드를 사용한 관리 
    - json, yaml 등으로 선언한 코드를 통해 배포하는 컨테이너 주변 리소스를 관리
2. 스케일링/ 오토 스케일링
    - 부하 분산 및 다중화 구조를 만들 수 있음 → 레플리카 수를 자동으로 늘리거나 줄일 수 있음
3. 스케쥴링
    - 컨테이너를 어떤 쿠버네티스 노드에 배포할 것인지 결정
    - 어플리케이션 워크로드의 특징이나 쿠버네티스 노드의 성능 기준으로 스케줄링 가능 (ex. 디스크 I/O가 많은 컨에티너는 ssd 노드에 배치하여 제어 가능)
4. 리소스 관리
    - 컨테이너 배치를 위한 특별한 지정이 없을 경우 노드의 cpu/ 메모리의 여유 리소스 상태에 따라 스케쥴링 됨→ 어떤 노드에 컨테이너를 배치할 지 관리할 필요가 없음
5. 자동화된 복구
    - self-healing : 컨테이너 프로세스를 모니터링/ 정지를 감지 시 다시 컨테이너 스케쥴링 실행하여 자동으로 재배포
    - 프로세스 모니터링 외에 http/tcp 나 셸 스크립트로 헬스체트 성공 여부 설정 가능
6. 로드 밸런싱/서비스 디스커버리
    - service/ingress
    - 로드 밸런서 기능
        - 사전에 정의한 조건과 일치하는 컨테이너 그룹에 라우팅하는 엔드포인트 할당 가능
        - 컨테이너 확장 시 엔드포인트가 되는 서비스에 컨테이너 자동 등록/삭제, 컨테이터 장애 시 분리, 롤링 업데이트 시 필요한 사전 분리 작업도 자동으로 실행.
    - 서비스 디스커버리 [https://bcho.tistory.com/1262](https://bcho.tistory.com/1262)
        - 기능 컴포넌트에 대한 엔드포인트(ip 주소) 를 찾는 기능
            
            pod는 지정되는 Ip가 랜덤하게 지정이 되고 리스타트 때마다 변하기 때문에 고정된 엔드포인트로 호출이 어렵
            
        
        - ms 구조에서 각 마이크로 서비스끼리 참조 시 디스커버리 기능 유용
        - 각각의 서비스가 정의된 복수의 manifest 이용하여 시스템 전체 연계 가능
        
        ```docker
        apiVersion: v1
        kind: Service
        metadata:
          name: hello-node-svc
        
        spec:
          selector:
            app: hello-node #라벨 셀렉터를 통해 관리하고자 하는 pod 지정 가느
        
          ports:
            - port: 80
              protocol: TCP
              targetPort: 8080
          type: LoadBalancer
        
        # 출처: https://bcho.tistory.com/1262 [조대협의 블로그](service/ingress) 
        ```
        
7. 데이터 관리
    - 백엔드 스토어로 etcd 를 사용
    - etcd
        - 클러스터 구성하여 이중화 가능
        - 컨테이너,서비스 manifest 파일도 이중화 구조로 저장
        - 컨테이너가 사용하는 설정 파일, 인증 정보 등의 데이터 저장 가능 → 컨테이너 공통 설정/어플리케이션에서 사용하는 db 인증 정보 등을 안전하고 이중화된 상태로 집중 관리 가능
        
        ![Untitled](https://user-images.githubusercontent.com/18088806/172280194-233c3fb5-370a-440a-9454-3c15c90d7291.png)


1. 외부 에코 시스템과의 연계
    - Fluentd: 쿠버네티스에 컨테이너 로그 전송
    - Prometheus : 쿠버네티스 모니터링
    - Spark : 잡을 k8s 네이티브로 실행(yarn 대체)
        - [https://eyeballs.tistory.com/82](https://eyeballs.tistory.com/82)
        - [https://www.slideshare.net/FerranGalReniu/yarn-by-default-spark-on-yarn](https://www.slideshare.net/FerranGalReniu/yarn-by-default-spark-on-yarn)
    - kubeflow : 쿠버네티스에 ml  플랫폼 배포
    
    ...
    

### 2.4 정리

- 컨테이너의 이동성/경량화를 활용한 빠른 개발과 전체 시스템의 배포 자동화를 구현할 수 있음.
- 자체적으로 만든 배포용 스크립트 등을 주변 에코 시스템에서 공통으로 사용 가능
