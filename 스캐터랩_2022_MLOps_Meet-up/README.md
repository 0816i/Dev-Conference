# 스캐터랩 2022 MLOps Meet-up

Property: 개인 과업
Status: 완료

# 밋업

- 전제조건
  - DAU 39만 총 5600여만건의 메시지
- ML의 특징
  - ML Code는 큰 비중을 차지하지 않음

## 데이터의 저장과 관리

- 모든 로그는 Cloud Logging과 Big Query로 저장

## 데이터 파이프라인

- Apache Beam을 사용한 설계 및 구현
  - 배치처리, 스트리밍 데이터파이프라인 가능
- GCP Cloud Dataflow를 통해 apache beam의 러너로 사용할 수 있음
  - 하둡 - 매니징이 어려움.
- TFX 파이프라인 내부에서도 beam을 통해 전처리 수행이 가능함
- raw데이터 형태로 쌓여있거나 머신러닝을 위한 데이터셋으로 저장
  - 데이터셋의 버전관리 등 관리의 문제가 생김
  - 사내용 Dataset Registry를 보유함.
    - 중복된 데이터셋의 경우 캐시을 통해 빠른 사용성을 제공함
    - 이름과 버전만으로 데이터셋을 불러올 수 있음.

## 학습과 평가

- 다양한 프레임워크를 사용
  - 연구와 실험 - pytorch
  - 빠른학습과 추론, 성능향상 - Tensor Flow
  - JAX를 사용하고자 하는 움직임 (구글의 새 머신러닝 라이브러리) for pre-training
- 학습에 필요한 가속기 리소스
  - On-premise v100 + 클라우드서비스 GPU와 TPU
  - TPU
    - 대규모 행렬연산이 가능하고 GPU보다 경제적임
    - Tensor, Pytorch, JAX기반 구동 모두 가능
      - JAX > Tensor> pytorch순으로 성능이 나옴
    - 루다팀은 이것으로 효율적인 학습
- 자동화된 학습과 평가
  - Kubeflow Pipelines (Vertex AI Pipelines)
    - pre-training 중에 체크포인트가 떨어지면 Finetuning의 결과수치를 반환
  - 정기적이고 자동화된 TFX사용
    - fetch부터 모델 반환까지 하나의 파이프라인으로 사용
    - 사내 TFX 컴포넌트들을 표준 라이브러리로 정의함 (Dataset, Model)
- 학습 로깅 MLflow를 사용함
  - 실험별 config와 artifact들을 하나의 시스템에서 관리
- 사내 모델 레지스트리에서 모델을 저장하고 관리함
  - Metadata는 github, 실제 파일은 CloudStorage
  - 서빙 형태의 모델들은 즉시 배포 가능하도록 별도 태깅해 저장

## 모델 서빙 최적화

- 이루다는 딥러닝 모델 추론이 전체 리소스시간의 대부분을 차지함
- 추론속도는 빠르게 성능 손실은 낮게
  - pre-training에서 quantization-aware Training으로 학습 (Mixed Precision)
  - 배포시 가속기에 맞춰 컴파일 후 배포
- AWS의 Inferentia
  - AWS의 추론용 ASIC
    - 추론만 빠름, GPU보다 6배 저렴함.
    - EKS 노드로 등록 가능해 Inf 인스턴스를 불러와 사용함.
    - 모델 배포 파이프라인도 Neuron SDK를 사용해 TF/Torch/MXNet기반 모델을 Inf1용으로 변환

## 포스트 스캐터랩

- Continual Learning
  - 대화주제는 아주 빠르게 달라지지만 학습 속도가 이를 따라갈 수 있을까?
    - ex) 범죄도시2에 대한 이야기에 포커스가 “범죄"로 집중됨
  - 방금까지의 세상을 학습해야 더 재밌는 대화상대가 될 수 있음
    - 효율적인 학습을 이어나갈 수 있는 파이프라인
    - Catastrophic Forgetting(과거 학습내용을 잊지 않게)을 최소화할 수 있는 학습 방법론의 구축
- A/B테스트 시스템
  - Accuracy가 높다고 재밌는 대화를 할 수 있는건 아니다.
  - 즉, 챗봇 모델 전체에 대한 A/B테스트가 필요함.
    - 리서치 결과에 대한 빠른 적용이 가능하게

# Apache Beam으로 머신러닝 파이프라인 구축하기

## 루다팀의 Data Proccessing

- 클라우드에서 발생하는 방대한 raw데이터를 PII와 같은 Regulations에 어긋나지 않으면서 Insight를 얻기
- 데이터 프로세싱이란?
  - 다량의 소스로 raw데이터를 가져와 의미있는형태로 변환 후 원하는 저장소에 저장
  - ETL
    - kafka, cloudStorage, S3 에서 Extract
    - hadoop, spark, flink, cloud Data flow 로 Transform
    - BigQuery등으로 Load
  - 배치 프로파일
    - 일정량이 쌓이면 트리거
  - 스트리밍 파이프라인
    - 수집되자마자 처리되는 실시간 Insight
- 학습 데이터셋 전처리 파이프라인
  - Dataset Registry/BigQuery 데이터를 TF레코드로 변환
- 코퍼스 필터링 파이프라인
  - raw 코퍼스를 연구용 공통정제된 데이터 또는 레이블링용 데이터로 변환하여 저장
- 비식별화/가명처리 파이프라인
  - 개인정보의 비식별화

## Apache Beam이란?

- 배치와 스트리밍 데이터 프로세싱 파이프라인을 위한 라이브러리
- Hadoop에 비한 장점
  - 하나의 코드로 Batch와 Streaming 모두를 위한 파이프라인을 작성
  - 하나의 파이프라인에서 여러 언어를 혼용하여 사용 (TFX in JAVA, Kafka In python)
  - 고수준 추상화
    - 분산 처리의 세부 구현을 신경쓰지 않고 작업할 수 있음
  - 우수한 재사용성
    - beamSDK로 파이프라인을 작성하면 Spark, Flink, Dataflow, Managed Service 등에서 실행이 가능함
    - AWS의 Kinesis Data Analytic
    - GCP의 Cloud Dataflow (서버리스 아키텍처)
      - Graph Optimizer
      - Monitoring
      - Centralized Logging
      - Job Summit (작업분배)
      - Dynamic Work Restriction

## Beam 파이프라인 첫걸음

- Beam의 컨셉
  - PCollection
    - 단일 머신, 여러머신에 분산된 Dataset을 In-memory에 올려 하나처럼 쓸 수 있는 추상체
  - PTransform
    - Data Collection에 원하는 비즈니스 로직을 가하기 위한 Transformatino이 추상화된 객체
    - PCollection을 사용하는 함수
    - ParDo
      - Map Reduce의 Map과정과 유사함
      - 하나의 element를 받아 원하는 처리를 하고 0개 이상의 element를 반환
    - Group By Key
      - MapReduce에서의 Shuffle과 유사함
      - 같은 key를 가진 것들끼리 묶어줌
    - Combiner
      - element를 합치는 변환, Reduce와 유사함
      - 분산적으로 합쳐야하기 때문에 연산이 교환법칙과 결합법칙이 만족해야함.
    - Extras
      - Flatten
        - 여러개 컬렉션을 합치기
      - Partition
        - 나누기
      - FlatMap
        - 1:N관계
  - PipeLine
    - 1개 이상의 PCollection에 대한 PTransform의 DAG
      - Node는 PTransform, Edge는 PCollection
    - Deferred Execution
      - 각 컴포넌트가 즉시 실행되는 것이 아닌 JobManager에게 선언형으로 알려줌
      - Optimization이 용이하고 유지보수가 쉬움
- Do It Yourself
  > 단어별 개수를 세서 BigQuery에 저장하세요.
  1. ReadFromText
     - 텍스트파일을 줄별로 읽어옴
  2. Flatten
     - 하나로 병합
  3. FlatMap
     - 각 줄에서 단어들만 찾아 새 컬렉션으로 생성
  4. Map
     - 각 단어를 키로 만들고 1을 할당
  5. CombinePerKey
     - key별로 합을 구함
  6. Map
     - 합과 키를 매핑
  7. WriteToBigQuery

## Beam 파이프라인의 배포

- CI/CD for Data Pipelines
  - Unit Test → Inte Test → Build → Deploy → Valid
  1. DirectRunner로 개발
     - 로컬 python으로 파이프라인 로직 테스트
  2. 유닛테스트
     - TestPipeline, assert-that을 통해 unittest나 pytest를 사용할 수 있음
     - Do Fn의 유닛테스트를 시작으로 Composite Transform의 테스트 진행
  3. DataflowRunner를 통한 통합테스트
     - runner 옵션 수정만으로 간단한 테스트 가능
  4. 파이프라인의 빌드 - TemplateSpecFile
     - 파이프라인을 컨테이너로 빌드하여 Flex Templae로 만들어서 실행환경의 의존성 이슈를 줄임
  5. AirFlow를 통한 배포
     - DataflowStartFlexTemplateOperator를 활용하여 Airflow에서 자동으로 주기적으로 실행

## Beam으로 ML하는 방법

- Beam과 TFX를 활용한 전처리
  - MLops의 전체 사이클을 TF생태계 안에서 하나의 파이프라인으로 구축 가능
  - ExampleGen
    - Beam파이프라인을 통해 TFExample로 변환후 TFRecord로 저장
    - custom exampleGen을 통해 DataSet으로 저장
- Beam을 통한 Inference
  - Beam은 내부적으로 여러개의 Worker Process를 띄우고, 각각의 프로세스는 여러개의 스레드를 한 Dofn에서 띄움
  - shared_handler로 acquire로 하면 하나의 스레드가 Model을 Initialize함. 그 뒤 다른 스레드는 이미 올라간 모델을 불러와서 inference할 수 있음.
  - BatchElements로 여러 element를 하나로 batching하면 GPU Utilization을 높일 수 있음.
- 추가적으로 [blog.pingpong.us](http://blog.pingpong.us) 에서.

## Q&A

- 파이프라인을 여러번 돌릴때 멱등성?
  - beam에서 transform은 여러개를 뱉을 수 있는데, 이때 태깅을 할 수 있음.
  - 에러처리가 중요한 경우 dlt queue를 통해 추후에 다시 처리
- 데이터셋의 버저닝
  - 데이터셋의 넘버링으로 합의

# 챗봇 모델의 A/B 테스트

## 이루다 서비스의 구조

- Nutty → Istio → Elastic Kubernetes Service → (Retriever ,Ranker, Embedding Search Server)
  - 사용자의 발화에 알맞은 답변 후보를 불러오고 알맞은 답변을 찾아 반환함
- 모델의 업데이트 방법
  - 대화 수준을 Metric으로 정의하고 평가한 성능으로 의사 결정 (SSA)
  - 리서치할 때 Metric은 잘 찍혔는데 실제로는 재미없는경우?
- 새로운 업데이트
  - 모델 자체가 변경되는 경우
  - Serving Configuration이 달라진 경우
  - 아예 새로운 구조거나 새 아키텍처가 적용되는 경우

## 사용자 피드백 테스트

- A/B테스트
  - 가설 설정
    - 통계적인 방법을 통해 참 거짓을 판별할 수 있는 명제
    - 귀무가설
      - 차이가 없거나 의미가 없는 경우
    - 대립가설
      - 차이가 분명히 발생한 경우 통상적으로 우리가 추측한 가설
  - 실험 설계
    - 사용자를 실험 그룹으로 분할
    - 평가 지표의 명확한 결정
    - 사용자 표본 추출
    - 실험 집행 기간의 결정 (외부변인의 배제를 위함)
  - 결과 도출
    - 귀무가설의 유의성 검증
      1. 귀무 가설을 기각할 수 있는지 검증함 (실패했는가?)
         - p Value, ~~
         - 귀무 가설을 기각할 수 없음
  - 문제점
    1. 손실에 매우 취약하다
       - B안의 품질이 맞으면 유저들의 크게 이탈
       - 각 시행이 가지는 가치가 클 수록 불리함.
  - 탐색과 활용의 딜레마와 해결방법
    - 지식을 늘리기 위해 새로운 행동을 시도, 알고있는 지식을 바탕으로 최적의 행동
    - 슬롯머신 알고리즘(bandit algorithm)
    - Multi-armed Bandit
      - 하나씩 시도해보면서 어떤게 가장 성공적인지 잡아가는 것
      - Agent: 최적의 솔루션을 찾아야 하는 자
        - Budget: 테스트 피용
      - Environment: Agent가 최적을 찾아야하는 대상
        - Agent가 Environment를 시도하면 Reward를 줌 r(t)함수 t→ 횟수
        - 누적보상
      - Regret: 가장 보상이 큰 경우와 나의 누적보상과 비교 → 이 regret을 최대한 줄임
      - 예시
        - 엡실론 그리디
          - 작은 확률값을 두고 그 확률에 해당하면 탐색, 아니면 활용함
          - 일정한 확률로 탐색하기 때문에 후회값이 큼
        - 소프트맥스 알고리즘
          - 슬롯머신에서 나오는 보상의 평균의 추론
          - 그 값을 활용해 상대적으로 좋은 선택지
        - 톰슨 샘플링
          - 각 슬롯머신의 보상 기댓값의 사후 분포를 추론 (베이즈추론)
          - 가장 큰 보상을 얻을 수 있는 슬롯머신을 선택
  - 쓰임새
    - Multi-armed Bandit - 광고
      - 전환률을 잃음으로써 비용이 너무 큰 경우
      - 촉박한 시간
      - 지속적인 최적화 (A/B대상이 자주바뀜)
      - 트래픽이 낮은 상태
    - NHST - 비즈니스 클로즈
      - 통계학적 유의미함이 강력하게 보장되어야할 경우
      - 실험 이후 그 결과를 수학적 논리로 계산
      - 비즈니스 결정을 위해 명확하게 수치로 산출

## 루다팀의 A/B테스트

- 서비스 자체가 ML
- 일반적인 방법론은 이산적일 경우에 매우 잘 동작함
  - 행위를 이산적으로 표현할 수 있을 때.
  - 하지만 루다팀은 “사용자가 더 좋아할까?”를 검정해야됨
  - → 매번 사용자가 보내는 답장이 만족했다면 실제 대화도 만족스럽지 않을까?
- MAB를 가정한다면?
  1. 사용자가 메시지 전송
  2. 정보에 따라 A 혹은 B로 답장
  3. 로그를 수합하고 라벨링을 통해 루다의 답장이 만족스러웠는지 측정
  4. 해당 값을 Reward로 사용해 MAB를 갱신함
- 대화경험이 만족스럽다라는건 뭐지?
  - 레이블러에게 가이드라인을 통지해야되는데..
  - 루다에 대한 답장이 만족스럽다는게 과연 대화 전체가 만족스럽다는 것일까?
  - 유저의 대화 한건마다 평가기준을 대입하여 평가하긴 어려움.
    - 즉 매 채팅마다 모델이 바뀌지 않고, 느리게 땡긴다면? 그냥 AB테스트 아님?
- 사용자 행동수치에 완전히 의존하자
  - Retention
    - Day N Retention 대화 경험이 좋았다면 사용자는 꾸준히 루다와 얘기할 것
    - 시간 단위 당 주고 받은 턴의 개수
    - 선톡에 대한 사용자의 답장시간 평균
  - 각각의 retention에 곱해지는 weight가 직관에 의해 설정
    - 어뷰징을 통한 통계적 검증의 위해
- 최종 결론
  - 만족스러움의 수치적 정의가 매우 힘듦
    - 즉 Reward에 의존한 모델이 불가능함
  - 가능한 많은 종류의 메트릭으로 대화를 분석하고 사람이 판단
    - 통계에 의존하지 않겠다.

## A/B system의 설계

### 요구사항

- 각 모델에 대한 테스트 시도가 쉽고 직관적
- 테스트 결과를 쉽게 수합해 분석 → 대화 내역에 대한 분석의 용이
- 서비스 백엔드와 결합이 최소화 → 모델의 변경이 Backend에 영향을 끼치면 안됨
- 서비스 안정성

### 설계 사항

- 모델 추론 파이프라인을 분리시킴
  - 차트 두개띄운다? 비용적으로 쉽지 않음.
  - FlowServer에 랭커와 리트리버를 경우에 따라 다르게 호출할 수 있어야함.
- A/B프록시 서버
  - N개의 모델로 사용자를 분산
- Config Storage
  - 실험이 시작되었을 때 실험군 및 사용자 그룹 관련 내용을 저장
  - A/B프록시가 쿠버네티스 API를 통해 변경사항을 감지하고 Hot Reload
- A/B TestSystem
  - 사용자의 명령을 받아 A/B실험을 Provision함

### Q&A

- 라벨링
  - 가이드라인을 선정하는 데 많은 고민을 한다.
  - 레이블러들의 집단지성화
