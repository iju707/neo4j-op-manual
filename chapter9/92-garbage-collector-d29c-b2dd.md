# 9.2 Garbage Collector 튜닝

> 이 섹션에서는 Neo4j의 성능에 관한 Java Virtual Machine의 Garbage Collector의 영향을 알아보겠습니다.

Heap 영역은 Old generation과 Young Generation으로 구분됩니다. 새로운 객체는 Young Generation에 생성이 되고 후에 충분히 사용되었다고 판단되는 경우에 Old Generation으로 이동하게 됩니다. Generation이 가득찰 경우에는 Garbage Collector가 다른 프로세스의 Thread를 모두 정지시키고 수집행동을 수행하게 됩니다. Young Generation은 Young Generation의 크기와 상관없이 유효한 객체의 집합과 연관되어 정지시간동안 빠르게 수집됩니다. Old Generation은 Heap Size에 따라 정지시간이 결정됩니다. 이러한 이유로, 트랜잭션과 쿼리의 상태가 절대 Old Generation으로 변경되지 않도록 Heap 영역이 이상적인 크기와 튜닝이 되어야 합니다.

Heap 크기는 neo4j.conf 파일의 `dbms.memory.heap.max_size` 파라미터에 설정됩니다. Heap의 초기 사이즈는 `dbms.memory.heap.initial_size` 파라미터, `-Xms??m` 플래그 또는 JVM 자체에서 자동\(미정의시\)으로 설정됩니다. JVM에서 필요에 따라 Heap영역을 최대 크기까지 확장시킵니다. Heap 영역의 크기가 확장될 경우에는 Full Garbage Collection이 발생하게 됩니다. 그래서 Heap 영역에 대한 초기값과 최대값을 동일하게 설정하길 권장합니다. 이렇게 되면 Heap 크기 증가에 따른 정지발생을 방지할 수 있습니다.

Heap에 대한 Old Generation, Young Generation의 비율은 `-XX:NewRatio=N` 플래그로 정의할 수 있습니다. `N` 값은 일반적으로 2 ~ 8\(기본값\) 사이에 설정됩니다. 비율이 2라는 의미는 Old Generation / New Generation의 값이 2라는 것 입니다. 달리말해, 2/3의 Heap 영역이 Old Generation 영역으로 설정되는 것 입니다. 3일 경우에는 3/4의 Heap 영역이 Old Generation 영역으로 설정되는 것 입니다. 비율이 1일 경우에는 Old와 New 영역이 같은 크기로 설정된다는 것 입니다. 비율이 1인 경우에는 매우 도전적인 설정이지만, 가끔 수많은 데이터가 자주 변경되는 경우 필요하기도 합니다. 매우 큰 New Generation 영역을 가지는 것이 대량의 데이터를 정렬하는 것 처럼 많은 데이터를 유지해야하는 쿼리를 수행할 때 중요합니다. 

만약 New Generation 영역이 너무 작을 경우, 짧은 주기의 객체들은 매우 빠르게 Old Generation 영역으로 이동할 것 입니다. 이것을 Premature Promotion이라고 부르며, 빈번한 Old Generation의 Garbage Collection을 일으켜 데이터베이스를 느리게 만듭니다. 만약 New Generation 영역이 너무 크면, Garbage Collector가 New 영역에서 Old 영역으로 이동하려는 모든 객체를 보관하기에는 Old Generation 영역이 작다고 판단하게 됩니다. 그럼 Old Generation Garbage Collection에서 New Generation Garbage Collection을 발생시키게 되며 역시 데이터베이스가 느려지게 됩니다. 동시다발적으로 Thread를 동작하는 것 주어진 시간동안 더 많은 할당이 발생하게 되어, New Generation 영역에 특히 부담이 증가하게 됩니다.

> JVM의 Compressed OOP 기능은 압축을 활용하여 32bit만 사용하여 객체 참조를 가능하게 합니다. 이 기능은 많은 메모리를 절약하게 하지만, Heap 영역이 32GB 초과할 수 는 없습니다. Heap을 32GB 초과하여 할당하도록 한 것이 증가시킴에도 불가하고 장점이 미미하거나 오히려 단점으로 적용될 수 있습니다.

Neo4j는 Java Process의 수명동안 효율적으로 Old Generation에 보관되는 다량의 장수하는 객체를 보유하고 있습니다. Garbage Collection의 정지시간에 대한 악영향없이 효율적인 프로세스 처리를 위해서는 동시다발적\(Concurrent\) Garbage Collector를 사용하길 권장합니다.

Garbage Collection 알고리즘에 대한 선택은 JVM 버전이나 작업부하량에 따라 결정됩니다. 며칠 또는 몇주 동안이라도 실환경과 유사한 부하가 있는 상태에서 Garbage Collection 설정을 테스트하길 권장합니다. Heap Fragmentaion과 같은 문제가 표면적으로 장기간 표출될 것 입니다.

좋은 성능을 위해서 살펴볼 것이 있습니다.

* JVM이 Garbage Collection을 수행하는데 있어서 많은 시간을 소요하면 안됩니다. 충분히 큰 Heap 영역을 보유해서 무겁거나 최고점의 부하가 발생해도 GC-Trashing이 발생되지 않도록 하는 것 입니다. 성능은 GC-Trashing이 발생하게 되면 배로 감소하게 됩니다. 그렇다고 너무 크게 Heap 영역을 설정하게 되면 역시 성능이 떨어지기 때문에 Heap 영역의 크기를 조정하면서 최적의 값을 찾아야 합니다.
* 동시다발적\(Concurrent\) Garbage Collector를 사용하는 것 입니다. 대부분의 상황에서 `-XX:+UseG1GC` 플래그를 통해 사용할 수 있습니다.
  * Neo4j의 JVM은 트랜젝션 상태와 쿼리 처리 및 Garbage Collector에 사용될 것을 감안하여 충분히 큰 Heap 메모리를 필요로 합니다. Heap 메모리의 요구량은 업무부하에 따라 상이하기 때문에 일반적으로 Heap 메모리를 1GB에서 32GB까지 설정합니다.
* `-server` 플래그와 알맞은 Heap 설정으로 JVM을 실행하는 것 입니다.
  * 전용 서버의 OS는 보통 1 ~ 2GB의 메모리를 사용합니다만, 더 많은 메모리를 보유할 경우에는 더 요구되기도 합니다.

이번 섹션에서 필요한 설정은 다음 파라미터에서 수정할 수 있습니다.

> **표 9.1 : JVM 튜닝에 관련된 neo4j.conf 설정 파라미터**
>
> | 파라미터 명 | 의미 |
> | :--- | :--- |
> | `dbms.memory.heap.initial_size` | 초기 Heap 메모리 크기 \(MB 단위\) |
> | `dbms.memory.heap.max_size` | 최대 Heap 메모리 크기 \(MB 단위\) |
> | `dbms.jvm.additional` | 추가적인 JVM 파라미터 |



