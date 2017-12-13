# 9.2 Garbage Collector 튜닝

> 이 섹션에서는 Neo4j의 성능에 관한 Java Virtual Machine의 Garbage Collector의 영향을 알아보겠습니다.

Heap 영역은 Old generation과 Young Generation으로 구분됩니다. 새로운 객체는 Young Generation에 생성이 되고 후에 충분히 사용되었다고 판단되는 경우에 Old Generation으로 이동하게 됩니다. Generation이 가득찰 경우에는 Garbage Collector가 다른 프로세스의 Thread를 모두 정지시키고 수집행동을 수행하게 됩니다. Young Generation은 Young Generation의 크기와 상관없이 유효한 객체의 집합과 연관되어 정지시간동안 빠르게 수집됩니다. Old Generation은 Heap Size에 따라 정지시간이 결정됩니다. 이러한 이유로, 트랜잭션과 쿼리의 상태가 절대 Old Generation으로 변경되지 않도록 Heap 영역이 이상적인 크기와 튜닝이 되어야 합니다.

Heap 크기는 neo4j.conf 파일의 `dbms.memory.heap.max_size` 파라미터에 설정됩니다. Heap의 초기 사이즈는 `dbms.memory.heap.initial_size` 파라미터, `-Xms??m` 플래그 또는 JVM 자체에서 자동\(미정의시\)으로 설정됩니다. JVM에서 필요에 따라 Heap영역을 최대 크기까지 확장시킵니다. Heap 영역의 크기가 확장될 경우에는 Full Garbage Collection이 발생하게 됩니다. 그래서 Heap 영역에 대한 초기값과 최대값을 동일하게 설정하길 권장합니다. 이렇게 되면 Heap 크기 증가에 따른 정지발생을 방지할 수 있습니다.

Heap에 대한 Old Generation, Young Generation의 비율은 `-XX:NewRatio=N` 플래그로 정의할 수 있습니다. `N` 값은 일반적으로 2 ~ 8\(기본값\) 사이에 설정됩니다. 

