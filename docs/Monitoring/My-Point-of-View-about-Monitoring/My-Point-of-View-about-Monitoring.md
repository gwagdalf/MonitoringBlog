# 모니터링에 대한 나의 관점
## 모니터링 구분
![Error-Metric-Trace](https://peter.bourgon.org/img/instrumentation/01.png)

크게 3가지, 때론 5가지의 유형으로 모니터링툴을 구분하여 다루고 있습니다. 

출처 : [Metrics, tracing, and logging](https://peter.bourgon.org/blog/2017/02/21/metrics-tracing-and-logging.html)

#### Logging
Events 기반으로 발생하는 error log / activity log 를 수집합니다.

#### Metrics
CPU, Memory, TPS, Response Time 등 grafana dashboard 에 표시할 데이터를 수집합니다. 

#### Trace
Web Request 를 처음부터 끝까지 추적하여 볼 수 있게 합니다. 주로 IO 중심의 실행시간을 표시하여, 병목구간을 찾는데 사용합니다.

#### Topology or Dependency Map
Server 간 호출관계등을 Graph 로 표현합니다.

#### Profiler
deep dive 하여, 미세한 병목구간을 찾을 때 사용합니다. 또 api / db parameter 도 확인 할 수 있습니다. 

---
## 선호하는 모니터링 툴
![Preferred Monitoring Tool](https://raw.githubusercontent.com/gwagdalf/images/master/MonitoringBlog/Monitoring/My-Point-of-View-about-Monitoring/Preferred-Monitoring-Tool.png)

[선호하는 모니터링 툴 google spread](https://docs.google.com/spreadsheets/d/1iaYxSQeSABVhddA3FoWU5kT9p-inykbiIME-utEszrs/edit?folder=10aSBCu8RSMjiyaU7w8HUlJjZBtlQAJQT#gid=0)

##### Zipkin 과 Pinpoint 를 같이 사용하는 이유
> 개인적으로 Pinpoint 를 선호하며, 강력한 모니터링 툴이라고 생각합니다. 

> 하지만, Zipkin 은 전체를 IO 중심으로 단순화 해서 보는데 장점이 있으며, plugin type 이기 때문에 customize 해서 사용하기 쉬운 장점이 있습니다.

> 또 Kafka 를 사용할 때, 각 Consumer 의 병렬처리를 Pinpoint 는 완벽하게 지원하지 않습니다.  

##### Jaeger + Kiali
근래 들어 가장 핫하고, 힘 받고 있는 Trace & Topology Tool 이라고 생각합니다만, Trace 의 경우 customize 사용할 일이 많은데, GO language 가 익숙하지 않아 사용하지 않고 있습니다.
 
##### Elastic APM
.net framework 등 다양한 언어를 지원하기 때문에, 관심을 가지고 있어야 하는 APM 툴입니다. 
