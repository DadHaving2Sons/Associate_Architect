# Architecture Styles

분석 범위: `（아키설계） 2.1 이론영상 교안（PDF）.pdf` 1-158페이지. 이 범위는 Architecture Styles 소개부터 Hierarchical Architecture의 Microkernel Pattern 장점까지 포함한다. 강의자료의 카탈로그에는 Distributed와 Deployment 스타일도 등장하지만, 상세 설명은 158페이지 이후이므로 이 문서에서는 다루지 않는다.

## 읽는 법

아키텍처 스타일은 반복적으로 발견되는 소프트웨어 구조의 추상화이며, 이름 붙은 설계 결정의 묶음이다. 강의자료는 스타일을 다음 기준으로 판단한다.

| 기준 | 확인 질문 |
|---|---|
| Elements | 어떤 컴포넌트 타입이 반복해서 등장하는가? |
| Relations | 컴포넌트 사이 연결 방식은 무엇인가? |
| Constraints | 토폴로지와 실행/동작에 어떤 제약이 있는가? |
| Consequences | 어떤 품질속성을 돕고, 어떤 품질속성에 비용을 만드는가? |

점수 범례: `++` 강하게 유리, `+` 유리, `0` 상황 의존 또는 직접 영향 작음, `-` 불리하거나 명확한 트레이드오프 있음.

강의자료에서 강조한 핵심 품질속성은 Availability, Modifiability, Performance, Security, Testability, Usability이다. 아래 표는 과제 비교를 쉽게 하기 위해 Deployability, Integrability, Energy Efficiency, Safety도 함께 평가한다.

| Quality Attribute | 강의자료 기준 요약 |
|---|---|
| Availability | 필요한 시점에 시스템이 지정된 운용 가능 상태에 있는 정도 |
| Modifiability | 시스템 변경을 얼마나 쉽게 수행할 수 있는지 |
| Performance | 이벤트 발생 후 시스템이 특정 동작을 수행하는 데 걸리는 시간 |
| Security | 악의적 행위, 정보 도난, 정보 손실 가능성을 낮추는 능력 |
| Testability | 테스트를 통해 결함을 드러내기 쉬운 정도 |
| Usability | 사용자가 원하는 작업을 쉽게 달성할 수 있는 정도 |

## 빠른 선택 신호

| 상황 | 우선 검토할 스타일 |
|---|---|
| 문제를 순차 단계로 나눌 수 있고 각 단계가 완료된 뒤 다음 단계가 시작된다. | Batch Sequential |
| 앞 단계가 끝나기 전에 뒤 단계가 증분 데이터 처리를 시작할 수 있다. | Pipe and Filter |
| 연속 데이터 스트림을 변환하거나 필터를 독립적으로 재사용하고 싶다. | Pipe and Filter |
| 외부 교란이 있는 환경에서 출력값을 목표값에 가깝게 유지해야 한다. | Process Control |
| 여러 컴포넌트가 공통 데이터 저장소를 통해서만 협력한다. | Shared Repository |
| 여러 지식 소스가 부분 해를 갱신하며 근사적/탐색적 해를 찾는다. | Blackboard |
| 이벤트 발생 즉시 반응해야 하며 이벤트 보관은 핵심이 아니다. | Non-Buffered Event Based |
| 생산자와 소비자를 시간/위치/식별자 관점에서 더 강하게 분리하고 싶다. | Buffered Message Based |
| 하나의 모델을 여러 화면으로 보여주고 UI 교체가 잦다. | MVC |
| 기능을 여러 협력 에이전트의 계층으로 나눌 수 있다. | PAC |
| 작은 프로그램의 흐름을 함수 호출 계층으로 단순하게 표현한다. | Main-Subroutine |
| 동일하거나 유사한 작업을 여러 처리자에게 분배해 신뢰성/성능을 얻는다. | Master-Slave |
| 안정적인 allowed-to-use 관계로 계층별 책임을 나누고 싶다. | Layered |
| 플랫폼 독립성, 실행 환경 추상화, 샌드박싱이 중요하다. | Virtual Machine |
| 최소 핵심 기능 위에 서버/플러그인으로 기능을 확장한다. | Microkernel |

### Data Flow

Data Flow Architecture는 입력 데이터가 일련의 변환 단계를 거쳐 출력으로 바뀌는 구조다. Component는 Data Transformer이고 Connector는 I/O stream, file, buffer 같은 Data Channel이다. 강의자료는 Data Flow 내부에서 Batch Sequential, Pipe and Filter, Process Control을 구분한다.

- Batch Sequential: 독립 프로그램들이 순서대로 실행되고, 데이터는 보통 파일 같은 aggregate 형태로 다음 프로그램에 전달된다. 각 프로그램은 완료된 뒤 다음 프로그램을 시작한다.
- Pipe and Filter: Filter가 독립적으로 증분 변환을 수행하고 Pipe는 단방향, 순서 보존, 데이터 보존 흐름을 제공한다. Active/Passive filter와 Active/Passive pipe 변형이 있다.
- Process Control: Set Point, Manipulated Variable, Input Variable, Controlled Variable을 통해 제어값을 목표값에 가깝게 유지한다. Cruise Control, QoS Control 같은 안정화 문제가 대표적이다.

| Quality Attribute | Batch Sequential - Effects | Batch Sequential - Justification | Pipe and Filter - Effects | Pipe and Filter - Justification | Process Control - Effects | Process Control - Justification |
|---|---|---|---|---|---|---|
| Availability | - | 한 단계가 실패하면 뒤 단계가 모두 지연되고, 강의자료도 limited availability를 약점으로 본다. | 0 | 필터는 분리되어 있지만 한 필터 장애가 전체 흐름을 막을 수 있고, fault tolerance가 낮다. | + | 피드백 제어로 이상 상태를 감지하고 보정해 운용 상태를 유지하기 쉽다. |
| Deployability | 0 | 단계별 교체는 가능하지만 배치 순서, 스케줄, 파일 계약에 의존한다. | + | 필터 단위로 교체하거나 배포하기 쉽고 I/O 계약이 맞으면 조합이 단순하다. | - | 하드웨어, 센서, 액추에이터, 제어 주기에 맞춘 배포 검증이 필요하다. |
| Integrability | + | 파일이나 중간 데이터 형식만 맞추면 단계 연결이 쉽다. | ++ | 표준 입출력 형식과 파이프로 여러 필터를 쉽게 조합할 수 있다. | - | 센서, 액추에이터, 제어 로직의 시간 의존성이 강하다. |
| Energy Efficiency | 0 | 대량 처리는 효율적이지만 중간 저장과 반복 I/O가 발생한다. | + | 스트리밍 처리로 불필요한 중간 저장을 줄일 수 있다. | + | 피드백으로 필요한 만큼만 동작하게 조절할 수 있다. |
| Modifiability | + | I/O 데이터가 맞는 경우 처리 단계를 교체하거나 재사용하기 쉽다. | ++ | 독립 필터를 추가, 제거, 교체하기 쉽고 유지보수/확장에 유리하다. | - | 제어 로직 변경은 안정성, 안전성, 외부 환경 영향까지 재검증해야 한다. |
| Performance | 0 | 대량 배치에는 유리하지만 순차 실행이라 지연 시간이 길고 throughput이 낮아질 수 있다. | + | 필터의 병렬/분산 처리로 처리량을 높일 수 있다. | ++ | 실시간 응답과 빠른 보정을 목표로 설계된다. |
| Safety | 0 | 처리 순서는 명확하지만 오류 발견이 다음 배치까지 늦을 수 있다. | + | 필터별 책임이 분리되어 오류 범위를 제한하기 쉽다. | ++ | 상태 감시와 보정이 가능해 안전 요구에 적합하다. |
| Security | 0 | 구조 자체는 단순하지만 중간 파일 보호와 단계별 권한 관리가 필요하다. | 0 | 연결 통로마다 접근 제어가 필요해 별도 설계에 좌우된다. | - | 제어 명령이나 센서값이 공격받으면 물리적 피해로 이어질 수 있다. |
| Testability | ++ | 입력과 출력이 명확하고 단계가 완료 단위로 나뉘어 재현 테스트가 쉽다. | + | 각 필터를 독립적으로 단위 테스트하기 좋다. | - | 실제 환경, 시간 조건, 외부 교란을 테스트로 재현하기 어렵다. |
| Usability | - | 사용자는 배치 완료까지 기다려야 해서 즉시성이 낮고 non-interactive하다. | 0 | 내부 처리 구조라 사용자 경험은 별도 UI 설계에 달려 있다. | 0 | 주로 내부 제어 구조라 사용자 편의성과 직접 관련은 작다. |

**예시**

- Batch Sequential: 야간 급여 정산 배치, 대용량 데이터 ETL, 컴파일러 단계 처리
- Pipe and Filter: Unix shell pipeline, Apache filters, AI training pipeline
- Process Control: 자동차 크루즈 컨트롤, QoS 제어, 공장 온도/압력 제어

### Data Centered

Data-Centered Architecture는 중앙 데이터 저장소를 공유하고, 주변 소프트웨어 컴포넌트가 저장소를 통해서만 상호작용하는 구조다.

- Shared Repository: Data store는 passive, data client는 active이다. DBMS, 정보시스템, 컴파일러 symbol table/AST 저장소가 예시다.
- Blackboard: Blackboard와 Controller가 active이고 Knowledge Source는 passive이다. Controller가 blackboard 상태를 감시하고 지식 소스 실행을 트리거한다. 음성/이미지 인식, 지식 기반 AI, 이상 탐지처럼 여러 가설을 병렬로 실험하는 문제에 맞다.

| Quality Attribute | Shared Repository - Effects | Shared Repository - Justification | Blackboard - Effects | Blackboard - Justification |
|---|---|---|---|---|
| Availability | - | 중앙 저장소 장애가 전체 시스템 장애로 이어질 수 있다. | - | Blackboard나 Controller가 멈추면 Knowledge Source가 함께 영향을 받는다. |
| Deployability | - | 저장소 스키마 변경이 여러 클라이언트 배포와 연결된다. | + | Knowledge Source는 공통 blackboard 규칙만 맞추면 따로 추가/배포할 수 있다. |
| Integrability | ++ | 여러 컴포넌트가 같은 데이터 모델로 쉽게 연동된다. | ++ | 서로 다른 Knowledge Source가 blackboard를 통해 간접적으로 협력한다. |
| Energy Efficiency | + | 데이터를 한곳에 모아 중복 저장과 중복 처리를 줄일 수 있다. | - | 여러 Knowledge Source가 상태를 반복 확인하고 후보 해를 갱신해 낭비가 생길 수 있다. |
| Modifiability | - | 공통 데이터 구조 변경이 많은 클라이언트에 파급된다. | ++ | 새로운 Knowledge Source를 추가해 기능과 가설 탐색 능력을 확장하기 쉽다. |
| Performance | - | 중앙 저장소에 요청이 몰리면 병목이 생기고, 분산 저장소라면 네트워크 데이터 이동 비용이 든다. | - | Blackboard 접근, Controller 조정, 추론 반복이 성능 부담을 만든다. |
| Safety | + | 단일 진실 원천으로 데이터 일관성과 추적성을 확보하기 쉽다. | 0 | 추론 과정은 추적 가능하지만 실행 순서와 종료 조건이 복잡해질 수 있다. |
| Security | 0 | 중앙 접근 제어는 유리하지만 저장소 공격의 영향 범위가 매우 크다. | - | 여러 Knowledge Source가 공유 지식에 접근해 권한 관리와 공격 표면이 커진다. |
| Testability | - | 테스트 데이터와 공유 상태를 준비하고 격리하기 어렵다. | - | Knowledge Source의 실행 순서, 조합 결과, 종료 시점을 예측하기 어렵다. |
| Usability | 0 | 데이터 구조는 내부 설계라 사용자 편의성은 별도 화면에 좌우된다. | 0 | 문제 해결 품질에는 도움 되지만 사용성 자체를 보장하지 않는다. |

**예시**

- Shared Repository: ERP/CRM의 중앙 데이터베이스, 사내 데이터 웨어하우스, RDBMS 기반 정보시스템
- Blackboard: 음성 인식 시스템, 의료 진단 전문가 시스템, 스마트홈 이상 탐지

### Implicit Invocation

Implicit Invocation은 프로시저를 직접 호출하는 대신, 이벤트 발표나 메시지 발행이 다른 모듈의 처리를 간접적으로 유발하는 구조다. 핵심 목적은 publisher/producer/sender/source와 subscriber/consumer/receiver/target을 느슨하게 결합하는 것이다.

- Non-Buffered Event Based: Event Producer, Event Channel, Event Consumer로 구성되며 channel에는 buffer가 없다. 이벤트는 생성 즉시 소비자에게 전달되어야 하고, 생산자와 소비자는 독립적으로 유지된다.
- Buffered Message Based: Message Producer와 Message Consumer가 queue 또는 pub-sub connector 같은 buffer를 통해 비동기로 연결된다. Point-to-Point는 queue 중심 one-to-one, Publish-Subscribe는 topic/subscription 중심 one-to-many에 가깝다.

| Quality Attribute | Non-Buffered Event Based - Effects | Non-Buffered Event Based - Justification | Buffered Message Based - Effects | Buffered Message Based - Justification |
|---|---|---|---|---|
| Availability | - | 수신자가 없거나 장애 중이면 이벤트가 사라질 수 있고 재처리 근거가 약하다. | ++ | 메시지를 큐에 보관해 소비자 장애 후에도 처리할 수 있다. |
| Deployability | + | 발행자와 구독자를 독립적으로 배포하기 쉽다. | ++ | 생산자와 소비자가 queue/pub-sub connector를 기준으로 독립 배포된다. |
| Integrability | + | 이벤트 계약과 등록 규칙만 맞추면 새로운 listener를 쉽게 연결할 수 있다. | ++ | 표준 메시지와 브로커로 이기종 시스템 연동이 쉽다. |
| Energy Efficiency | + | 저장 공간 없이 즉시 전달해 메시지 보관 비용이 적다. | 0 | 부하 평준화는 가능하지만 queue와 broker 운영 비용이 든다. |
| Modifiability | ++ | 발행자를 바꾸지 않고 listener를 동적으로 등록/해제할 수 있다. | ++ | 라우팅, topic, 소비자를 바꿔도 생산자 영향이 작다. |
| Performance | + | 버퍼 없이 즉시 전달되어 지연 시간이 짧다. | + | 비동기로 부하를 흡수하지만 queue 대기 시간이 추가될 수 있다. |
| Safety | - | 이벤트 손실이나 순서 문제가 상태 오류로 이어질 수 있다. | + | 내구성 있는 메시지, acknowledgement, priority, expiration으로 손실 위험을 줄인다. |
| Security | - | 이벤트 흐름이 분산되어 인증과 감사 범위가 넓어진다. | + | broker에서 인증, 권한, 감사를 집중 관리할 수 있다. |
| Testability | - | listener 응답 순서와 타이밍을 예측하기 어려워 테스트와 디버깅이 어렵다. | + | 메시지를 저장하고 재처리할 수 있어 시나리오 검증이 쉽다. |
| Usability | 0 | 빠른 반응은 가능하지만 사용자 경험은 이벤트 처리 설계에 달려 있다. | + | 오래 걸리는 작업을 백그라운드로 처리해 화면 응답성을 높인다. |

**예시**

- Non-Buffered Event Based: Java Swing/AWT listener, IDE 자동 빌드 트리거, 주식 임계값 알림
- Buffered Message Based: MQTT broker, Kafka 기반 주문 처리, RabbitMQ 기반 이메일 발송 큐

### Interaction-Oriented

Interaction-Oriented Architecture는 사용자 상호작용을 데이터 추상화와 비즈니스 처리에서 분리한다. 강의자료는 Data module, Control module, View presentation module이라는 세 역할을 먼저 설명하고, 이를 MVC와 PAC로 구체화한다.

- MVC: Model은 데이터와 비즈니스 로직을 캡슐화하고 View/Controller에 의존하지 않는다. View는 표시와 입력 인터페이스를 담당하고 application logic을 갖지 않는다. Controller는 입력을 받아 Model 동작으로 변환하고 Model-View 사이 흐름을 관리한다. Document-View, MVP, MVVM 변형도 함께 소개된다.
- PAC: 각 agent가 Presentation, Abstraction, Control 세 부분을 가지며 tree-like hierarchy를 이룬다. Presentation과 Abstraction은 직접 연결되지 않고 Control이 중재한다. 많은 협력 agent를 느슨하게 결합해야 하는 인터랙티브 시스템에 적합하다.

| Quality Attribute | MVC - Effects | MVC - Justification | PAC - Effects | PAC - Justification |
|---|---|---|---|---|
| Availability | 0 | 화면 구조 분리는 도움 되지만 장애 복구를 직접 보장하지는 않는다. | + | Agent 단위로 분리되어 일부 장애를 격리하기 쉽다. |
| Deployability | + | Model, View, Controller를 나누어 UI와 로직 변경을 따로 배포하기 쉽다. | + | 독립 agent 단위로 기능을 추가하거나 배포하기 좋다. |
| Integrability | + | Model과 Controller의 인터페이스를 통해 다른 화면이나 기능과 연결하기 쉽다. | + | Agent 간 Control 인터페이스로 독립 기능을 조합할 수 있다. |
| Energy Efficiency | 0 | change propagation, observer, data binding 비용이 생길 수 있다. | - | 여러 agent와 제어 계층이 통신해 추가 비용이 발생한다. |
| Modifiability | ++ | UI, 입력 처리, 데이터 로직을 분리해 UI 교체와 다중 view 확장이 쉽다. | ++ | Agent별 책임이 분리되어 기능 추가, 교체, 확장이 쉽다. |
| Performance | 0 | 역할 분리와 갱신 전파 비용은 있지만 일반적으로 큰 병목은 아니다. | - | Agent 간 조정과 메시지 전달이 응답 시간을 늘릴 수 있다. |
| Safety | 0 | 입력 검증 위치는 명확하지만 안전 기능 자체를 제공하지는 않는다. | + | Agent 경계로 오류 전파를 줄이고 제어 책임을 분리할 수 있다. |
| Security | + | Controller에서 입력 검증과 접근 제어를 집중 적용하기 쉽다. | 0 | Agent별 보안 적용은 가능하지만 통신 경로 관리가 필요하다. |
| Testability | + | Model과 Controller를 View와 분리해 단위 테스트하기 쉽다. MVP/MVVM 변형은 View를 더 수동화해 테스트성을 높일 수 있다. | 0 | 개별 agent는 테스트 가능하지만 상호작용과 계층 통합 테스트는 복잡하다. |
| Usability | ++ | 여러 View, 동기화된 화면, look and feel 교체를 지원해 사용자 경험 개선에 유리하다. | + | 복잡한 인터랙션을 agent별로 나누어 화면 구성이 유연하다. |

**예시**

- MVC: Spring MVC 웹 애플리케이션, Django/Rails 기반 웹 서비스, 다중 화면 편집기
- PAC: 데스크톱 프레젠테이션 도구, 네트워크 트래픽 관리, 모바일 로봇 제어 UI

### Hierarchical - Main-Subroutine / Master-Slave

Hierarchical Architecture는 시스템 전체를 계층 구조로 보고, 하위 모듈이 인접 상위 모듈에 서비스를 제공하는 방식이다. 모듈 간 연결은 명시적 또는 암묵적 method invocation, 즉 call-and-return 관계가 중심이다.

- Main-Subroutine: 기능을 procedure/function 계층으로 분해한다. Connector는 procedure call이고, 중앙집중형 단일 제어 흐름을 갖는다. 작은 프로그램에는 단순하고 테스트가 쉽지만 global shared data와 tight coupling에 취약하다.
- Master-Slave: Master가 replicated service 호출을 구성하고, 여러 Slave 결과를 받아 selection strategy로 결과를 선택한다. Fault tolerance, parallel computing, scalability, accuracy를 얻을 수 있지만 Master가 single point of failure가 될 수 있다.

| Quality Attribute | Main-Subroutine - Effects | Main-Subroutine - Justification | Master-Slave - Effects | Master-Slave - Justification |
|---|---|---|---|---|
| Availability | - | 상위 호출 흐름이 멈추면 하위 기능도 함께 멈추기 쉽다. | 0 | Slave 장애는 대체 가능하지만 Master가 단일 장애점이 될 수 있다. |
| Deployability | - | 기능이 하나의 호출 구조와 배포 단위에 묶여 부분 배포가 어렵다. | + | Slave를 추가하거나 교체해 처리 능력을 단계적으로 배포할 수 있다. |
| Integrability | - | 상위와 하위 모듈이 호출 관계와 공유 데이터로 강하게 연결된다. | 0 | Master-Slave 프로토콜과 selection strategy가 맞아야 하므로 연동 자유도는 제한된다. |
| Energy Efficiency | + | 직접 함수 호출 중심이라 구조적 실행 오버헤드가 작다. | 0 | 병렬 처리로 효율을 높일 수 있지만 여러 Slave 운영 비용과 통신 비용이 든다. |
| Modifiability | - | 호출 계층 변경과 global data 변경이 ripple effect를 만들기 쉽다. | 0 | Slave 교체는 쉽지만 Master의 분배/선택 로직 변경 영향이 크다. |
| Performance | + | 단순한 제어 흐름과 직접 호출로 실행 속도가 빠르다. | ++ | 작업을 여러 Slave에 나누어 병렬 처리 성능을 높일 수 있다. |
| Safety | 0 | 실행 흐름은 예측 가능하지만 오류 격리 장치는 약하다. | + | 여러 Slave 결과를 비교하거나 대체해 오류를 줄일 수 있다. |
| Security | 0 | 구조가 단순해 분석은 쉽지만 별도 보안 메커니즘은 없다. | - | Master가 공격받으면 전체 Slave 제어와 결과 선택이 영향을 받을 수 있다. |
| Testability | + | 각 subroutine의 입력과 출력이 명확해 단위 테스트가 쉽다. | 0 | Slave 단위 테스트는 쉽지만 분산 실행, majority vote, 실패 전환 테스트는 추가 노력이 든다. |
| Usability | 0 | 내부 구현 구조라 사용자 경험에는 직접 영향이 작다. | 0 | 처리 방식은 내부 구조라 사용자 편의성은 별도 UI에 달려 있다. |

**예시**

- Main-Subroutine: 전통적인 C 프로그램의 `main()` 함수 구조, 단순 파일 변환 도구
- Master-Slave: Hadoop MapReduce, Primary-Replica 데이터베이스 복제, 다중 알고리즘 결과 비교

### Hierarchical - Layered / Virtual Machine / Microkernel

- Layered: Layer는 같은 추상화 수준의 컴포넌트 묶음이고, 관계는 allowed-to-use이다. 강의자료는 순환 의존 금지와 최소 2개 layer를 제약으로 둔다. Relaxed layered system은 성능과 유지보수성 사이 타협이고, layering through inheritance는 하위 서비스를 상속으로 수정할 수 있지만 base class 변경 파급이 크다.
- Virtual Machine: 하드웨어나 OS 위에서 Virtual Execution Environment를 제공하는 추상화 계층이다. 가상 CPU, 메모리, I/O 인터페이스, 실행/메모리 관리/JIT 등을 제공하고 Host OS/Hypervisor가 실제 자원을 관리한다.
- Microkernel: 최소 기능의 Core System 위에 Internal Server, External Server, Adapter, Client를 붙여 확장한다. 158페이지까지의 강의자료는 Flexibility/Extensibility, Availability, Portability를 주요 장점으로 설명한다.

| Quality Attribute | Layered - Effects | Layered - Justification | Virtual Machine - Effects | Virtual Machine - Justification | Microkernel - Effects | Microkernel - Justification |
|---|---|---|---|---|---|---|
| Availability | 0 | 하위 계층 장애는 전파되지만 계층별 격리로 영향 범위를 줄일 수 있다. | + | 실행 환경 격리와 복구 기능으로 장애 영향을 줄이기 쉽다. | + | 분산 Microkernel에서는 같은 server를 여러 machine에서 실행해 가용성을 높일 수 있다. |
| Deployability | + | 계층 인터페이스가 안정적이면 계층별 배포, 유지보수, 갱신이 가능하다. | ++ | 동일한 VM 위에서 다양한 환경에 쉽게 배포할 수 있다. | + | Internal/External Server나 plug-in component를 독립적으로 추가하거나 갱신하기 쉽다. |
| Integrability | + | 계층별 표준 인터페이스로 상위 기능을 쉽게 연결할 수 있다. | + | VM API를 기준으로 여러 프로그램을 같은 방식으로 실행할 수 있다. | ++ | Microkernel 통신 메커니즘과 adapter로 다양한 server/client를 연결하기 좋다. |
| Energy Efficiency | - | 여러 계층을 거치며 호출과 데이터 변환 비용이 생긴다. | - | 가상화, 해석 실행, JIT, 자원 스케줄링 비용이 발생할 수 있다. | - | 서비스 간 IPC와 컨텍스트 전환 비용이 반복해서 발생한다. |
| Modifiability | ++ | 특정 계층을 교체해도 인터페이스가 유지되면 영향이 작고 병렬 개발이 쉽다. | + | 플랫폼 차이를 VM이 숨겨 하위 환경 변경 영향을 줄인다. | ++ | 핵심을 작게 유지하고 server를 추가해 기능과 view를 확장하기 쉽다. |
| Performance | - | 계층 수가 많으면 직접 호출보다 지연과 복잡도가 커진다. | - | interpreter nature와 추가 계층 때문에 실행 오버헤드가 발생할 수 있다. | - | 메시지 전달, IPC, server 경유가 성능 병목이 될 수 있다. |
| Safety | + | 계층별 책임과 접근 경로가 명확해 오류 전파를 줄일 수 있다. | ++ | 샌드박스와 격리로 위험한 실행을 제한하기 좋다. | + | Minimal core와 server 격리로 오류 범위를 줄일 수 있다. |
| Security | + | 각 계층이 private information을 숨기고 보안 정책을 특정 계층에 집중 적용하기 쉽다. | ++ | VM 샌드박스가 실행 권한과 자원 접근을 제한할 수 있다. | ++ | 작은 신뢰 기반과 adapter/server 격리로 공격 표면을 줄일 수 있다. |
| Testability | + | 계층별 Mock을 사용해 단위와 통합 테스트를 나누기 쉽다. | + | 동일한 VM 환경에서 테스트를 재현하기 쉽다. | + | 작은 core와 개별 server를 분리해 테스트하기 좋다. |
| Usability | 0 | 내부 구조라 사용자 편의성은 상위 기능 설계에 좌우된다. | + | 사용자에게 일관된 실행 환경을 제공해 사용 혼란을 줄인다. | 0 | 내부 시스템 구조라 최종 사용성에는 직접 영향이 작다. |

**예시**

- Layered: Android Architecture, OSI 7 Layer, 웹 애플리케이션의 Presentation/Business/Data 계층
- Virtual Machine: CLR, JVM, KVM, Docker 기반 microservices 실행 환경
- Microkernel: Eclipse Architecture, Hydra OS 예시, MINIX/QNX 계열 운영체제
