# Session 4.

## AKS 컨테이너 배포

### Yaml 종류

- Deployment.yaml
- Serivce.yaml
- configmap.yaml
- secret.yaml
- ingress.yaml

### 실습) Deployment.yaml 수정

MasterClassKT에서 vivaldi-ops repository에 vivaldi-ops/charts/.. 하위에 서비스에 해당하는 디렉토리들이 있다.

예) vivaldi-ops/chats/be-chgbill 폴더를 확인해보면

![image.png](Session%204%20c71c26857b01417c99a951cd4e0a6afd/image.png)

![image.png](Session%204%20c71c26857b01417c99a951cd4e0a6afd/image%201.png)

6개의 파일이 있다. 이 중 deployment.yaml 을 수정할 계획입니다.

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: be-chgbill
spec:
  replicas: 1
  selector:
    matchLabels:
      app: be-chgbill
  template:
    metadata:
      labels:
        app: be-chgbill
    spec:
      containers:
      - name: be-chgbill
      ##수정 여기요~###
        image: aksaz01devvvd01registry.azurecr.io/masterclasskt/be-chgbill:20241013200951-teacher ## 각자 교육생의 이미지 이름으로 변경
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
          name: http
        envFrom:
        - configMapRef:
            name: be-chgbill-env-configmap
        - secretRef:
            name: be-chgbill-secret
        # - secretRef:
        #     name: vivaldi-redis-cluster
        env:
        - name: NODE_IP
          valueFrom:
            fieldRef:
              fieldPath: status.hostIP
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
      ### Whatap JVM 추가 ####
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: OKIND  
          value: be-chgbill ## 각 서비스의 deployment 명으로 기재 
        - name: whatap
          value: 10.241.10.36/10.241.10.37
        # resources: 
        #   requests:
        #     cpu: 200m
        #     memory: 1250Mi
        #   limits:
        #     cpu: 500m
        #     memory: 1250Mi
        livenessProbe:
          httpGet:
            path: /api/be/chgbill/actuator/health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 1
          successThreshold: 1
          failureThreshold: 15 
        readinessProbe:
          httpGet:
            path: /api/be/chgbill/actuator/health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 15
          timeoutSeconds: 1
          successThreshold: 1
          failureThreshold: 15
        securityContext:
          privileged: true
        volumeMounts:
        - mountPath: /var/log/WASLOG/
          name: waslog-volume 
        - mountPath: /var/log/LAMP/
          name: lamp-volume
      dnsPolicy: ClusterFirst
      hostAliases:
      - hostnames:
        - inisafenet
        ip: 147.6.37.57 ## 여기 주소를 바꿔야함
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30 
      volumes:  
      - hostPath:
          path: /usr/share/zoneinfo/Asia/Seoul
        name: timezone-config
      - hostPath:
          path: /logs/vivaldi/be-chgbill/WASLOG/
          type: DirectoryOrCreate
        name: waslog-volume
      - hostPath:
          path: /logs/vivaldi/LAMP/
          type: DirectoryOrCreate
        name: lamp-volume
      # nodeSelector: 
      #   agentpool: 'nodevvd'  ## Azure 노드풀 USER 타입의 이름으로 변경
```

### 실습) 배포 해보기

![image.png](Session%204%20c71c26857b01417c99a951cd4e0a6afd/image%202.png)

- 배포 configmap.yaml, secret.yaml, service.yaml, deployment.yaml (ingress.yaml, kustomization.yaml 제외)

![image.png](Session%204%20c71c26857b01417c99a951cd4e0a6afd/image%203.png)

- yaml 파일 하나씩 첨부하여 진행해야함!!

![image.png](Session%204%20c71c26857b01417c99a951cd4e0a6afd/image%204.png)

- 명령어 모음

```bash
kubectl apply -f deployment.yaml -n {{교육생 네임스페이스}}
```

```bash
kubecl apply -f configmap.yaml -n {{교육생 네임스페이스}}
```

```bash
kubectl apply -f secret.yaml -n {{교육생 네임스페이스}}
```

```bash
kubectl apply -f service.yaml -n {{교육생 네임스페이스}}
```

### 실습) 서비스 be-chgbill, be-pay, be-common, fe-chat, fe-web 모두 올려보기

결과모습

![image.png](Session%204%20c71c26857b01417c99a951cd4e0a6afd/image%205.png)

### 실습) 본인이 만든 Eventhub, PostgreSQL, Redis에 연동이 잘 되었는지 확인

```bash
kubectl logs {{pod이름}} -n {{교육생 네임스페이스}}
```