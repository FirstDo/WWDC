## ✏️ Data Flow Through SwiftUI

### 2 Principles
1. every time you read a piece of data in your view, you're creating a dependency for that view.
View에서 데이터를 읽을 때마다, 해당 View에 대한 의존성을 생성한다. (데이터가 변경될 때마다 새 값을 반영하기 위해 View를 변경해야 하기 때문)

![](https://velog.velcdn.com/images/marisol/post/07809ca8-0644-4158-a402-70caf3137f24/image.png)

파란색 PlayerView가 보라색 데이터를 읽어야 한다고 할 때,
보라색 값이 변경될 때마다 View를 업데이트 해야 한다.

SwiftUI는 수동 동기화 또는 무효화의 과정이 없기 때문에, SwiftUI로 몇가지 tool을 사용하여 프레임워크에 대한 의존성을 간단히 선언하면 프레임워크가 나머지를 모두 처리하고, 사용자에게 최상의 환경을 제공하는 데에 집중할 수 있다.

2. 뷰 계층 구조에서 사용하는 모든 데이터가 source of truth를 가지고 있다. 
source of truth가 어디에 있든, 항상 하나의 source of truth를 가져야 한다. source of truth가 중복되면 버그와 불일치가 발생할 수 있기 때문에 동기화를 유지할 때 항상 주의해야 한다.
![](https://velog.velcdn.com/images/marisol/post/e41bb7ae-0de8-452a-9866-1c9ad02ae518/image.png)
![](https://velog.velcdn.com/images/marisol/post/0a46b53f-ccf0-49d8-84a6-4f2801eb5236/image.png)

코드 예시
![](https://velog.velcdn.com/images/marisol/post/96b6fb47-60fd-453c-b6d0-01c7258ecbf7/image.png)

"Cannot use mutating member on immutable value: 'self' is immutable" 오류 발생
![](https://velog.velcdn.com/images/marisol/post/0f1e550d-418d-4302-bd50-831ab2038222/image.png)

UI가 업데이트될 때마다 뷰의 body가 달라지는데, 이런 경우를 처리하기 위해 @State를 사용한다.
isPlaying 프로퍼티를 @State 프로퍼티 래퍼를 사용함으로써, 시스템에게 isPlaying이 계속 변할 수 있는 값이며, PlayerView가 isPlaying에 의존하고 있다는 것을 알려줄 수 있다.

![](https://velog.velcdn.com/images/marisol/post/b5d97685-06f0-4266-9c57-64661275922e/image.png)

빌드하면 컴파일러 오류가 발생하지 않고, 사용자가 버튼을 누르면 isPlaying의 상태값이 변경되고, 프레임워크는 이 View를 위한 새로운 body를 생성한다.

@State를 private으로 선언하는 것은 좋은 습관인데, 그 뷰에 의해서만 소유되고 관리된다는 것을 강조할 수 있기 때문이다.
어떤 State를 정의하면, 프레임워크는 영구 스토리지를 할당한다.

상태가 변경되고, 그 상태를 소유하는 View의 유효성을 검사하면서, 해당 View의 body와 관련된 모든 하위 항목들이 recompute된다.
하지만 프레임워크가 View를 비교하고, 변경된 것만 다시 렌더링하기 때문에 매우 효율적으로 처리된다.

# "Every @State is a source of truth."
# "Views are a function of state, not a sequence of events."

View는 이벤트의 연속이 아니라, 상태의 기능이다.
기존에는 View 계층을 직접 변경하여 일부 이벤트에 응답한다. (하위 View를 추가하거나, 제거하는 등)
SwiftUI는 View 선언적인 언어로서, 현재의 상태를 고려하여 View를 설명한다.
app을 user와 device 사이의 지속적인 피드백 루프로 생각할 수 있다.

![](https://velog.velcdn.com/images/marisol/post/0ed16f95-5218-4aab-b32a-2f6ea0b8818c/image.png)

모든 것은 유저로부터 시작된다.
유저는 앱과 상호 작용하여 action을 생성한다.
action은 프레임워크에 의해 실행되고, 일부 State를 변형시킨다.
시스템은 State가 변경되었음을 감지하므로, State에 따사 달라지는 View를 업데이트해야 함을 알 수 있다.
이 업데이트는 사용자가 상호 작용할 수 있는 새로운 버전의 UI를 생성한다.

데이터가 항상 단일 방향으로 흐르는 이 모델은, 뷰 업데이트를 예측 가능하고 쉽게 이해할 수 있다.

+ 재생 중일 때와 일시중지 중일 때 타이틀 색을 다르게 하고 싶다면? 코드를 간단하게 추가할 수도 있지만, 뷰에서 의미있는 데이터를 더 작고 재사용 가능한 요소로 그룹화할 수 있다.

![](https://velog.velcdn.com/images/marisol/post/e31b257f-831e-4f37-bdb6-2755223c05ea/image.png)
![](https://velog.velcdn.com/images/marisol/post/a0a275de-007f-4dea-95e3-3f355c99fbd6/image.png)

![](https://velog.velcdn.com/images/marisol/post/8241a72b-5581-4140-8e54-57a2437a7d62/image.png)


PalyButton에도 @State로 isPlaying 변수를 생성해버리면, source of truth가 복제된다.
이럴 때 사용하는 프로퍼티 래퍼 -> **@Binding**

![](https://velog.velcdn.com/images/marisol/post/eb26ed5c-e022-44ab-94fd-b82edb0903d1/image.png)

- 읽고 쓸 수 있지만, 해당 프로퍼티를 소유하지 않으면서도 해당 소스에 대한 명시적인 종속성을 정의한다.
- @State로부터 가져다 사용할 수 있다. (초기값을 제공할 필요가 없다)

![](https://velog.velcdn.com/images/marisol/post/13673a3b-da4b-453e-9b39-0e0811af28c3/image.png)
![](https://velog.velcdn.com/images/marisol/post/cb8dc8d3-285f-4b0d-b0e2-bcf3cc7ea5af/image.png)

PlayerView가 여전히 source of truth인 state를 가지고 있고, PlayButton은 해당 isPlaying에 $ prefix를 사용하여 바인딩해올 수 있다.

![](https://velog.velcdn.com/images/marisol/post/3637b808-1d27-4bb0-8e12-a80affa03cd3/image.png)

UIKit에서는 수동으로 target action을 설정하거나, delegate을 정의해서 유저 인터랙션에 대응해야 하는 여러 뷰를 소유하는 ViewController가 있다.
모델 변화를 관찰하고, 그 event에 대응해야 한다.
값이 변경될 때마다 값을 읽고, 필요한 모든 곳에 설정해야 한다.
ViewController의 목적은 View와 데이터를 동기화 하는 것이다.

이 모든 복잡성을 관리해야 하지만, SwiftUI에서는 그렇지 않다.
👏👏👏👏👏

data dependency를 정의하는 간단한 도구가 있으며, 프레임워크가 나머지를 처리한다. 
### ViewController가 더이상 필요하지 않다.

이 아이디어는 매우 강력하며, 프레임워크 전반에 걸쳐 적용되어 있다.

Toggle, TextField, Slider와 같은 요소에 대한 API를 보면 모두 바인딩 값을 기대한다.
프레임워크는 source of truth가 어디에 있는지 당신이 통제할 수 있게 한다.
데이터를 생성한 후, 해당 정보를 복제하거나 수동으로 동기화하지 않고, 참조를 제공한다.

만약 액션에 애니메이션을 추가하고 싶다면?

![](https://velog.velcdn.com/images/marisol/post/1c721054-27fc-42aa-98d5-be54d67d7fed/image.png)

액션을 withAnimation 블럭으로 감싸주면 된다.

---

## ✏️ Working With External Data

![](https://velog.velcdn.com/images/marisol/post/7a76aaf3-4e9a-4c1b-8881-8e15faca5f6f/image.png)
![](https://velog.velcdn.com/images/marisol/post/fb46fe25-0397-4a06-8d53-8f3e11ebd37d/image.png)

타이머나 알림같은 일부 이벤트는 외부에서 시작된다.
따라서 타이머가 작동하거나, 알림이 수신될 때 프로세스는 거의 동일하다.
어떤 action을 만들고, 어떤 state에 대한 변화를 수행하고, view의 새로운 복사본을 얻고, 다시 사용자에게 렌더링 된다.
SwiftUI에서 이러한 외부 이벤트를 표현하기 위한 단일 추상화를 가지고 있다.
"Publisher"
Publisher는 combine 프레임워크에서 왔으며, Combine은 시간 경과에 따른 값 처리를 위한 통합되고 선언적인 API이다.
(Combine is a unified declarative API for processing values over time.)

현재 시간을 나타내는 State 값과 텍스트를 추가하고,
onReceive 모디파이어를 사용해서 현재 시간이 바뀔 때마다 currentTime을 업데이트하도록 만들었다.

![](https://velog.velcdn.com/images/marisol/post/3faf8546-c312-407e-9f11-f1e835c07625/image.png)

BindableObject는 이미 가지고 있는, 잘 캡슐화된 모델을 사용하는 편리한 방법이다. (=> ObservableObject)

SwiftUI는 데이터의 변화에 어떻게 반응해야 하는지만 알면 된다.

팟캐스트가 모든 기기에서 동기화되기를 원한다면?

![](https://velog.velcdn.com/images/marisol/post/c36ce6e0-9cc4-4359-80cf-21500b1f84ae/image.png)

Model을 팟캐스트 플레이어 뷰에 가져와야 하는데, 
BindableObject 프로토콜을 채택하기만 하면 된다.

![](https://velog.velcdn.com/images/marisol/post/9956d35d-b140-4645-9d53-df43d89afd1d/image.png)

![](https://velog.velcdn.com/images/marisol/post/05b73026-e8f6-485b-813e-8d5f2ad36081/image.png)


BindableObject를 사용하면 Publisher만 제공하면 된다.
Publisher는 데이터에 대한 변경 사항을 나타낸다.
그리고 Publisher는 SwiftUI에게 외부의 변화를 나타내는 단일 추상화이다.

여기서는 "didChange" 프로퍼티의 Publisher를 제공한다. ("PassthroughSubject"는 Publisher)
SwiftUI가 이 Publisher를 구독하기 때문에, 뷰 계층을 업데이트할 시기를 알 수 있다.
그리고 advanced 메서드에서는 모델을 변경할 때 send 메서드를 호출한다.

정확성을 위해 모델이 변경될 때마다 이 작업을 수행하여 뷰 계층을 최신 상태로 유지해야 하며, SwiftUI가 알아서 처리해준다.

BindableObject에 종속성을 생성하는 것은 매우 쉬운데, 
뷰를 만들 때 ObjectBinding 프로퍼티 래퍼를 View의 프로퍼티에 추가한다. (=> ObservedObject)
그런 다음 View를 인스턴스화 할 때는 이미 가지고 있는 모델에 참조를 전달하기만 하면 된다.
이것은 View의 이니셜라이저에 명시적인 의존성을 생성한다.
위와 같이 코드를 작성하면, 프로퍼티래퍼가 있는 각 뷰는 모델에 따라 달라진다.
State와 마찬가지로 ObjectBinding 프로퍼티 래퍼를 사용하여 View에 추가하면, 프레임워크에서 종속성이 있음을 인식한다.
그래서 View를 언제 업데이트할지 자동으로 알아낸다.

### Creating Dependencies Indirectly

BindableObject을 사용하여 전체 앱을 구축할 수는 있지만, 모델을 한 번에 하나씩 이동하는 것은 번거로울 수 있다.
-> 이 때 EnvironmentObject 사용. 계층 구조를 통해 데이터를 간접적으로 전달하는 데 매우 유용하다.

![](https://velog.velcdn.com/images/marisol/post/c60926b2-046a-4bec-899a-fd50612ebdca/image.png)
![](https://velog.velcdn.com/images/marisol/post/5d573aca-88e4-4cf8-a413-d4a5cca43106/image.png)
![](https://velog.velcdn.com/images/marisol/post/bf20a5f5-1f18-4197-963f-5ddf477c8271/image.png)

![](https://velog.velcdn.com/images/marisol/post/f142cbdc-f2c9-44b9-854e-409292c170f7/image.png)

![](https://velog.velcdn.com/images/marisol/post/8f68d700-9132-48ef-8fe3-dfcd34431d52/image.png)


