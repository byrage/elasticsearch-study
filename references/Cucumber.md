## Cucumber
https://cucumber.io/docs

- 쿠컴버는 여러가지 언어, 프레임워크를 지원하는데 설치방법은 다르지만 제공하는 기능은 공통적이다.
- 쿠컴버에서 각 "Feature"는 "gherkin"이라는 plain-text english언어(DSL?)로 작성되어 있다.

### Gherkin
#### Gherkin
https://cucumber.io/docs/reference
- 쿠컴버는 '.feature' 파일을 실행하는데, 각 파일은 Gherkin으로 작성된 실행가능한 명세를 가지고 있다.
- Gherkin은 non-programmer도 배우기 쉽게 설계되었다.
```
Feature: Refund item

  Scenario: Jeff returns a faulty microwave
    Given Jeff has bought a microwave for $100
    And he has a receipt
    When he returns the microwave
    Then Jeff should be refunded $100
```
- 걸킨에서 공백이 라인 행은 걸킨 키워드로 시작해야 된다.
- Gherkin Keywords
 - Feature
 - Scenario
 - Given, When, Then, And, But (Steps)
 - Background
 - Scenario Outline
 - Examples
- Extra keywords
 - """ (Doc Strings)
 - | (Data Tables)
 - @ (Tags)
 - \# (Comments)

#### Feature
- '.feature' 파일은 시스템의 단일 기능을 명시한다.
- feature에는 3가지 구성 요소가 있다.
 - keyword
 - name (같은 라인에)
 - (optional) description (무조건 써라)
- cucumber는 name, description은 신경쓰지 않지만 문서화를 위해서 작성하는 것이 좋다.

```
Feature: Refund item

  Sales assistants should be able to refund customers' purchases.
  This is required by the law, and is also essential in order to
  keep customers happy.

  Rules:
  - Customer must present proof of purchase
  - Purchase must be less than 30 days ago
```

#### Descriptions
- 키워드 다음에 아무줄도 없다면 니가 원하는 맘대로 작성해도 된다.

#### Scenario
- 시나리오는 비지니스 규칙을 설명하는 구체적인 예시이다.
- 시나리오를 원하는 만큼 가질 수 있지만, 3-5개의 시나리오를 가지는 것을 추천한다. 더 길어지면 명세나 문서로서의 효력을 잃는다.
- 명세나 문서가 되는 것 이외에도 시나리오는 테스트이다. 너의 시나리오는 시스템의 실행 가능한 명세이다.
- 시나리오의 패턴
 - initial context
 - event
 - expected outcome

#### Steps
- 시나리오는 보통 ** Given**, ** When**, ** Then**으로 시작한다. 만약 여러개의 Given, When 스텝이 있다면 And나 But 키워드를 사용할 수 있다.

#### Given
- 시스템의 초기상태를 묘사. 과거에 일어난 것
- 쿠컴버에서 Given을 실행한다면 시스템의 상태를 설정할 것이다. ex) 객체 생성 및 설정, 테스트 데이터에 데이터 추가

#### When
- 이벤트나 액션을 묘사. 사용자 또는 다른 시스템에 의해 시작된 이벤트
- 시나리오당 하나의 When이 있는 것을 추천함. 여러개가 있어야 될 것 같다면 시나리오를 나눠야 한다는 신호임

#### Then
- 기대하는 결과를 묘사.
- 시스템 작동 후에 결과와 기대 결과를 비교하는데 사용

### Background
- feature 파일 내에 given이 계속 반복 될 경우에 각 시나리오에 적용할 수 있다. -> @Before
```
Background:
  Given a $100 microwave was sold on 2015-11-03
  And today is 2015-11-18
```
#### Scenario Outline
- 값만 다른 여러개의 시나리오를 생성하게 되는 경우, 하나의 시나리오로 생성
- 가독성이 좋다. <$VAR>

```
Scenario: feeding a small suckler cow
  Given the cow weighs 450 kg
  When we calculate the feeding requirements
  Then the energy should be 26500 MJ
  And the protein should be 215 kg

Scenario: feeding a medium suckler cow
  Given the cow weighs 500 kg
  When we calculate the feeding requirements
  Then the energy should be 29500 MJ
  And the protein should be 245 kg

# There are 2 more examples - I'm already bored
```

```
Scenario Outline: feeding a suckler cow
  Given the cow weighs <weight> kg
  When we calculate the feeding requirements
  Then the energy should be <energy> MJ
  And the protein should be <protein> kg

  Examples:
    | weight | energy | protein |
    |    450 |  26500 |     215 |
    |    500 |  29500 |     245 |
    |    575 |  31500 |     255 |
    |    600 |  37000 |     305 |
```

#### Examples
- Scenario Outline에서 Examples를 사용하게 된다.
- 테이블의 헤더는 Scenario Outline의 변수와 일치해야 한다.

#### Scenario Outlines and UI
- 셀레니움같은 자동화 시나리오 아웃라인은 bad price로 여겨진다.
- 시나리오 아웃라인을 사용하는 하나의 이유는 각각의 인풋 파라미터가 동작하는 비즈니스 룰을 검증하기 위함이다.
- UI를 통한 비즈니스 룰 검증은 느리고, 장애포인트 추적이 힘듬

#### Step Arguments
싱글라인에 적합하지 않은 변수 - chunk 데이터나 테이블을 전달할 경우

** Doc String**
```
Given a blog post named "Random" with Markdown body
  """
  Some Title, Eh?
  ===============
  Here is the first paragraph of my blog post. Lorem ipsum dolor sit amet,
  consectetur adipiscing elit.
  """
```
- 오프닝 """의 indentation은 중요하지 않으나, 닫을때는 two space 필수

** Data Tables**
```
Given the following users exist:
  | name   | email              | twitter         |
  | Aslak  | aslak@cucumber.io  | @aslak_hellesoy |
  | Julien | julien@cucumber.io | @jbpros         |
  | Matt   | matt@cucumber.io   | @mattwynne      |
```

#### Tags
- 예약어를 그루핑함. Feature, Scenario, Scenario Outline or Examples
- 공백 허용하지 않음. 태그는 자식 엘리먼트로 상속된다. ex) Feature에 태그를 달면 아래의 모든 시나리오에 태그 적용

#### Step Definitions
- 쿠컴버는 블랙박스로 실행한다. 이때, Gerkin plain-text를 시스템과 상호작용하기 위해서는 스텝 데피니션 기능이 필요하다.
- 쿠컴버가 시나리오 내의 스텝을 실행할때, 걸킨 스텝을 찾아서 실행한다

```
Scenario: Some cukes
  Given I have 48 cukes in my belly
```

```
@Given("I have (\\d+) cukes in my belly")
public void I_have_cukes_in_my_belly(int cukes) {
  System.out.format("Cukes: %n\n", cukes);
}
```

- 쿠컴버가 스텝에 매칭하면, 모든 캡처그룹의 값을 파라미터로 전달함.
- 캡처그룹은 스트링(\d+)이다. 정적언어에서는 자동으로 적절한 타입을 transform하고, 동적언어에서는 타입정보 없이는 기본적으로 transform하지 않는다.
- Cucumber does not differentiate between the five step keywords Given, When, Then, And and But.
- java, cucumber에서 키워드타입은 중요하지 않고, 어노테이션안에 텍스트로 쿠컴버 스텝을 찾고 스텝에서는 파라미터만 받는다.(인풋 파라미터, 결과 파라미터) 이를 자바로 구현함. 한마디로 어노테이션 타입, 쿠컴버 타입은 가독성을 위한것이지 별다른 기능이 없다고 봄. 
