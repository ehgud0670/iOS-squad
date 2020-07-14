![RIBs](https://user-images.githubusercontent.com/38216027/85385331-2258b600-b57d-11ea-9acc-77e8f5a3901f.png)

# RIBs는 무엇입니까? 

RIBs는 Uber의 크로스 플랫폼 아키텍처 프레임 워크입니다. 이 프레임 워크는 많은 중첩 상태(many nested states)를 포함하는 대규모 모바일 애플리케이션(복잡도가 높은 어플리케이션)을 위해 설계되었습니다.

Uber를 위해 이 프레임 워크를 설계할 때 우리는 다음의 원칙을 준수했습니다.

* 플랫폼 간 협업(Collaboration) 장려 : 앱의 복잡한 부분은 대부분 iOS와 Android에서 비슷합니다. **RIBs는 Android 및 iOS에 대한 유사한 개발 패턴을 제공합니다.** RIBs를 사용함으로서, iOS 및 Android 플랫폼의 엔지니어들은 그들의 기능에 대해, 공동 디자인된 단일한 아키텍처를 공유할 수 있습니다. 
<br>=> 기능을 추상적인 수준에서 의논하는 것이 아니라, 구체적인 부분까지 협업할 수 있습니다.
 
* 전역 상태(Global state) 및 결정들을 최소화한다 : 전역 상태 변경으로 인해 예기치 않은 동작이 발생하며, 엔지니어가 상태의 변경사항으로 인한 전체 영향을 알지 못할 수 있습니다. RIBs는 잘 격리된(well - isolated) 개별 RIB의 깊은 계층 구조 내에서 상태(states)를 캡슐화하도록 하여 전역 상태 문제를 방지합니다. ( 싱글톤을 없앨 수 있다?, Scope을 줄일 수 있다? )

* 테스트 가능성 및 격리(Isolation) : 클래스는 단위 테스트가 쉬워야 하며 격리에 대한 이유가 있어야 합니다. 개별 RIB 클래스에는 별도의 책임(SRP)이 있습니다 (예 : 라우팅, 비즈니스 로직, 뷰 로직, 다른 RIB 클래스 생성). 또한 부모 RIB 로직은 대부분 자식 RIB 로직와 분리(결합도가 낮다)되어 있습니다. 이를 통해 RIB 클래스를 쉽게 테스트하고 독립적으로 추론할 수 있습니다.

* 개발자 생산성을 위한 툴링 : 대단한 아키텍처 패턴을 채택한다고해서 강력한 툴링 없이는 작은 앱를 넘어 확장 할 수 없습니다. **RIB에는 코드 생성, 정적 분석 및 런타임 통합에 대한 IDE 도구가 포함되어 있습니다.** 이 도구는 모두 크고 작은 팀의 개발자 생산성을 향상시킵니다.

* OCP 원칙 : 가능한 경우 개발자는 기존 코드를 수정하지 않고 새로운 기능을 추가할 수 있어야 합니다. RIB를 사용할 때 몇 곳에서 볼 수 있습니다. 예를 들어, **부모 RIB를 거의 변경하지 않고 부모의 종속성이 필요한 복잡한 자식 RIB**를 연결(attach)하거나 작성(build)할 수 있습니다.

* 비즈니스 로직을 중심으로 구조화됨 : 앱의 비즈니스 로직 구조는 UI의 구조(예를 들어, CoCoa Framework를 따라야 한다는지)를 엄격하게 반영할 필요는 없습니다. 예를 들어, 애니메이션 및 뷰 성능을 용이하게 하기 위해 뷰 계층 구조는 RIB 계층 구조보다 얕길 원할 수 있습니다. 또는 단일 기능 RIB가 UI의 다른 위치에 나타나는 세 가지 뷰의 모양을 제어 할 수 있습니다.

* 명시적 계약 : 요구사항들은 컴파일 타임 안전 계약으로 선언되어야 합니다. 클래스 종속성과 순서 종속성이 충족되지 않으면 클래스는 컴파일되지 않아야합니다. 우리는 ReactiveX를 사용하여 순서 의존성을 나타내며, 클래스 의존성을 나타내기 위해, 또 데이터 변수의 생성을 장려하는 많은 DI Scope을 나타내기 위해 DI (type safe dependency injection) 시스템을 사용합니다.

# Parts of a RIB

RIB는 일반적으로 아래 그림과 같은 요소(Element)로 구성되며 모든 요소는 자체 클래스로 구현됩니다.

![ribs (1)](https://user-images.githubusercontent.com/38216027/85391324-f93c2380-b584-11ea-82c6-28792adeec41.png)

## Interactor

Interactor는 비즈니스 로직을 포함합니다. 여기에서 Rx 구독(subscription)을 수행하고, 상태 변경 결정을 내리고, 데이터를 저장할 위치를 결정하고, 어떤 다른 RIB를 자식으로서 attach 해야 하는지 결정합니다.

Interactor가 수행하는 모든 작업은 Interactor의 수명주기에 국한되어야 합니다. Interactor가 활성화된(살아있는) 경우에만 비즈니스 로직이 실행되도록 툴링을 구축했습니다. **이렇게하면 Interactor는 죽어있지만(deactivated) 구독(rx subscriptions)은 여전히 ​​실행되어 비즈니스 로직이나 UI 상태에 대해 원치 않는 업데이트를 유발하는 시나리오를 예방할 수 있습니다.**

## Router

Router는 Interactor로부터 듣고(listens), Interactor의 output을 하위 RIB 연결 및 분리로 변환합니다. 라우터는 다음과 같은 세 가지 간단한 이유로 존재합니다.

* Router는 겸손한 객체로서 Interactor의 복잡한 로직을 더 쉽게 테스트하게 만드는 역할을 합니다, 자식 Interactor를 mock 할 필요없이, 또 mock 한다고 해도 그들의 존재에 신경 쓸 필요가 없이 말이죠.

* Router는 부모 Interactor와 그 자식 Interactor 사이에 추가적인 추상 계층을 생성합니다. 이로 인해 Interactor 간의 동기적인(synchronous) 통신이 조금 더 어려워지고 RIB 간의 직접적인 커플링 대신 반응성 통신의 채택이 권장됩니다.

* Router는 단순하고 반복적인 라우팅 로직을 포함합니다, 그렇지 않으면 Interactor에 의해 구현되어 집니다. 이 boilerPlate 코드를 Router에 두면, Interactor 객체는 작게 유지될 수 있고. RIB에 의해 제공된 핵심 비지니스 로직에 더욱 집중할 수 있다. 

## Builder

빌더의 책임은 **모든 RIB의 구성 요소 클래스**뿐만 아니라 **각 RIB의 자식에 대한 빌더**를 **인스턴스화** 하는 것입니다.

빌더에서 클래스 생성 로직을 분리하는 것은 iOS에서 mockability에 대한 지원을 더해줍니다, 그리고 나머지 RIB 코드가 DI(Dependency Injection) 구현의 세부 사항과 무관하게 만들 수 있습니다. 빌더는 프로젝트에 사용된 **DI 시스템을 인식해야 하는 RIB의 유일한 부분**입니다. 다른 빌더를 구현하면, 다른 DI 메커니즘을 사용하여 프로젝트에서 나머지 RIB 코드를 재사용할 수 있습니다.

## Presenter

Presenter는 비즈니스 모델을 뷰모델로 또는 뷰모델을 비지니스 모델로 변환(translate)하는 상태 비 저장(statelass) 클래스입니다. 그들은 뷰모델 변환(transformation) 테스트를 용이하게 하는데 사용할 수 있습니다. 그러나 종종 이 translation은 너무나 사소하여 헌신적인 Persenter Class의 생성을 보장하진 않습니다. 만약 Presenter를 생략하면, 뷰 모델 translating은 View (Controller) 또는 Interactor의 책임이 됩니다.

## View(Controller)

뷰는 UI를 빌드하고 업데이트 합니다. 여기에는 UI components 들을 인스턴스화 및 레이아웃, 사용자 상호 작용 처리, UI 구성 요소에 데이터 채우기 및 애니메이션이 포함됩니다. 뷰는 가능한 한 "멍청(dumb)"하게 설계되었습니다. **그들은 단지 정보를 표시합니다.** 일반적으로 단위 테스트가 필요한 코드는 포함되어 있지 않습니다.

## Component

Components는 RIB 종속성을 관리하는데 사용됩니다. 이들은 RIB를 구성하는 다른 유닛을 인스턴스화하여 빌더를 지원합니다. Components는 RIB를 build 하는데 필요한 외부 종속성에 대한 접근을 제공할 뿐만 아니라 RIB 자체에 의해 생성된 종속성을 소유하고 다른 RIB으로부터 이들에 대한 접근을 제어(Control)합니다. 부모 RIB의 Component는 일반적으로 자식 RIB의 빌더에 주입되어 자식에게 부모 RIB의 종속성에 대한 접근 권한을 부여합니다.

# State Management

응용 프로그램 상태는 대규모로(largely) 관리되고 RIB 트리에 현재 연결된(attached) RIB에 의해 표시됩니다. 예를 들어, 단순화 된 탑승 공유 앱에서 사용자가 다른 상태를 진행함에 따라 앱은 다음 RIB를 첨부(attach) 및 분리(detach)합니다 (아래 GIF 참조).

![state](https://user-images.githubusercontent.com/38216027/85400433-5dfe7a80-b593-11ea-80a5-7236ce5acdaf.gif)

RIB는 그들의 해당 범위 내에서만 상태(state) 결정을 내립니다. 예를 들어 LoggedIn RIB는 Request, Menu, OnTrip으로 전환하는 것에 대해서만 상태를 결정합니다. LoggedIn RIB은 OnTrip 화면에서 작동하는 방법에 대해서는 결정하지 않습니다.

RIBs를 추가하거나 제거하여 모든 상태를 저장할 수 있는 것은 아닙니다. 예를 들어 사용자의 프로필 설정이 변경되면, RIB가 연결(attached)되거나 분리(detached)되지 않습니다. 일반적으로 이 상태는 세부 사항이 변경 될때 값을 다시 편집(re-emit)하는 불변 모델의 스트림 안에 저장됩니다. 예를 들어, 사용자 이름은 LoggedIn 범위 내에있는 ProfileDataStream에 저장될 수 있습니다. 네트워크 응답만이 스트림에 대한 쓰기 권한을 갖습니다. DI 그래프 아래로 이러한 스트림에 대한 읽기 접근을 제공하는 인터페이스를 전달합니다.

RIB에는 RIB state에 대한 진실의 단일 소스를 강요하는 것은 없습니다(RIB을 여러가지 방법으로 만들수 있다). 이것은 React와 같은 더 많은 의견을 가진 프레임 워크가 이미 기본적으로 제공하는 것과 대조적입니다. 각 RIB의 맥락 내에서 단방향 데이터 흐름을 촉진하는 패턴을 채택하거나 효율적인 플랫폼 애니메이션 프레임 워크의 이점을 활용하기 위해 비즈니스 상태 및 뷰 상태를 일시적으로 분산시킬 수 있습니다.

# Communication Between RIBs

Interactor가 비즈니스 로직을 결정할 때 다른 RIB에게 완료(completion)와 같은 이벤트를 알리고 데이터를 보내야 할수도 있습니다. RIB 프레임 워크에는 RIB 간에 데이터를 전달하는 단일(single) 방법이 포함되어 있지 않습니다. 그럼에도 불구하고 일반적인 패턴을 용이하게하기 위해 만들어졌습니다.

일반적으로 통신이 하위 RIB로 하향 전달되는 경우, 이 정보를 Rx 스트림으로 배출로서 전달합니다. 또는, 데이터가 하위 RIB의 build () 메소드에 매개 변수로 포함될 수 있으며,
이 경우에 매개 변수는 자녀의 수명(lifetime) 동안 변하지 않습니다.

![stream](https://user-images.githubusercontent.com/38216027/85407217-e550eb80-b59d-11ea-96af-6ead1f234224.png)

Example of downwards communication via Rx. Lines denote RIB hierarchy.

통신이 RIB 트리를 상위의 RIB Interactor로 진행하는 경우, 부모가 자식보다 오래 살수 있으므로(outlive) 리스너 인터페이스를 통해 통신이 수행됩니다. 부모 RIB 또는 해당 DI 그래프의 일부 객체는 리스너 인터페이스를 채택하고 자식 RIB가 이를 호출 할 수 있도록 DI 그래프에 배치합니다. 부모가 자녀의 Rx 스트림을 직접 구독하는 대신 이 패턴을 사용하여 데이터를 위쪽으로 전달하면 몇 가지 이점이 있습니다. 메모리 누수를 방지하고, 어떤 child가 부착되어 있는지에 대한 지식 없이 부모를 작성, 테스트 및 유지 관리할 수 ​​있으며(의존성 역전원칙을 통해 가능하다), child RIB를 첨부(attach) / 분리(detach)하는 데 필요한 의식(ceremony)을 줄입니다. child RIB를 이런 방식으로 연결한다면, Rx 스트림 또는 리스너를 등록 취소 / 재 등록 할 필요가 없습니다.

![listener](https://user-images.githubusercontent.com/38216027/85409012-3661df00-b5a0-11ea-9489-e5ff324fe3a9.png)

Example of upwards communication with a listener interface. Lines denote RIB hierarchy.

