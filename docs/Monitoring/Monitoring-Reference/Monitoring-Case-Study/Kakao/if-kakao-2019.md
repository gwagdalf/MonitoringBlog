# 카카오의 모니터링 사례 조사
## NEO - apm
#### naming
Neo from matrix<br/>
<img src="https://t1.daumcdn.net/thumb/R720x0/?fname=http://t1.daumcdn.net/brunch/service/user/15UX/image/0c95gagmlLaILF2c0TQ-u_rKzzQ.jpeg" width="150px" title="matrix Neo" alt="matrix Neo" /><br/>
\+Neo from kakao character<br/>
<img src="http://blog.snackfever.com/wp-content/uploads/2017/01/Screen-Shot-2017-01-19-at-11.07.31-AM-300x281.png" width="150px" title="kakao Neo" alt="kakao Neo" /><br/>

#### Neo 의 장점
##### 1. TPS/Response time(Error rate %) 이 나오는 Topology
> * __215 TPS / Response time 14.0ms (Error rate 0.09%)__ 의 정보가 Topology 화면에 효과적으로 표시 됩니다.
* Service Grouping 도 지원하는 것으로 추정합니다.<br/>
![Topology-1](https://raw.githubusercontent.com/gwagdalf/images/master/MonitoringBlog/Monitoring/Monitoring-Reference/Monitoring-Case-Study/Kakao/neo-topology-1.png)

##### 2. Observability
> * 요새 모니터링의 트렌드인 Observability(관측가능성)를 강화하기 위해, x-view 에서 __daily compare__ 를 지원합니다.<br/>

![x-view daily compare](https://raw.githubusercontent.com/gwagdalf/images/master/MonitoringBlog/Monitoring/Monitoring-Reference/Monitoring-Case-Study/Kakao/neo-daily-compare-view.png)<br/>

```
Observability is the ability to infer internal states of a system based on the system’s external outputs. In control theory, observability is a mathematical dual (follows a direct conceptual mapping) to controllability, which is the ability to control internal states of a system by manipulating external inputs.
; output log 로부터 시스템 내부의 상태를 유추. 단순한 데이터의 나열이 아닌, 변화를 인식할 수 있는 의미있는 추론
```

##### 3. Adaptive Sampling 
> * __응답시간별 샘플링__ : agent 에서 response time 이 빠른 건은 적게 샘플링하고, response time 이 느린건은 많이 샘플링 합니다.
* 예상컨데, Application 에서는 interceptor 가 100% 실행되고, agent -> collector 로 보낼때 sampling 이 적용되는 것으로 추정합니다. cpu 는 희생하고, network 부하 감소를 위한 sampling 방법으로 추정합니다.<br/>
![응답시간별 샘플링](https://raw.githubusercontent.com/gwagdalf/images/master/MonitoringBlog/Monitoring/Monitoring-Reference/Monitoring-Case-Study/Kakao/neo-horizontal-sampling.png)

* vertical sampling 은 window 기법으로 특정 시간대만 sampling 하는 방법을 사용한 듯 합니다. 위의 응답시간별 샘플링은 network 부하를 경감하기 위한 방법이라면, vertical sampling 은 cpu/memory 부하를 경감하기 위한 sampling 으로 보입니다. 순서상으로는
  - 1.vertical sampling ( 시간별, cpu/memory 부하경감 ) 이 먼저 진행되고
  - 2.horizontal sampling ( 응답시간별, network 부하경감 ) 이 나중에 진행되리라<br/>
  추정합니다만 응답시간별 샘플링은 kakao 가 처음 시도하는 것이여서 강조하기 위해 먼저 소개한 것 같습니다.<br/> 

![double sampling](https://raw.githubusercontent.com/gwagdalf/images/master/MonitoringBlog/Monitoring/Monitoring-Reference/Monitoring-Case-Study/Kakao/neo-vertical-sampling-with-horizontal-sampling.png)

> * __Traffic limit 과 Backpressure__(queue?) 를 지원합니다. <br/>

![limit & backpressure](https://raw.githubusercontent.com/gwagdalf/images/master/MonitoringBlog/Monitoring/Monitoring-Reference/Monitoring-Case-Study/Kakao/neo-limit-and-backpressure.png)  

-------
#### feature
##### Main dashboard
많은 사람들이 좋아하는 X-view 를 지원합니다. 함께 볼 수 있는 지표는 아래의 표와 같습니다.
  
| Transaction Window(X-view) | Active Transaction | Response Time |
|:--------|:--------:|:--------:|
|  | Visitor Count  | TPS |

| CPU(node)  | Process Cpu(pod) | Memory | Used Heap |
|:--------|:--------:|:--------:|:--------:|
| Service count | Active Count | Error Rate | Open FileDescriptor |

![main dashboard](https://raw.githubusercontent.com/gwagdalf/images/master/MonitoringBlog/Monitoring/Monitoring-Reference/Monitoring-Case-Study/Kakao/neo-main-dashboard-1.png)

#### Spec & Data

* Yesterday Message Size : 1.1 TiB
* Yesterday Message Count : 89억 ( 10만/s )
* Cassandra
  - tsp : 13만/s
  - db spec server 사용 ( 발표하신 김은수님이 당시, 자세한 spec 은 모르셨어요 )
  - idc 당 5대 * 2 = 10대
  - memory 64G 중 50% JVM  


#### Cassandra
* High performance __writing__
  - __WRITE__ > read
* Scalibility
  - idc 지원 -> dc=pg|gs,   pg 판교, gs 가산<br/>
  
![cassandra idc properties](https://raw.githubusercontent.com/gwagdalf/images/master/MonitoringBlog/Monitoring/Monitoring-Reference/Monitoring-Case-Study/Kakao/neo-cassandra-idc-properties.png)<br/>  
  
* Time series data store
  - partition key : 어느 파티션으로 -> Application ID + __Time Bucket__
  - cluster key : 어느 순서로 -> Time

* 장애경험 공유 : LZ4Compressor 가 Disk IO 를 많이 먹어서, DeflateCompressor 를 사용

<img src="https://raw.githubusercontent.com/gwagdalf/images/master/MonitoringBlog/Monitoring/Monitoring-Reference/Monitoring-Case-Study/Kakao/neo-cassandra-disk-io-issue-1.png" width="650px" alt="cassandra disk io issue 1" /><br/>

<img src="https://raw.githubusercontent.com/gwagdalf/images/master/MonitoringBlog/Monitoring/Monitoring-Reference/Monitoring-Case-Study/Kakao/neo-cassandra-disk-io-issue-2.png" width="650px" alt="cassandra disk io issue 2" /><br/>
 

### KEMI - efk
NEO 에서 너무 많은 시간을 사용해서, KEMI 쪽은 나중에 다시 정리해야 되겠습니다.

##### KEMI-Stat
![KEMI-stat](https://tech.kakao.com/files/kemi-stats.jpg)<br/>


##### KEMI-Log
![KEMI-log](https://tech.kakao.com/files/kemi-log.jpg)<br/>

* EFK  Elasticsearch + Fluentd + Kibana


##### PostgreSQL : JSON
* json 을 text 로 저장

##### PostgreSQL : JSONB
> * key-value 형태의 binary로 저장
> * indexing 지원 ( B-Tree, GIN )



## Reference
##### KEMI efk
[KEMI(Kakao Event Metering & monItoring) tech blog 2016](https://tech.kakao.com/2016/08/25/kemi/)

[Microservice에서 쏟아지는 로그를 Perl5를 사용하여 로그수집기들로 잘 보내고 활용하기 pdf 2019](https://mk.kakaocdn.net/dn/if-kakao/conf2019/%EB%B0%9C%ED%91%9C%EC%9E%90%EB%A3%8C_2019/T03-S03.pdf)

[Microservice에서 쏟아지는 로그를 Perl5를 사용하여 로그수집기들로 잘 보내고 활용하기 mp4 2019](https://mk-v1.kakaocdn.net/dn/if-kakao/conf2019/conf_video_2019/1_102_03_m1.mp4)

##### NEO apm
[카카오 애플리케이션 모니터링 NEO apm pdf 2019](https://mk.kakaocdn.net/dn/if-kakao/conf2019/%EB%B0%9C%ED%91%9C%EC%9E%90%EB%A3%8C_2019/T05-S05.pdf)

[카카오 애플리케이션 모니터링 NEO apm mp4](https://mk-v1.kakaocdn.net/dn/if-kakao/conf2019/conf_video_2019/1_105_05_m1.mp4)

##### if kakao 2019
[if kakako 2019 program](https://if.kakao.com/2019/program) 