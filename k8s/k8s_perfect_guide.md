
- - -
```Index```
 
* [CH1. 도커 복습과 hello,k8s](#CH1)

- - - 

   

## CH1
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
