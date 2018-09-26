---
title: CMDeviceMotion
author: Nate Cook, Mattt Zmuda
translator: 김필권
category: Cocoa
excerpt: >
  Beneath the smooth glass of each iPhone
  an array of sensors sits nestled on the logic board,
  sending a steady stream of data to a motion coprocessor.
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


Let's say we want to give the splash page of our app a fun effect, such that the background image remains level no matter how the phone is tilted.


Consider the following code:


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


First, we check to make sure our device makes accelerometer data available.
Next we specify a high update frequency.
And then finally, we begin updates to a closure that will rotate a `UIImageView` property:


Each `CMAccelerometerData` object includes an `x`, `y`, and `z` value --- each of these shows the amount of acceleration in G-forces (where 1G = the force of gravity on Earth) for that axis.
If your device were stationary and standing straight up in portrait orientation, it would have acceleration `(0, -1, 0)`;
laying flat on its back on the table, it would be `(0, 0, -1)`;
tilted forty-five degrees to the right, it would be something like `(0.707, -0.707, 0)` _(dat √2 tho)_.


We calculate the rotation with the [two-argument arctangent function (`atan2`)](https://en.wikipedia.org/wiki/Atan2) using the `x` and `y` components from the accelerometer data.
We then initialize a `CGAffineTransform` using that calculate rotation.
Our image should stay right-side-up, no matter how the phone is turned --- here, it is in a hypothetical app for the _National Air & Space Museum_ (my favorite museum as a kid):


![Rotation with accelerometer]({% asset cmdm-accelerometer.gif @path %})


The results are not terribly satisfactory --- the image movement is jittery, and moving the device in space affects the accelerometer as much as or even more than rotating.
These issues _could_ be mitigated by sampling multiple readings and averaging them together, but instead let's look at what happens when we involve the gyroscope.


## Adding the Gyroscope


Rather than use the raw gyroscope data that we would get by calling the `startGyroUpdates...` method, let's get composited gyroscope _and_ accelerometer data by requesting the unified"device motion" data.
Using the gyroscope, Core Motion separates user movement from gravitational acceleration and presents each as its own property of the `CMDeviceMotion` object.
The code is very similar to our first example:


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


_Much better!_


![Rotation with gravity]({% asset cmdm-gravity.gif @path %})


## UIClunkController


We can also use the other, non-gravity portion of this composited gyro / acceleration data to add new methods of interaction.
In this case, let's use the `userAcceleration` property of `CMDeviceMotion` to navigate backward whenever the user taps the left side of the device against their hand.


Remember that the X-axis runs laterally through the device in our hand, with negative values to the left.
If we sense a _user_ acceleration to the left of more than 2.5 Gs, that's our cue to pop the view controller from the stack.
The implementation is only a couple lines different from our previous example:


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


_Works like a charm!_


Tapping the device in a detail view immediately takes us back to the list of exhibits:


![Clunk to go back]({% asset cmdm-clunk.gif @path %})


## Getting an Attitude


Better acceleration data isn't the only thing we gain by including gyroscope data:
we now also know the device's true orientation in space.
This data is accessed via the `attitude` property of a `CMDeviceMotion` object and encapsulated in a `CMAttitude` object.
`CMAttitude` contains three different representations of the device's orientation:


- Euler angles,
- A quaternion,
- A rotation matrix.


Each of these is in relation to a given reference frame.


### Finding a Frame of Reference


You can think of a reference frame as the resting orientation of the device from which an attitude is calculated.
All four possible reference frames describe the device laying flat on a table, with increasing specificity about the direction it's pointing.


- `CMAttitudeReferenceFrameXArbitraryZVertical` describes a device laying flat (vertical Z-axis) with an "arbitrary" X-axis. In practice, the X-axis is fixed to the orientation of the device when you _first_ start device motion updates.
- `CMAttitudeReferenceFrameXArbitraryCorrectedZVertical` is essentially the same, but uses the magnetometer to correct possible variation in the gyroscope's measurement over time.
- `CMAttitudeReferenceFrameXMagneticNorthZVertical` describes a device laying flat, with its X-axis (that is, the right side of the device in portrait mode when it's facing you) pointed toward magnetic north. This setting may require the user to perform that figure-eight motion with their device to calibrate the magnetometer.
- `CMAttitudeReferenceFrameXTrueNorthZVertical` is the same as the last, but it adjusts for magnetic / true north discrepancy and therefore requires location data in addition to the magnetometer.


For our purposes, the default "arbitrary" reference frame will be fine (you'll see why in a moment).


### Euler Angles


Of the three attitude representations, Euler angles are the most readily understood, as they simply describe rotation around each of the axes we've already been working with.


- `pitch` is rotation around the X-axis, increasing as the device tilts toward you, decreasing as it tilts away
- `roll` is rotation around the Y-axis, decreasing as the device rotates to the left, increasing to the right
- `yaw` is rotation around the (vertical) Z-axis, decreasing clockwise, increasing counter-clockwise.


> Each of these values follows what's called the "right hand rule": make a cupped hand with your thumb pointing up and point your thumb in the direction of any of the three axes.
> Turns that move toward your fingertips are positive, turns away are negative.


### Keep It To Yourself


Lastly, let's try using the device's attitude to enable a new interaction for a flash-card app designed to be used by two study buddies.
Instead of manually switching between the prompt and the answer, we'll automatically flip the view as the device turns around, so the quizzer sees the answer while the person being quizzed sees only the prompt.


Figuring out this switch from the reference frame would be tricky.
To know which angles to monitor, we would somehow need to account for the starting orientation of the device and then determine which direction the device is pointing.
Instead, we can save a `CMAttitude` instance and use it as the "zero point" for an adjusted set of Euler angles, calling the `multiply(byInverseOf:)` method to translate all future attitude updates.


When the quizzer taps the button to begin the quiz, we first configure the interaction (note the "pull" of the deviceMotion for `initialAttitude`):


```swift
// get magnitude of vector via Pythagorean theorem
func magnitude(from attitude: CMAttitude) -> Double {
    return sqrt(pow(attitude.roll, 2) +
            pow(attitude.yaw, 2) +
            pow(attitude.pitch, 2))
}

// initial configuration
var initialAttitude = manager.deviceMotion.attitude
var showingPrompt = false

// trigger values - a gap so there isn't a flicker zone
let showPromptTrigger = 1.0
let showAnswerTrigger = 0.8
```


Then, in our now familiar call to `startDeviceMotionUpdates`, we calculate the magnitude of the vector described by the three Euler angles and use that as a trigger to show or hide the prompt view:


```swift
if manager.isDeviceMotionActive {
    manager.startDeviceMotionUpdates(to: .main) {
        // translate the attitude
        data.attitude.multiply(byInverseOf: initialAttitude)

        // calculate magnitude of the change from our initial attitude
        let magnitude = magnitude(from: data.attitude) ?? 0

        // show the prompt
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

        // hide the prompt
        if showingPrompt && magnitude < showAnswerTrigger {
            showingPrompt = false
            self?.dismiss(animated: true, completion: nil)
        }
    }
}
```


Having implemented all that, let's take a look at the interaction.
As the device rotates, the display automatically switches views and the quizee never sees the answer:


![Prompt by turning the device]({% asset cmdm-prompt.gif @path %})


### Further Reading


I skimmed over the [quaternion](https://en.wikipedia.org/wiki/Quaternions_and_spatial_rotation) and [rotation matrix](https://en.wikipedia.org/wiki/Rotation_matrix) components of `CMAttitude` earlier, but they are not without intrigue.
The quaternion, in particular, has [an interesting history](https://en.wikipedia.org/wiki/History_of_quaternions), and will bake your noodle if you think about it long enough.


## Queueing Up


To keep the code examples readable, we've been sending all of our motion updates to the main queue.
A better approach would be to schedule these updates on their own queue and dispatch back to main to update the UI.


```swift
let queue = DispatchQueue(label: "motion")
manager.startDeviceMotionUpdates(to: queue) {
    [weak self] (data, error) in

    // motion processing here

    DispatchQueue.main.async {
        // update UI here
    }
}
```

---


Remember that not all interactions made possible by Core Motion are good ones.
Navigation through motion can be fun, but it can also be hard to discover, easy to accidentally trigger, and may not be accessible to all users.
Similar to purposeless animations, overuse of fancy gestures can make it harder to focus on the task at hand.


Prudent developers will skip over gimmicks that distract and find ways to use device motion that enrich apps and delight users.
