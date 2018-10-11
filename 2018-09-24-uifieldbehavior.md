---
title: UIFieldBehavior
author: Jordan Morgan
translator: 김필권
category: Cocoa
excerpt: >
  iOS 7에서 디자인 변경과 함께 스큐어모피즘 디자인은 유명한 석양처럼 사라졌습니다.
  그 대신 UI 컨트롤이 실제보다 물리적인 객체로 보이도록 만들어지는 새로운 패러다임이 생겨났습니다.
status:
  swift: 4.2
---

지난 10년간 iOS에는 많은 것들이 생겼다가 사라졌습니다. 수공예품 같았던 처음과는 달리 이젠 많은 것들이 바뀌었습니다.

테이블뷰, 레이블, 버튼처럼 익숙한 영역 말고 다른 영역에 발을 딛는 순간 저는 걸림돌에 걸리는데 그럴때면 제가 간과하거나 완벽히 잊고 있던 Cocoa Touch의 영역이 있었다는 것을 깨닫게 해줍니다.

`UIFieldBehavior` 는 먼지 쌓인 책의 일부분이었습니다. UI 요소에 대한 복잡한 필드 물리학을 모델링하기 위해 만들어진 API는 일반적인 사례가 아니며 동료 엔지니어들 사이에서 뜨거운 논점이 될 수도 있습니다. 이는 NSHipster의 글 주제가 되기에 아주 적합하다는 뜻이죠.

---

iOS 7에서 디자인 변경과 함께 스큐어모피즘 디자인은 유명한 석양처럼 사라졌습니다.
그 대신 UI 컨트롤이 실제보다 물리적인 객체로 보이도록 만들어지는 새로운 패러다임이 생겨났습니다.
새로운 시대를 위한 새로운 API는 [UIKit Dynamics](https://developer.apple.com/documentation/uikit/animation_and_haptics/uikit_dynamics)에서 소개되었습니다.

OS 전체에서 그 예를 찾을 수 있습니다. 톡톡 튀는 잠금 화면, 휙 볼 수 있는 사진, 버블버블 메세지 거품 등 이것들을 포함한 상호작용들은 UIKit Dynamics의 향이 납니다.

- `UIAttachmentBehavior`: 두 아이템이나 아이템과 주어진 anchor point와의 관계를 생성합니다.
- `UICollisionBehavior`: 하나 이상의 객체가 서로 덮어씌우는 것 대신에 튕겨나가도록 하는 상호작용입니다.
- `UIFieldBehavior`: 필드 기반 물리학을 아이템이나 공간에 적용합니다.
- `UIGravityBehavior`: 중력(아래로) 또는 위로 뜨도록 합니다.
- `UIPushBehavior`: 즉각적이거나 지속적인 힘을 생성합니다.
- `UISnapBehavior`: 지속적으로 꺾는 모션을 생성합니다.

이 글에선 FaceTime에서 PiP 기능을 만들 때 사용한 Cupertino의 좋은 친구인 `UIFieldBehavior` 에 대해 알아보도록 하겠습니다.

{% asset facetime-picture-in-picture.png alt="FaceTime" title="Image: Apple Inc. All Rights reserved." %}

## Field Behavior 이해하기

Apple은 `UIFieldBehavior` 가 "필드 기반" 물리학을 적용한다고 언급헀지만 정확히 그게 무슨 뜻일까요? 감사히도 그것은 우리가 생각하는 것과 매우 비슷합니다.

자석의 당김이나 스프링의 띠용(sproing)이나 우리를 아래로 누르는 지구의 중력처럼 현실 세계에는 필드 기반 물리학의 예제가 수없이 많습니다. `UIFieldBehavior` 를 사용하면 뷰의 공간을 조정해서 아이템이 언제든 그 공간에 들어와도 특정 물리 효과를 적용할 수 있게 됩니다.

API 디자인이 접근하기 쉽게 만들어졌기 때문에 복잡한 물리학이라도 팩토리 메소드에 불과하게 됩니다.

```swift
let drag = UIFieldBehavior.dragField()
```

```objc
UIFieldBehavior *drag = [UIFieldBehavior dragField];
```

우리가 다룰 수 있는 공간을 갖게 되면 화면에 배치하고 영향을 끼칠 영역을 정의해야 합니다.

```swift
drag.position = view.center
drag.region = UIRegion(size: bounds.size)
```

```objc
drag.position = self.view.center;
drag.region = [[UIRegion alloc] initWithSize:self.view.bounds.size];
```

필드의 행동을 넘어서 더 세분화된 조정이 필요하다면 `strength` 와 `falloff` 를 설정할 수 있습니다. 이를 사용하면 필드 타입에 추가적인 속성을 설정할 수 있습니다.

---

모든 UIKit Dynamics의 행동들은 영향을 끼치기 전에 약간의 설정이 필요하고 `UIFieldBehavior` 도 예외는 없습니다. 플로우는 다음과 같습니다.

- `UIDynamicAnimator` 의 인스턴스를 만들어서 다이나믹 아이템에 적용될 애니메이션의 컨텍스트를 제공합니다.
- 사용하고자 하는 행동으로 초기화합니다.
- 각 행동이 적용될 뷰를 추가합니다.
- 그 행동들을 처음 만들었던 dynamic animator에 추가합니다.

```swift
lazy var animator:UIDynamicAnimator = {
    return UIDynamicAnimator(referenceView: view)
}()

let drag = UIFieldBehavior.dragField()

// viewDidLoad:
drag.addItem(anotherView)
animator.addBehavior(drag)
```

```objc
@property (strong, nonatomic, nonnull) UIDynamicAnimator *animator;
@property (strong, nonatomic, nonnull) UIFieldBehavior *drag;

// viewDidLoad:
self.animator = [[UIDynamicAnimator alloc] initWithReferenceView:self.view];
self.drag = [UIFieldBehavior dragField];

[self.drag addItem:self.anotherView];
[self.animator addBehavior:self.drag];
```

{% warning do %}

`UIKitDynamicAnimator` 객체에 강한 참조를 유지하는 것을 관리해주세요. animator는 행동이 추가되면 오너쉽을 가져서 보통 꼭 할 필요는 없습니다.

{% endwarning %}


`UIFieldBehavior` 의 _bona fide_ 를 살펴보겠습니다. FaceTime은 어떻게 정면 카메라의 작은 직사각형 뷰를 화면의 경계에 붙일 수 있을까요?

## Spring Field로 만든 Face to Face

FaceTime을 하는 동안 우리는 픽쳐-인-픽쳐를 화면의 네 구석 중 하나에 옮길 수 있습니다. 어떻게 이렇게 유동적이면서 딱 붙어있도록 만들 수 있을까요?

한 가지 접근법은 제스처 인식기의 끝 상태를 확인하고 어떤 코너에 붙을 지 계산하며 필수적으로 애니메이션까지 하는 것입니다. 이 방법의 문제는 아바타가 코너에 옮겨질때마다 보간(interpolation)과 축임(dampening) 같은 상호작용 때문에 Apple이 고심해서 넣은 "비밀 소스"를 잃을 가능성이 있다는 것입니다.

이는 `UIFieldBehavior` 의 스프링 필드에서 있는 교과서적인 상황입니다. 말그대로 스프링이 작동하는 방식에 대해 생각한다면 원래 자리로 돌아오도록 선형적인 힘을 동등하게 줄 것입니다. 코일형 스프링을 아래로 당기면 원래 자리로 돌아가려는 것을 생각하면 됩니다.

이것은 스프링 필드가 우리의 UI에도 도움을 줄 수 있는 이유가 됩니다. 복싱 링에서 누군가 밖으로 나가려고 하면 다시 튕겨서 들어오는 상황을 생각하면 됩니다.

스프링 필드도 이것과 매우 비슷하게 작동합니다. 뷰가 네 영역으로 나눠져서 스프링이 각 영역에 달려있다고 상상해보세요. 그 스프링들은 사각형의 가운데에서 코너로 당길 것입니다.

{% info do %}

스프링 필드는 [Hooke's Law](https://phys.org/news/2015-02-law.html)를 복제해서 필드 안에서 객체에게 얼마만큼의 힘이 적용되는지 계산하기 위해서 만들어졌습니다.

{% endinfo %}

우리는 다음과 같이 똑똑하게 아바타 세팅을 관리할 수 있습니다.

```swift
let scale = CGAffineTransform(scaleX: 0.5, y: 0.5)

for vertical in [\UIEdgeInsets.left,
                 \UIEdgeInsets.right]
{
    for horizontal in [\UIEdgeInsets.top,
                       \UIEdgeInsets.bottom]
    {
        let springField = UIFieldBehavior.springField()
        springField.position =
            CGPoint(x: layoutMargins[keyPath: horizontal],
                    y: layoutMargins[keyPath: vertical])
        springField.region =
            UIRegion(size: view.bounds.size.applying(scale))

        animator.addBehavior(springField)
        springField.addItem(facetimeAvatar)
    }
}
```

```objc
UIFieldBehavior *topLeftCornerField = [UIFieldBehavior springField];

// 왼쪽 위 코너
topLeftCornerField.position = CGPointMake(self.layoutMargins.left, self.layoutMargins.top);
topLeftCornerField.region = [[UIRegion alloc] initWithSize:CGSizeMake(self.bounds.size.width/2, self.bounds.size.height/2)];

[self.animator addBehavior:topLeftCornerField];
[self.topLeftCornerField addItem:self.facetimeAvatar];

// 각 코너에 대한 스프링 필드를 계속 생성하면 됩니다
```

## 물리학 디버깅하기

보이지 않는 힘과 상호작용하는 것의 개념을 익히는 것은 쉬운 일이 아닙니다. 고맙게도 Apple은 이러한 문제를 예상했고 이를 풀기위한 격이 다른 방법들을 제공합니다.
It's not easy to conceptualize the interactions of invisible field forces.

`UIDynamicAnimator` 의 숨은 불리언 속성인 `debugEnabled` 입니다. 이를 `true` 로 설정하는 것은 필드 기반 효과들과 그것들이 영향을 끼치는 곳들을 빨간 줄로 시각화 해줍니다. 이는 계속 작업하는데 있어서 아주 좋은 동반자가 될 것입니다.

이 API는 공개적으로 알려지진 않았지만 키밸류 코딩이나 카테고리를 사용해서 잠금 해제 할 수 있습니다.

```objc
@import UIKit;

#if DEBUG

@interface UIDynamicAnimator (Debugging)
@property (nonatomic, getter=isDebugEnabled) BOOL debugEnabled;
@end

#endif
```

또는

```swift
animator.setValue(true, forKey: "debugEnabled")
```

```objc
[self.animator setValue:@1 forKey:@"debugEnabled"];
```

카테고리를 생성하는 것은 귀찮은 일이지만 더 안전한 옵션인 것은 변함 없습니다.
키밸류 코딩이 안전하지 않은 이유는 미래의 iOS에서 그 키가 쓰일 수 있기 때문입니다. 그것만 아니면 사용하기도 편해서 사용하기 좋습니다.

디버깅 설정을 켜면 어떤 코너가 스프링 효과를 가지고 있는지도 보여줍니다. 그러나 우리의 풋내기 앱을 실행해보면 우리가 찾고 있는 효과를 완전하게 소화하고 있지 않다는 것을 알 수 있습니다.

{% asset uidynamicanimator-debug.jpg %}

## 행동 정리하기 (Aggregating Behaviors)

필드 물리학에 대해 더 깊이 이해하기 위해 현재 상황을 살펴보겠습니다.
지금 우리가 가지고 있는 문제는 다음과 같습니다.

1. 아바타는 스프링 필드가 아니면 아무것도 유지하지 않고 화면에서 벗어날 수 있어야 합니다.
2. 원형으로 회전하고 있습니다.
3. 또 아주 느립니다.

UIKit Dynamics는 물리학을 시뮬레이션합니다. 아주 잘이요.

다행히도 이러한 바람직하지 않은 부작용을 모두 완화시킬 수 있습니다. 위트있게 말하자면 사소하지만 왜 그것이 핵심인지에 대한 이유입니다.

첫번째 이슈는 UIKit Dynamics의 가장 쉬운 행동 중 하나인 collision을 사용하면 아주 쉽게 해결할 수 있습니다. 스프링 필드가 작동되면 아바타 뷰가 어떻게 반응해야 하는지 더 잘 이해하기 위해 더 의도적으로 물리 속성을 설명해야 합니다. 이상적으로 우리는 중력과 마찰이 기세를 늦추기 위해 현실 세계에서와 같이 행동하기를 원합니다.

어떤 경우에선 `UIDynamicItemBehavior` 는 이상적입니다. 이는 물리적인 특성을 물리 엔진과 상호작용하는 단순한 뷰 인스턴스로 연결할 수 있습니다. UIKit은 물리엔진과 상호작용할 때 이러한 속성들에 대해 각각 기본 값을 제공하고 있습니다. 그것들은 여러분의 경우에 맞춰져 있지 않을 것입니다. 그리고 UIKit Dynamics는 거의 매번 "특정 경우" 버킷에 포함됩니다.

그러한 API 부족이 문제로 바뀔 수 있다고 예상하는 것은 그리 어렵지 않습니다. 밀거나 당기거나 속도와 같은 것을 모델링하고 싶지만 객체의 질량이나 밀도를 지정할 방법이 없다면 그것은 중요한 퍼즐 조각을 생략하는 것입니다.

```swift
let avatarPhysicalProperties= UIDynamicItemBehavior(items: [facetimeAvatar])
avatarPhysicalProperties.allowsRotation = false
avatarPhysicalProperties.resistance = 8
avatarPhysicalProperties.density = 0.02
```

```objc
UIDynamicItemBehavior *avatarPhysicalProperties = [[UIDynamicItemBehavior alloc] initWithItems:@[self.facetimeAvatar]];
avatarPhysicalProperties.allowsRotation = NO;
avatarPhysicalProperties.resistance = 8;
avatarPhysicalProperties.density = 0.02;
```

이제 아바타 뷰는 현실 세계를 더 많이 반영해서 스프링 필드로 밀린 후 가볍게 느려지는 효과를 낼 것입니다. `UIDynamicItemBehavior` 가 이런 일이 가능하게 해주는 것은 정말 놀랍운 기능이며 마음에 들때까지 설정을 바꿀 수도 있습니다.

게다가 그것은 객체에 선형 속도 또는 각속도에 대한 지원을 포함하고 있습니다. 이것은 `UIDynamicItemBehavior` 는 FaceTime 아바타에게 제스처 인식기의 마지막에 친근한 넛지를 추가해서 가까운 모서리로 보내는 여정의 끝을 내는 역할을 할 것입니다.

```swift
// 제스처 인식기 안의 switch문의 내용입니다
case .canceled, .ended:
let velocity = panGesture.velocity(in: view)
facetimeAvatarBehavior.addLinearVelocity(velocity, for: facetimeAvatar)
```

```objc
// 제스처 인식기 안의 switch문의 내용입니다
case UIGestureRecognizerStateCancelled:
case UIGestureRecognizerStateEnded:
{
CGPoint velocity = [panGesture velocityInView:self.view];
[facetimeAvatarBehavior addLinearVelocity:velocity forItem:self.facetimeAvatar];
break;
}
```

가짜 FaceTime UI를 만드는 것도 거의 다 끝났습니다!

모든 경험을 다같이 겪기 위해서 우리는 FaceTime 아바타가 animator의 뷰의 코너에 닿았을 때 어떤 행동을 해야하는지 알아야합니다. 우리는 그것이 계속 화면 안에 있기를 원합니다. UIKit Dynamics는 이러한 상황을 위해 `UICollisionBehavior` 를 제공합니다.

일관성있는 API 디자인덕분에 우리는 collision을 생성할 때도 다른 UIKit Dynamics 행동과 비슷한 패턴으로 생성할 수 있습니다.

```swift
let parentViewBoundsCollision = UICollisionBehavior(items: [facetimeAvatar])
parentViewBoundsCollision.translatesReferenceBoundsIntoBoundary = true
```

```objc
UICollisionBehavior *parentViewBoundsCollision = [[UICollisionBehavior alloc] initWithItems:@[self.facetimeAvatar]];
parentViewBoundsCollision.translatesReferenceBoundsIntoBoundary = YES;
```

`translatesReferenceBoundsIntoBoundary` 를 주목하세요. 이것의 값이 `true` 라면 우리의 animator 뷰의 바운드를 collision의 경계로 취급할 것입니다.

```swift
lazy var animator:UIDynamicAnimator = {
    return UIDynamicAnimator(referenceView: view)
}()
```

```objc
self.animator = [[UIDynamicAnimator alloc] initWithReferenceView:self.view];
```

여러가지 행동을 하나로 모았으니 이제 즐길 일만 남았습니다.

<video preload="none" src="{% asset uifieldbehavior-demo.mp4 @path %}" poster="{% asset uifieldbehavior-demo.png @path %}" width="320" controls></video>

<br/>

FaceTime의 "끈적한" 코너에서 벗어나고 싶다면 이상적인 태도입니다.
`UIFieldBehavior` 는 스프링말고도 많은 필드 물리학을 가지고 있습니다.
우리는 이를 자석 효과로 변경하거나 계속 회전하게 만들수도 있습니다.

---

iOS는 많은 스큐어모피즘을 겪어와서 사용자 경험에선 먼 길을 걸어왔습니다.
우리는 더 이상 Game Center에 어떤 게임이 있는지 알고 그것을 관리하기 위해 [초록색 펠트]({% asset game-center-felt.jpg @path %})를 필요로 하지 않습니다.

대신에 UIKit Dynamics는 사용자에게 완전히 새로운 방식의 상호작용을 소개합니다.
UI 컴포넌트를 현실 세계처럼 보이게 만드는 것은 사용자 경험이 2007년 이후로 얼마나 많이 발전했는지를 보여주는 좋은 예입니다.

OS 전반에 그러한 레이어가 벗겨지면서 UIKit Dynamics가 시각적 요소들이 우리의 행동에 어떤 반응을 해야하는지에 연결하는 문이 열렸습니다. 이러한 작은 연결은 처음 봤을 땐 그렇게 중요하지 않아 보일 수 있지만 시간이 지날수록 그렇지 않다는 것을 알게 될 것입니다.

UIKit Dynamics는 실제로 사용할 수 있는 여러가지 실제 동작을 제공하며 필드 행동들은 정말 흥미롭고 다양합니다.
다음에 앱에서 기회가 생긴다면 `UIFieldBehavior` 로 시작하는 것은 어떨까요?
