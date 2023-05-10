# How SOCAR handles large IoT data with Amazon MSK and Amazon ElastiCache for Redis 읽고나서 간단 정리

## Solution Overview

### Architecture

이 아키텍처는 세 가지 요소로 구성됩니다.

- **Streaming data**

    - `Amazon MSK` 는 스트리밍 데이터를 위한 확장 가능하고 안정적인 플랫폼 역할을 하며, 메시지가 여러 주제와 파티션으로 구성된 `AWS IoT Core` 를 비롯한
      다양한 `Source` 에서
      메시지를 수신하고 저장할 수 있습니다.

- **Consumer application**

    - 소비자 애플리케이션을 통해 사용자는 필요에 따라 데이터 변환 규칙을 정의하는 동시에 `Amazon MSK` 에서 `Target` 데이터베이스 또는 데이터 스토리지로
      데이터를 원활하게 가져올 수 있습니다.

- **Target databases**

    - 소비자 애플리케이션을 통해 `Amazon MSK` 의 데이터를 각각 특정 워크로드를 제공하는 두 개의 개별 데이터베이스로 로드할 수 있었습니다.

### Components of the consumer application

소비자 애플리케이션은 `Amazon MSK` 에서 대상 데이터베이스로 메시지를 `소비`, `변환` 및 `로드` 하기 위해 함께 작동하는 세 가지 주요 부분으로 구성됩니다. 다음 다이어그램은 핸들러 구성 요소의 데이터
변환 예를
보여줍니다.

- **Consumer**

    - `Amazon MSK` 의 메시지를 소비한 다음 메시지를 다운스트림 `Handler` 로 전달합니다

- **Loader**

    - 대상 데이터베이스를 지정합니다. 예를 들어 대상 데이터베이스에는 `ElastiCache(Redis)` 및 `DynamoDB` 가 포함됩니다.

- **Handler**

    - 사용자는 들어오는 메시지를 대상 데이터베이스에 로드하기 전에 `데이터 변환 규칙` 을 적용할 수 있습니다.

### Features of the consumer application

- **Scalability(확장성)**

    - 소비자 애플리케이션은 확장 가능하도록 설계되어, 소비자 애플리케이션이 증가하는 데이터 양을 처리하고 향후 추가 애플리케이션을 수용할 수 있도록 합니다.
    - 예를 들어 약 20,000대의 차량에서 현재 데이터를 처리할 수 있을 뿐만 아니라 비즈니스와 데이터가 계속해서 빠르게 성장함에 따라 더 많은 양의 메시지를 처리할 수 있는 솔루션을
      개발하고자 했습니다.

- **Performance(성능)**

    - 소비자 애플리케이션을 사용하면 `Source` 메시지 및 `Target` 데이터베이스의 양이 증가하더라도 사용자는 일관된 성능을 얻을 수 있습니다.
    - 소비자 애플리케이션은 `멀티 스레딩` 을 지원하여 `동시 데이터 처리` 를 허용하고 컴퓨팅 리소스를 쉽게 늘려 예상치 못한 데이터 볼륨 급증을 처리할 수 있습니다.

- **Flexibility(유연성)**

    - 소비자 애플리케이션은 전체 소비자 애플리케이션을 다시 빌드하지 않고도 새 항목에 재사용할 수 있습니다. 소비자 애플리케이션을 사용하여 `Handler` 에서 다른 구성 값으로 새 메시지를 수집할 수
      있습니다.
    - 다양한 메시지를 수집하기 위해 `여러 Handler` 를 배포했습니다. 또한 이 소비자 애플리케이션을 통해 사용자는 추가 대상 위치를 추가할 수 있습니다.
    - 예를 들어, 처음에 `ElastiCache(Redis)` 전용 솔루션을 개발한 다음 `DynamoDB` 전용 소비자 애플리케이션을 복제했습니다.

### Design considerations of the consumer application

- **Scale out(확장)**

    - 이 솔루션의 핵심 설계 원칙은 `확장성` 입니다.
    - 이를 달성하기 위해 소비자 애플리케이션은 `Amazon Elastic Kubernetes Service(Amazon EKS)` 와 함께 실행됩니다. 사용자가 소비자 애플리케이션을 쉽게 늘리고 복제할 수
      있기 때문입니다.

- **Consumption patterns(소비 패턴)**

    - 데이터를 효율적으로 수신, 저장 및 소비하려면 메시지 및 소비 패턴에 따라 `Kafka` `주제(Topic)` 를 설계하는 것이 중요합니다.
    - 마지막에 사용된 메시지에 따라 메시지는 서로 다른 스키마의 여러 `주제(Topic)` 로 수신될 수 있습니다.
    - 다양한 워크로드에서 사용되는 다양한 주제가 있습니다.

- **Purpose-built database(목적에 맞게 구축된 데이터베이스)**

    - 소비자 애플리케이션은 특정 사용 사례를 기반으로 여러 대상 옵션으로 데이터 로드를 지원합니다.
    - `ElastiCache(Redis)` 에 `실시간 IoT 데이터` 를 저장하여, 실시간 대시보드 및 웹 애플리케이션을 구동하는 동시에 `DynamoDB` 에 `실시간 처리가 필요하지 않은 최근 여행 정보`
      를
      저장했습니다.

## Walkthrough overview

- 이 솔루션의 `Producer` 는 `GPS` 라는 `주제(Topic)` 로 메시지를 보내는 `AWS IoT Core` 입니다.
- 이 솔루션의 `Target` 데이터베이스는 `ElastiCache(Redis)` 입니다.
- `ElastiCache(Redis)` 는 인터넷 규모의 실시간 애플리케이션에 밀리초 미만의 지연 시간을 제공하는 빠른 인 메모리 데이터 스토어입니다.
- `ElastiCache(Redis)` 는 오픈 소스 `Redis` 의 속도, 단순성 및 다용성을 `Amazon` 의 관리 용이성, 보안 및 확장성과 결합하여 가장 까다로운 실시간 애플리케이션을 지원합니다.
- `Target` 위치는 사용 사례 및 워크로드에 따라 다른 데이터베이스 또는 스토리지가 될 수 있습니다.
- `Amazon EKS` 를 사용하여 컨테이너화된 솔루션을 운영하여 확장성, 성능 및 유연성을 달성합니다.
- `Amazon EKS` 는 컨테이너 예약, 애플리케이션 가용성 관리, 클러스터 데이터 저장 및 기타 주요 작업을 담당하는 `Kubernetes` 제어 플레인 노드의 가용성과 확장성을 자동으로 관리합니다.

## Reference Document

- [How SOCAR handles large IoT data with Amazon MSK and Amazon ElastiCache for Redis](https://aws.amazon.com/blogs/big-data/how-socar-handles-large-iot-data-with-amazon-msk-and-amazon-elasticache-for-redis/)
