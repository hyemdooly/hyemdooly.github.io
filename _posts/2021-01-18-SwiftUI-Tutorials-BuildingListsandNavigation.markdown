---
layout: post
title:  "Building Lists and Navigation"
date:   2020-01-18 17:40:00 +0900
categories: iOS
---
두 번째 튜토리얼. 안드로이드의 Recycler View가 생각나는 파트였다.

Swift 문법을 공부하지 않고 바로 튜토리얼을 봐서 그런지 이해하기 어려운 문법이 존재했다.

## 디렉토리 구조

![Directory]({{"assets/img/iOS/BuildingListsandNavigation/Directory.png"| relative_url}})

이번 프로젝트에서는 다음과 같이 디렉토리 구조를 정리한다.

Views는 보여줄 화면과 그 요소들, Resources는 json파일, Model은 데이터 structure와 데이터 로딩 관련한 파일 디렉토리다.

## 전체 소스 코드

### [ContentView.swift]

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        LandmarkList()
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```

### [CircleImage.swfit]

지난 튜토리얼과 중복되므로 설명은 생략한다.

```swift
import SwiftUI

struct CircleImage: View {
    var image: Image
    var body: some View {
        image
            .clipShape(Circle()) // 원 모양으로 clip
            .overlay(Circle().stroke(Color.white, lineWidth: 4)) // 테두리
            .shadow(radius: 7) // 그림자
    }
}

struct CircleImage_Previews: PreviewProvider {
    static var previews: some View {
        CircleImage(image: Image("turtlerock"))
    }
}
```

### [MapView.swift]

지난 튜토리얼의 연장선으로 코드가 조금 수정된다.

```swift
import SwiftUI
import MapKit

struct MapView: View {
    var coordinate: CLLocationCoordinate2D // LandmakrDetail.swift에서 값을 넣어줌
    @State private var region = MKCoordinateRegion()
    
    var body: some View {
        Map(coordinateRegion: $region)
            .onAppear { // View가 나타날 때 trigger 역할
                setRegion(coordinate)
            }
    }
    
    private func setRegion(_ coordinate: CLLocationCoordinate2D) {
        region = MKCoordinateRegion(
            center: coordinate,
            span: MKCoordinateSpan(latitudeDelta: 0.2, longitudeDelta: 0.2)
        )
    }
}

struct MapView_Previews: PreviewProvider {
    static var previews: some View {
        MapView(coordinate: CLLocationCoordinate2D(latitude: 34.011_286, longitude: -116.166_868))
    }
}
```

### [LandmarkRow.swift]

List에 들어갈 Item UI를 의미하는 코드이다.

```swift
import SwiftUI

// List Item
struct LandmarkRow: View {
    var landmark: Landmark
    
    var body: some View {
        HStack {
            landmark.image
                .resizable()
                .frame(width: 50, height: 50)
            
            Text(landmark.name)
            Spacer()
        }
    }
}

struct LandmarkRow_Previews: PreviewProvider {
    static var previews: some View {
        Group {
            LandmarkRow(landmark: landmarks[0])
            LandmarkRow(landmark: landmarks[1])
        }
        .previewLayout(.fixed(width: 300, height: 70))
    }
}
```

### [LandmarkList.swift]

List를 의미하는 코드이다. List를 Dynamic으로 만든다.

```swift
import SwiftUI
// List를 Dynamic(동적)으로 만드는 부분
struct LandmarkList: View {
    var body: some View {
        NavigationView {
            List(landmarks) { landmark in // landmarks : ModelData.swift에 정의된 변수
                NavigationLink(destination: LandmarkDetail(landmark: landmark)) {
                    LandmarkRow(landmark: landmark) // LandmarkRow : Item 하나하나의 레이아웃 부분
                    // LandmarkRow에 목적지를 LandmarkDetail로 Link
                }
            }
            .navigationTitle("Landmarks")
        }
    }
}

struct LandmarkList_Previews: PreviewProvider {
    static var previews: some View {
        ForEach(["iPhone SE", "iPhone XS Max"], id: \.self) { deviceName in
                    LandmarkList()
                        .previewDevice(PreviewDevice(rawValue: deviceName))
        }
    }
}
```

#### ForEach와  `\`에 대하여

먼저 SwiftUI의 ForEach는, 우리가 흔히 사용하는 foreach와 다른 개념이다. 우리가 아는 foreach는 말 그대로 반복문이지만, SwiftUI의 ForEach는 **반복되는 UI**를 **특정 배열**로 그리기 위한 수단이다.

먼저 `id`는 Item 하나하나에 대한 고윳값이다. 만약 `id`를 명시하지 않는다면 Item 하나가 바뀌었을 때, SwiftUI 입장에서는 어떤 Item이 바뀌었는지 알 수 없기 때문에 일부만 바뀌지 않고 UI 전체가 렌더링되어버린다. 이것이 ForEach에 `id` 를 넣어주는 이유이다.

여기서 ForEach의 `id` 는 **KeyPath**로 작동한다. `\`는 **KeyPath**를 가져오는 문법이다. 예시 코드를 통해 이해해보자.

Swift에는 "특정 프로퍼티의 값이 변경되면 이벤트를 수신하라(KVO : Key-Value Observing)"는 개념이 존재한다.

``` swift
class Landmark {
    id: String,
    location: Location
}

class Location {
    latitude: double
    longitude: double
}

var landmark: Landmark
```

다음과 같은 코드가 있다고 가정할 때, Landmark 안에 location 프로퍼티 안에 latitude 객체가 변경되면 이벤트를 받고 싶을 때 다음과 같이 작성한다.

``` swift
landmark.observe(\.location.latitude) { // 앞 부분 타입 생략 가능, 원래는 \Landmark.location.latitude => latitude의 KeyPath 반환
    // location.latitude가 변경되면 실행되는 내부
}
```

앞 부분 타입은 생략이 가능하기 때문에, `id`에서도 `\.self`로 작성했던 것이다. 풀로 쓴다면 `\String.self`가 된다.

> "엥, 그러면 그냥 Type에 대한 KeyPath가 나오는 것이 아닌가요?"

사실 이 문구의 정확한 뜻은 String형 객체의 KeyPath이다. 즉, 배열로 들어갔던 ["iPhone SE", "iPhone XS Max"]에서 "iPhone SE"이라는 String 객체에 대한 KeyPath, "iPhone XS Max"이라는 String 객체에 대한 KeyPath를 의미하는 것이다.

self에 대하여 더 정리하자면,

> * 타입.self → Type 반환
> * 변수.self → 변수 자기 자신 반환

위와 같이 반환이 되는데, 이 경우에는 타입.self 이다. Type이 반환이 되고, `\`가 붙었으므로 그 Type의 객체에 대한 KeyPath가 반환된다.

또한, 우리가 보는 코드와는 상관없는 내용이지만 변수.self에 관해 추가하자면

```swift
var name = "혜민"
name = "송혜민" // 1
name.self = "송혜민" // 2
```

1과 2는 같은 동작을 하는 코드이다.

### [LandmarkDetail.swift]

List의 요소를 눌렀을 때 보이게 되는 화면 코드이다.

```swift
import SwiftUI

struct LandmarkDetail: View {
    var landmark: Landmark
    
    var body: some View {
    // view <-> data 연동해주는 부분
        ScrollView {
            MapView(coordinate: landmark.locationCoordinate)
                .ignoresSafeArea(edges: .top)
                .frame(height: 300)
            
            CircleImage(image: landmark.image)
                .offset(y: -130)
                .padding(.bottom, -130)
            
            VStack(alignment: .leading) {
                Text(landmark.name)
                    .font(.title)
                    .foregroundColor(.primary)
                
                HStack {
                    Text(landmark.park)
                    Spacer()
                    Text(landmark.state)
                }
                .font(.subheadline)
                .foregroundColor(.secondary)
                
                Divider()
                // Text안에다가 변수 값을 가져와서 쓰고 싶을 때는 \(변수명)
                Text("About \(landmark.name)")
                    .font(.title2)
                Text(landmark.description)
            }
            .padding()
        }
        .navigationTitle(landmark.name)
        .navigationBarTitleDisplayMode(.inline)
        // automatic, inline, large
    }
}

struct LandmarkDetail_Previews: PreviewProvider {
    static var previews: some View {
        LandmarkDetail(landmark: landmarks[0])
    }
}
```

#### navigationBarTitleDisplayMode

navigationBarTitleDisplayMode에는 `automatic`, `inline`, `large`라는 3가지 옵션이 존재한다.

- automatic ![displayMode-automatic]({{"assets/img/iOS/BuildingListsandNavigation/navigationBarTitleDisplayMode-automatic.png"| relative_url}})

- inline ![displayMode-inline]({{"assets/img/iOS/BuildingListsandNavigation/navigationBarTitleDisplayMode-inline.png"| relative_url}})

- large ![displayMode-large]({{"assets/img/iOS/BuildingListsandNavigation/navigationBarTitleDisplayMode-large.png"| relative_url}})

automatic과 large의 차이점은 거의 없다. automatic은 플랫폼에 알아서 적응하는 형태이다. 즉, 아이폰과 아이패드 앱을 동시에 개발할 때 유용하게 사용한다. large는 고정이라고 보면 된다.

### [Landmark.swift]

데이터의 Structure 파일이다. 안드로이드에서 Data Class와 비슷한 듯 하다.

```swift
import Foundation
import SwiftUI
import CoreLocation

// 데이터를 담는 용기 역할
struct Landmark: Hashable, Codable, Identifiable { // 구조체이름: 프로토콜이름, 프로토콜은 JAVA의 인터페이스와 비슷한 느낌
    var id: Int
    var name: String
    var park: String
    var state: String
    var description: String
    
    private var imageName: String
    var image: Image {
        Image(imageName)
    }
    
    private var coordinates: Coordinates
    var locationCoordinate: CLLocationCoordinate2D {
        CLLocationCoordinate2D(
            latitude: coordinates.latitude,
            longitude: coordinates.longitude
        )
    }
    
    struct Coordinates: Hashable, Codable {
        var latitude: Double
        var longitude: Double
    }
}
```

### [ModelData.swift]

Data JSON 파일을 로딩하는 부분이다. Generic 개념이 사용된다.

```swift
import Foundation

var landmarks: [Landmark] = load("landmarkData.json") // Landmark 타입의 배열 변수
// Generic
// 정의를 func funcName<T>이라고 했다면
// var a: [T]

// Decoable : decoder가 존재해서 JSON형태의 String과 내가 만든 객체를 왔다갔다할 수 있는 타입
func load<T: Decodable>(_ filename: String) -> T {
    // (_ filename) : 기본적으로 Swift는 Named Parameter(a: 1, b: 2)지만 _을 쓰면 파라미터 이름을 써주지 않고 넣을 수 있음, (1, b: 2)처럼
    let data: Data
    
    guard let file = Bundle.main.url(forResource: filename, withExtension: nil) // nil : null, file이 nil이면 else로 이동, file open
    else {
        fatalError("Couldn't find \(filename) in main bundle.")
    }
    
    do {
        data = try Data(contentsOf: file)
    } catch {
        fatalError("Couldn't load \(filename) from main bundle:\n\(error)")
    }
    
    do {
        let decoder = JSONDecoder()
        return try decoder.decode(T.self, from: data) // T.self : T.Type
    } catch {
        fatalError("Couldn't parse \(filename) as \(T.self):\n\(error)")
    }
}
```

## 결과물

![UI-1]({{"assets/img/iOS/BuildingListsandNavigation/UI-1.jpg"| relative_url}}){: width="50%" height="50%"}

![UI-2]({{"assets/img/iOS/BuildingListsandNavigation/UI-2.jpg"| relative_url}}){: width="50%" height="50%"}

Preview로 다음과 같은 결과를 확인할 수 있다.