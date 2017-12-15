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

만약 설치가 안되어있다면, Java 8을 자동으로 설치할 것이기 때문에 Neo4j 3.3.1 설치를 위한 준비가 끝났습니다. 설치가 끝난 뒤, "설치된 Java 버전 제어하기" 섹션을 참고하여 맞게 설정한 뒤 Neo4j를 시작할 수 있습니다.

### Ubuntu 14.04에서 Java 8 설치





