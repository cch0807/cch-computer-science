# 들어가기 앞서서.
MSA 아키텍쳐를 구성하기 어려운 이유 중 하나는 트랜잭션의 문제이다.

기존의 Monolithic 환경에서는 DBMS가 기본적으로 제공해주는 트랜잭션 기능을 통해서 데이터 commit이나 rollback을 통해서 일관성있게 관리하였다. 그러나 Application 과 DB가 분산되면서 해당 트랜잭션 처리를 단일 DBMS에서 제공하는 기능으로는 해결할 수 없다.

# 대안 : Two-Phase Commit?
여러 서비스 간에 데이터 일관성을 유지하기 위해서 전통적인 방법인 Two-Phase commit 과 같은 방법을 사용했다.

다만 이 방법은 하나의 서비스가 장애가 있는 경우나 각각의 서비스에 동시에 Rocking이 걸리게되면 성능의 문제가 발생하기 때문에 비효율적이다. 더 나아가 각각의 서비스가 다른 instance에 있기 때문에 이를 통제하는데 어려움이 있다.

* 트랜잭션이란?
- 데이터베이스의 상태를 변화시키기 위해서 수행하는 작업의 단위를 의미.
- 트랜잭션은 4가지 특성(원자성, 일관성, 독립성, 지속성)을 지켜야한다.

# SAGA 패턴의 정의
- 위의 문제를 해결하기 위해 SAGA 패턴 등장.

SAGA 패턴이란 마이크로서비스들끼리 이벤트를 주고 받아 특정 마이크로서비스에서의 작업이 실패하면 이전까지의 작업이 완료된 마이크서비스들에게 보상 (complemetary) 이벤트를 소싱함으로써 분산 환경에서 원자성(atomicity)을 보장하는 패턴이다.

- SAGA패턴의 이벤트 실패 시는 실패 이벤트를 주어 처리한다.

해당 SAGA 패턴의 핵심은 트랜잭션의 관리주체가 DMBS에 있는 것이 아닌 Application에 있다.
Application이 분산되어 있을 때는 각 Application은 하위에 존재하는 DB는 local 트랜잭션만 담당한다.

즉, 각각의 Application의 트랜잭션 요청의 실패로 인한 Rollback 처리(보상 트랜잭션)은 Application에서 구현한다.

이러한 과정을 통해서 순차적으로 트랜잭션이 처리되며, 마지막 트랜잭션이 끝났을 때 데이터가 완전히 영속되었음을 확인하고 종료된다. 이 방법을 통해서 최종 일관성(Eventually Consistency)를 달성할 수 있다.

# SAGA 패턴의 종류
일반적으로 SAGA 패턴은 크게 2가지로 나뉜다.
하나는 Choreography based SAGA pattern이고 다른 하나는 Orchestration based SAGA pattern 이다.

# Choreography based SAGA pattern
Choreography-based Saga 패턴은 보유한 서비스 내의 Local 트랜잭션을 관리하며 트랜잭션이 종료하게 되면 완료 Event를 발행한다. 만약 그 다음 수행해야할 트랜잭션이 있으면 해당 트랜잭션을 수행해야하는 App으로 이벤트를 보내고, 해당 App은 완료 Event를 수신받고 다음 작업을 진행한다. 이를 순차적으로 수행한다. 이때 Event는 Kafka와 같은 메시지 큐를 통해서 비동기 방식으로 전달할 수 있다.

Choreography-base Saga 패턴에서는 각 App별로 트랜잭션을 관리하는 로직이 있다. 이를 통해서 중간에 트랜잭션이 실패하면 해당 트랜잭션 취소 처리를 실패한 App에서 보상 Event를 발행해서 Rollback 처리를 시도한다.

# 장점
- 구성하기 편하다.

# 단점
- 운영자 입장에서 트랜잭션의 현재 상태를 확인하기 어렵다.

# Orchestration based SAGA pattern
Orchestration-Based Saga 패턴은 트랜잭션 처리를 위해 Saga 인스턴스(Manager)가 별도로 존재한다.
트랜잭션에 관여하는 모든 App은 Manager에 의해 점진적으로 트랜잭션을 수행하며 결과를 Manager에게 전달하게 되고, 비지니스 로직상 마지막 트랜잭션이 끝나면 Manager를 종료해서 전체 트랜잭션 처리를 종료한다. 만약 중간에 실패하게 되면 Manager에서 보상 트랜잭션을 발동하여 일관성을 유지한다.

모든 관리를 Manager가 호출하기 때문에 분산트랜잭션의 중앙 집중화가 이루어진다.

# 장점
- 서비스간의 복잡성이 줄어들어서 구현 및 테스트가 쉬워진다.
- 트랜잭션의 현재 상태를 Manager가 알고 있으므로 롤백을 하기 쉽다.

# 단점
- 관리를 해야하는 Orchestrator 서비스가 추가되어야하기 때문에 인프라 구현이 복잡해진다.
