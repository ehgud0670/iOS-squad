# Swift Monads, Functors and Applicatives with examples

해당 [블로그][Medium_블로그_링크]를 번역했음을 알려드립니다. 

![monad_image](https://user-images.githubusercontent.com/38216027/87918981-1bf12780-cab2-11ea-862f-1c5db69deae9.png)

함수형 리액티브 프로그래밍을 배우기 시작하는 모든 개발자는 결국 functor, applicative, monad 와 같은 개념을 마주하게 될 것입니다. 처음으로, 비기너들에게는 위의 키워드가 무섭게 들릴수도 있지만, 이 단어들은 전혀 복잡하지 않습니다. 오늘의 수업에서 나는 여러분에게 명령형 프로그래밍과는 전혀 친숙하지 않은 Functor, Applicative, Monad에 대해 단순하게 설명해보도록 하겠습니다. 

## Values in context

명령형 프로그래머라면 기본적으로 속성(프로퍼티)에 익숙합니다. 각 속성에는 type이 있으며 고유한 value가 있습니다. 하지만 함수형 세계에서 이 개념은 약간 다릅니다. 우선, 우리는 그것이 context에서 value가 무엇을 의미하는지 이해해야 합니다. 간단히 하기 위해 var number = 7과 같은 변수를 상상한 다음, 값을 넣을 수있는 상자(box)를 상상해보십시오.

이 상자는 context의 간단한 개념입니다. 다음으로 받아들여야 하는 것은 상자가 값을 가질 수 있거나 비어있을 수(empty) 있다는 것입니다(물리적 세계의 배열 또는 실제 상자처럼). 컨텍스트에 싸인 값이라는 것은 이 값에 직접 도달 할 수 없는 것을 의미하지만, 컨테이너가 비어 있지 않은 경우(즉 값이 있는 경우) 어떻게든 추출(extract)할 수 있습니다. 여기서 복잡한 것은 없습니다. 모든 것이 매우 단순하고 이해하기 쉽습니다.

이 박스 개념은 Swift의 Optional type과 비슷하다는 것을 알 수 있습니다. 그리고 그것은 옳습니다! 상자와 마찬가지로, Optional은 값을 가질 수 있거나 값이 없으면 nil 일 수 있습니다.



### A little spoiler: Optional is a functor and even a monad.



그러나 먼저 해야 할 일이 있습니다. 우리 자신의 Optional type을 만들고, 그 예에서 Functor, Monad 및 Applicative을 다루는 방법을 배웁시다. 추가 실험을 위해 Box\<T> 타입의 generic enum을 정의합니다. some은 값이 있을 때, empty는 box가 비어있을 때입니다.

```swift
// Type Box<T> that represents value in a context
// It can have some value either be empty
enum Box<T> {
    case some(T)
    case empty
}
```



## Functor

### 간단히 말해서 functor는 type이며 map 함수를 구현합니다. Swift의 Optional, Collection 및Result 타입이 이에 해당합니다. 

자, 이제 여러분은 Int 타입의 변수가 7과 같다고 상상해보십시오. 그리고 3을 늘려서 10을 얻으려고합니다. 명령형 패러다임에서 무엇을하십니까? 맞습니다, 당신은 7 + 3을 쓰고 10을 얻습니다. 하지만 7이라는 값이 Context에 쌓여있다면, 어떻게 그냥 3과 더할 수 있겠습니까?

> *Box(7) + 3 -> What? Type Error.*



```swift 
extension Box where T == Int {
    func add(_ value: Int) -> Box<Int> {
        switch self {
        case .some(let t):
            return .some(t + value)
        case .empty:
            return .empty
        }
    }
}

let intBox: Box<Int> = .some(7)
let result: Box<Int> = intBox.add(3)
```



여기서 수행한 작업 : Int 값을 매개 변수로 받아들이고 일부 계산을 수행하고 동일한 유형 Int의 Box를 반환하는 함수를 정의합니다. 꽤 직설적이죠. 이제 이 표현식을 다시 작성하여 보다 generic하게 만들 수 있습니다. 바로 map 함수입니다. 우리가 달성하고자 하는 것 : **시그니처(signiture) (T) -> U 가있는 함수를 map의 매개 변수로 전달**하고 특정 유형과 독립적으로 되게 하는 것입니다. 그것이 우리가 하는 방법입니다.



```swift
// Functor applies a function to a wrapped value
extension Box {
    func map<U>(_ f: @escaping (T) -> U) -> Box<U> {
        switch self {
        case .some(let t):
            return .some(f(t))
        case .empty:
            return .empty
        }
    }
}
```

#### 여기서 우리가 한 것: 

* generic map 함수:  타입 (T) -> U인 함수를 파라미터로 받아들이고 Box\<U>가 리턴타입인

* function body에, 우리는 단순한 switch case를 작성했다. 

* 만약 박스에 값이 들어있다면, 우리는 U 타입의 리턴타입인 f - function을 호출하고, 이후에 U 타입값을 wrap해서 Box\<U> 타입으로 만들고 리턴한다. 
* 만약에 값이 없다면 새 타입 U의 empty case 인 Box\<U>로 리턴합니다.



이제 map-function을 사용하면서, 우리는 우리의 "box"에 T를 파라미터로 받고, 새 U 타입을 반환하는 어떠한 함수도 적용할 수 있습니다. **따라서  context 안에서 포장된(wrapped) 값을 처리할 수 있는 능력이 생긴 겁니다.(이게 포인트, wrapping 된 값을 처리하기 위해 map을 사용한다.)**

>  **A functor applies a function to a value wrapped in a context.**



=> 일례로 Int는 map 함수를 지원할 수 없는 Functor가 아닌 타입이다. (context(x), map(x))



## Applicative

### Applicative는 wrapped 값에 wrapped 함수를 적용합니다. 

```swift
// Applicative applies wrapped function to a wrapped value
extension Box {
    func apply<U>(_ f: Box<((T) -> U)>) -> Box<U> {
        switch f {
        case .some(let transform):
            return self.map(transform)
        case .empty:
            return .empty
        }
    }
}
```



## Monad

### 모나드는 wrapped 값에 wrapped 값을 리턴하는 함수를 적용합니다.

```swift
// Monad applies a function that returns wrapped value to a wrapped value
extension Box {
    func flatMap<U>(_ f: (T) -> Box<U>) -> Box<U> {
        switch self {
        case .some(let t):
            return f(t)
        case .empty:
            return .empty
        }
    }
}
```


[Medium_블로그_링크]: https://medium.com/flawless-app-stories/swift-monad-functor-applicative-806bb34c68c5







