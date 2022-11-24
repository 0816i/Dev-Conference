# Shall We NestJS?

# 1. NestJS 시작하기 - 남병관

## 오늘 할 이야기

- Controllers, Provider, Modules, Pipes
- nestJS로 사이드프로젝트를 가볍게 시작하기

## 1. Nest.js 시작하기

누구나 손쉽게 서버 아키텍처를 구축할 수 있도록 하자! (이전엔 Node진영에서 아키텍처에 대한 문제를 풀어낸 적이 없다. 그래서 우리가 함.)
source, controller, module, service라는 기본 아키텍처를 제공함.

> nest g resource sample
> 

이것 만으로도 기본적인 dto, entity, controller, module, service를 다 만들어줌

### Controller

우리가 아는 **그 컨트롤러**

- 각 HTTP 메서드별 데코레이터를 제공함
- @Param, @Body를 통해 request 객체의 정보 읽기를 가능하도록 함
- 이 외에도 다양한 데코레이터를 제공함

### Provider

**의존성 주입**의 대상

- **의존성 주입이란?**
    - 컨트롤러는 서비스에 의존하고 있고 구체화된 명세를 모름!
    - 생성자에 직접 서비스 객체를 생성해주고 있다면, 객체가 바뀔 때마다 코드가 수정되어야함.
    - 추상과 구현의 분리를 통해 유연한 설계가 가능하도록 함.
    - Next에서는 `@Inject()` 데코레이터를 사용함.
    - Module에서 어떤 것을 주입할 지를 결정할 수 있음! 즉, 클래스가 바뀐다고 컨트롤러를 수정할 필요 없음!
- 프로바이더를 선언하는 방법
    1. `@Injectable()` 라는 데코레이터를 통해 프로바이더를 관리하고 있음.
    2. Module에서 Provider로 선언해야됨

### Module

쪼개기 위한 것!

imports, providers, controllers, exports를 제공함

- `imports`
    - 의존하고 있는 다른 모듈을 이곳에 선언함
- `controllers`
    - 밖으로 노출할 것
- `providers`
    - 이 모듈에서 의존성을 주입할 대상들
- `exports`
    - 다른 모듈에게 제공할 것

루트모듈로서 `AppModule` 이 존재하며 각각 하위모듈을 import함으로써 사용함.

`@Global()` 데코레이터를 통해 글로벌 프로바이더를 만들 수 있음. 다만, 의존관계가 복잡해질 수 있기 때문에 제한적으로 사용해야 함.

### Pipe

입력의 형태를 **변환**하거나 **검증**을 하기 위한 것

기본적으로 Nest가 제공하는 Pipe가 존재함.

`@Param()` 의 두번째 인자로 주입하는 등의 방식으로 검증할 수 있음

그리고 커스텀으로 만들 수 있음. → `class-validator` 와 결합할 수 있음

`PipeTransform`을 상속받아 `transform`을 오버라이딩해서 구현할 수 있음.

## 2. 기술스택을 교체하며 Nest.js를 선택한 이유

- 기존 스택의 문제점 (feat. Flask)
    1. 내부 의존성의 복잡함 → 코드에 따라 복잡도가 지수적으로 늘어남
- 고민한 해결 사항
    1. 로그 스케일, 적어도 선형적인 복잡도의 증가를 지향하자 → 그래프를 트리로 바꾼다면?
    → 답은 클린아키텍처다!
- 시작한 방법
    1. 의존성 역전! → hexagonal 아키텍처를 사용하자
- 구현체의 구상
    1. 의존성 주입을 기본적으로 사용할 수 있어야 함
        - ~~Spring~~ ⇒ 사람이 없어서 힘들다, 언어 교육이 힘들다, 스프링은 너무 크다
        - nest ⇒ 의존성 역전의 도구를 내장, 타입 언어 생태계

## 3. Q&A

- 졸라 많아서 내용 줄임
    1. Inject데코레이터 유무의 차이
        - Inject는 실제로 주입할 대상을 결정함. 자료형만 지정해도 넣어주긴 하나, 내가 의존하는 대상을 수기로 지정할 수 있음
    2. DB랑 ORM은 뭐쓰나
        - mongoDB, mongoose를 쓰고 있다.
        - mongoDB → 테이블구조에 종속될 필요가 없어서 사용하고 있다
    3. fastapi를 DI를 프레임워크 레벨에서 지원하는데 nest를 선택한 이유는?
        - fastAPI를 써봤지만, 문제점
            1. 언어가 타입을 강제하지 않는다! (타입언어를 쓰고 싶다)
            2. 언어자체의 비동기성
    4. 아직 남아있는 기술부채가 있다면?
        - MSA에 대한 대응 등
    5. 의존성 주입과 import의 차이
        - 의존성 주입: 나는 추상적으로 객체를 주입받겠고 구현은 생각 안할꺼다.
        - import: 내가 이걸 쓰겠다는 선언
    6. DI와 IoC의 차이점
        - IoC: 제어의 역전, 내가 의존하고 있는 객체들에 대해 신경쓰지 않겠다
        - Di: IoC를 구현하기 위한 하나의 기법
    7. 모듈을 사용하는 이유? 
        - 개발자의 명세를 위함
        - nestJS에서는 모듈간의 토플로지를 그리게 되기 때문에 쓸 일이 없는건 빼두자.
    8. NestJS를 공부하는 방법
        - 만들어보자!
        - NestJS korea에서 하자..
        - [NestJS로 배우는 백엔드 프로그래밍 - WikiDocs](https://wikidocs.net/book/7059) (feat. Dexter-한용재)
        - [Awesome Nestjs (awesomeopensource.com)](https://awesomeopensource.com/project/nestjs/awesome-nestjs)
    9. 파이썬도 모듈을 탑다운으로 관리하는데 왜 문제가 생겼냐
    - 플라스크의 문제는 아니고, 프로그래머의 문제다.
    1. 인터페이스 Nestjs DI
    2. NestJS로 트래픽
        1. MAU 50만 나오는데, 잘 된다.
    3. 쌩초보 Nest 공부
        - 오피션 비추, 저 위키독스 함 보셈;;
    4. 팀스파르타의 서비스 구성
        1. B2C로는 좀 많다.
    5. 테스트코드의 작성
        - 두번째 강의를 보자
    6. nest를 선택한 회사
        - 네카라쿠배당토, 모두싸인, 팀스파르타.. etc
    7. Spring에 익숙한 사람들이 있었다면 nest를 썼을거냐?
        1. 당연히 수푸링쓴다;;
    8. 개발 중 참고한 거
        - 참고할 만한 것이 없고 직접 nest안에 까보고 열심히 써봤음..
    9. 프로젝트 구조는 spring과 비슷하게 하셨나요?
        - hexagonal아키텍처를 채택함
    10. 채용 풀에서 nestjs를 할 줄 아는 사람이 있냐?
        - 흠,, 잘 몰?루, 다른 언어 잘하면 뽑아갈 것 같음.
    11. nest의 배포
        1. dist형태로 js배포 가능하도록 배포한다.
    12. nest의 단점?
        - 스스로 만들어 가야 할 부분이 많은 것 같다.
    13. NestJS로 MSA 모노레포 운영
        - 모노레포를 검토하고는 있음.

# 2. Angular 프론트엔드 개발자가 본 Nest.js - 이상훈

## 1. Nest.js로 전환

처음엔, 장고와 Angular로 프로젝트를 시작했다.

- 장고의 단점
    1. JSON 데이터 처리, Serializer 지옥
    2. 개발속도가 느림
    3. 빈약한 코드 어시스턴트와 인텔리센스
    4. 인터페이스, 클래스, DTO의 중복
- 찾아본 프레임워크들의 단점.. (난 타입스크립트를 쓰겠다.)
    1. 너무 높은 자유도
    2. 미들웨어, 로깅, 데이터베이스 통합 등 하위 레이어 구현
    3. Angular로 개발하다보니 Node.js 프레임워크들이 아쉽다.
    4. Node.js계의 스프링을 찾고 있었다.
    - 답은.. NEST다.

## 2. Angular와 Nest.js 궁합

1. 타입스크립트를 채용해서 백엔드 프론트 코드의 재사용이 가능함.
    - 기타 DTO, Interface 등등..
2. 도메인이 비슷해서 개발자가 접근하기 매우 편함.

## 3. Nest.js를 이루는 구성요소

- Module
    1. 앵귤러의 모듈
        - **Declarations → 요거 백엔드에선 필요없다.**
            - View를 구성하는 컴포넌트의 선언
        - Imports
            - 다른 모듈을 import
        - Providers
            - DI 대상의 선언
        - Exports
            - 다른 모듈에게 export
    2. Nest의 모듈
        - Controllers
            - 프론트와 연결하기 위한 인터페이스
        - Imports
        - Providers
            - → Injectable을 통해 선언하는 것도 동일한 방식임.
        - Exports
- Guard
    1. at 앵귤러
        - 페이지가 이동될 때 조건을 걸 수 있음 (인증 받았냐)
    2. Nest에서
        - AuthGuard등을 통해 api요청이 guard를 거치도록 해서 인증할 수 있음.
- Interceptor
→ 요청을 가로챔
    1. Angular
        - 백으로 갈 때, 인증 토큰 등을 검증
    2. Nest
        - 백으로 들어올 때 인증 토큰 등을 검증
- Pipe
    1. Angular
        - 각각의 객체를 유효한 값으로 바꿔줄 수 있음
    2. Nest
        - 들어오는 값을 유효한 객체로 바꿔줄 수 있음
        - bootstrap에서 전역 파이프를 설정할 수 있음
- Nest 요청에서 응답까지
    
    미들웨어 → 가드 → 인터셉터 → 파이프 → 라우트 핸들러 → 컨트롤러
    

### 4. Q&A

1. 모노레포로 관리하나요?
    - 프로젝트는 별도로 관리하고 있습니다. 레포지토리만 모노입니다.

# 3. Nest.js에서 Hexagonal Architecture 구현하기

## 1. Hexagonal Architecture란?

### 클린아키텍처

![좌: 더러운 아키텍처, 우: 클린아키텍처](assets/Untitled.png)

좌: 더러운 아키텍처, 우: 클린아키텍처

왼쪽의 모습에서 컴퍼스를 바꿔볼까? 라고 한다면 세상이 망한다. 우측처럼 단방향의 의존성 방향을 지향하는 구조가 좋다.

### 헥사고날 아키텍처

![좌: 헥사고날 아키텍처, 우: 클린 아키텍처](assets/Untitled%201.png)

좌: 헥사고날 아키텍처, 우: 클린 아키텍처

![레이어드 아키텍처에서 영속성 계층으로 향하는 의존성을 역전!](assets/Untitled%202.png)

레이어드 아키텍처에서 영속성 계층으로 향하는 의존성을 역전!

도메인이 인프라스트럭처에 의존하기 때문에 문제가 생김!

- 따라서 내부와 외부를 구분하라!
    - 내부: 도메인 계층으로 순수 비즈니스 로직이 캡슐화
    - 외부: 기존 아키텍처에서 도메인을 분리한 나머지

다른 말로 **포트/어댑터 패턴**으로도 말함.

![Untitled](assets/Untitled%203.png)

따라서 좌측 모습과 같이 영속성 계층을 분리하거나 테스트하거나 도메인 모델의 수정이 쉬움.

그리고, 헥사고날(6)에 대해 큰 의미가 있지는 않음.

## 2. Nest.js에서 Hexagonal Architecture를 적용하려면?

> Port는 인터페이스이다.
> 

내부 영역이 외부영역에 노출할 유스케이스 선정이 중요함.

> Adapter는 인프라와 포트 사이에 커뮤니케이션을 담당한다.
> 

다른 인프라에 대해 다른 어댑터가 필요함.

### 구체적인 코드를 통해 알아보자.

- 채용서비스를 가정함.
    - recruiter는 이력서를 열람, 저장, 면접제안 요청을 보낼 수 있음
1. 포트를 만들자
    - In-Port: 도메인으로 들어오는 요청에 대한 것
        1. use case에 대한 interface를 작성함
    - Out-Prot: 도메인이 외부로 요청하는 것
        1. 영속성 계층 등에 대해 도메인이 물어보는 것
2. 어댑터를 만들자
    - In-Adapter
        - 실제 인터페이스의 구현이 도메인 내부에 있어야 한다!
        - 컨트롤러에서 포트에 의존성을 주입하여 사용하면 됨.
    - Out-Adapter
        - 어댑터에서 실제 인터페이스 구현이 발생함.
        - 도메인에서 이것을 주입받아 사용함.
3. 그렇다면 서비스계층의 역할은?
    - Out Port의 사용처이며 In Port의 구현체이다.

## 3. 먼저 삽질한 사람의 소소한 제안

1. 도메인을 먼저 작성할 수 있음
2. 이벤트 버스..

## 4. Q&A

- 졸라많아서..
    1. 레이어드 아키텍처와 헥사고날 아키텍처
        - 클린아키텍처의 구현 방법중 하나인 포트-어댑터 아키텍처

# 3. NestJS와 함께한 3년간의 고군분투기

## 1. Hello, nest!

옛날 옛적에 JS와 koa로 개발했었음..
⇒ 구조의 설계와 타입을 사용할 수 있었으면 좋겠다. 객체지향 혜택도 누리고 싶다.

그래서! inversify를 사용해서 해봤다.
→ 어느 정도 객체지향 프로그래밍의 혜택을 누림
→ 그러나 계층화된 DI System을 직접 관리하고 애플리케이션에 필요한 모든 것들을 직접 추가해야 했음.

NestJS로 얻게 된 것
⇒ 3년, 30개의 서비스, 10개의 패키지 개발

## 2. NestJS, 근데 이제 DDD를 곁들인

### DDD란?

![Untitled](assets/Untitled%204.png)

- Value Object: 값 객체
- Entity: app에서 식별 가능한 개체
- Aggregate: 연관있는 Entity와 ValueObject의 집합
- Repository: Aggregate를 영속화 하기 위한 저장소 인터페이스

### 간단한 예제 만들기

User는 Aurora에 Product는 DynamoDB에 저장한다고 가정했을 때,

![Untitled](assets/Untitled%205.png)

![Untitled](assets/Untitled%206.png)

![Untitled](assets/Untitled%207.png)

![Untitled](assets/Untitled%208.png)

![Untitled](assets/Untitled%209.png)

여기까지는 일반적인 방법..

![Untitled](assets/Untitled%2010.png)

![Untitled](assets/Untitled%2011.png)

![Untitled](assets/Untitled%2012.png)

![Untitled](assets/Untitled%2013.png)

![Untitled](assets/Untitled%2014.png)

→ 데이터베이스는 다르지만 구현은 유사하다!.

![Untitled](assets/Untitled%2015.png)

서비스가 실제 DB를 뭘 쓰는지 알 방법은 없다!

**만약에 여기서 DB를 바꾼다면!**

→ 인프라 영역의 구현체만 살짝 바꿔주면 된다.

### Transaction관리는 어디서 할까요?

- 서비스?
    - 서비스에서는 어떤 DB를 쓰는지 몰라야 하기 때문에 DDD계층 규칙을 위반함
- 레포지토리?
    - 정의된 고수준 인터페이스에서는 여러 aggregate를 저장하는 로직을 구현할 때 트랜잭션 전달을 구현할 수 없음.
- 횡단관심사의 대표적인 사례 → aop를 통해 해결할 수 있다!
    - cls-hooked
        - 정해진 context내에서 접근할 수 있는 namespace를 생성하고 관리할 수 있도록 하는 라이브러리
        - 요청단위로 네임스페이스를 만들도록 함. → at 미들웨어에서
        - 이후 Transactional 데코레이터를 통해 transaction안에서 해당 메서드가 실행될 수 있도록 후킹해줌

## 3. SNS, SQS와 마이크로서비스

메시지를 원하는 핸들러로 뿌려주고 싶다.

![Untitled](assets/Untitled%2016.png)

_container : 실제 메시지를 제목과 핸들러로 맵

route: 실제로 핸들러를 실행

![Untitled](assets/Untitled%2017.png)

메타데이터를 통해서 targetEntity, target을 스토리지에 매핑하고 다시 가지고 오는 메서드가 있었음.

![Untitled](assets/Untitled%2018.png)

저 Handler데코레이터에서 set을 호출해서 매핑을 함.

ModulesContainer → Di시스템을 담고있는거

비슷한 원리로 뭐 SQS도 하고 있슴

근데 내 서비스에서 커밋이 안되고 msa발행이되거나 하면 어떡하지?

MessageOutBox테이블에 저장해서 원자성을 띄게 하면 됨.