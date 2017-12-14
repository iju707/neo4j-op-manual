# 2.1 시스템 요구사항

> 이번 섹션에서는 Neo4j 인스턴스에 대한 시스템요구사항을 알아보겠습니다.

## CPU

그래프 규모에 관한 성능은 메모리나 I/O에 영향을 받고, 그래프 자체의 성능은 CPU 성능에 영향을 받습니다.

### 최소요구사항

Intel Core i3

### 권장사항

Intel Core i7

IBM POWER8

## Memory

메모리가 클수록 큰 규모의 그래프를 사용할 수 있습니다. 다만, Garbage Collection 동작으로 인한 성능저하를 피하기 위해서는 속성을 설정할 필요가 있습니다. [9.1 메모리 설정](/chapter9/chapter9_1.md) 부분을 참고하시기 바랍니다.

### 최소요구사항

2GB

### 권장사항

16 ~ 32GB 또는 그이상

## Disk

Disk를 선택할 때 용량외에 Disk의 성능적 특성이 가장중요합니다. Neo4j의 작업은 무작위 읽기\(Random Read\)에 크게 영향을 받습니다. 낮은 평균 탐색시간을 가지는 미디어\(일반 디스크보다 SSD\)를 선택해야합니다.

### 최소요구사항

10GB SATA

### 권장사항

SSD w/SATA

## Filesystem

적절한 ACID 행위를 위해서는 Filesystem은 flush\(fsync, fdatasync와 같은\)를 지원해야합니다. Linux에서 추가적인 성능을 위해 Filesystem을 설정하는 방법은 [9.4 Linux 파일시스템 튜닝](/chapter9/chapter9_4.md)에서 알아보겠습니다.

### 최소요구사항

ext4 \(또는 유사한 시스템\)

### 권장사항

ext4, ZFS

## 소프트웨어

Neo4j는 실행하기 위하여 Java Virtual Machine\(JVM\)이 필요합니다. Windows나 Mac OS용 Community Edition 인스톨러에는 편의성을 위하여 JVM이 포함되어 있습니다. Enterprise Edition을 포함한 다른 배포버전의 경우에는 JVM이 사전에 설치되어있어야 합니다.

### Java

OpenJDK 8 \([Zulu OpenJDK](https://www.azul.com/downloads/zulu/) 또는 [Debian Distribution](http://openjdk.java.net/)\)

[Oracle Java 8](http://www.oracle.com/technetwork/java/javase/downloads/index.html)

[IBM Java 8](http://www.ibm.com/developerworks/java/jdk/)

### 운영체제

Ubuntu 14.04, 16.04

Debian 8, 9

CentOS 6, 7

Fedora, Red Hat, Amazon Linux

Windows Server 2012, 2016

Mac OS

### 아키텍쳐

x86

OpenPOWER \(POWER8\)

