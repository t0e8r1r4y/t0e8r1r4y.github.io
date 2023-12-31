---
title: Paper Reading. Apache Druid - A Real-time Analytical Data store
date: 2021-06-09
author: Jung Hyun Shin
layout: post
tags:
  - Paper-reading
category: theory
---
    
### 1. 드루이드 개발 목적
<br/>
     
#### 1-1.드루이드란? ( 논문의 결론 부분 )

드루이는 다음을 충족하는 데이터 스토어이다.    

- 분산
- 컬럼 지향형
- 리얼타임 분석 데이터 스토어 (처리하는 데이터를 drill-down하게 aggregates 할 수 있도록 설계 됨)
- 고성능, 쿼리 지연이 낮도록 설계 됨.
- 스트리밍 데이터 처리와 fault-tolerant를 지원함.

<br/>

#### 1-2. 드루이드 개발 목적 ( 논문의 도입 부분과 문제 정의 부분 )

Druid was originally designed to solve problems around ingesting and exploring large quantities of transactional events (log data). → <strong>OLAP</strong>

드루이드는 분산,  컬럼 지향형, 리얼타임 분석 데이터 스토어 ( 쿼리 수행 속도가 더 빠른 )를 개발하고자 만들었다.

이는 기존의 하둡이 혁신적이고 데이터 저장과 대용량의 데이터에 대한 접근은 우수하지만, 데이터에 대한 접근 속도(실시간 쿼리 등), 병렬로 수많은 데이터가 쏟아지는 경우 드루이드 개발자들의 니즈에 충족되지 못하였다고 함. ( Hadoop wasn’t going to meet our needs ).

그리고 NoSql 아키텍처나 다른 솔루션들도 개발진이 원하는 수준이 아니거나, 비용타당성이 떨어짐.

<img data-width="1038" data-height="188" src="https://hissing-language-ca5.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F3ab27e22-3ff6-4e08-bd53-5cca1f5ce836%2FUntitled.png?table=block&id=521e699d-9457-4e95-adf2-3da665b9ca51&spaceId=9fd6fd24-6311-4602-ac31-2081abcd6ece&width=1300&userId=&cache=v2" />

이러한 메타데이터는 Timestamp가 있고, Dimension 컬럼이 많고, Metric 컬럼이 많다.

- 원문    
    This metadata is comprised of 3 distinct components. First, there is a timestamp column indicating when the edit
    was made. Next, there are a set dimension columns indicating various attributes about the edit such as the page that was edited, the
    user who made the edit, and the location of the user. Finally, there
    are a set of metric columns that contain values (usually numeric)
    that can be aggregated, such as the number of characters added or
    removed in an edit.
    

드루이드 개발진은 이러한 데이터를 drill-down하게 계산하고, aggregates 하는 것을 목표로 하였다. 그래서 논문의 결론을 한번더 언급하지만

- 분산
- 컬럼 지향적
- 리얼타임 데이터 분석 ( 리얼타임으로 drill-down하게 계산하고, 집계(agrregates) 하는 데이터 스토어

이런 것을 만들고자 Druid를 만들었다.

<br/>
---
<br/>

### 2. 드루이드 아키텍처

<img data-width="1038" data-height="616" src="https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fa543a8d6-b42a-4e8e-973c-a85bf031a0cf%2FUntitled.png?table=block&id=d8f5e497-5391-46e9-b381-efb29914246b&spaceId=9fd6fd24-6311-4602-ac31-2081abcd6ece&width=2000&userId=5fa4730b-7a5a-4f31-a0bb-db83c33e4454&cache=v2" />

#### 2-1. Real-time Nodes

Real-time Nodes는 실시간 데이터를 흡수하는 계층이면서 스트림에 쿼리 이벤트를 수행하는 노드라고 보임. ( 흡수하는 계층이면서 데이터 컨슈머 역할을 함 )

Real-time Nodes에서는 이벤트들을 흡수하고 실시간 쿼리가 가능한데, 이를 위해서 in-memory index 버퍼를 유지함. ( 그리고 이를 위해서 JVM 내부 힙 기반의 버퍼를 사용한 row store를 가지고 있다고 하는데 요대목이 조금 이해가 안됨 )

그리고 유지하는 index는 주기적으로 디스크에 유지하여 불변성을 유지함. 그리고 요때 컬럼지향형으로 데이터가 converting 된다.

디스크에 저장된 index는 불변이고, 이를 논문에서는 persisted-indexer라고 명명함. 그리고 disk에 저장된 index들을 Off-heap memory에 불러온다.

논문에서 제시한 도식처럼

쿼리가 Real-time Node에 들어오면( Events indexed via these nodes are immediately available for querying ) in-memory index와 Off-heap memory and persisted indexex 둘 다 searching 해서 결과를 return한다.

<img data-width="1038" data-height="821" src="https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F09a0b89e-b8bd-4243-a957-24e4ca16fb4d%2FUntitled.png?table=block&id=d3299dde-e0a7-4754-81b7-d0413df42b19&spaceId=9fd6fd24-6311-4602-ac31-2081abcd6ece&width=2000&userId=5fa4730b-7a5a-4f31-a0bb-db83c33e4454&cache=v2" />

그리고 Real-time nodes는 일정 기간동안 영구적으로 저장된 index들을 하나의 불변의 block으로 묶고 ( 이를 segment라고 명명함 ) 영구적인 백업스토리지에 업로드를 한다.

이런 백업 스토리지는 S3나 HDFS와 같은 파일시스템을 주로 쓰게되며, 이러한 행위를 드루이드에서는 deep storage라고 한다.

이 과정을 요약하자면

1. 실시간 스트리밍 데이터를 흡수하여 in-memory-index를 생성
2. in-memory-index를 컬럼 지향형 포맷으로 converting하고 Disk에 저장
3. 일정 기간(some span of time) 모인 데이터를 묶어서 segment 생성
4. 이 세그먼트를 백업 스토리지 저장 → deep store

이 과정을 다시 논문 도식에 추가해보면...

<img data-width="1038" data-height="821" src="https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F2b5860aa-1638-4544-9ee0-cfcd4d240b09%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-01-16_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9.02.04.png?table=block&id=07f9841e-970a-4fe3-a8f2-72d4e4d49cea&spaceId=9fd6fd24-6311-4602-ac31-2081abcd6ece&width=2000&userId=5fa4730b-7a5a-4f31-a0bb-db83c33e4454&cache=v2" />

(데이터 indexing 방법에 대한 일반적인 내용도 조금 참고가 필요함)

[검색 데이터 색인 방법론](https://searchtool.tistory.com/11)

<br/>

##### 2-1-1. Availability and Scalability

위에서 Real-time-node는 컨슈머라는 표현을 썻는데... 논문의 맥락상 그 이야기는 여기 나온다. 컨슈머는 별도의 데이터 스트림을 생성해 줄 프로듀서가 필요하다.

주로 이럴 때 kafka를 쓴다. 데이터 버스 역할로 카프카를 쓰는 이유는 데이터 버퍼의 역할과 데이터 손실 났을 때 카프카 broker에 저장 된 데이터 소스에서 데이터를 가져다 쓰기 때문에 손쉬운 복구가 가능하다는 의미로 보인다.

<img data-width="1038" data-height="657" src="https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fecad64c5-7214-4ad4-af16-5a2183df12c2%2FUntitled.png?table=block&id=cef26f2c-047c-4a1a-9e76-20bcd52852f4&spaceId=9fd6fd24-6311-4602-ac31-2081abcd6ece&width=2000&userId=5fa4730b-7a5a-4f31-a0bb-db83c33e4454&cache=v2" />

논문에서 제시하는 카프카 + 드루이드 조합의 성능은 원문 그대로를 가지고 왔다

<div class="highlight">🔥 원문 - In practice, this model has allowed one of the largest production Druid clusters to be able to consume raw data at approximately 500 MB/s (150,000 events/s or 2 TB hour).</div>
<br/>

#### 2-2. Historical Nodes

Historical Nodes는 불변 데이터의 읽기 일관성을 제공하는 노드다. 

<img data-width="1038" data-height="780" src="https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F424d29b8-70e8-44bf-bc5f-fc59b73e5134%2FUntitled.png?table=block&id=d2abbcd5-3b83-40fb-8205-e56650120990&spaceId=9fd6fd24-6311-4602-ac31-2081abcd6ece&width=2000&userId=5fa4730b-7a5a-4f31-a0bb-db83c33e4454&cache=v2" />

해당 노드는 Deep Storage로 부터 세그먼트를 다운로드하거나 메모리로 로드하는 역할을 한다. Real-time-nodes 설명에서 동작하는 과정에서 위의 역할을 해주는 것이 historical nodes다.

<br/>

##### 2-2-1. Tiers

Tiers는 Historical nodes의 그룹으로 정의한다. 티어들은 각각의 티어단위로 설정되는데, 

Different performance and fault-tolerance parameters can be set for each tier

라고 원문에서 밝힌다. ( 원문을 한글로 더 좋게 번역을 못하겠음 )

이렇게 하는 목적은 중요도에 따른 세그먼트들의 높고 낮은 우선순위 때문이다. 논문에서는 예시를 들어서 설명했는데, 액세스가 더 많이 되는 세그먼트들에게 더 큰 캐쉬 메모리를 할당하기 위한 목적 정도로 간단히 이해하면 될 듯.

<br/>

##### 2-2-2. Availability

Historical node는 주키퍼를 통해 세그먼트를 로드 하거나 언로드하는 명령을 받는다. 그래서 주키퍼가 unavailable 상태가 되면 더 이상 새로운 세그먼트는 받지 못함. 그러나 서빙은 가능함.

This means that Zookeeper outages do not impact current data availability on historical nodes.

<br/>

#### 2-3. Broker Nodes

Broker Nodes는 real-time nodes와 historical nodes 사이에서 ‘쿼리 라우터’ 역할을 한다. 두가지 nodes 노드에게 질의하여 결과를 merge하는 역할을 하고 있다.

<br/>

##### 2-3-1. Caching

브로커 노드는 [LRU 캐시 전략](https://0th-lab.tistory.com/6)을 사용한다. ( 캐시 전략은 별도 링크 )

<img data-width="1038" data-height="224" src="https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Ffe494747-cb02-4206-a9ae-ddbaaf31bdbc%2FUntitled.png?table=block&id=58b456b1-f11a-40cb-ba31-7bf640bfb23d&spaceId=9fd6fd24-6311-4602-ac31-2081abcd6ece&width=2000&userId=5fa4730b-7a5a-4f31-a0bb-db83c33e4454&cache=v2" />

1. Broker nodes는 쿼리를 받으면 일단 세그먼트 셋에서 먼저 일치하는 것이 있는지 확인
2. 일치하는 것이 있다면 다시 계산할 필요 없음
3. 일치하는 것이 없다면, broker nodes는 historical nodes와 real time nodes에  쿼리를 각각 날림
4. broker nodes는 historical nodes에서 온 결과 값이 있으면 이를 cache로 저장 ( 이렇게 하는 이유는 real time nodes는 perpetually changing and caching the results is unreliable 하기 때문.

<br/>

##### 2-3-2. Availability

상태를 관리하는 주키퍼가 unavailable하게 되어도 쿼리가 가능하다. 기존에 클러스터와 통신이 정상이라고 하면 쿼리 수행을 하기 때문.

<br/>

#### 2-4. Coordinator Nodes

Coordinaotr Nodes는 드루이드 아키텍처 내부에서 ZooKeeper와 같은 역할을 하는 nodes이다. Historical nodes들의 분산 코디네이터 역할을 하며, historical nodes와 주키퍼 사이에서  load new data, drop outdated data, replicate data, and move data to load balance 이런 명령을 전달하는 역할을 한다.

Coordinator Nodes들은 주키퍼에 의해 메인이 선출되고 나머지는 backup의 형태로 동작함

Coordinator Nodes들은 주기적으로 현재 클러스터들의 상태를 결정한다. MySql에는 클러스터들의 내부 파라미터, config 정보가 있다. 이런 정보를 바탕으로 예상되는 클러스터의 상태와 실제 상태를 비교하여 상태를 결정한다.

상태를 결정하면 MySql db에 저장된 rule table에 기반하여 세그먼트와 클러스터를 관리하게 된다.

<br/>

##### 2-4-1. Rules

rules은 historical segment들이 로드되고 드랍되어야 하는지에대한 것이며 다음과 같이 정의 된다.

1. Segment들이 어떻게 각기 다른 historical tier에 할당이 되어야 하는가
2. Segment들이 어떻게 replication 되어야 하는가
3. 언제 Segment가 drop 되어야 하는가. 주로 시간으로 한다고 함

그리고 이러한 rule은 MySQL db에 저장하고 load해서 사용함. 모든 Segment들에 매칭되는 Rule에 대해 적용한다.

<br/>

##### 2-4-2.  Load Balancing

production 환경에서는, historical nodes는 제한적인 리소스를 가지고 있기 때문에, segment들은 반드시 클러스터들에게 골고루 배분되어야 한다. 

이를 위해서 Druid는 a cost-based optimization procedure that takes into account the segment data source, recency,
and size 를 사용한다고 한다.

( 세부 알고리즘에 대한 설명은 없는데, 위 설명을 보자면 일반적으로 쿼리 패턴은 최근의 세그먼트를 자주 참조하는 경향이 있기 때문에, 최근 참조를 많이하는 segment는 복제를 더하는게 좋다고 논문에는 되어 있는데 맥락이 조금 이해가 안됨. )

<br/>

##### 2-4-3. Replication

coordinator nodes는 historical nodes에게 같은 segment를 복제하라고 할 수 있다. 복제하는 숫자는 설정을 통해 계산이 가능하다. 

높은 레벨의 fault tolerance를 위해서는 많은 수의 레플리카를 설정하면 된다고 하면서, 사례를 논문에서는 제시함.

<br/>

##### 2-4-4. Availability

coordinator nodes는 주키퍼와 MySql을 외부 의존성으로 가진다.

- 주키퍼 : historical nodes에 대한 클러스터 상태
- MySql : Operation에 필요한 management information

주키퍼가 unavailable 되면 현재 클러스터간 통신 상태 유지 ( 위 다른 nodes 와 동일 )

MySql이 unavailable 되면 새로운 segment의 생성, 및 drop 불가능.


<br/>
---
<br/>


### 3. Storage Format

드루이드의 데이터 테이블은 시계열 이벤트이며 파티셔닝 된 segment들의 집합임.

<div class="highlight">
🔥 segment에 대한 정의는 아래 원문을 가져왔다.

we define a segment as a collection of rows of data that span some period of time.
Segments represent the fundamental storage unit in Druid and replication and distribution are done at a segment level.

</div>

드루이드의 데이터 포맷에서는 timestamp 컬럼이 반드시 있어야 된다. (논문에서는 always requires 라고 표현 함 ) 그 이유인 즉슨, 시간의 세분성을 기준으로 데이터의 볼륨과 시간의 범위를 결정하기 때문이다.

Segment들은 데이터 소스의 식별자 역할을 한다. 새로운 segment가 추가 됨에 따라 time-interval, version이 증가되어 이 정보를 바탕으로 segment들의 최신버전???(freshness)를 반영함.

Druid의 Segment는 컬럼 지향형이다. 그래서 aggregate에 best used 되어 있다. row 지향형보다 집계를 함에 있어 column 지향은 해당 컬럼만 로드하면 되기 때문에 더 효율적이다.

Druid는 다중 컬럼 타입이 있고, 이는 다양한 데이터를 나타낸다고 함. 이러한 컬럼 타입에 의존하여 디스크에 데이터를 스토리징하는 비용이 줄어든다고 함.

<div class="highlight">
🔥 위 내용에 대한 예시와 ‘Indicaes for Filtering Data’ 그리고 ‘Storage Engine’에 대한 부분은 이해가 부족하여 차후 정리. OS 관련 지식이 조금 요구되는 부분.
</div>

<br/>
---
<br/>

### 4. Query API

<pre>
  <code class="json">
    {
      "queryType" : "timeseries",
      "dataSource" : "wikipedia",
      "intervals" : "2013-01-01/2013-01-08",
      "filter" : {
      "type" : "selector",
      "dimension" : "page",
      "value" : "Ke$ha"
    },
      "granularity" : "day",
      "aggregations" : [{"type":"count", "name":"rows"}]
    }
  </code>
</pre>

    

위에서 설명한 각각의 노드들은 RestFul API가 존재하며, POST 메서드 형식에 body에 위와 같은 JSON 형태가 들어간다.

그리고 드루이드는 조인 쿼리를 제공하지 않는다 .( A join query for Druid is not yet implemented )

이는 컬럼 지향형 데이터 구조에서 Aggregation에 최적화 된 드루이드에서는 아래 두가지 이유로 조인을 제공하지 않는다.

1. Scaling join queries has been, in our professional experience,
a constant bottleneck of working with distributed databases.
2. The incremental gains in functionality are perceived to be
of less value than the anticipated problems with managing
highly concurrent, join-heavy workloads.

조인에 대해서 여러가지 전략이 있겠지만, Druid에서는 여러가지를 조인하는 것이 복잡한 메모리 관리가 필요한 큰 테이블이 요구되어, 메모리 관리의 복잡성이 증폭된다고 한다.

<div class="highlight">🔥 결론 ! 조인을 제공하지 않는다.</div>
<br>
      


해당 부분은 아래 Druid 쿼리 관련한 내용을 함께 보면 좋을 듯.

[SQL · Apache Druid](https://druid.apache.org/docs/latest/querying/sql.html)

---

논문의 다음 내용은 성능과 Production Level에서의 드루이두 사용 결과에 대한 내용이다.

일단 druid에 대한 주요 내용은 위 내용이 중요하다고 판단되어 우선 정리하였고, 빠진 개념들을 해당 글에 계속 보강하고자 함.