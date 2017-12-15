# 2.2.1 Debian

> 이번 섹션에서는 Debian 또는 Debian기반\(Ubuntu 등\) 환경에서 Neo4j Debian Package를 활용하여 설치하는 방법을 알아보겠습니다.

## 2.2.1.1 설치

Debian에서 Neo4j를 설치하기 위해서는 다음을 확인해야 합니다.

* Java 8 runtime이 설치되었는가?
* Neo4j Debian Package의 Repository 정보가 Package Manager에 등록되어있는가?

### 사전요구사항 \(Ubuntu 14.04와 Debian 8에 한정\)

Neo4j 3.3은 Java 8 버전의 런타임을 요구합니다. Java 8은 Ubuntu 14.04 LTS 버전 또는 Debian 8 \(jessie\) 버전에는 포함되어있지 않기 때문에 Neo4j 3.3을 설치 또는 업그레이드 하기 위해서는 수동으로 설치해야 합니다. Debian 사용자는 일반적으로 [Backports](https://packages.debian.org/jessie-backports/openjdk-8-jdk)에 있는 OpenJDK 8 버전을 설치합니다.

### Debian 8에서 Java 8 설치

`/etc/apt/sources.list.d/` 경로에 `deb http://httpredir.debian.org/debian jessie-backports main`이라는 정보를 가지는 `.list` 확장자로 끝나는 파일을 생성합니다. 그 후 `apt-get update` 명령을 실행합니다.

```
echo "deb http://httpredir.debian.org/debian jessie-backports main" | sudo tee -a /etc/apt/sources.list.d/jessie-backports.list
sudo apt-get update
```

Neo4j 설치를 위한 Java 8을 위하여, 먼저 `ca-certificates-java` 패키지를 설치합니다.

```
sudo apt-get -t jessie-backports install ca-certificates-java
```

만약 설치가 안되어있다면, Java 8을 자동으로 설치할 것이기 때문에 Neo4j 3.3.1 설치를 위한 준비가 끝났습니다. 설치가 끝나고 Neo4j를 시작하기 전에, [설치된 Java 버전 제어하기](#설치된-java-버전-제어하기) 섹션을 참고하여 맞는 Java 버전이 선택되었는지 확인하시기 바랍니다.

### Ubuntu 14.04에서 Java 8 설치

Ubuntu 14.04의 사용자는 webupd8을 통하여 Oracle Java 8 버전을 설치할 수 있습니다. Neo4j를 설치하기 전에 webupd8 또는 다른 PPA를 활용하여 Java 8을 수동으로 설치해야 합니다. 만약 Java 9 버전이 설치되어있다면 Neo4j에 맞지 않아서 위험성이 발생할 수 있습니다.

```
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
```

설치가 끝나고 Neo4j를 시작하기 전에, [설치된 Java 버전 제어하기](#설치된-java-버전-제어하기) 섹션을 참고하여 맞는 Java 버전이 선택되었는지 확인하시기 바랍니다.

### 설치된 Java 버전 제어하기

설치된 Java 버전이 여러개일 때 기본값으로 Java 8 버전을 선택하지 않으면 Neo4j 3.3.1을 실행할 수 없습니다. `update-java-alternatives` 명령을 사용하여 확인하시면 됩니다.

먼저 설치된 모든 Java 버전을 `update-java-alternatives --list` 명령으로 확인하시기 바랍니다.

환경에 따라 상이하겠지만 다음과 비슷한 결과가 나올 것 입니다.

```
java-1.7.0-openjdk-amd64 1071 /usr/lib/jvm/java-1.7.0-openjdk-amd64
java-1.8.0-openjdk-amd64 1069 /usr/lib/jvm/java-1.8.0-openjdk-amd64
```

Java 버전을 확인한 뒤\(위 경우에는  `java-1.8.0-openjdk-amd64`\) 아래 명령에서 `<java8name>` 부분을 바꿔서 실행하여 기본값으로 설정하시면 됩니다.

```
sudo update-java-alternatives --jre --set <java8name>
```

### Repository 추가

Debian 패키지는 [http://debian.neo4j.org](http://debian.neo4j.org) 에서 사용할 수 있습니다. 해당 Repository를 사용하기 위해 아래 명령어를 수행하시면 됩니다.

```
wget -O - https://debian.neo4j.org/neotechnology.gpg.key | sudo apt-key add -
echo 'deb http://debian.neo4j.org/repo stable/' | sudo tee -a /etc/apt/sources.list.d/neo4j.list
sudo apt-get update
```

### 설치

Neo4j Community Edition의 경우에는

```
sudo apt-get install neo4j=3.3.1
```

Neo4j Enterprise Edition의 경우에는

```
sudo apt-get install neo4j-enterprise=3.3.1
```

Neo4j Enterprise Edition을 설치할 때 라이센스 동의 확인이 표시됩니다. 설치 도입부분에 한번의 동의가 있으며, 동의된 정보는 동일한 시스템에서 추후 다시 설치할 경우를 위하여 저장됩니다.

저장된 답변을 지우고 동의절차를 설치 단계에 표시하려면 `debconf-communicate` 명령을 사용하여 답변을 삭제하시면 됩니다.

### 상호작용없는 Neo4j Enterprise Edition 설치

Neo4j Enterprise Edition에서 라이센스 동의는 사용자의 입력을 대기하게 됩니다. 만약 Neo4j Enterprise Edition설치때 사용자입력을 제외시키려면, 라이센스를 읽고 동의한다는 정보를 `debconf-set-selections`를 활용하여 지정해야놔야 합니다.

```
echo "neo4j-enterprise neo4j/question select I ACCEPT" | sudo debconf-set-selections
echo "neo4j-enterprise neo4j/license note" | sudo debconf-set-selections
```

## 2.2.1.2 업그레이드

Neo4j 3.x 버전에서 3.3.1로 업그레이드 하는 경우에는 [챕터 5. 업그레이드](/chapter5.md)를 참고하시기 바랍니다.

아래의 설명은 Neo4j 2.3에서 3.3.1로 업그레이드 하는 방법입니다.

### Neo4j 2.3에서 업그레이드

Neo4j의 Debian/Ubuntu 설치버전을 업그레이드하려면 3단계 절차를 거쳐야 합니다. 첫번째로 설정파일을 통합하고, 데이터베이스를 가져오기 한뒤, 데이터베이스 저장방식을 업그레이드 해야합니다.

### 설정파일

3.3.1 버전에서는 설정파일이 변경되었습니다. 만약 설정파일을 수정하지 않았다면, Debian 패키지가 자동으로 더이상 쓰지 않는 파일은 삭제하고 새로운 기본 설정파일로 변경할 것 입니다.

만약 2.3 버전에서 설정파일을 수정하였다면, 제공되는 설정 통합 도구를 활용하여야 합니다. 두개의 파라미터가 필요한데, 통합할 설정파일이 보관된 conf의 대상과 목적 경로값입니다. Debian 패키지의 파일시스템 구성에 맞게 두가지 모두 제공되어야 합니다.

Debian의 Neo4j에 관련된 파일은 `neo4j` 사용자와 `adm` 그룹에 속해있기 때문에 `sudo` 명령을 사용하여 권한을 변경한 뒤 실행해야 합니다.

```
sudo -u neo4j -g adm java -jar /usr/share/neo4j/bin/tools/2.x-config-migrator.jar /var/lib/neo4j /var/lib/neo4j
```



