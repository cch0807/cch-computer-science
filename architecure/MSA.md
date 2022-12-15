# Microservice Architecture
크게 아키텍처는 Monolithic Architecture과 Microservice Architecture 등으로 구성된다.
많은 회사들이 Monolithic 아키텍처의 단점으로 인해 MSA 아키텍처로 넘어가기 위해 노력하려 한다.

# MSA의 등장
MSA는 microservice architecture의 약자로서, 하나의 큰 어플리케이션을 여러개의 작은 어플리케이션으로 쪼개어 변경과 조합이 가능하도록 만든 아키텍처이다.

# 기존 Monolithic의 한계 (왜 필요할까?)
Monolithic Architecture은 소프트웨어의 모든 구성요소가 한 프로젝트에 통합되어 있는 서비스이다.
현재 많은 회사들의 소프트웨어가 레거시 또는 필요로 인해서 Monolithic 형태로 구현되어 있다.

소규모의 프로젝트에서는 Monolithic 형태는 간단하며, 유지보수가 편하기 때문에 선호된다.

그러나 일정 규모 이상을 넘어가면 Monolithic은 많은 한계점에 봉착한다.

-