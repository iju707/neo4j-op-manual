# 9.1 메모리 설정

> 이 섹션에서는 Neo4j 인스턴스에 대한 메모리설정을 어떻게 하는지 알아봅니다.
>
> 이 섹션에서 Neo4j의 메모리 관련 설정에 대해 살펴볼 것 입니다. 먼저 다른 사용환경에 따라 메모리사용이 아주 상이한 특성을 가짐을 알아야합니다. 데이터 집합의 크기나 인덱스의 존재 등이 메모리 사용량에 영향을 끼치는 요소의 예가 될 수 있습니다. 그러므로 아래에 제시한 수치는 일반적인 목적에만 해당함을 알려드립니다. Neo4j 설치 환경 및 상황에 따라서 각자에 맞게 사용하셔야 합니다.

아래 이미지를 참조하여 Neo4j 서버의 RAM은 3가지 영역으로 구성되어있습니다.

![](https://neo4j.com/docs/operations-manual/3.3/images/neo4j-memory-management.png)

메모리 설정을 할 때 다음 단계로 진행하길 권장합니다.

1. OS 메모리
   * [OS 메모리](#os-memory) - OS에 사용되는 메모리 추산
   * [Lucene Index Cache](#lucene-index-cache) - Lucene 인덱스에서 사용되는 메모리 추산
2. [Page Cache](#page-cache) - Page Cache 영역 사이즈 결정
3. [Heap Size](#heap-size) - Heap 사이즈 결정
4. [메모리 설정 검증](#메모리-설정-검증) - 설정된 메모리가 사용가능한 RAM에 맞는지 확인

위 단계를 좀 더 자세히 알아보겠습니다.

여기서 설명하는 것은 서버가 Neo4j 전용으로 운영되있음을 가정하였습니다. 만약 다른 서비스가 함께 동작하고 있는 경우에는 각각의 서비스에 대한 메모리 요구사항을 고려하여야 합니다.

## OS 메모리 {#os-memory}

시스템 일부 메모리는 OS 동작을 위하여 할당되어야 합니다. Page Cache와 Heap Space를 할당한 뒤 남은 RAM에 대하여 OS에 특정 수치만큼 할당할 수는 없습니다. 그러나, OS를 위하여 충분한 메모리를 남기지 않는 경우에는 계속적으로 Disk Swapping이 발생하며 성능에 심각한 영향을 끼칩니다.

1GB 정도가 독립적으로 Neo4j를 수행하는 서버에 대한 OS 영역 메모리로 적당합니다.

> **예제 9.1 : OS 영역에 사용될 메모리 계산**
>
> 간단하게 OS 메모리를 결정하도록 하겠습니다. 이 섹션에서는 OS 메모리로 1GB 정도가 사용된다고 하겠습니다.

시스템 사용 및 환경\(예로, 매우 큰 RAM 용량을 가진 시스템\)에 따라 1GB 이상의 메모리가 OS에 사용되는 경우도 있습니다.

## Lucene Index Cache {#lucene-index-cache}

Neo4j는 인덱싱 기능을 위하여 [Apache Lucene](https://lucene.apache.org/)을 사용합니다. 인덱스를 메모리 크기에 맞게 설정하여 인덱스 검색 성능을 최적화 할 것 입니다 OS 메모리와 유사하게, Lucene Index Cache 또한 특정한 수치로 설정할 수 는 없습니다. 대신 필요한 메모리를 측정하여 Page Cache와 Heap Space를 설정하기 전에 선점할 메모리로 계산합니다.

다음 파일명 규칙으로 해당하는 파일의 사이즈 합계를 통하여 Lucene Index Cache에 사용할 메모리 용량을 측정할 것 입니다.

* NEO4J\_HOME/data/databases/&lt;database-name&gt;/index
* NEO4J\_HOME/data/databases/&lt;database-name&gt;/schema/index/\*/\*/lucene\*

> **예제 9.2 : Lucene Index에 사용할 메모리 측정**
>
> 이 섹션에 사용할 Database 이름을 graph.db라고 하겠습니다. 아래 방법으로 index와 schema/index/lucene 디렉터리의 사이즈를 계산하겠습니다.
>
> ```bash
> $neo4j-home> ls data/databases/graph.db/index | du -ch | tail -l
> 500M    total
> $neo4j-home> find data/databases/graph.db/schema/index -regex '.*/lucene.*' | du -ch | tail -l
> 500M    total
> ```
>
> 위 예제로 보면 예약할 Lucene Index Cache 크기는 500MB + 500MB인 1GB 입니다.

## Page Cache

Page Cache는 Neo4j에서 Disk에 저장된 데이터를 캐시하는데 사용됩니다. 디스크에 있는 모든 또는 대부분의 Graph 데이터를 메모리로 캐시하게 되면 디스크 접속빈도를 줄이고 최적화된 성능으로 결과를 보여줄 수 있습니다.

추가적으로 Neo4j의 자체 인덱스 또한 Page Cache에 캐시됩니다. 직전에도 언급하였듯, 인덱스 또한 캐시되면 성능적인 측면으로 유용합니다.

Neo4j가 Page Cache 용도로 메모리를 어느정도 사용할지는 다음 파라미터에 정의합니다.

`dbms.memory.pagecache.size`

현재 운영중인 시스템인지 또는 배포를 계획하고 있는 시스템인지에 따라 아래 두가지 방법으로 Page Cache의 크기를 계산할 수 있습니다.

1. 운영중인 시스템의 경우 다음에 언급할 파일의 크기를 계산하고 확장될 비율을 곱하시면 됩니다. 예로 20% 확장성이 있는 경우 1.2를 곱하시면 됩니다.
2. Neo4j로 신규 시스템을 운용할 계획인 경우에는 일부 데이터를 적재한 뒤 데이터 비중과 확장성을 곱하여 계산하시면 됩니다. 예로 20%의 확장성을 가진 전체 데이터베이스 크기를 측정하기 위해서 1/100의 데이터를 적재한 뒤, 파일의 크기를 계산하고 120을 곱하시면 됩니다.

다음 파일명 규칙으로 해당하는 파일의 사이즈 합계를 통하여 Page Cache에 사용할 메모리 용량을 측정할 것 입니다.

* NEO4J\_HOME/data/databases/&lt;database-name&gt;/\*store.db\*
* NEO4J\_HOME/data/databases/&lt;database-name&gt;/schema/index/\*/\*/native\*

아래의 예제로 Page Cache에 사용될 메모리를 어떻게 계산하고 설정하는지 기존시스템과 신규시스템에 따라 각각 알아보겠습니다.

> **예제 9.3 : 기존 Neo4j 시스템에서 사용할 Page Cache 측정**
>
> 예제에서 사용되는 Neo4j의 데이터베이스를 graph.db라고 하겠습니다.
>
> 데이터베이스 파일의 전체 크기를 계산해보겠습니다. Posix 시스템 기반에서는 다음 명령어를 입력합니다.
>
> ```
> $neo4j-home> ls data/databases/graph.db/*store.db* | du -ch | tail -l
> 33G    total
> $neo4j-home> find data/databases/graph.db/schema/index -regex '.*/native.*' | du -ch | tail -l
> 2G    total
> ```
>
> 전체 크기는 33G + 2G 해서 총 35G 입니다. 여기에 20%의 확장성을 가진다고 가정하겠습니다. 그럼 최종적으로 설정될 정보는
>
> `dbms.memory.pagecache.size` = 1.2 \* \(35GB\) = 42GB
>
> 실제 neo4j.conf 파일에 다음과 같이 설정하시면 됩니다.
>
> ```
> dbms.memory.pagecache.size=42GB
> ```
>
> **예제 9.4 : 신규 Neo4j 시스템에서 사용할 Page Cache 측정**
>
> 예제에서 사용되는 Neo4j의 데이터베이스를 graph.db라고 하겠습니다.
>
> 새로운 데이터베이스의 경우에는 일부 데이터\(1/100\)를 적재한 뒤 비율\(1/100만큼 입력하였으므로 X 100\)만큼 곱해서 전체 데이터베이스 사이즈를 예상하는 것이 좋습니다. 향후 확장성까지 고려하여 최종 사이즈를 측정하시면 됩니다. 여기서는 20%의 확장성을 가진다고 하겠스빈다.
>
> 이미 1/100의 데이터가 입력되었다고 가정을 하고 데이터베이스 크기를 측정해보겠습니다. Posix 시스템 기반에서는 다음 명령어를 입력합니다.
>
> ```
> $neo4j-home> ls data/databases/graph.db/*store.db* | du -ch | tail -l
> 330M    total
> $neo4j-home> find data/databases/graph.db/schema/index -regex '.*/native.*' | du -ch | tail -l
> 20M    total
> ```
>
> 계산된 결과는 330M + 20M = 350M 입니다. 그럼 최종 결과를 20% 확장성을 포함하여 계산하겠습니다.
>
> `dbms.memory.pagecache.size` = 120 \* \(350MB\) = 42GB
>
> 실제 neo4j.conf 파일에 다음과 같이 설정하시면 됩니다.
>
> ```
> dbms.memory.pagecache.size=42GB
> ```

## Heap Size

Heap Space는 쿼리 수행, 트랜잭션 상태관리, 그래프 관리 등에 사용됩니다. Heap에 사용되는 메모리 크기는 Neo4j 사용방식에 따라 상이합니다. 예로들어 장기간 수행되는 쿼리, 매우 큰 데이터 사이의 Catesian Production 쿼리는 단순한 쿼리보다 더 많은 Heap 영역을 필요로 합니다. 이러한 경우에는 쿼리를 튜닝하거나 사용되는 Heap 영역의 사이즈를 모니터링 해서 증가시키게 됩니다.

Heap 영역으로 사용가능한 메모리의 크기는 Neo4j 성능에 있어서 매우 중요합니다. 일반적으로, 동시다발적인 동작을 위해서 충분한 Heap 영역으로 설정하길 권장드립니다. 만약 Heap의 크기가 작을 경우 성능에 심각한 영향을 끼칠 것 입니다.

Heap 메모리 크기는 2개의 파라미터로 설정이 됩니다. `dbms.memory.heap.initial_size`, `dbms.memory.heap.max_size`

원치 않는 Full Garbage Collection이 동작하지 않도록 두개의 파라미터 값을 동일하게 지정함을 권장합니다.

> **예제 9.5 : Heap Size 설정**
>
> 일반적인 설정에서 Neo4j가 안정적으로 동작하기 위해서는 Heap의 크기를 8GB ~ 16GB로 지정해야 합니다.
>
> 이번 예제에서는 Heap Size를 16GB로 설정하겠습니다. Heap에 대한 최초 크기와 최대크기를 동일한 수치로 지정하여 불필요한 Garbage Collection 동작을 방지하도록 합니다.
>
> ```
> dbms.memory.heap.initial_size=16G
> dbms.memory.heap.max_size=16G
> ```

## 메모리 설정 검증

여기에서 설명한 것처럼 Page Cache와 Heap Space에 대한 파라미터를 설정한 뒤, 남은 메모리 용량이 OS와 Index Cache에 충분한지 확인해야 합니다. 다음의 검증과정을 보겠습니다.

실제 OS를 위한 남은 메모리 = 시스템의 사용가능한 전체 메모리 - Index Cache 크기 - Page Cache 크기 - Heap 크기

> 예제 9.6 : 메모리 설정 검증
>
> 이번 섹션에서 다른 예제를 기준으로, 시스템의 전체 메모리를 64GB라고 하겠습니다. 그리고 이전의 예제에 다뤘던 설정값으로 설정하였다고 하겠습니다.
>
> 그러면,
>
> * \[1\] 시스템의 전체 RAM = 64GB
> * \[2\] OS에 할당하기 원하는 최소 메모리 = 1GB
> * \[3\] Lucene Index Cache 크기 = 1GB
> * \[4\] Page Cache = 42GB
> * \[5\] Heap Space = 16GB
>
> 그럼 실제 검증을 해보겠습니다.
> 실제 OS를 위한 남은 메모리 = 64GB \[1\] - 1GB \[3\] - 42GB \[4\] - 16GB \[5\] = 4GB &gt; 1GB \[2\]
>
> 그럼 우리가 예상했던 OS에 할당하기 원하는 메모리보다 큰 여유분의 메모리가 있으므로 Disk Swapping에 대한 위험을 회피할 수 있습니다.

> 시스템에 대한 성능을 잘 제어하기 위해서는 neo4j.conf 파일에 Page Cache와 Heap 크기 파라미터 설정을 꼭 하시길 권장드립니다. 해당 파라미터가 정의되어있지 않는 경우에는 기동할 때 남은 자원을 가지고 대략적으로 계산하여 설정할 것 입니다.



