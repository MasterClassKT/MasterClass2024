# Session 1.

## Github 설치

블로그 참고: https://codingapple.com/unit/git-install-windows-mac/

## Docker Desktop 설치

1. Mac사용자 참고: https://blog.naver.com/ys00214/223178384364
2. Window사용자 참고: https://csm-kr.tistory.com/102
    1. Window사용자의 경우 WSL2설치가 필요합니다 - https://csm-kr.tistory.com/101

### 이번 실습에서는 Docker를 활용하지 않을 계획이니 참고만 하시길 바랍니다.

## Github Personal Access Token 발급

1. 우측 상단 계정 아이콘 클릭 → Setting

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image.png)

1. Developer Settings 진입

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%201.png)

1. Personal Access Token(PAT) → Tokens classic 클릭

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%202.png)

1. Generate new token(classic) 클릭

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%203.png)

1. 원하는 Token이름 기재 & 체크박스 모두 체크 → 하단 “Generate token” 클릭

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%204.png)

1. Token 메모장 저장

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%205.png)

## Github Repository Clone

1. MasterClassKT의 Repository Clone 하기

### 모든 프론트/백앤드 레파지토리는 Private repo로 설정 되어있다.

```bash
cd ~/
mkdir masterclasswork
cd masterclasswork
git clone https://{{Personal Access Token}}@github.com/MasterClassKT/be-common.git
```

## Github branch 나누기

1. Organization에 조직원 추가하기

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%206.png)

### 고수의 클래스 Slack 채널에 github계정(이메일주소) 올려주세요!!

1. git 브랜치 생성

```bash
git branch teacher
```

1. git 브랜치 변경

```bash
git checkout teacher
```

1. git 커밋 및 푸시

```bash
git add .
git commit -m "new branch"
git push origin teacher
```

1. git branch 상태 확인

```bash
jungchihoon@jungchihoonui-MacBookPro fe-web % git branch
  main
* teacher
```

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%207.png)

1. git CLI가 안돼시는 분들은 github사이트에서 UI로 가능합니다.

## Github Action Secret 변수 저장

1. 소스 Repository → settings

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%208.png)

1. Secrets and variables → Actions 클릭

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%209.png)

1. New repository secret 클릭 

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%2010.png)

1. Secret 등록 

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%2011.png)

1. AZURE_URL, ACR_USERNAME, ACR_PASSWORD

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%2012.png)

위 그림과 같이 각각의 Secret 변수 값은 Azure의 ACR 설정 → “엑세스키”탭에 진입하면 확인이 가능하다

1. ACTION_TOKEN 도 추가

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%2013.png)

위에서 생성한 Github Personal Access Token도 Secret 변수에 등록해준다.

### 7. 실습) ACTION_TOKEN_{{이름} Secret 변수 만들기

- be-chgbill
- be-pay
- be-common
- fe-chat
- fe-web

## Github Action 이란?

GitHub Actions는 GitHub에서 제공하는 CI/CD(Continuous Integration/Continuous Deployment) 도구로, 소프트웨어 개발의 워크플로우를 자동화할 수 있는 기능이다. 이를 통해 빌드, 테스트, 배포와 같은 반복적인 작업을 코드로 정의하고 실행할 수 있다. GitHub Actions를 사용하면 리포지토리 내에 있는 코드를 커밋하거나 풀 리퀘스트를 생성할 때 자동으로 특정 작업을 수행하도록 설정할 수 있다.

### GitHub Actions의 주요 개념

1. **Workflow (워크플로우)**:
    - 워크플로우는 특정 이벤트에 따라 자동으로 실행되는 작업의 집합이다. 워크플로우는 리포지토리의 `.github/workflows` 디렉토리에 `YAML` 파일 형식으로 정의된다.
    - 예: `push`, `pull request`, `issue` 생성 등과 같은 이벤트가 발생할 때 실행된다.
2. **Job (작업)**:
    - 워크플로우 내에서 여러 작업(Job)을 정의할 수 있으며, 각 작업은 하나 이상의 단계로 이루어진다.
    - 예를 들어, `build`, `test`, `deploy` 작업을 각기 다른 Job으로 나눌 수 있다.
3. **Step (단계)**:
    - 각 Job은 여러 단계(Step)로 구성되며, 각 단계는 실행할 명령어나 스크립트를 나타낸다.
    - Step에서는 명령어를 직접 실행하거나, Action이라는 재사용 가능한 스크립트를 사용할 수 있다.
4. **Action**:
    - Action은 특정 기능을 수행하는 재사용 가능한 코드이다. 예를 들어, 코드 스타일 검사를 하거나, 도커 이미지를 빌드하는 등의 작업을 수행하는 Action을 사용할 수 있다.
    - GitHub Actions 마켓플레이스에서 다양한 공개된 Action을 가져와 사용할 수 있으며, 직접 Action을 만들어 사용할 수도 있다.

## Github Workflow(워크플로우)

1. 폴더 경로

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%2014.png)

1. 워크플로우 파일 위치

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%2015.png)

1. 예) workflow.yaml

```yaml
name: Deploy Azure with Gitops-Kustomize

on:      
  workflow_dispatch:
    inputs:
      name:
        description: "Docker TAG"
        required: true
        default: "master"

env:
  GIT_OPS_NAME: vivaldi-ops
  OPS_DIR: charts/${{ github.event.repository.name }}
  
jobs:
  ecr-build-push-and-deploy:
    name: azr-build-push-and-deploy
    runs-on: ubuntu-latest
    #runs-on: self-hosted
    environment: production

    steps:
    - name: Check out the repo
      uses: actions/checkout@v4
      
    - name: Log in to Docker Hub
      uses: azure/docker-login@v1
      with:
        login-server: ${{ secrets.AZURE_URL }}
        username: ${{ secrets.ACR_USERNAME }}
        password: ${{ secrets.ACR_PASSWORD }}
      
    - name: Set Timezone
      uses: zcong1993/setup-timezone@master
      with:
        # Desired timezone for Linux
        timezoneLinux: Asia/Seoul
        
    - name: set env  # TAG 를 현재 시간으로 설정
      run: echo "NOW=$(date +'%Y%m%d%H%M%S')" >> $GITHUB_ENV
      
    - name: Extract metadata (tags, labels) for Docker
      id: meta
      uses: docker/metadata-action@98669ae865ea3cffbcbaa878cf57c20bbf1c6c38
      with:
        images: ${{ github.repository }}
        tags: ${{ env.NOW }} # ${{ github.event.inputs.name }}
  
    - name: Build and Push to ACR
      uses: docker/build-push-action@v2
      with:
        context: .
        push: true
        file: ./Dockerfile
        platforms: linux/amd64
        build-args: |
          WHATAP_HOST: ${{ secrets.WHATAP_HOST }}
        tags: ${{ secrets.AZURE_URL }}/${{ steps.meta.outputs.tags }}
        
    # - name: Run Trivy vulnerability scanner
    #   uses: aquasecurity/trivy-action@master
    #   with:
    #     image-ref: "${{ secrets.AZURE_URL }}/${{ steps.meta.outputs.tags }}"
    #     format: "table"
    #     exit-code: "0"
    #     ignore-unfixed: true
    #     vuln-type: "os,library"
    #     severity: "CRITICAL,HIGH"
          
    # kustomize 명령을 가져온다.
    - name: Setup Kustomize
      uses: imranismail/setup-kustomize@v1

    - name: Checkout kustomize repository
      uses: actions/checkout@v2
      with:
        # kubernetes 설정정보 저장소
        repository: ${{ github.repository_owner }}/${{ env.GIT_OPS_NAME }}
        ref: main
        # 다른 저장소에 push 하려면 Personal Access Token이 필요.
        token: ${{ secrets.ACTION_TOKEN }} # ${{ secrets.GITHUB_TOKEN }} 
        path: ${{ env.GIT_OPS_NAME }}
        
    # 새 이미지 버전으로 파일 수정
    - name: Update Kubernetes resources
      run: |
        pwd
        cd ${{ env.GIT_OPS_NAME }}/${{ env.OPS_DIR }}
        kustomize edit set image ${{ secrets.AZURE_URL }}/${{ steps.meta.outputs.tags }}
        cat kustomization.yaml
        
# kustomize edit set image ${{ github.repository }}=${{ secrets.AZURE_URL }}/${{ steps.meta.outputs.tags }}
   
    # 수정된 파일 commit & push
    - name: Commit manifest files
      run: |
        cd ${{ env.GIT_OPS_NAME }}/${{ env.OPS_DIR }}
        git checkout HEAD
        git config --global user.email "giglepeople@gmail.com"
        git config --global user.name "jungchihoon"
        git commit -am 'update image tag  ${{ env.NOW }} from Github Action'
        cat kustomization.yaml
        git push origin HEAD
```

소스 수정은 Session 3에 다시 합니다..

## 🌲Azure 리소스 구성 & 생성

### 1. Azure Kubernetes Service(AKS)

AKS는 kubernetes Vanila 버전을 사용함으로 기존 Redhat Openshift 같은 PaaS 에서 지원하는 기능은 많지 않음

- Kubernetes API는 랜딩존에서는 Private 으로 설정이 되고 Private Subnet 영역에 위에서 동작하며 Application Gateway for Containers 를 통해 Ingress를 구성하여 외부에서 도메인으로 접근 가능
- Worker Node 구성 : Azure에서는 Worker Node Group을 Node Pool 로 지칭
    - Node Pool : 시스템과 사용자의 차이는 시스템은 kubernetes 동작을 위해 필요한 control plane 이 위치
    - kube-system namespace가 시스템 Node Pool 에서 동작

```yaml
[ktadmin@vm-az01-dev-vvd-jumpbox-01 ~]$ kubectl get nodes -o wide
NAME                                STATUS   ROLES    AGE     VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
aks-nodepool1-31400841-vmss000002   Ready    <none>   6d1h    v1.29.4   10.241.163.134   <none>        Ubuntu 22.04.4 LTS   5.15.0-1067-azure   containerd://1.7.15-1
aks-nodevvd-49840854-vmss000002     Ready    <none>   6d19h   v1.29.4   10.241.163.137   <none>        Ubuntu 22.04.4 LTS   5.15.0-1067-azure   containerd://1.7.15-1
aks-nodevvd-49840854-vmss000003     Ready    <none>   6d19h   v1.29.4   10.241.163.136   <none>        Ubuntu 22.04.4 LTS   5.15.0-1067-azure   containerd://1.7.15-1
[ktadmin@vm-az01-dev-vvd-jumpbox-01 ~]$ kubectl get po -o wide --all-namespaces | grep aks-nodepool1
kube-system                     azure-cns-tfs6f                                  1/1     Running            0                6d1h    10.241.163.134   aks-nodepool1-31400841-vm
ss000002   <none>           <none>
kube-system                     azure-ip-masq-agent-sd7ff                        1/1     Running            0                6d1h    10.241.163.134   aks-nodepool1-31400841-vm
ss000002   <none>           <none>
kube-system                     cloud-node-manager-bsl88                         1/1     Running            0                6d1h    10.241.163.134   aks-nodepool1-31400841-vm
ss000002   <none>           <none>
kube-system                     csi-azuredisk-node-xkfwl                         3/3     Running            0                6d1h    10.241.163.134   aks-nodepool1-31400841-vm
ss000002   <none>           <none>
kube-system                     csi-azurefile-node-7zp99                         3/3     Running            0                6d1h    10.241.163.134   aks-nodepool1-31400841-vm
ss000002   <none>           <none>
kube-system                     extension-agent-c7f4d9567-2t4sz                  2/2     Running            0                5d20h   100.64.2.53      aks-nodepool1-31400841-vm
ss000002   <none>           <none>
kube-system                     extension-operator-79c58b9bc5-v92kl              2/2     Running            0                5d20h   100.64.2.241     aks-nodepool1-31400841-vm
ss000002   <none>           <none>
kube-system                     kube-proxy-88w86                                 1/1     Running            0                6d1h    10.241.163.134   aks-nodepool1-31400841-vm
ss000002   <none>           <none>
```

- 비발디의 경우 nodepool1 이 시스템 노드이고 1개의 VM이 할당, nodevvd 는 사용자 노드이고 최대 2개까지 할당 가능

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%2016.png)

- nodepool에 들어가면 subnet 구성을 볼수 있고 뒤에 app로 끝나는 서브넷을 확인 할 수 있다.

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%2017.png)

- node를 클릭하면 Node의 사이즈를 볼 수 있다.
- 기본적으로 시스템 노드와 사용자 노드 사이즈는 다르고 향후에 업무의 성격에 따라서 Node Pool 을 나누어 설정을 하면 같은 사이즈로 Provisioning이 된다.

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%2018.png)

- 사용자 노드 현황

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%2019.png)

- Subnet 을 클릭하면 네트웍 대역을 확인 할 수 있다. **해당 네트웍 대역으로 Legacy 방화벽도 오픈 해야 하고 Azure SaaS 상품과도 Private Network 으로 연결된다**.

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%2020.png)

### 고려 및 주의 사항

OKD ( FlyingCube )  는  Namespace 에서 NodeSelector를 설정을 하지만 kubernetes에서는 POD 레벨에서 NodeSelector를 설정해야만 한다.

별도의 node selector를 설정 하지 않으면 아래와 같이  POD가  시스템과 유저 모드의  워커 노드에 분산이 되고

시스템  모드  Node의 리소스가 부족할수 있어 장애가 발생 할 수 있음 . 따라서 일반 업무용 POD와 System 용 POD는 반드시 분리 필요

• 유저 모드 NODE의 Label 을 확인한다.  

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%2021.png)

• 비발디의 경우 deployment.yaml에 아래와 같이 NodeSelector 설정함.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: be-bill
spec:
  replicas: 1
  selector:
    matchLabels:
      app: be-bill
  template:
    metadata:
      labels:
        app: be-bill
    spec:
      containers:
      - name: be-bill
        image: aksaz01devvvd01registry.azurecr.io/bg012401-vivaldi/be-bill ## 올바른 이미지 이름으로 변경
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
          name: http
      nodeSelector:
        agentpool: 'nodevvd'  ## Azure 노드풀 USER 타입의 이름으로 변경
```

## 2. Azure Container Registry

- ACR은 AKS 에서 접근 하기 위해서 AcrPull 역할이 할당 되어 있어야 한다.

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%2022.png)

- ACR은 Public Endpoint 로 설정이 되어 Github Action에서 도커 이미지 Push시에도 방화벽 오픈이 별도로 필요 없다.
- Private Network 는 AKS 대역으로 구성이 되어 있다.

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%2023.png)

## 3. EventHub 구성

- EventHub는 Kafka 호환 모드로 작동하기 위하여 Standard 이상의 요금제가 필요 하다.

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%2024.png)

- JSON 보기를 클릭하여 End Point 를 확인 할 수 있다.

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%2025.png)

- AccessKey는 기본 생성된 RootManageSharedAccessKey를 사용하여 연동 한다.

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%2026.png)

- AKS 와 연동하기 위해서는 프라이빗 엔드포인트 연결이 필요하고 AKS의 네트웍 대역과 연결이 되어야 함.

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%2027.png)

- 프라이빗 엔드포인트 가 AKS 네트웍크 대역인 sbn-az01-dev-vvd-app 로 연결 되어 있는 것 확인.

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%2028.png)

## 4. Azure Cache for redis

- 메모리는 표준 타임 250M로 설정이 되고 SSL 적용 하지 않고 6379 포트로 연동. Private Endpoint설정 하기 때문에 SSL 반드시 필요하지는 않음

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%2029.png)

- 패스 워드 설정

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%2030.png)

AKS와 연동하기 위하여 Private Endpoint 를 구성한다.

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%2031.png)

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%2032.png)

### 5. DNS 구성

- Azure 랜딩존은 기본적으로 Public 환경 이기 때문에 Public DNS 로  설정됨.
- kt 내부 시스템을 도메인으로 연동하기 위해서는 kt dns 가 적용이 되어야 하고 기존에는 PHZ 에 Private DNS Zone 을 구성하여 도메인을 하나씩 등록 하였으나 8/8일 부터

 KT DNS <-> Azure DNS Forwarder 구성이 완료 되어 VNET 에 연결만 하면 됨.

- 비발디의 경우 vnet-az01-dev-vvd-01 에  Azure DNS Forwarder 설정 됨.

### 6. Azure 리소스 생성 - Eventhub 생성 실습

- **포털 중앙 상단에 검색: Eventhub**

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%2033.png)

**리소스를 설정하고 namespace를 설정한다.**

- **Eventhub의 namespace 는 kafka의 cluster를 의미한다**
- **Price plan은 "Standard"를 선택한다. Standard 이상에서만 kafka 라이브러리와 호환이 지원 된다.**

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%2034.png)

- TLS 버전은 Java 버전에 따라 지정한다. (권장은 버전 1.2)
- KT의 백앤드 서비스는 노후화된 java 버전이 많아 TLS 버전을 꼭 체크해야한다.

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%2035.png)

- 네트워킹은 정해진 가이드에 따라 설정 한다.(실습은 공용 엑세스로)

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%2036.png)

- 태그는 정해진 가이드에 따라 설정 한다. (이번 실습에서는 패스)

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%2037.png)

- 유효성 검사 + 만들기 진행

![image.png](Session%201%200a526e8c94dc478a8b48b5143af9552c/image%2038.png)

### 만들기 후 약 5분의 소요시간이 걸린다…Session 2에서 계속