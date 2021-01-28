---
layout: post
title:  "Handling User Input"
date:   2020-01-28 15:21:00 +0900
categories: iOS

---

세 번째 튜토리얼은 User의 Input값을 받아 UI를 변경하는 부분이다. 변경된 코드만 다루고, 나머지 코드는 이전 튜토리얼 코드와 같다.

## 디렉토리 구조

![Directory]({{"assets/img/iOS/HandlingUserInput/Directory.png"| relative_url}})

이번 프로젝트에서는 다음과 같이 디렉토리 구조를 정리한다.

View 내부에서 Landmarks와 Helpers로 나뉘었는데, Landmarks는 UI의 큰 구조(List, List의 Item, Item 클릭 시 보일 Detail View), Helpers는 UI 내의 요소 하나하나(이미지, 지도, 버튼)를 모아둔 Directory이다.

## 전체 소스 코드

### [LandmarkRow.swift]

`landmark.isFavorite`은 `landmark.json` 파일을 확인하면 알 수 있다. input 파일에 isFavorite 여부가 존재한다. 만약 True일 경우 star 이미지에 노란색으로 foreground를 채우는 것을 확인할 수 있다.

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
            
            // User isFavorite
            if landmark.isFavorite {
                Image(systemName: "star.fill")
                    .foregroundColor(.yellow)
            }
        }
    }
}

struct LandmarkRow_Previews: PreviewProvider {
    static var landmarks = ModelData().landmarks
    static var previews: some View {
        Group {
            LandmarkRow(landmark: landmarks[0])
            LandmarkRow(landmark: landmarks[1])
        }
        .previewLayout(.fixed(width: 300, height: 70))
    }
}
```

### [LandmarkList.swfit]

`@EnvironmentObject`이 무엇인지는 맨 뒤에서 설명한다.

```swift
import SwiftUI

struct LandmarkList: View {
    @EnvironmentObject var modelData: ModelData
    @State private var showFavoriteOnly = false
    
    var filteredLandmarks: [Landmark] {
        modelData.landmarks.filter { landmark in
            (!showFavoriteOnly || landmark.isFavorite)
            // 필터링, showFavoriteOnly를 true로 바꾸면 isFavorite인 Item만 모은 List로 보이게 함
        }
    }
    
    var body: some View {
        NavigationView {
            List {
                Toggle(isOn: $showFavoriteOnly) {
                    Text("Favorites only")
                }
                // ForEach로 수정해서 data의 collection을 passing
                ForEach(filteredLandmarks) { landmark in
                    NavigationLink(destination: LandmarkDetail(landmark: landmark)) {
                        LandmarkRow(landmark: landmark)
                    }
                }
            }
            .navigationTitle("Landmarks")
        }
    }
}

struct LandmarkList_Previews: PreviewProvider {
    static var previews: some View {
        LandmarkList()
            .environmentObject(ModelData())

    }
}

```

#### 왜 Toggle을 List 내부에 넣었을까?

```swift
VStack {
    Toggle(isOn: $showFavoriteOnly) {
        Text("Favorites only")
    }
    List(filteredLandmarks) { landmark in
        NavigationLink(destination: LandmarkDetail(landmark: landmark)) {
            LandmarkRow(landmark: landmark)
        }
    }
}
```

위와 같이 Toggle을 List 내부에 넣지 않고 VStack으로 쌓을 수도 있다는 생각을 하였다. 하지만 UI가 내가 원하는 대로 나오지 않는다는 점을 확인할 수 있었다.

![Directory]({{"assets/img/iOS/HandlingUserInput/Result-VStack-Toggle.png"| relative_url}})

### [LandmarkDetail.swift]

`modelData.landmarks.firstIndex` 뒤에 붙은 `!`은 nil을 반환하지 않는다는 것을 의미한다. nill과 null은 같은 의미고, `!`은 Kotlin의 `!!`과 같은 역할을 한다고 볼 수 있다.

`$숫자`는 보통 인자를 의미한다. 예를 들어 `$0`이라고 하면 첫 번째 인자, `$1`이라고 하면 두 번째 인자를 의미한다. 코드를 축약하는데 유용하다.

`@EnvironmentObject`이 무엇인지는 맨 뒤에서 다른 코드와 함께 설명한다.

```swift
import SwiftUI

struct LandmarkDetail: View {
    @EnvironmentObject var modelData: ModelData
    var landmark: Landmark
    
    var landmarkIndex: Int {
        modelData.landmarks.firstIndex(where: { $0.id == landmark.id })!
        // landmarks 값들 중에서 landmark.id에 해당하는 id값들의 firstIndex 반환
        // 위 코드는 아래 코드의 축약형, self.landmark.id는 바깥쪽 landmark를 의미함
				// modelData.landmarks.firstIndex(where: { landmark in
				//     landmark.id == self.landmark.id
				// })!
    }
    
    var body: some View {
        ScrollView {
            MapView(coordinate: landmark.locationCoordinate)
                .ignoresSafeArea(edges: .top)
                .frame(height: 300)
            
            CircleImage(image: landmark.image)
                .offset(y: -130)
                .padding(.bottom, -130)
            
            VStack(alignment: .leading) {
                HStack {
                    Text(landmark.name)
                        .font(.title)
                        .foregroundColor(.primary)
                    FavoriteButton(isSet: $modelData.landmarks[landmarkIndex].isFavorite)
                }
                
                HStack {
                    Text(landmark.park)
                    Spacer()
                    Text(landmark.state)
                }
                .font(.subheadline)
                .foregroundColor(.secondary)
                
                Divider()
              
                Text("About \(landmark.name)")
                    .font(.title2)
                Text(landmark.description)
            }
            .padding()
        }
        .navigationTitle(landmark.name)
        .navigationBarTitleDisplayMode(.inline)
    }
}

struct LandmarkDetail_Previews: PreviewProvider {
    static let modelData = ModelData()
    
    static var previews: some View {
        LandmarkDetail(landmark: ModelData().landmarks[0])
            .environmentObject(modelData)
    }
}
```

### [FavoriteButton.swift]

LandmarkDetail에 포함되는 Button UI이다. `@Binding`은 간단히 말하자면 `@Binding` 어노테이션으로 선언하고 초기화할 때 State값을 받는다. 이 때 여러 개의 View가 동시에 State값을 참조할 수 있다. 따라서, isSet이 변경되면 참조하고 있는 State값도 변경되고 UI에서도 바뀌는 것을 확인할 수 있다. 뒤에서 더 자세하게 살펴보도록 하자.

```swift
import SwiftUI

struct FavoriteButton: View {
    @Binding var isSet: Bool
    
    var body: some View {
        Button(action: {
            isSet.toggle()
        }) {
            Image(systemName: isSet ? "star.fill" : "star")
                .foregroundColor(isSet ? Color.yellow : Color.gray)
        }
    }
}

struct FavoriteButton_Previews: PreviewProvider {
    static var previews: some View {
        FavoriteButton(isSet: .constant(true))
    }
}
```

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
            .environmentObject(ModelData())
    }
}
```

### [ModelData.swift]

```swift
import Foundation
import Combine

// ObservableObject : View가 외부객체를 감지하게 함
// @ObservedObject : ObservableObject를 구독하고 값이 업데이트 될 때마다 View를 Refresh시킴
final class ModelData: ObservableObject {
    @Published var landmarks: [Landmark] = load("landmarkData.json") // @Published : 값의 변화 감지를 위해서 필요, 값이 업데이트 되면 View(구독자, Subscriber)를 Refresh 시킴
}

func load<T: Decodable>(_ filename: String) -> T {
    let data: Data
    
    guard let file = Bundle.main.url(forResource: filename, withExtension: nil)
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
        return try decoder.decode(T.self, from: data)
    } catch {
        fatalError("Couldn't parse \(filename) as \(T.self):\n\(error)")
    }
}
```

### [LandmarksApp.swift]

```swift
import SwiftUI

@main
struct LandmarksApp: App {
    // @State는 private로 선언
    @StateObject private var modelData = ModelData()
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(modelData)
        }
    }
}
```

전체적인 코드와 함께 설명해야하기에 앞에서 `@binding`, `@EnvironmentObject` 설명을 건너뛰었다.

#### 이 튜토리얼에서 Data를 선언하고 넘기는 구조 (Data-Flow)

```swift
// ModelData.swift
final class ModelData: ObservableObject {
    @Published var landmarks: [Landmark] = load("landmarkData.json")
}
```

먼저 `ModelData.swift`의 `ModelData` 클래스를 살펴보자. 이 클래스는 `ObservableObject`를 상속받는다. 이는 View가 외부 객체를 감지할 수 있도록 한다. `@Published`는 값의 변화 감지를 위해서 필요한데, 이 어노테이션이 붙어있는 landmarks 값이 업데이트 되면 `ModelData` 객체를 바라보는 View(구독자, Subscriber)를 Refresh시킨다.

```swift
// LandmarkApp.swift
@StateObject private var modelData = ModelData()
// ...
ContentView()
    .environmentObject(modelData)
```

`LandmarksApp.swift` 코드를 보면 `@StateObject` 어노테이션이 붙은 modelData 변수를 확인할 수 있다. 기존 `@State`가 `bool`, `String`, `float` 등의 Primitive 타입에 대해서 많이 사용하는데, 특정 모델을 **상태**로 쓰고 싶을때 `@StateObject` 를 사용한다. 예로 들자면 User 정보가 있겠다.

그리고 `ContentView().environmentObject(modelData)`는 modelData를 하위 View인 ContentView로 넘겨주는 역할을 한다.

> 예? 갑자기 넘겨요?

state를 저장하는 방법으로는 `@State`, `@ObservedObject`, `@EnvironmentObject`가 존재한다. `@State`는 앞에서 설명했기 때문에 나머지 두 가지를 비교해보자.

- `@ObservedObject` : 하위 View들이 쓰려면 일일히 넘겨줘야함
- `@EnvironmentObject` : Shared-Data, 최상위 View에서 넘겨주면 하위 View들이 알아서 접근 가능

기본적으로 둘 다 `@Published`로 값이 변경되었음을 알려주는 것과 View 업데이트가 되는 것도 같다. 하위 View들이 모두 접근이 가능한가, 가능하지 않은가의 차이가 있다.

이번 튜토리얼에서는 `@EnvironmentObject`를 사용한 경우이다.

```swift
// ContentView.swift
struct ContentView: View {
    var body: some View {
        LandmarkList()
    }
}
```

```swift
// LandmarkList.swift
struct LandmarkList: View {
    @EnvironmentObject var modelData: ModelData // ... (1)
    @State private var showFavoriteOnly = false
  // ... 생략
    var body: some View {
        ScrollView {
            MapView(coordinate: landmark.locationCoordinate)
                .ignoresSafeArea(edges: .top)
                .frame(height: 300)
            
            CircleImage(image: landmark.image)
                .offset(y: -130)
                .padding(.bottom, -130)
            
            VStack(alignment: .leading) {
                HStack {
                    Text(landmark.name)
                        .font(.title)
                        .foregroundColor(.primary)
                    FavoriteButton(isSet: $modelData.landmarks[landmarkIndex].isFavorite) // ... (2)
                }
  		// ... 생략
```

ContentView 내부에 LandmarkList가 존재하기 때문에 environmentObject가 자동으로 LandmarkList에 넘어간다. 따라서 (1)을 보면, `@EnvironmentObject` 어노테이션이 붙어있는 modelData는 최상위 View에서 넘겼던 modelData로 초기화된다. (LandmarkApp → ContentView → LandmarkList)

(2)를 보면 modelData는 state라서 `$`가 붙는다.

```swift
// FavoriteButton.swift
struct FavoriteButton: View {
    @Binding var isSet: Bool
  // ... 생략
```

Favorite 여부를 state로 받아오기 때문에 `@Binding` 어노테이션을 붙인다. modelData와 연결해준다고 생각하면 된다.

## 결과물

![UI-1](/Users/songhyemin/Documents/GitHub/hyemdooly.github.io/_posts/{{"assets/img/iOS/HandlingUserInput/UI-1.png"| relative_url}}){: width="50%" height="50%"}

![UI-2](/Users/songhyemin/Documents/GitHub/hyemdooly.github.io/_posts/{{"assets/img/iOS/HandlingUserInput/UI-2.png"| relative_url}}){: width="50%" height="50%"}

![UI-3](/Users/songhyemin/Documents/GitHub/hyemdooly.github.io/_posts/{{"assets/img/iOS/HandlingUserInput/UI-3.png"| relative_url}}){: width="50%" height="50%"}

![UI-4](/Users/songhyemin/Documents/GitHub/hyemdooly.github.io/_posts/{{"assets/img/iOS/HandlingUserInput/UI-4.png"| relative_url}}){: width="50%" height="50%"}

Preview로 다음과 같은 결과를 확인할 수 있다.

마지막 이미지는 3번째 이미지에서 별 버튼을 눌러 Favorite을 false로 바꾼 후 화면이다.

