---
title: CMDeviceMotion
author: Nate Cook, Mattt Zmuda
translator: 김필권
category: Cocoa
excerpt: "iPhone의 액정 뒷면에선 수 많은 센서들이 데이터의 흐름을 꾸준히 모션 보조 프로세서에 보내고 있습니다."
status:
  swift: 4.2
---

iPhone의 액정 뒷면에선 수 많은 센서들이 데이터의 흐름을 꾸준히 모션 보조 프로세서에 보내고 있습니다.

[Core Motion 프레임워크](https://developer.apple.com/documentation/coremotion)는 센서들을 이용해서 사용자가 탭이나 스와이프 하기 전과 후의 상호작용을 가능하게 해주는 것을 놀랍도록 쉽게 만들었습니다.

---

Core Motion은 iOS나 watchOS 기기의 위치(position)와 방향(orientation)의 변화를 관찰하고 반응하는 것을 가능하게 해줍니다.
헌신적인 모션 보조 프로세서덕분에 iPhone, iPad 그리고 Apple Watch는 CPU나 배터리의 큰 소모 없이 내장 센서들의 입력을 읽고 처리할 수 있게 되었습니다.

가속도계와 자이로스코프 데이터는 기기의 중심을 기준으로 3D 좌표 공간에 투영됩니다.

![기기의 X축, Y축, Z축]({% asset cmdm-axes.png @path %})

iPhone을 세로로 들고 있을 때:

- X축은 기기의 가로만큼 왼쪽(음수 값)에서 오른쪽(양수 값)의 값을 가집니다.
- Y축은 아래(-)에서 위(+)로 기기의 높이만큼의 값을 가집니다.
- Z축은 화면을 수직으로 뒤(-)에서 앞(+)의 값을 가집니다.

## CMMotionManager

`CMMotionManager` 클래스는 해당 기기의 모션에 대한 데이터를 제공합니다.
퍼포먼스를 가장 높은 수준으로 유지하려면 하나의 공유된 `CMMotionManager` 를 만들고 사용하는 것이 좋습니다.

`CMMotionManager` 는 센서 정보에 따라 네 가지 다른 인터페이스를 제공합니다.

- `가속도계(accelerometer)` 는 속도의 변화인 가속도를 측정합니다.
- `자이로스코프(gyroscope)` 는 기기의 자세나 방향을 측정합니다.
- `자기계(magnetometer)` 는 기기에 대한 지구 자기장을 측정하는 나침반입니다.

각 센서가 개별적으로 읽는 것 말고도 `CMMotionManager` 는 하나로 합쳐진 "device motion" 인터페이스를 제공하며 이는 센서 융합 알고리즘을 사용해서 각 센서에서 읽어온 값을 조합하고 하나로 합쳐진 뷰를 제공합니다.

### 사용 가능성 확인하기

오늘날의 Apple 기기들 대부분 표준 센서를 가지고 나옴에도 불구하고 모션 데이터를 읽기 전에 해당 기기가 그 기능을 사용할 수 있는지 확인하는 것은 훌륭한 아이디어입니다.

다음 예제는 가속도계를 사용하고 있지만 "accelerometer" 단어를 원하는 모션 데이터("gyro", "magnetometer", "deviceMotion" 등)로 변경하면 그에 맞게 사용가능합니다.

```swift
let manager = CMMotionManager()
guard manager.isAccelerometerAvailable else {
    return
}
```

### Push vs Pull

Core Motion은 모션 데이터에 대한 "pull"과 "push" 접근을 제공합니다.

모션 데이터에 "pull"하면 `CMMotionManager` 의 읽기 전용 속성 중 하나를 가져옵니다.

"pushed" 데이터를 받으려면 업데이트를 받고 원하는 데이터에 대한 클로저를 생성해야 합니다.

#### "pull" 데이터 업데이트하기

```swift
manager.startAccelerometerUpdates()
```

위와 같이 호출 이후에 `manager.accelerometerData` 는 기기의 현재 가속도계 데이터이며 언제든지 접근할 수 있게됩니다.

```swift
manager.accelerometerData
```

또한 "is active" 속성을 이용해서 지금 사용이 가능한지 확인할 수 있습니다.

```swift
manager.isAccelerometerActive
```

#### "push" 데이터 업데이트하기

```swift
manager.startAccelerometerUpdates(to: .main) { (data, error) in
    guard let data = data, error != nil else {
        return
    }

    // ...
}
```

전달된 클로저는 업데이트 주기마다 호출됩니다.
(실제로 Core Motion은 최소와 최대 주기를 강요하며 범위 바깥의 값을 지정하면 값이 조정될 수 있습니다. 모션 이벤트가 생성되는 시간을 확인해서 최적의 주기를 결정할 수도 있습니다.)

#### 업데이트 멈추기

```swift
manager.stopAccelerometerUpdates()
```

## 가속도계 실제로 사용해보기

폰이 기울어져도 항상 제자리를 지키는 배경화면을 만들어보겠습니다.

다음과 같이 짤 수 있을 것입니다:

```swift
if manager.isAccelerometerAvailable {
    manager.accelerometerUpdateInterval = 0.01
    manager.startAccelerometerUpdates(to: .main) {
        [weak self] (data, error) in
        guard let data = data, error != nil else {
            return
        }

        let rotation = atan2(data.acceleration.x,
                             data.acceleration.y) - .pi
        self?.imageView.transform =
            CGAffineTransform(rotationAngle: CGFloat(rotation))
    }
}
```

먼저 확인해야 할 사항은 기기의 가속도계 데이터가 사용 가능한지입니다.
다음은 잦은 주기로 업데이트하도록 하는 것입니다.
마지막으로 `UIImageView` 속성을 회전해야 할 클로저를 업데이트하는 것입니다.

각 `CMAccelerometerData` 객체는 `x`, `y` 그리고 `z` 값을 가지고 있습니다. 각 값은 그 축에 맞는 중력장(G-forces)의 가속도값을 의미합니다. (1G = 지구의 중력)
만약 기기가 가만히 있고 똑바로 서 있으며 세로 방향이라면 `(0, -1, 0)` 값을 가질 것입니다.
테이블에 등을 대고 평평하게 누워있다면 `(0, 0, -1)` 값일 것입니다.
45도 오른쪽으로 기울어있다면 `(0.707, -0.707, 0)` 값을 가질 것입니다.

회전을 계산할 때는 [2인자 아크탄젠트 함수(`atan2`)](https://en.wikipedia.org/wiki/Atan2)를 사용한 다음 `CGAffineTransform` 을 초기화했습니다.
우리의 이미지는 폰이 기울어져도 항상 올바른 방향을 유지해야 합니다. 아래 예제는 어릴때부터 좋아하던 _National Air & Space Museum_ 의 앱입니다.

![Rotation with accelerometer]({% asset cmdm-accelerometer.gif @path %})

결과는 심각하게 만족스럽지 않습니다. 이미지의 움직임은 지직거리며 기기를 움직이는 것이 회전하는 것에 비해 가속도계에 동일하거나 그 이상의 영향을 미칩니다.
이 이슈는 여러 값을 합치면 완화할 수 있는 일이지만 그 대신에 우리는 자이로스코프를 더 좋게 만들것입니다.

## 자이로스코프 추가하기

가공하지 않은 자이로스코프 데이터를 쓰는 것 대신에 우리는 `startGyroUpdates...` 메소드를 호출할 것입니다. 그 이후에 자이로스코프 값과 가속도계 값을 합쳐서 하나의 "device motion" 데이터를 만들 것입니다.
자이로스코프를 쓰면 Core Motion이 사용자의 움직임을 중력 가속도에서 분리할 수 있고 `CMDeviceMotion` 객체의 속성에 표현할 수 있게 됩니다.
코드 자체는 처음 예제와 크게 다르지 않습니다.

```swift
if manager.isDeviceMotionActive {
    manager.deviceMotionUpdateInterval = 0.01
    manager.startDeviceMotionUpdates(to: .main) {
        [weak self] (data, error) in

        guard let data = data, error != nil else {
            return
        }

        let rotation = atan2(data.acceleration.x,
                             data.acceleration.y) - .pi
        self?.imageView.transform =
            CGAffineTransform(rotationAngle: CGFloat(rotation))
    }
}
```

_훨씬 낫네요!_


![Rotation with gravity]({% asset cmdm-gravity.gif @path %})


## UIClunkController

또한 우리는 합쳐진 자이로 / 가속도계 데이터의 비 중력 영역을 새로운 상호작용 메소드로 사용할 수 있습니다.
예를 들면 `CMDeviceMotion` 의 `userAcceleration` 속성을 사용하면 사용자가 왼손으로 왼쪽 화면을 눌렀을 경우 뒤로가게 만들 수도 있습니다.

X축은 우리의 손에서 음의 값에서 양의 값을 가진다는 것을 기억하세요. 그리고 가속도가 2.5 G이상을 감지하면 뷰 컨트롤러를 뒤로가게 합시다. 결과 코드는 이전의 예제와 몇 줄 다르지 않습니다.

```swift
if manager.isDeviceMotionActive {
    manager.deviceMotionUpdateInterval = 0.01
    manager.startDeviceMotionUpdates(to: .main) {
        [weak self] (data, error) in

        guard let data = data, error != nil else {
            return
        }
        if data.userAcceleration.x < -2.5 {
            self?.navigationController?.popViewControllerAnimated(true)
        }
    }
}
```

_잘 작동하네요!_

상세 화면을 누르면 즉시 전시회 목록으로 가게될 것입니다.

![Clunk to go back]({% asset cmdm-clunk.gif @path %})

## 또 다른 접근법 알아보기

자이로스코프 데이터를 포함하면서 얻은 것은 더 나은 가속도계 데이터뿐만이 아닙니다.
우리는 기기의 실제 방향도 알 수 있게 되었습니다.
그 데이터는 `CMDeviceMotion` 객체의 `attitude` 속성으로 접근 가능하며 `CMAttitude` 객체로 압축되어 있습니다.
`CMAttitude` 는 기기의 방향에 대한 세 가지 서로 다른 표현을 가지고 있습니다.

- 오일러 각,
- 쿼터니언,
- 회전 행렬 입니다.

각 표현법은 주어진 참조 프레임에 관련이 있습니다.

### 참조할만한 프레임 찾기

참조 프레임을 계산되는 장치의 방향이라고 생각하실 수도 있습니다. 4개 참조 프레임은 모두 테이블에 누워있는 기기를 설명하고 있어서 향하는 방향에 대해 더 구체적으로 표현합니다.

- `CMAttitudeReferenceFrameXArbitraryZVertical` 은 평평한 바닥(Z축 수직)에 누워있으며 "임의의" X축 값으로 기기의 위치를 표현합니다. 실제로 X축은 기기 모션 업데이트를 시작한 _처음_ 위치에 고정되어 있습니다.
- `CMAttitudeReferenceFrameXArbitraryCorrectedZVertical` 은 거의 똑같지만 자기계를 사용해서 자이로스코프의 측정의 다양함을 바로잡기 위해 사용했습니다.
- `CMAttitudeReferenceFrameXMagneticNorthZVertical` 은 바닥에 누워있고 X축이 자기장의 북쪽을 가리키는 값을 표현합니다. (마주보고 있다면 세로 모드일때를 뜻합니다) 이 설정을 사용하려면 사용자가 자기계를 보정하기 위해 해당 기기로 8자 모션을 수행해야 합니다.
- `CMAttitudeReferenceFrameXTrueNorthZVertical` 은 위와 동일하지만 자기와 실제 북쪽 불일치가 일어나지 않도록 조정하기 때문에 추가적인 위치 데이터가 필요합니다.

우리의 경우엔 기본값인 "임의의" 참조 프레임도 괜찮을 것 같네요. (왜그런지는 곧 보여드리겠습니다.)

### 오일러 각

오일러 각은 기기의 자세를 표현하는 세 가지 방법 중 가장 읽기 좋은 형태를 가지고 있습니다. 앞에서 우리가 같이 알아봤던 각 축에 대한 회전을 간단하게 표시하기 때문입니다.

- `pitch` 는 X축의 회전을 의미합니다. 앞으로 기울이면 값이 증가하고 뒤로 기울이면 값이 감소합니다.
- `roll` 은 Y축의 회전을 의미합니다. 오른쪽으로 돌리면 값이 증가하고 왼쪽으로 돌리면 값이 감소합니다.
- `yaw` 는 Z축(수직)의 회전을 의미합니다. 반시계방향으로 돌리면 값이 증가하고 시계방향으로 돌리면 감소합니다.

> 각각의 값은 "오른손 법칙"을 따릅니다. 손을 컵 모양으로 만들고 엄지를 세 축 중 원하는 축의 위쪽을 향합니다. 손끝이 향하는 곳이 증가하는 방향이고 그 반대가 감소하는 방향입니다.

### 내 것으로 만들기

마지막으로 기기의 자세를 사용해서 두 명이서 같이 공부할 수 있는 스피드 퀴즈 앱에 새로운 상호작용을 넣어봅시다.
문제와 정답을 수동으로 변경하는 것 대신에 기기가 돌아갈 때 자동으로 문제와 정답을 변경하게 해보겠습니다. 그렇게 하면 문제를 내는 사람은 문제와 정답을 다 같이 보고 문제를 푸는 사람은 문제만 보게 될 것입니다.

문제와 정답의 변경을 위해 참조 프레임을 사용하는 것은 곤란할 수 있습니다. 어떤 각을 감시해야 할 지 알기 위해서 우리는 기기의 시작 위치(orientation)과 기기가 바라보고 있는 방향(direction)을 알아야합니다.
이렇게 하는 것 대신에 `CMAttitude` 인스턴스를 저장하고 이를 "영점"으로 사용하겠습니다. 그리고 `multiply(byInverseOf:)` 메소드를 사용해서 미래에 생길 모든 자세 업데이트에 대해 오일러 각을 조정하겠습니다.

문제 내는 사람이 퀴즈 버튼을 눌러서 시작할 때 우리는 기기의 시작점을 이용해 상호작용을 설정하겠습니다. (`initialAttitude` 의 deviceMotion의 "pull"을 기억하세요)

```swift
// Pythagorean 이론을 통해 규모를 알아냅니다
func magnitude(from attitude: CMAttitude) -> Double {
    return sqrt(pow(attitude.roll, 2) +
            pow(attitude.yaw, 2) +
            pow(attitude.pitch, 2))
}

// 초기 설정
var initialAttitude = manager.deviceMotion.attitude
var showingPrompt = false

// 트리거할 값을 지정합니다
let showPromptTrigger = 1.0
let showAnswerTrigger = 0.8
```

그 다음에 익숙한 `startDeviceMotionUpdates` 을 호출하고 오일러 각에서 설명된 벡터값의 규모를 계산합니다. 그리고 문제 화면을 보여줄지 말지를 트리거합니다.

```swift
if manager.isDeviceMotionActive {
    manager.startDeviceMotionUpdates(to: .main) {
        // attitude값 가져오기
        data.attitude.multiply(byInverseOf: initialAttitude)

        // 초기 attitude의 변화를 계산합니다
        let magnitude = magnitude(from: data.attitude) ?? 0

        // 문제를 보여줍니다
        if !showingPrompt && magnitude > showPromptTrigger {
            if let promptViewController =
                self?.storyboard?.instantiateViewController(
                    withIdentifier: "PromptViewController"
                ) as? PromptViewController
            {
                showingPrompt = true

                promptViewController.modalTransitionStyle = .crossDissolve
                self?.present(promptViewController,
                              animated: true, completion: nil)
            }
        }

        // 문제를 숨깁니다
        if showingPrompt && magnitude < showAnswerTrigger {
            showingPrompt = false
            self?.dismiss(animated: true, completion: nil)
        }
    }
}
```

다 작성하셨다면 결과물을 확인해봅시다.
기기가 회전될때마다 화면은 자동으로 변경되어서 문제 푸는 사람은 절대 정답을 볼 수 없게됩니다.

![Prompt by turning the device]({% asset cmdm-prompt.gif @path %})

### 더 읽을거리

저는 `CMAttitude` 의 [quaternion](https://en.wikipedia.org/wiki/Quaternions_and_spatial_rotation)과 [rotation matrix](https://en.wikipedia.org/wiki/Rotation_matrix) 컴포넌트를 건너뛰었지만 의도한 것은 아닙니다.
특히 quaternion에는 [흥미로운 역사](https://en.wikipedia.org/wiki/History_of_quaternions)가 있고 생각하면 할수록 머리가 아프실 수도 있습니다.

## 큐 잘 사용하기

코드 예제를 더 읽기 쉽게 만들기 위해서 모션 업데이트들을 모두 메인 큐에 올렸었습니다.
실제로 더 나은 접근법은 이 업데이트들을 각자의 큐에 올려두고 UI 업데이트시에만 메인 큐를 사용하는 것입니다.

```swift
let queue = DispatchQueue(label: "motion")
manager.startDeviceMotionUpdates(to: queue) {
    [weak self] (data, error) in

    // 여기서 모션 처리

    DispatchQueue.main.async {
        // 여기서 UI 처리
    }
}
```

---

Core Motion으로 모든 상황에 맞는 상호작용을 만들 수는 없다는 사실을 알아주세요. 목적없는 애니메이션처럼 팬시한 제스쳐를 과하게 사용하는 것은 오히려 작은 화면에 집중하기 어려워집니다.

신중한 개발자라면 앱을 풍부하게 만들고 사용자를 기쁘게하는 기기 모션을 적절하게 잘 사용할 것입니다.
