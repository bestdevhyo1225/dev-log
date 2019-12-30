## Java 빌드 도구 - Maven 그리고 Gradle

<br>

### :book: 개발환경에서 빌드(Build)란?

소스 코드 파일들을 컴퓨터에서 실행할 수 있는 소프트웨어로 변환하는 일련의 과정이다. 그리고 빌드 도구는 아래와 같은 작업을 도와준다.

* 프로그램 빌드

* 프로그램 테스트와 실행

* 라이브러리 관리

* 배포 작업

<br>

### :book: Maven

프로젝트에 필요한 모든 종속성을 리스트 형태로 Maven에게 알려 관리 할 수 있도록 돕는 방식을 말한다.

**`장점`**

* Dependency를 관리, 표준화된 프로젝트 제공

* XML, Remote Repository 를 가져올 수 있다. (개발에 필요한 종속되는 'jar', 'classpath' 를 다운로드 할 필요 없이 선언만으로 사용이 가능)

* 상속형 (하위 XML이 필요 없는 속성도 모두 표기)

**`단점`**

* 라이브러리가 서로 종속할 경우 XML이 복잡해짐

* 계층적인 데이터를 표현하기에 좋지만, 플로우나 조건부 상황을 표현하기엔 어렵다.

* 맞춤하된 로직 실행이 어렵다.

<br>

### :book: Gradle

Groovy 언어 기반의 빌드 도구이며, Groovy는 JVM에서 실행되는 스크립트 언어이다. 따라서 로직을 개발자의 의도에 따라 설계할 수 있다. 초기 프로젝트 설정에 드는 시간을 절약할 수 있으며, 기존의 Maven과 같은 빌드 도구와 호환이 가능하다.

**`정리`**

* 오픈 소스 기반의 Build 자동화 시스템으로 Groovy 기반의 DSL(Domain-Specific Language)로 작성한다.

* Build-by-Convention 을 바탕으로 한다. (스크립트 규모가 작고 읽기 쉽다.)

* Mutli 프로젝트의 빌드를 지원하기 위해 설계 되었다.

* 설정 주입 방식이다. (Configuration Injection)

<br>

### :bookmark: 참고 자료

* [https://jj-one-a-week.blogspot.com/2017/05/ant-maven-gradle.html](https://jj-one-a-week.blogspot.com/2017/05/ant-maven-gradle.html)
