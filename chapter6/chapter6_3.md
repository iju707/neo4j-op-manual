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

> 예제 6.3 단일



