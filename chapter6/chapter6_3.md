# 6.3 백업 복구

> Enterprise Edition 전용입니다.
>
> 이번 섹션에서는 Neo4j 데이터베이스의 백업 복구를 수행하는 방법에 대하여 알아보겠습니다.

## 6.3.1 복구 명령

Neo4j 데이터베이스는 `neo4j-admin`의 `restore` 명령을 통해 복구할 수 있습니다.

### 문법

```
neo4j-admin restore --from=<backup-directory> [--database=<name>] [--force[=<true|false>]
```

### 옵션

| 옵션 | 기본값 | 설명 |
| :--- | :--- | :--- |
| --from |  | 복구할 백업 위치 |
| --database | graph.db | 데이터베이스 이름 |
| --force | false | 기존 데이터베이스 대체 여부 |

## 6.3.2 단일 데이터베이스 환경에서의 복구

> 예제 6.3 단일 데이터베이스 복구
>
> `/mnt/backup/graph.db-backup` 에 있는 백업을 `graph.db` 데이터베이스로 복구하겠습니다. 복구전 필히 Neo4j를 중지시켜야 합니다.
>
> ```
> neo4j-home> bin/neo4j stop
> neo4j-home> bin/neo4j-admin restore --from=/mnt/backup/graph.db-backup --database=graph.db --force
> neo4j-home> bin/neo4j start
> ```

## 6.3.3 Causal Cluster 환경에서의 복구

Causal Cluster 환경에서 복구할 때는 다음 절차를 따릅니다.

1. Cluster의 모든 데이터베이스 인스턴스를 중지시킵니다.
2. [단일 데이터베이스 환경에서의 복구](#632-단일-데이터베이스-환경에서의-복구) 방법에 따라 각각의 인스턴스 모두를 복구합니다.
3. 만약 새로운 하드웨어에서 복구를 진행할 경우, `neo4j.conf`에 있는 Causal Clustering 설정을 확인해야합니다. `causal_clustering.initial_discovery_members`에 복구를 진행할 서버가 반영되어있는지 확인합니다. `causal_clustering.expected_core_cluster_size`에 새로운 설정에 맞는 서버 개수가 반영되어있는지 확인합니다.
4. 데이터베이스 인스턴스를 시작합니다.

## 6.3.4 HA Cluster 환경에서의 복구

HA Cluster 환경에서 복구할 때는 다음 절차를 따릅니다.

1. Cluster의 모든 데이터베이스 인스턴스를 중지시킵니다.
2. [단일 데이터베이스 환경에서의 복구](https://www.gitbook.com/book/iju707/neo4j-operations-manual/edit#)방법에 따라 각각의 인스턴스 모두를 복구합니다.
3. 만약 새로운 하드웨어에서 복구를 진행할 경우, neo4j.conf에 있는 HA 설정을 확인해야합니다. `ha.initial_hosts`에 백업을 반영할 서버정보가 있는지 확인합니다.
4. 데이터베이스 인스턴스를 시작합니다.



