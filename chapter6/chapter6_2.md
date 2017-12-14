# 6.2 백업 수행

> Enterprise Edition 전용입니다.
>
> 이번 섹션에서는 Neo4j Database를 어떻게 백업하는지 알아보겠습니다.

## 6.2.1 백업 명령 {#chapter621}

Neo4j 데이터베이스는 `neo4j-admin`의 `backup` 명령을 통해서 온라인 모드로 백업을 실행할 수 있습니다.

### 문법

```shell
neo4j-admin backup --backup-dir=<backup-path> --name=<graph.db-backup>
                    [--from=<address>] [--fallback-to-full[=<true|false>]]
                    [--pagecache=<pagecache>]
                    [--timeout=<timeout>]
                    [--check-consistency[=<true|false>]]
                    [--additional-config=<config-file-path>]
                    [--cc-graph[=<true|false>]]
                    [--cc-indexes[=<true|false>]]
                    [--cc-label-scan-store[=<true|false>]]
                    [--cc-property-owners[=<true|false>]]
                    [--cc-report-dir=<directory>]
```

### 옵션

| 옵션 | 기본값 | 설명 |
| :--- | :--- | :--- |
| --backup-dir |  | 백업된 결과가 저장될 디렉터리 |
| --name |  | 백업의 이름, 만약 동일한 이름이 있는 경우에는 증분백업이 수행됩니다. |
| --from | localhost:6362 | Neo4j의 Host, Port |
| --fallback-to-full | true | 만약 증분백업이 실패할 경우에는 &lt;name&gt;.err.&lt;N&gt;의 경로로 이동하고 전체백업을 수행합니다. |
| --pagecache | 8M | 백업처리에 사용할 Page Cache 사이즈 |
| --timeout | 20m | &lt;time&gt;[ms&vert;s&vert;m&vert;h] 형식의 타입아웃 시간\(기본값은 초\). 이 항목은 일반적으로 Debug 옵션이며, Neo4j Professional Service에서 사용하라 할 경우에만 사용하시면 됩니다. |
| --check-consistency | true | 일관성검사 수행여부 |
| --additional-config |  | 추가적인 설정정보를 위한 파일. 이 항목은 더이상 사용하지 않습니다. |
| --cc-graph | true | 노드, 관계, 속성, 타입, 토큰 간의 일관성검사 수행여부 |
| --cc-indexes | true | 인덱스 일관성검사 수행여부 |
| --cc-label-scan-store | true | Label Scan Store 일관성검사 수행여부 |
| --cc-property-owners | false | 속성에 대한 소유자 일관성검사 수행여부. 이 확인과정은 소요시간 및 메모리사용량이 매우 큽니다. |
| --cc-report-dir | . | 일관성검사 결과가 작성될 디렉터리 |

> **예제 6.1 : 데이터베이스 백업**
>
> 이번 예제에서는 메모리 사용을 명시적으로 정의하여 백업을 실행하겠습니다. Page Cache의 크기는 명령의 `--pagecache` 옵션을 통하여 정의하겠습니다. 추가로, `HEAP_SIZE` 환경변수를 통해서 백업에 사용할 Heap 크기를 정의하겠습니다.
>
> 전체 백업을 수행하겠습니다.
>
> ```
> $neo4j-home> export HEAP_SIZE=2G
> $neo4j-home> mkdir /mnt/backup
> $neo4j-home> bin/neo4j-admin backup --from=192.168.1.34 --backup-dir=/mnt/backup --name=graph.db-backup --pagecache=4G
> Doing full backup...
> 2017-02-01 14:09:09.510+0000 INFO  [o.n.c.s.StoreCopyClient] Copying neostore.nodestore.db.labels
> 2017-02-01 14:09:09.537+0000 INFO  [o.n.c.s.StoreCopyClient] Copied neostore.nodestore.db.labels 8.00 kB
> 2017-02-01 14:09:09.538+0000 INFO  [o.n.c.s.StoreCopyClient] Copying neostore.nodestore.db
> 2017-02-01 14:09:09.540+0000 INFO  [o.n.c.s.StoreCopyClient] Copied neostore.nodestore.db 16.00 kB
> ...
> ...
> ...
> ```
>
> 백업이 완료되면 `graph-db.backup` 이라는 이름의 Neo4j 백업이 `/mnt/backup` 폴더에 생성됨을 볼 수 있습니다.

## 6.2.2 증분 백업 {#chapter622}

증분 백업은 백업에 지정한 백업디렉터리가 존재하고, 백업 이후 트랜잭션 로그가 존재하는 경우에 수행됩니다. 백업 명령은 Neo4j로부터 신규 트랜잭션 로그를 복사하고 백업에 적용합니다. 적용이 완료되면 현재 서버 상태와 동일하게 백업정보가 업데이트 됩니다.

> 예제 6.2 : 증분 백업 수행
>
> 이번 예제에서는 직전에 수행한 전체 백업을 동일하게 다시 수행하겠습니다. 동일하게 메모리 설정도 하겠습니다.
>
> 지정한 경로에 백업이 존재하므로 증분백업을 수행하게 됩니다.
>
> ```
> $neo4j-home> export HEAP_SIZE=2G
> $neo4j-home> bin/neo4j-admin backup --from=192.168.1.34 --backup-dir=/mnt/backup --name=graph.db-backup --fallback-to-full=true --check-consistency=true --pagecache=4G
> Destination is not empty, doing incremental backup...
> Backup complete.
> ```

증분 백업을 진행하는데 만약 기존의 백업이 유효하지 않고, `--fallback-to-full=false` 옵션이 지정된 경우에는 실패하게 됩니다. 요구되는 트랜잭션 로그가 삭제되고, `--fallback-to-full=false` 옵션이 지정된 경우 또한 실패합니다. `--fallback-to-full=true`로 지정해야 증분 백업이 실행될 수 없을 때 전체 백업을 수행함으로 안정성을 확보할 수 있습니다.

`--check-consistency` 옵션을 `true`로 기본 지정하는 것 또한 중요합니다. 만약 빠르게 증분백업이 필요할 경우에는 `--check-consistency=false`와 `--fallback-to-full=false` 옵션을 사용하시면 됩니다.

> 적용되지 않은 트랜잭션을 복사하기 위해서 서버는 트랜잭션 로그에 접근하게 됩니다. 이 로그는 Neo4j에 의하여 관리되며, `dbms.tx_log.rotation.retention_policy`에 정의된 특정 시간 이후에는 자동으로 삭제됩니다. 백업 전략을 구상할 때 증분 백업 간격 사이에 트랜잭션 로그가 유지될 수 있도록 dbms.tx\_log.rotation.retention\_policy를 맞게 설정하는 것이 중요합니다.

## 6.2.3 종료 코드

`neo4j-admin backup` 은 성공이냐 실패냐, 실패에 따른 원인에 따라 다른 코드를 표출하며 종료합니다.

> **테이블 6.1 : Neo4j Admin 백업 종료 코드**
>
> | 코드 | 설명 |
> | :--- | :--- |
> | `0` | 성공 |
> | `1` | 백업 실패 |
> | `2` | 백업은 성공하였으나 일관성 확인 실패 |
> | `3` | 백업은 성공하였으나 일관성 확인에서 불일치 발생 |



