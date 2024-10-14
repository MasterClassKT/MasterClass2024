# Session 3.

## Github 소스 수정

기존 IPC 환경에서 연동하던 Kafka, Redis, PPAS(postgreSQL DB) 등 모두 Azure PaaS상품으로 대체하여 사용하기 때문에 소스의 Config가 수정 되어야 한다. 

### 1. PPAS → Azure PostgreSQL flexible server

- be-chgbill/src/main/resources/application.yml 파일 열어보기

```yaml
spring:
  profiles:
    active: local #기본환경

management:
  endpoints:
    web:
      exposure:
        include: health, prometheus       
# 로컬 환경 
---
##생략
. 
.
.
# 개발 환경
---
server:
  servlet.context-path: /api/be/chgbill
spring:
  profiles: dev
  application:
    name: be-chgbill
  ######우리가 바꿔야 하는 부분 시작 ########
  datasource:
    #url: jdbc:postgresql://pgdb-az01-dev-vvd-01.postgres.database.azure.com:5432/dtowndev
    url: jdbc:postgresql://pgdb-az01-dev-vvd-02.postgres.database.azure.com:5432/dtowndev
    username: ktadmin
    password: dtnew1234!
    #url: jdbc:postgresql://10.217.34.150:5444/dtowndev
    #url: jdbc:postgresql://10.217.47.143:5444/dtowndev
    #url: jdbc:postgresql://kmosdb.postgres.database.azure.com:5432
    #username: ktsapp
    #password: ${DB_PWD}
    hikari:
      minimum-idle: 5
      maximum-pool-size: 10 
      idle-timeout: 400000
      max-lifetime: 600000
      connection-timeout: 30000
      validation-timeout: 5000
  redis:
    lettuce:
      pool:
        max-active: 10
        max-idle: 10
        min-idle: 2
    host: vvdredis.redis.cache.windows.net
    port: 6379
    ssl:
      enabled: false
    password: 2mhV9eMF0OTRXNnauz7NPRs2ep41chGoEAzCaK6CJXw= #KcyekqMSxJMto2skDn86VW7oFmCdpzyA8AzCaG490Rk= # 패스워드는 Deployment에 redis Sercet을 추가해서 받아옴.(values에 추가)       
    timeout:
      seconds: 10
  main.allow-bean-definition-overriding: true
  #### 실습 ####
  ###kafka의 부트스트랩 서버의 주소를 session1에서 만든 주소로 대체해보기
    kafka:
    bootstrap-servers: vvdeventhub.servicebus.windows.net:9093 
    # vivaldi-kafka.vivaldi.svc:9092
    properties:
      sasl.jaas.config: org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="Endpoint=sb://vvdeventhub.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=DBY1Wadj+5Qu5jdqKBP5U4sQlVhBrvKnL+AEhDcuJ2g=";
      sasl.mechanism: PLAIN
      security.protocol: SASL_SSL
    consumer:
      group-id: $Default
    #password: KAFKA_PWD  
    session:
      timeout: 10000
    topic:
      hit: vivaldi-log-hit
      chg: vivaldi-log-chg
      inf: vivaldi-log-inf
      inf-fail: vivaldi-log-inffail  
mybatis:
  type-aliases-package: com.kt.vivaldi.be.chbill.mybatis.bean 
  configuration:
    map-underscore-to-camel-case: true
initech:
  prop.path: /initech/inisafenet/dev/properties
  enc.type: 0011 
wsdl:
  om:
    url.http: http://10.217.155.85:5000/SoapGateway
    corp.url.http: http://10.217.155.85:5000/SoapGateway
  omso:
    url.http: http://10.217.155.85:5000/SoapDynamicGateway
  cdm:
    url.http: http://10.217.155.85:5000/SoapGateway
    corp.url.http: http://10.217.155.172:5000/SoapGateway
  cdmso:
    url.http: http://10.217.155.85:5000/SoapDynamicGateway
    corp.url.http: http://10.217.155.172:5000/SoapDynamicGateway
  onbill:
    url.http: http://10.217.155.85:5000/SoapGateway
    corp.url.http: http://10.217.155.172:5000/SoapGateway
  onbillso:
   url.http: http://10.217.155.85:5000/SoapDynamicGateway
  crm:
    url.http: http://10.217.155.190:5000/SoapGateway
    promotion.domain: http://10.217.41.55:80
    promotion.emailcheck.address: /BIZIWS2/services/EmailCheckService.EmailCheckServiceHttpSoap11Endpoint
  crmso:
    url.http: http://10.217.155.190:5000/SoapDynamicGateway 
  rdsso:
    url.http: http://10.217.157.90:5000/SoapDynamicGateway
cris:
  url.http: https://10.217.213.158:8183
  sys.cd: "104"
  sys.ip: ${NODE_IP}
  end.sys.cd: VI
  sys.id: NCRIS_VIVALDI #암호화 필요
  sys.key: ${CRIS_SYS_KEY} #암호화 필요

locktimestamp.sec: 20
docker.node.ip: ${NODE_IP}
cntpnt.div.nm: DP

api.client:
  connect.timeout: 30000
  read.timeout: 30000
  common:
    name: be-common
    ##아래 URL도 바꿔야하는데 be-common은 우리가 Pod로 띄울 서비스 이기 때문에 띄워논 서비스의DNS 주소를 사용해야한다.
    ## http://서비스이름.네임스페이스이름.svc.cluster.local
    url: http://be-common.vivaldi.svc.cluster.local
    path: /api/be/common
application:
  lamp:
    logging: true
    local:
      logging: false
    except:
      signatures: index,error
#Spring Boot Admin 관련 설정 추가
management:
  endpoints:
    web:    
      exposure:
        include: "*"
        exclude: "session"
  endpoint:
    health:
      show-details: always  
logging:
  file: /var/log/WASLOG/spring-${HOSTNAME}.log
  path: /var/log

```

### 2. 추가적으로 변경해줘야 하는 소스 확인해보기

- be-chgbill/src/main/java/com/kt/vivaldi/be/core/conf/SenderConfig.java

```java
/*
* VIVALDI Platform version 1.0
*
*  Copyright ⓒ 2019 kt corp. All rights reserved.
*
*  This is a proprietary software of kt corp, and you may not use this file except in
*  compliance with license agreement with kt corp. Any redistribution or use of this
*  software, with or without modification shall be strictly prohibited without prior written
*  approval of kt corp, and the copyright notice above does not evidence any actual or
*  intended publication of such software.
*/
package com.kt.vivaldi.be.core.conf;

import java.util.HashMap;
import java.util.Map;

import org.apache.kafka.clients.CommonClientConfigs;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.common.config.SaslConfigs;
import org.apache.kafka.common.serialization.StringSerializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.core.DefaultKafkaProducerFactory;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.core.ProducerFactory;

@Configuration
public class SenderConfig {

    @Value("${spring.kafka.bootstrap-servers}")
    private String bootstrapServers;
    
////아래 Bean으로 호출해야하는 kafka config를 변경해줘야 함
    @Value("${spring.kafka.properties.sasl.jaas.config}")
    private String saslJaasConfig;

    @Bean
    public Map<String, Object> producerConfigs() {
//     // BootstrapServer 주소는 고정값이며 Arsenal Event Stream 가 설치되어 있는  Cluster 에서만 동작합니다.
//        String bootstrapServer = "arsenal-cluster-kafka-bootstrap.kafka.svc:9092";
//        // username과 password 는 namespace 당 하나씩 부여되며 Arsenal portal 에서 확인이 가능합니다.
//        String username = "vivaldi-user";
//
//        // Arsenal Event Stream 은 SASL platform의 SCRAM-SHA-512 인증방식을 이용합니다.
//        Map<String, Object> props = new HashMap<>();
//        props.put("bootstrap.servers", bootstrapServer);
//        props.put("security.protocol", "SASL_PLAINTEXT");
//        props.put("sasl.mechanism", "SCRAM-SHA-512");
//
//        String jaasTemplate = "org.apache.kafka.common.security.scram.ScramLoginModule required username=\"%s\" password=\"%s\";";
//        
//        String jaasCfg = String.format(jaasTemplate, username, password);
//        props.put("sasl.jaas.config", jaasCfg);
//
//        /* [ 3.Producer ] */
//        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
//        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        
    	Map<String, Object> props = new HashMap<>();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        props.put(ProducerConfig.RETRIES_CONFIG, 0); // 전송중 오류시 재시도 횟수
        props.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384); // 한번에 전송하는 byte(16384-16KB 기본값)
        props.put(ProducerConfig.LINGER_MS_CONFIG, 1); // 배치그룹전송시 대기시간 (밀리초) 대기시간이 길어지면 해당 대기시간동안 배치를 채움
        props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 33554432); // 전송시 메모리 버퍼사이즈(32메가 기본값)
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
    	
        //SASL로 통신하기 때문에, Event Hub SASL 연결 설정 추가
        props.put(CommonClientConfigs.SECURITY_PROTOCOL_CONFIG, "SASL_SSL");
        props.put(SaslConfigs.SASL_MECHANISM, "PLAIN");
        //추가----//
        props.put(SaslConfigs.SASL_JAAS_CONFIG, saslJaasConfig);
        //-------//
        return props;
    }

    @Bean
    public ProducerFactory<String, ?> producerFactory() {
        return new DefaultKafkaProducerFactory<>(producerConfigs());
    }

    @Bean
    public KafkaTemplate<String, ?> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }
}

```

## 모니터링 구성

### 1. 모니터링 JVM 추가 - WhaTap

- WhaTap은 보안 관계상 깃허브에 내용을 올리지 않고, 사내 컨플루언스를 확인해 보자.
- be-chgbill에 WhaTap 에이전트와 연동 세팅이 되어있기 때문에 추후 Pod를 배포하고 Agent가 검색되는지 확인

### 2. Azure insight 구성 - Container Monitoring

Azure에는 모니터링 기능을 제공한다. Azure insight라 칭하고, 이번 실습은 Application insight를 세팅하여 컨테이너의 메트릭, 로그 모니터링을 구성 할 예정이다.

- Azure insight의 vvd 구독에서는 이미 구성이 완료되어 어떻게 구성되었는지 아래 그림 첨부를 확인한다.

![image.png](Session%203%20ae1184d30d924f6faeb8e2fd02232eb8/image.png)

- 모니터링 구성 클릭

![image.png](Session%203%20ae1184d30d924f6faeb8e2fd02232eb8/image%201.png)

- 컨테이너 로그 사용

![image.png](Session%203%20ae1184d30d924f6faeb8e2fd02232eb8/image%202.png)

![image.png](Session%203%20ae1184d30d924f6faeb8e2fd02232eb8/image%203.png)

![image.png](Session%203%20ae1184d30d924f6faeb8e2fd02232eb8/image%204.png)

- 설치 결과

![image.png](Session%203%20ae1184d30d924f6faeb8e2fd02232eb8/image%205.png)

- Prometheus & Grafana 설정

![image.png](Session%203%20ae1184d30d924f6faeb8e2fd02232eb8/image%206.png)

- 로그 수집 및 모니터링을 위한 설정 확인

![image.png](Session%203%20ae1184d30d924f6faeb8e2fd02232eb8/image%207.png)

![image.png](Session%203%20ae1184d30d924f6faeb8e2fd02232eb8/image%208.png)

- Application insight 설정

![image.png](Session%203%20ae1184d30d924f6faeb8e2fd02232eb8/image%209.png)

![image.png](Session%203%20ae1184d30d924f6faeb8e2fd02232eb8/image%2010.png)

- Azure monitor에서 사용하는 log analystics table 리스트

![image.png](Session%203%20ae1184d30d924f6faeb8e2fd02232eb8/image%2011.png)

![image.png](Session%203%20ae1184d30d924f6faeb8e2fd02232eb8/image%2012.png)

- 모니터 → 진단 설정

![image.png](Session%203%20ae1184d30d924f6faeb8e2fd02232eb8/image%2013.png)

![image.png](Session%203%20ae1184d30d924f6faeb8e2fd02232eb8/image%2014.png)

- kubernetes 로그 수집에 대한 범주를 지정 할 수 있다.

![image.png](Session%203%20ae1184d30d924f6faeb8e2fd02232eb8/image%2015.png)

- 비용 조회도 가능하다(비용이 상당히 많이 나와 로그 수집에 대한 커스터마이징이 필요하다.)

![image.png](Session%203%20ae1184d30d924f6faeb8e2fd02232eb8/image%2016.png)

### 로그 수집 Configmap Customizing

- stdout 로그는 수집 하지 않도록 설정하고, stderr 로그는 kube-system, argocd 네임스페이스를 제외하고 수집하도록 설정 해본다.

```yaml
## container-azm-ms-agentconfig.yaml

kind: ConfigMap
apiVersion: v1
data:
  schema-version:
    #string.used by agent to parse config. supported versions are {v1}. Configs with other schema versions will be rejected by the agent.
    v1
  config-version:
    #string.used by customer to keep track of this config file's version in their source control/repository (max allowed 10 chars, other chars will be truncated)
    ver1
  log-data-collection-settings: |-
    # Log data collection settings
    # Any errors related to config map settings can be found in the KubeMonAgentEvents table in the Log Analytics workspace that the cluster is sending data to.

    [log_collection_settings]
      #  This feature in private preview and not ready for production use,please reach out us to try out this feature.
      #  [log_collection_settings.multi_tenancy]
      #    # High log scale MUST be enabled to use this feature. Refer to https://aka.ms/cihsmode for more details on high log scale mode
      #    enabled = true
      ###   Below advanced settings MUST be provided when ama-logs-multitenancy service deployed through AKS Monitoring Addon
      ##    advanced_mode_enabled = false # Default value is false
      ##    namespaces = ["app-team-1","app-team-2"]
      ##    namespace_settings = ["app-team-1:throttle_rate=2000","app-team-2:throttle_rate=5000"]

       [log_collection_settings.stdout]
          # In the absense of this configmap, default value for enabled is true
          enabled = true
          # exclude_namespaces setting holds good only if enabled is set to true
          # kube-system,gatekeeper-system log collection are disabled by default in the absence of 'log_collection_settings.stdout' setting. If you want to enable kube-system,gatekeeper-system, remove them from the following setting.
          # If you want to continue to disable kube-system,gatekeeper-system log collection keep the namespaces in the following setting and add any other namespace you want to disable log collection to the array.
          # In the absense of this configmap, default value for exclude_namespaces = ["kube-system","gatekeeper-system"]
          exclude_namespaces = ["kube-system","gatekeeper-system"]
          # If you want to collect logs from only selective pods inside system namespaces add them to the following setting. Provide namepace:controllerName of the system pod. NOTE: this setting is only for pods in system namespaces
          # Valid values for system namespaces are: kube-system, azure-arc, gatekeeper-system, kube-public, kube-node-lease, calico-system. The system namespace used should not be present in exclude_namespaces
          # collect_system_pod_logs = ["kube-system:coredns"]

       [log_collection_settings.stderr]
          # Default value for enabled is true
          enabled = true
          # exclude_namespaces setting holds good only if enabled is set to true
          # kube-system,gatekeeper-system log collection are disabled by default in the absence of 'log_collection_settings.stderr' setting. If you want to enable kube-system,gatekeeper-system, remove them from the following setting.
          # If you want to continue to disable kube-system,gatekeeper-system log collection keep the namespaces in the following setting and add any other namespace you want to disable log collection to the array.
          # In the absense of this configmap, default value for exclude_namespaces = ["kube-system","gatekeeper-system"]
          exclude_namespaces = ["kube-system","argocd"]
          # If you want to collect logs from only selective pods inside system namespaces add them to the following setting. Provide namepace:controllerName of the system pod. NOTE: this setting is only for pods in system namespaces
          # Valid values for system namespaces are: kube-system, azure-arc, gatekeeper-system, kube-public, kube-node-lease, calico-system. The system namespace used should not be present in exclude_namespaces
          # collect_system_pod_logs = ["kube-system:coredns"]

       [log_collection_settings.env_var]
          # In the absense of this configmap, default value for enabled is true
          enabled = true
       [log_collection_settings.enrich_container_logs]
          # In the absense of this configmap, default value for enrich_container_logs is false
          enabled = false
          # When this is enabled (enabled = true), every container log entry (both stdout & stderr) will be enriched with container Name & container Image
       [log_collection_settings.collect_all_kube_events]
          # In the absense of this configmap, default value for collect_all_kube_events is false
          # When the setting is set to false, only the kube events with !normal event type will be collected
          enabled = false
          # When this is enabled (enabled = true), all kube events including normal events will be collected
       [log_collection_settings.schema]
          # In the absence of this configmap, default value for containerlog_schema_version is "v1" if "v2" is not enabled while onboarding
          # Supported values for this setting are "v1","v2"
          # See documentation at https://aka.ms/ContainerLogv2 for benefits of v2 schema over v1 schema
          containerlog_schema_version = "v2"
       #[log_collection_settings.enable_multiline_logs]
          # fluent-bit based multiline log collection for .NET, Go, Java, and Python stacktraces. Update stacktrace_languages to specificy which languages to collect stacktraces for(valid inputs: "go", "java", "python", "dotnet").
          # NOTE: for better performance consider enabling only for languages that are needed. Dotnet is experimental and may not work in all cases.
          # If enabled will also stitch together container logs split by docker/cri due to size limits(16KB per log line) up to 64 KB.
          # Requires ContainerLogV2 schema to be enabled. See https://aka.ms/ContainerLogv2 for more details.
          # enabled = "false"
          # stacktrace_languages = []
       #[log_collection_settings.metadata_collection]
          # kube_meta_cache_ttl_secs is a configurable option for K8s cached metadata. Default is 60s. You may adjust it in below section [agent_settings.k8s_metadata_config]. Reference link: https://docs.fluentbit.io/manual/pipeline/filters/kubernetes#configuration-parameters
          # if enabled will collect kubernetes metadata for ContainerLogv2 schema. Default is false.
          # enabled = false
          # if include_fields commented out or empty, all fields will be included. If include_fields is set, only the fields listed will be included.
          # include_fields = ["podLabels","podAnnotations","podUid","image","imageID","imageRepo","imageTag"]
       #[log_collection_settings.filter_using_annotations]
          # if enabled will exclude logs from pods with annotations fluenbit.io/exclude: "true".
          # Read more: https://docs.fluentbit.io/manual/pipeline/filters/kubernetes#kubernetes-annotations
          # enabled = false

  prometheus-data-collection-settings: |-
    # Custom Prometheus metrics data collection settings
    [prometheus_data_collection_settings.cluster]
        # Cluster level scrape endpoint(s). These metrics will be scraped from agent's Replicaset (singleton)
        # Any errors related to prometheus scraping can be found in the KubeMonAgentEvents table in the Log Analytics workspace that the cluster is sending data to.

        #Interval specifying how often to scrape for metrics. This is duration of time and can be specified for supporting settings by combining an integer value and time unit as a string value. Valid time units are ns, us (or µs), ms, s, m, h.
        interval = "1m"

        ## Uncomment the following settings with valid string arrays for prometheus scraping
        #fieldpass = ["metric_to_pass1", "metric_to_pass12"]

        #fielddrop = ["metric_to_drop"]

        # An array of urls to scrape metrics from.
        # urls = ["http://myurl:9101/metrics"]

        # An array of Kubernetes services to scrape metrics from.
        # kubernetes_services = ["http://my-service-dns.my-namespace:9102/metrics"]

        # When monitor_kubernetes_pods = true, prometheus sidecar container will scrape Kubernetes pods for the following prometheus annotations:
        # - prometheus.io/scrape: Enable scraping for this pod
        # - prometheus.io/scheme: Default is http
        # - prometheus.io/path: If the metrics path is not /metrics, define it with this annotation.
        # - prometheus.io/port: If port is not 9102 use this annotation
        monitor_kubernetes_pods = false

        ## Restricts Kubernetes monitoring to namespaces for pods that have annotations set and are scraped using the monitor_kubernetes_pods setting.
        ## This will take effect when monitor_kubernetes_pods is set to true
        ##   ex: monitor_kubernetes_pods_namespaces = ["default1", "default2", "default3"]
        # monitor_kubernetes_pods_namespaces = ["default1"]

        ## Label selector to target pods which have the specified label
        ## This will take effect when monitor_kubernetes_pods is set to true
        ## Reference the docs at https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors
        # kubernetes_label_selector = "env=dev,app=nginx"

        ## Field selector to target pods which have the specified field
        ## This will take effect when monitor_kubernetes_pods is set to true
        ## Reference the docs at https://kubernetes.io/docs/concepts/overview/working-with-objects/field-selectors/
        ## eg. To scrape pods on a specific node
        # kubernetes_field_selector = "spec.nodeName=$HOSTNAME"

    [prometheus_data_collection_settings.node]
        # Node level scrape endpoint(s). These metrics will be scraped from agent's DaemonSet running in every node in the cluster
        # Any errors related to prometheus scraping can be found in the KubeMonAgentEvents table in the Log Analytics workspace that the cluster is sending data to.

        #Interval specifying how often to scrape for metrics. This is duration of time and can be specified for supporting settings by combining an integer value and time unit as a string value. Valid time units are ns, us (or µs), ms, s, m, h.
        interval = "1m"

        ## Uncomment the following settings with valid string arrays for prometheus scraping

        # An array of urls to scrape metrics from. $NODE_IP (all upper case) will substitute of running Node's IP address
        # urls = ["http://$NODE_IP:9103/metrics"]

        #fieldpass = ["metric_to_pass1", "metric_to_pass12"]

        #fielddrop = ["metric_to_drop"]

  metric_collection_settings: |-
    # Metrics collection settings for metrics sent to Log Analytics and MDM
    [metric_collection_settings.collect_kube_system_pv_metrics]
      # In the absense of this configmap, default value for collect_kube_system_pv_metrics is false
      # When the setting is set to false, only the persistent volume metrics outside the kube-system namespace will be collected
      enabled = false
      # When this is enabled (enabled = true), persistent volume metrics including those in the kube-system namespace will be collected

  alertable-metrics-configuration-settings: |-
    # Alertable metrics configuration settings for container resource utilization
    [alertable_metrics_configuration_settings.container_resource_utilization_thresholds]
        # The threshold(Type Float) will be rounded off to 2 decimal points
        # Threshold for container cpu, metric will be sent only when cpu utilization exceeds or becomes equal to the following percentage
        container_cpu_threshold_percentage = 95.0
        # Threshold for container memoryRss, metric will be sent only when memory rss exceeds or becomes equal to the following percentage
        container_memory_rss_threshold_percentage = 95.0
        # Threshold for container memoryWorkingSet, metric will be sent only when memory working set exceeds or becomes equal to the following percentage
        container_memory_working_set_threshold_percentage = 95.0

    # Alertable metrics configuration settings for persistent volume utilization
    [alertable_metrics_configuration_settings.pv_utilization_thresholds]
        # Threshold for persistent volume usage bytes, metric will be sent only when persistent volume utilization exceeds or becomes equal to the following percentage
        pv_usage_threshold_percentage = 60.0

    # Alertable metrics configuration settings for completed jobs count
    [alertable_metrics_configuration_settings.job_completion_threshold]
        # Threshold for completed job count , metric will be sent only for those jobs which were completed earlier than the following threshold
        job_completion_threshold_time_minutes = 360
  integrations: |-
    [integrations.azure_network_policy_manager]
        collect_basic_metrics = false
        collect_advanced_metrics = false
    [integrations.azure_subnet_ip_usage]
        enabled = false

# Doc - https://github.com/microsoft/Docker-Provider/blob/ci_prod/Documentation/AgentSettings/ReadMe.md
  agent-settings: |-
    # High log scale option for container logs high log volume scenarios. This mode is optimized for high log volume scenarios and have little higher resource utilization on low scale which will be addressed in subsequent agent releases.
    # Refer to public documentation for more details - https://aka.ms/cihsmode before enabling this setting as this setting has additional dependencies such as Data Collection endpoint and Microsoft-ContainerLogV2-HighScale stream instead of Microsoft-ContainerLogV2.
    [agent_settings.high_log_scale]
      enabled = false

    # prometheus scrape fluent bit settings for high scale
    # buffer size should be greater than or equal to chunk size else we set it to chunk size.
    # settings scoped to prometheus sidecar container. all values in mb
    [agent_settings.prometheus_fbit_settings]
      tcp_listener_chunk_size = 10
      tcp_listener_buffer_size = 10
      tcp_listener_mem_buf_limit = 200

    # prometheus scrape fluent bit settings for high scale
    # buffer size should be greater than or equal to chunk size else we set it to chunk size.
    # settings scoped to daemonset container. all values in mb
    # [agent_settings.node_prometheus_fbit_settings]
      # tcp_listener_chunk_size = 1
      # tcp_listener_buffer_size = 1
      # tcp_listener_mem_buf_limit = 10

    # prometheus scrape fluent bit settings for high scale
    # buffer size should be greater than or equal to chunk size else we set it to chunk size.
    # settings scoped to replicaset container. all values in mb
    # [agent_settings.cluster_prometheus_fbit_settings]
      # tcp_listener_chunk_size = 1
      # tcp_listener_buffer_size = 1
      # tcp_listener_mem_buf_limit = 10

    # The following settings are "undocumented", we don't recommend uncommenting them unless directed by Microsoft.
    # They increase the maximum stdout/stderr log collection rate but will also cause higher cpu/memory usage.
    ## Ref for more details about Ignore_Older -  https://docs.fluentbit.io/manual/v/1.7/pipeline/inputs/tail
    # [agent_settings.fbit_config]
    #   log_flush_interval_secs = "1"                 # default value is 15
    #   tail_mem_buf_limit_megabytes = "10"           # default value is 10
    #   tail_buf_chunksize_megabytes = "1"            # default value is 32kb (comment out this line for default)
    #   tail_buf_maxsize_megabytes = "1"              # default value is 32kb (comment out this line for default)
    #   enable_internal_metrics = "true"              # default value is false
    #   tail_ignore_older = "5m"                      # default value same as fluent-bit default i.e.0m

    # On both AKS & Arc K8s enviornments, if Cluster has configured with Forward Proxy then Proxy settings automatically applied and used for the agent
    # Certain configurations, proxy config should be ignored for example Cluster with AMPLS + Proxy
    # in such scenarios, use the following config to ignore proxy settings
    # [agent_settings.proxy_config]
    #    ignore_proxy_settings = "true"  # if this is not applied, default value is false

    # Disables fluent-bit for perf and container inventory for Windows
    #[agent_settings.windows_fluent_bit]
    #    disabled = "true"

    # The following settings are "undocumented", we don't recommend uncommenting them unless directed by Microsoft.
    # Configuration settings for the waittime for the network listeners to be available
    # [agent_settings.network_listener_waittime]
    #   tcp_port_25226 = 45                           # Port 25226 is used for telegraf to fluent-bit data in ReplicaSet
    #   tcp_port_25228 = 60                           # Port 25228 is used for telegraf to fluentd data
    #   tcp_port_25229 = 45                           # Port 25229 is used for telegraf to fluent-bit data in DaemonSet

    # The following settings are "undocumented", we don't recommend uncommenting them unless directed by Microsoft.
    # [agent_settings.mdsd_config]
    #   monitoring_max_event_rate = "50000" # default 20K eps
    #   backpressure_memory_threshold_in_mb = "1500" # default 3500MB
    #   upload_max_size_in_mb = "20"  # default 2MB
    #   upload_frequency_seconds = "1" # default 60 upload_frequency_seconds
    #   compression_level = "0"  # supported levels 0 to 9 and 0 means no compression

    # Disables fluentd and uses fluent-bit for all data collection
    # [agent_settings.resource_optimization]
    #   enabled = false  # if this is not applied, default value is false

    # The following settings are "undocumented", we don't recommend uncommenting them unless directed by Microsoft.
    # [agent_settings.telemetry_config]
    #   disable_telemetry = false  # if this is not applied, default value is false

    # The following setting is for KubernetesMetadata CacheTTL Settings.
    # [agent_settings.k8s_metadata_config]
    #   kube_meta_cache_ttl_secs = 60  # if this is not applied, default value is 60s

    # [agent_settings.chunk_config]
    #    PODS_CHUNK_SIZE = 10  # default value is 1000 and for large clusters with high number of pods, this can reduced to smaller value if the gaps in KubePodInventory/KubeNodeInventory data.

metadata:
  name: container-azm-ms-agentconfig
  namespace: kube-system
```

### 🖐️보안 취약점 조치

Container image 빌드 시 trivy를 통해서 보안 취약점을 확인 할 수 있다.

![image.png](Session%203%20ae1184d30d924f6faeb8e2fd02232eb8/image%2017.png)

- trivy scan 결과

```bash
## 너무 많아서 몇개만.. 올렸습니다..
Java (jar)
==========
Total: 167 (HIGH: 124, CRITICAL: 43)

┌──────────────────────────────────────────────────────────────┬──────────────────┬──────────┬────────┬───────────────────┬─────────────────────────────────────────────────┬──────────────────────────────────────────────────────────────┐
│                           Library                            │  Vulnerability   │ Severity │ Status │ Installed Version │                  Fixed Version                  │                            Title                             │
├──────────────────────────────────────────────────────────────┼──────────────────┼──────────┼────────┼───────────────────┼─────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ ch.qos.logback:logback-classic (app.jar)                     │ CVE-2023-6378    │ HIGH     │ fixed  │ 1.2.3             │ 1.3.12, 1.4.12, 1.2.13                          │ logback: serialization vulnerability in logback receiver     │
│                                                              │                  │          │        │                   │                                                 │ https://avd.aquasec.com/nvd/cve-2023-6378                    │
├──────────────────────────────────────────────────────────────┤                  │          │        │                   │                                                 │                                                              │
│ ch.qos.logback:logback-core (app.jar)                        │                  │          │        │                   │                                                 │                                                              │
│                                                              │                  │          │        │                   │                                                 │                                                              │
├──────────────────────────────────────────────────────────────┼──────────────────┼──────────┤        ├───────────────────┼─────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ com.fasterxml.jackson.core:jackson-databind (app.jar)        │ CVE-2018-14718   │ CRITICAL │        │ 2.9.6             │ 2.9.7, 2.8.11.3, 2.7.9.5, 2.6.7.3               │ jackson-databind: arbitrary code execution in slf4j-ext      │
│                                                              │                  │          │        │                   │                                                 │ class                                                        │
│                                                              │                  │          │        │                   │                                                 │ https://avd.aquasec.com/nvd/cve-2018-14718                   │
│                                                              ├──────────────────┤          │        │                   ├─────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│                                                              │ CVE-2018-14719   │          │        │                   │ 2.9.7, 2.8.11.3, 2.7.9.5                        │ jackson-databind: arbitrary code execution in blaze-ds-opt   │
│                                                              │                  │          │        │                   │                                                 │ and blaze-ds-core classes                                    │
│                                                              │                  │          │        │                   │                                                 │ https://avd.aquasec.com/nvd/cve-2018-14719                   │
│                                                              ├──────────────────┤          │        │                   │                                                 ├──────────────────────────────────────────────────────────────┤
│                                                              │ CVE-2018-14720   │          │        │                   │                                                 │ jackson-databind: exfiltration/XXE in some JDK classes       │
│                                                              │                  │          │        │                   │                                                 │ https://avd.aquasec.com/nvd/cve-2018-14720                   │
│                                                              ├──────────────────┤          │        │                   │                                                 ├──────────────────────────────────────────────────────────────┤
│                                                              │ CVE-2018-14721   │          │        │                   │                                                 │ jackson-databind: server-side request forgery (SSRF) in      │
│                                                              │                  │          │        │                   │                                                 │ axis2-jaxws class                                            │
│                                                              │                  │          │        │                   │                                                 │ https://avd.aquasec.com/nvd/cve-2018-14721                   │
│                                                              ├──────────────────┤          │        │                   ├─────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│                                                              │ CVE-2018-19360   │          │        │                   │ 2.9.8, 2.8.11.3, 2.7.9.5                        │ jackson-databind: improper polymorphic deserialization in    │
│                                                              │                  │          │        │                   │                                                 │ axis2-transport-jms class                                    │
│                                                              │                  │          │        │                   │                                                 │ https://avd.aquasec.com/nvd/cve-2018-19360                   │
│                                                              ├──────────────────┤          │        │                   ├─────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│                                                              │ CVE-2018-19361   │          │        │                   │ 2.7.9.5, 2.9.8, 2.8.11.3                        │ jackson-databind: improper polymorphic deserialization in    │
│                                                              │                  │          │        │                   │                                                 │ openjpa class                                                │
│                                                              │                  │          │        │                   │                                                 │ https://avd.aquasec.com/nvd/cve-2018-19361                   │
│                                                              ├──────────────────┤          │        │                   ├─────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│                                                              │ CVE-2018-19362   │          │        │                   │ 2.9.8, 2.8.11.3, 2.7.9.5, 2.6.7.3               │ jackson-databind: improper polymorphic deserialization in    │
│                                                              │                  │          │        │                   │                                                 │ jboss-common-core class                                      │
│                                                              │                  │          │        │                   │                                                 │ https://avd.aquasec.com/nvd/cve-2018-19362                   │
│                                                              ├──────────────────┤          │        │                   ├─────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│                                                              │ CVE-2019-14379   │          │        │                   │ 2.9.9.2, 2.8.11.4, 2.7.9.6                      │ jackson-databind: default typing mishandling leading to      │
│                                                              │                  │          │        │                   │                                                 │ remote code execution                                        │
│                                                              │                  │          │        │                   │                                                 │ https://avd.aquasec.com/nvd/cve-2019-14379                   │
│                                                              ├──────────────────┤          │        │                   ├─────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────┤
```

### #위 취약점들을 조치 하기 위해선 dependency 버전을 업그레이드 해야하지만 그전에 springboot, jdk 버전이 너무 낮아 버전을 올리고 취약점 조치를 해야한다.

### spring-boot & jdk 버전 업 수행

- springboot: 2.1.2 → 3.3.3
- jdk 1.8. → 17
- be-chgbill → be-chgbill2

### 라이브러리 취약점 제거

![image.png](Session%203%20ae1184d30d924f6faeb8e2fd02232eb8/image%2018.png)

- 도커 이미지 JAVA 8 에서 JAVA 17로  변경한다. (  이번 예제는 eclipse temurin 버전을 별로 가공을 하지 않았기 때문에 eclipse-temurin:17.0.12_7-jre-alpine 이미지를  사용해도 된다. )

```bash
##Dockerfile

#### 1) Maven build
FROM  maven:3.8.4-openjdk-17 AS MAVEN_BUILD
.
.
.
### 2) Package Stage
FROM aksaz01devvvd01registry.azurecr.io/bg012401-vivaldi/eclipse-temurin:v1
```

- Springboot 버전 3.x.x로 수정 시 application.yml 을 환경 별로 3개로 분리 한다.

![image.png](Session%203%20ae1184d30d924f6faeb8e2fd02232eb8/image%2019.png)

- 코드내에 Profile 가져오기 또한 아래와 같이 변경 된다.

[Before] @Value(”${spring.profiles}”) private String profiles;

[After] @Value(”${spring.profiles.active}”) private String profiles

- Redis config도 변경이 필요하다. spring.redis → spring.data.redis

![image.png](Session%203%20ae1184d30d924f6faeb8e2fd02232eb8/image%2020.png)

![image.png](Session%203%20ae1184d30d924f6faeb8e2fd02232eb8/image%2021.png)

- Swagger 2.x(SwaggerConfig.java 삭제) → Swagger 3.x(OpenApiConfig.java추가) 변경이 가장 변동 사항이 크다.
- SpringSecurity 변경 (5.x.0 → 6.x)

![image.png](Session%203%20ae1184d30d924f6faeb8e2fd02232eb8/image%2022.png)

- Frontend 서비스를 지원하는 경우 5가지를 변경해야 한다.
    - SpringSecurity 변경 (5.x.0 → 6.x)
    - XssConfig 추가
    - 3개 필터 추가 (HTMLCharacterEscapes, XSSFilter, XSSFilterWrapper)

![image.png](Session%203%20ae1184d30d924f6faeb8e2fd02232eb8/image%2023.png)

- javax를 jakarta로 변경한다.

| 구분 | Before | **After** |
| --- | --- | --- |
| servlet | import javax.servlet.http.HttpServletRequest; | **import jakarta.servlet.http.HttpServletRequest;** |
| annotation | import javax.annotation.PostConstruct | **import jakarta.annotation.PostConstruct;** |
| xml.ws.Holder | import javax.xml.ws.Holder; | **import jakarta.xml.ws.Holder;** |

- 이번 실습에서는 이 모든 코드를 수정 하지 않습니다. 사전에 미리 준비해드렸고, 실습시간에 수행해야할 과제가 있습니다.

### 💯과제) be-chgbill2의 Trivy에서 검출된 Dependency 수정

- 본인의 branch로 수정 - gitworkflow.yaml
- Trivy에서 출력된 dependency 확인 후, pom.xml 에서 수정
- Github action 수행 성공 시, 과제 클리어!!