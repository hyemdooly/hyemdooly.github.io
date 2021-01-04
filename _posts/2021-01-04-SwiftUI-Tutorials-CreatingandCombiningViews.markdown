---
layout: post
title:  "Creating and Combining Views"
date:   2020-01-04 17:34:00 +0900
categories: iOS
---
iOS 공부를 위해서 SwiftUI Tutorials부터 차근차근 해보기로 했다. 생각보다 도큐먼트가 잘 되어있다. 다른 친구들의 이야기를 들어보니, UI쪽은 정말 잘 되어있는데 그 외에는 샘플 코드도 존재하지 않는 부분이 있다고...

첫 번째 튜토리얼부터 차근차근 해보자.

코드 느낌은 React가 생각나는 구조였다. 각 View 요소를 만들고 조립하는 느낌이다. 코드 설명은 주석으로 첨부한다.

## 전체 소스 코드

### [CircleImage.swift]

사각 이미지를 원형으로 만들어 보여주는 View이다.

```swift
import SwiftUI

struct CircleImage: View {
    var body: some View {
        Image("turtlerock")
            .clipShape(Circle()) // 원 모양으로 clip
            .overlay(Circle().stroke(Color.white, lineWidth: 4)) // 테두리
            .shadow(radius: 7) // 그림자
    }
}

struct CircleImage_Previews: PreviewProvider {
    static var previews: some View {
        CircleImage()
    }
}
```

clipShape로 원 모양으로 뻥 뚫린 Mask 비스무리한 것으로 덮어 씌우고, overlay로 테두리, shadow로 그림자를 표현한다. SwiftUI Tutorials의 설명 이미지는 아래에.

![imageview-clip]{{"assets/img/iOS/ImageView-Clip.png"| relative_url}}


오른쪽 그림을 참고하면 된다. 안드로이드에서는 복잡하게 구현해야하는 부분이었는데 iOS에서는 단 3줄이면 가능하다는게 새로웠다.

### [MapView.swift]

지도 View이다.

```swift
import SwiftUI
import MapKit

struct MapView: View {
    @State private var region = MKCoordinateRegion(
        center: CLLocationCoordinate2D(latitude: 34.011_286, longitude: -116.166_868), span: MKCoordinateSpan(latitudeDelta: 0.2, longitudeDelta: 0.2)
    ) // 좌표 관련 정보
    var body: some View {
        Map(coordinateRegion: $region)
    }
}

struct MapView_Previews: PreviewProvider {
    static var previews: some View {
        MapView()
    }
}
```

### [ContentView.swift]

body에 해당하는 ContentView이다.

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        VStack { // Vertical로 쌓기
            MapView() // 따로 구현한 MapView
                .frame(height: 300) // height => 300으로 조정
                .ignoresSafeArea(edges: .top) // 기본 top margin 삭제
            
            CircleImage() // 따로 구현한 ImageView
                .offset(y: -180) // 위치 조정
                .padding(.bottom, -180)
            
            VStack(alignment: .leading){ // leading : 좌측 정렬, center : 가운데 정렬, trailing : 우측 정렬
                Text("Turtle Rock")
                    .font(.title)
              
                HStack { // Horizontal로 쌓기
                    Text("Joshua Tree National Park")
                    Spacer() // 여백, 위 Text와 아래 Text 사이에 공백을 두어 펼쳐지도록 함
                    Text("California")
                }
                .font(.subheadline)
                .foregroundColor(.secondary)
                
                Divider() // line
                
                Text("About Turtle Rock").font(.title2)
                Text("Descriptive text goes here.")
                
            }
            .padding()
            Spacer()
        }
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```

LandmarksApp.swift는 기본과 달라진 코드가 없으므로 생략한다.

## 결과물

![Result]({{"assets/img/iOS/Result.png"| relative_url}})

Preview로 다음과 같은 결과를 확인할 수 있다.
