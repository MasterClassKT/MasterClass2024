# Session 2.

## Eventhub Event 만들기

- 실습) Event 4종 만들기
- vivaldi-log-chg
- vivaldi-log-hit
- vivaldi-log-inf
- vivaldi-log-inffail

![image.png](Session%202%20a1333648ab244ad6ba1e73905f97261a/image.png)

## 🛫CI 파이프라인 만들기

### 1. Github workflow 탐구 (Step by Step)

- workflow 이름 정하기

```yaml
name: Deploy Azure with Gitops-Kustomize by Teacher ## 원하는 워크플로우 이름을 기재

on:      
  workflow_dispatch:
    inputs:
      name:
        description: "Docker TAG"
        required: true
        default: "main"
## on.workflow_dispatch: 에 대한 부분은 아래 그림의 빨간 칸에 있는 설정을 해주는 역할 
```

![image.png](Session%202%20a1333648ab244ad6ba1e73905f97261a/image%201.png)

- env(환경변수) 셋팅

```yaml
env:
  GIT_OPS_NAME: vivaldi-ops      ##env: 고유 명사로 쓰일만한 환경변수를 셋팅
  OPS_DIR: charts/${{ github.event.repository.name }}  
```

- Job 생성

```yaml
jobs:
  ecr-build-push-and-deploy:
    name: azr-build-push-and-deploy
    runs-on: ubuntu-latest #ubuntu 최신버전의 가상머신을 생성 
    environment: production #dev #local
```

- Job 첫 번째 step

```yaml
steps:
    - name: Check out the repo
      uses: actions/checkout@v4
```

- Job 두 번째 step

```yaml
- name: Log in to Docker Hub
      uses: azure/docker-login@v1
      with:
        login-server: ${{ secrets.AZURE_URL }}
        username: ${{ secrets.ACR_USERNAME }}
        password: ${{ secrets.ACR_PASSWORD }}
```

- Job 세 번째 step

```yaml
- name: Set Timezone
      uses: szenius/set-timezone@v2.0
      with:
        # Desired timezone for Linux
        timezoneLinux: Asia/Seoul
```

- Job 네 번째 step

```yaml
- name: set env  # TAG 를 현재 시간으로 설정
      run: echo "NOW=$(date +'%Y%m%d%H%M%S')" >> $GITHUB_ENV
```

- Job 다섯번째 step

```yaml
- name: Extract metadata (tags, labels) for Docker
      id: meta
      uses: docker/metadata-action@98669ae865ea3cffbcbaa878cf57c20bbf1c6c38
      with:
        images: ${{ github.repository }}
        tags: ${{ env.NOW }} # ${{ github.event.inputs.name }}
```

- Job 여섯번째 step (중요)

```yaml
- name: Build and Push to ACR
      uses: docker/build-push-action@v2
      with:
        context: .
        push: true
        file: ./Dockerfile  ## 나의 레파지토리의 최상위 디렉토리의 Dockerfile을 불러온다.
        platforms: linux/amd64
        tags: ${{ secrets.AZURE_URL }}/${{ steps.meta.outputs.tags }}
```

- Job 일곱번째 step

```yaml
- name: Run Trivy vulnerability scanner
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: "${{ secrets.AZURE_URL }}/${{ steps.meta.outputs.tags }}"
        format: "table"
        exit-code: "0"
        ignore-unfixed: true
        vuln-type: "os,library"
        severity: "CRITICAL,HIGH"
```

- Job 여덟번째 step

```yaml
# kustomize 명령을 가져온다.
    - name: Setup Kustomize
      uses: imranismail/setup-kustomize@v1
```

- Job 아홉번째 step

```yaml
- name: Checkout kustomize repository
      uses: actions/checkout@v2
      with:
        # kubernetes 설정정보 저장소
        repository: Azure-Vivaldi/${{ env.GIT_OPS_NAME }}
        ref: main ## 본인이 원하는 브랜치를 명명
        # 다른 저장소에 push 하려면 Personal Access Token이 필요.
        token: ${{ secrets.ACTION_TOKEN }} # ${{ secrets.GITHUB_TOKEN }} 
        path: ${{ env.GIT_OPS_NAME }}
```

- Job 열번째 step

```yaml
# 새 이미지 버전으로 파일 수정
    - name: Update Kubernetes resources
      run: |
        pwd
        cd ${{ env.GIT_OPS_NAME }}/${{ env.OPS_DIR }}
        kustomize edit set image ${{ github.repository }}=${{ secrets.AZURE_URL }}/${{ steps.meta.outputs.tags }}
        cat kustomization.yaml
```

- Job 마지막 step

```yaml
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

### 나만의 워크플로우 만들기 (실습)

- 나의 브랜치에 들어가서 “create new file” 클릭

![image.png](Session%202%20a1333648ab244ad6ba1e73905f97261a/image%202.png)

- 기존 워크플로우.yaml 붙여넣기 후 만들기

![image.png](Session%202%20a1333648ab244ad6ba1e73905f97261a/image%203.png)

### 실습) 깜짝 과제!!!

 1.  나의 브랜치를 사용하도록 수정 (필수)

1. Docekr image 태그에 {{시간}}-{{교육생이름}} 이름이 붙도록 설정 (필수)
    1. 예) masterclasskt/fe-web:20241013200951-teacher
2. Step 하단에 giglepeople@gmail.com, jungchihoon의 계정이 아닌 본인의 계정으로 CI 해보기!! (선택)

## ☁️Azure Container Registry Docker Image 확인

- Azure portal에 접속하여 이미지가 푸쉬되었는지 확인한다.

![image.png](Session%202%20a1333648ab244ad6ba1e73905f97261a/image%204.png)

- 이미지 태그명을 확인한다. (과제를 수행 하셨다면 태그명 뒤에 기재한 명칭이 추가로 붙는다.)

![image.png](Session%202%20a1333648ab244ad6ba1e73905f97261a/image%205.png)

### 🤩원하는 이름의 이미지가 푸쉬되었다면 Github Action을 통한 CI 구성은 완료

## ⛅AKS 네임스페이스 만들기

- Azure portal에서 AKS 접속 (aks-az01-dev-vvd-01)
- Kubernetes 리소스 → 실행명령 진입

![image.png](Session%202%20a1333648ab244ad6ba1e73905f97261a/image%206.png)

- 네임스페이스 생성

```bash
kubectl create namespace teacher
```

![image.png](Session%202%20a1333648ab244ad6ba1e73905f97261a/image%207.png)

- 네임스페이스 생성 확인

![image.png](Session%202%20a1333648ab244ad6ba1e73905f97261a/image%208.png)

```bash
kubectl get ns 
```

![image.png](Session%202%20a1333648ab244ad6ba1e73905f97261a/image%209.png)