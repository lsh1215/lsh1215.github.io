---
title: "프로젝트에 모니터링 툴 구축하기"
date: 2024-05-29 10:00:00 +0900
categories: [DevOps]
tags: [모니터링, Prometheus, Grafana, Docker, DevOps]
---

# 프로젝트에 모니터링 툴 구축하기

## 모니터링이란?

![시스템 모니터링 도구](/assets/img/posts/2024-05-29-project-monitoring-tool-setup/moni_1.webp)

다들 위 사진과 같은 툴로 자기 컴퓨터의 cpu, memory 사용량 등에 대해 한번쯤 확인해본적이 있을 것입니다. 우리가 컴퓨터에서 렉이 걸릴 때 cpu 사용량이 과다한지, 아니면 메모리 사용량이 과다한지를 그냥은 알지 못하기 때문에 다음과 같은 툴(작업관리자, htop) 등을 이용해서 직관적으로 문제를 찾고자 합니다. 그러나 이러한 툴은 과거의 데이터까지 제공해주지 않고 때로는 문제를 확인하기에는 부족합니다.

그래서 모니터링(Monitoring) 시스템을 구축하여 시스템, 네트워크, 애플리케이션, 서버 등 다양한 IT 자산의 성능과 상태를 지속적으로 감시하고 분석하여 문제를 발견하고 해결하고자 합니다. 이를 통해 서비스 가용성을 높이고, 성능을 최적화하며, 보안 위협을 사전에 차단할 수 있습니다.

## 모니터링의 범주

결국 대부분의 모니터링은 이벤트에 대한 것이고 다음과 같은 상황이 이벤트에 해당합니다.

- HTTP 요청 수신
- HTTP 400 응답 송신
- 함수 시작
- if 문의 else에 도달
- 함수종료
- 사용자 로그인
- 디스크에 데이터 쓰기
- 네트워크에서 데이터 읽기
- 커널에 추가 메모리 요청

모든 이벤트에는 컨텍스트가 있다. HTTP 요청에는 들어오고 나가는 IP 주소, 요청 URL, 설정된 쿠기, 요청한 사용자 정보가 들어 있습니다.

모든 이벤트에 대한 컨텍스트를 파악하면, 디버깅에 큰 도움이 되고 기술적인 관점과 비즈니스 관점 모두에서 시스템의 수행 방법을 이해할 수 있지만, 처리 및 저장해야 하는 데이터 양이 늘어납니다.

따라서, 데이터의 양을 감소시킬 수 있는 방법으로 프로파일링, 트레이싱, 로깅, 메트릭의 네 가지가 있습니다.

## 메트릭과 로그의 개념

메트릭은 컨텍스트를 대부분 무시하고 다양한 유형의 이벤트에 대해 시간에 따른 집계(aggregation)를 추적합니다. 아마도 우리가 경험할 수 있는 Metric으로는 수신된 HTTP 요청 횟수, 요청을 처리하는 데 걸린 시간, 현재 진행 중인 요청 수 등이다. 컨텍스트 정보를 제외하면 필요한 데이터 양과 처리는 합리적인 수준으로 유지된다.

메트릭을 이용하면 애플리케이션의 각 서브시스템에서의 대기 시간과 처리하는 데이터 양을 추적해서 성능 저하의 원인이 정확히 무엇인지 쉽게 알아낼 수 있습니다. 로그에는 많은 필드를 기록할 수는 없지만 어떤 서브시스템에 문제의 원인이 있는지를 찾아낸 다음, 로그를 통해 해당 문제에 관련된 사용자 요청을 정확하게 파악할 수 있습니다.

심플하게 생각하면 메트릭은 특정 기준에 대한 수치를 나타내는 것이라면 로그는 어떤 오류인지 파악하기 위해 사용하는 데이터를 의미한다. 예를 들면 어플리케이션의 레이턴시가 높아지는 상황을 메트릭을 통해 파악한다면 이로 인해 발생하는 오류에 대한 내용을 파악하기 위해서는 로그를 사용합니다.

### 컨테이너 인프라 환경에서 metric 구분

- **system metrics**: 파드 같은 오브젝트에서 측정되는 CPU와 메모리 사용량을 나타내는 메트릭
- **service metrics**: HTTP 상태 코드 같은 서비스 상태를 나타내는 메트릭

## Metric 수집 방식에서의 2가지 방식

Metric 수집 방식의 모니터링에는 아래와 같은 두가지 방식이 있습니다.

### 1. 가져오거나(pull-based monitoring system)

![Pull-based 모니터링](/assets/img/posts/2024-05-29-project-monitoring-tool-setup/moni_2.webp)

### 2. 보내거나 (push-based monitoring system)

![Push-based 모니터링](/assets/img/posts/2024-05-29-project-monitoring-tool-setup/moni_3.webp)

대상(Target)으로부터 메트릭 값을 받아오는 모니터링 시스템을 Pull-based monitoring system이라고 하고, 이후 다루게 될 Prometheus도 이러한 Pull 방식을 사용합니다.

Pull 기반은 Target에 직접적으로 접근하여 데이터를 Scraping 하며 그 대상이 되는 Target은 데이터를 노출 시킬 방법이 필요하며 Prometheus는 이러한 니즈에 대해 생태계가 잘 구축되어있습니다.

## 프로메테우스 + 그라파나

![Prometheus + Grafana 아키텍처](/assets/img/posts/2024-05-29-project-monitoring-tool-setup/moni_4.webp)

### 프로메테우스 + 그라파나 아키텍처

시간이 지남에 따라 추이가 변하는 데이터를 메트릭(Metric)이라고 부른다. CPU 사용률, 메모리 사용률, 트래픽 등이 메트릭(Metric)에 해당된다. 메트릭은 시간별로 데이터가 수집되기에 그 양이 많다. 그래서 외부에 메트릭을 저장하는 DB를 두는데, 대표적으로 프로메테우스(Prometheus)가 있다.

프로메테우스는 일정시간 간격으로 App에 접근하여 메트릭 데이터를 수집한다. 프로메테우스는 메트릭을 저장하는 기능에 특화되어 있지만 추이를 시각화하는데는 특화되어 있지 않다. 그래서 프로메테우스 DB에 원격으로 쿼리를 날려 데이터를 가져와 시각화하는 툴(Tool)이 필요한데, 그것이 **그라파나(Grafana)**이다.

즉, prometheus는 아키텍트에서 Back-end 역할을, grafana는 아키텍트에서 Front-end 역할을 합니다.

## 구현

실제로 프로젝트(spring boot)에서 모니터링 툴을 구축하려면 어떻게 해야하는 지 알아보자.

아래 코드는 metric을 수집할 때 actuator와 node-exporter 두 가지 모두 사용하고 있으니 자신의 프로젝트 상황에 맞게 적용하도록 하자.

### (docker)compose_dev

```yaml
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - '9090:9090'
    restart: unless-stopped
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - ./prometheus/data:/prometheus/data
    #      - prometheus_data : /prometheus
    command:
      - '--web.enable-lifecycle'
      - '--storage.tsdb.path=/prometheus'
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--web.console.libraries=/usr/share/prometheus/console_libraries'
      - '--web.console.templates=/usr/share/prometheus/consoles'
    privileged: true
    networks:
      - monitoring-network

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - '3001:3000'
    restart: always
    #    unless-stopped
    environment:
      GF_INSTALL_PLUGINS: "grafana-clock-panel,grafana-simple-json-datasource"
      GF_AUTH_ANONYMOUS_ENABLED: 'true'
      GF_AUTH_ANONYMOUS_ORG_ROLE: 'Admin'
      GF_AUTH_DISABLE_LOGIN_FORM: 'true'
    depends_on:
      - prometheus
      - node-exporter
    networks:
      - monitoring-network
    volumes:
      - ./monitoring/grafana:/var/lib/grafana
      - ./monitoring/grafana/provisioning/datasources:/etc/grafana/provisioning/datasources
      - ./monitoring/grafana/provisioning/dashboards:/etc/grafana/provisioning/dashboards

  node-exporter:
    image: prom/node-exporter
    container_name: node-exporter
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
      - /etc/hostname:/etc/nodename
      - ./node-exporter/data:/etc/node-exporter
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--collector.textfile.directory=/etc/node-exporter/'
    ports:
      - 9100:9100
    restart: always
    deploy:
      mode: global
    networks:
      - monitoring-network
  
networks:
  monitoring-network:
    driver: bridge
```

### build.gradle

```gradle
//Spring Actuator
implementation 'org.springframework.boot:spring-boot-starter-actuator'
//Prometheus
implementation 'io.micrometer:micrometer-registry-prometheus'
// runtimeOnly 'io.micrometer:micrometer-registry-prometheus'
```

### application.yml

```yaml
#server:
#  port:8080  # 필요하면 추가

management:
  endpoints:
    web:
      exposure:
        include: "*" #  이 설정은 모든 Actuator 엔드포인트를 외부에 노출
      base-path: /actuator # Actuator 엔드포인트의 기본 경로 지정
```

### /prometheus/prometheus.yml

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: [
        'prometheus:9090',
        'host.docker.internal:9090'
      ]

  - job_name: 'node-exporter'
    static_configs:
      - targets: [
        'host.docker.internal:9100',
        'node-exporter:9100'
      ]

  # job 기준으로 메트릭을 수집한다.
  - job_name: "spring-actuator" # job_name 은 모든 scrap 내에서 고유해야한다
    metrics_path: '/actuator/prometheus' # 스프링부트에서 설정한 endpoint [metrics_path의 기본 경로는 '/metrics'], 뒤에 prometheus를 붙이면 prometheus의 정보를 가져온다.
    scrape_interval: 15s # global 값과 다르게 사용할려면 따로 정의하자.
#    scheme: 'http' # request를 보낼 scheme 설정 [scheme의 기본값은 `http`]
    static_configs:
      - targets: [
                  'host.docker.internal:8080']
    # request를 보낼 server ip 그리고 actuator port를 적어주면 된다. (위에서 지정한 포트는 9090이며, 서버 IP는 EC2의 Public Ip를 적어주자)
    # 보안을 위해서 JWT를 사용. (내 Spring 서버에서 사용할 수 있는 JWT를 발급받아서 사용)
#    bearer_token: 'JWT Token'

#  - job_name: 'nginx'
#    scrape_interval: 10s
#    static_configs:
#      - targets: ['nginxexporter:9113']
```

### /monitoring/dashboards/dashboard.yml

```yaml
apiVersion: 1

providers:
  - name: 'Prometheus'
    orgId: 1
    folder: Monitor
    type: file
    disableDeletion: false
    editable: true
    allowUiUpdates: true
    options:
      path: /etc/grafana/provisioning/dashboards
```

같은 path에 자신이 사용할 적당한 grafana dashboard json file을 만듭니다.

### /monitoring/datasources/datasource.yml

```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    orgId: 1
    url: http://prometheus:9090
    basicAuth: false
    isDefault: true
    editable: true
```
