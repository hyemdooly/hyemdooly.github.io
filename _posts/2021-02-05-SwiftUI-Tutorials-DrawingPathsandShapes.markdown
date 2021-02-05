---
layout: post
title:  "Drawing Paths and Shapes"
date:   2020-02-05 15:21:00 +0900
categories: iOS

---

Badge를 그리는 튜토리얼이다. 수학스러운 내용이 많아서 머릿속이 굉장히 복잡했던 부분이다.

## 디렉토리 구조

![Directory]({{"assets/img/iOS/DrawingPathsandShapes/Directory.png"| relative_url}})

이번 프로젝트에서는 다음과 같이 디렉토리 구조를 정리한다. Badge 디렉토리만 집중해서 보면 된다.

## 전체 소스 코드

### [HexagonParameter.swift]

`line`, `curve`, `control`가 정의되어 있고, 육각형을 만들기 위한 요소 set이 담긴 코드이다.

```swift
import CoreGraphics

struct HexagonParameters {
    struct Segment {
        let line: CGPoint
        let curve: CGPoint
        let control: CGPoint
    }
    static let adjustment: CGFloat = 0.085 // 모양 조정을 위한 값
    
    static let segments = [
        Segment(
            line:    CGPoint(x: 0.60, y: 0.05),
            curve:   CGPoint(x: 0.40, y: 0.05),
            control: CGPoint(x: 0.50, y: 0.00)
        ),
        Segment(
            line:    CGPoint(x: 0.05, y: 0.20 + adjustment),
            curve:   CGPoint(x: 0.00, y: 0.30 + adjustment),
            control: CGPoint(x: 0.00, y: 0.25 + adjustment)
        ),
        Segment(
            line:    CGPoint(x: 0.00, y: 0.70 - adjustment),
            curve:   CGPoint(x: 0.05, y: 0.80 - adjustment),
            control: CGPoint(x: 0.00, y: 0.75 - adjustment)
        ),
        Segment(
            line:    CGPoint(x: 0.40, y: 0.95),
            curve:   CGPoint(x: 0.60, y: 0.95),
            control: CGPoint(x: 0.50, y: 1.00)
        ),
        Segment(
            line:    CGPoint(x: 0.95, y: 0.80 - adjustment),
            curve:   CGPoint(x: 1.00, y: 0.70 - adjustment),
            control: CGPoint(x: 1.00, y: 0.75 - adjustment)
        ),
        Segment(
            line:    CGPoint(x: 1.00, y: 0.30 + adjustment),
            curve:   CGPoint(x: 0.95, y: 0.20 + adjustment),
            control: CGPoint(x: 1.00, y: 0.25 + adjustment)
        )
    ]
}
```

### [BadgeBackground.swfit]

Badge의 배경을 담당하는 코드이다.

```swift
import SwiftUI

struct BadgeBackground: View {
    var body: some View {
        GeometryReader { geometry in
            Path { path in
                var width: CGFloat = min(geometry.size.width, geometry.size.height)
                let height = width
                let xScale: CGFloat = 0.832
                let xOffset = (width * (1.0 - xScale)) / 2.0 // 가운데 정렬하기 위함
                width  *= xScale // 비율 조정
                path.move (
                    to: CGPoint(
                        x: width * 0.95 + xOffset,
                        y: height * (0.20 + HexagonParameters.adjustment)
                    )
                )
                
                HexagonParameters.segments.forEach { segment in // addLine + addQuadCurve => 선을 그리고 꺾는 역할
                    path.addLine(
                        to: CGPoint(
                            x: width * segment.line.x + xOffset,
                            y: height * segment.line.y
                        )
                    )
                    
                    path.addQuadCurve(
                        to: CGPoint(
                            x: width * segment.curve.x + xOffset,
                            y: height * segment.curve.y
                        ),
                        control: CGPoint(
                            x: width * segment.control.x + xOffset,
                            y: height * segment.control.y
                        )
                    )
                }
            }
            .fill(LinearGradient(
                gradient: Gradient(colors: [Self.gradientStart, Self.gradientEnd]),
                startPoint: UnitPoint(x: 0.5, y: 0),
                endPoint: UnitPoint(x: 0.5, y: 0.6)
            ))
        }
        .aspectRatio(1, contentMode: .fit)
    }
    static let gradientStart = Color(red: 239.0 / 255, green: 120.0 / 255, blue: 221.0 / 255)
    static let gradientEnd = Color(red: 239.0 / 255, green: 172.0 / 255, blue: 120.0 / 255)
}

struct BadgeBackground_Previews: PreviewProvider {
    static var previews: some View {
        BadgeBackground()
    }
}
```

#### GeometryReader?

SwiftUI를 작성할 때 View를 배치하기 위해 여러가지 방법을 사용하게 되는데, 그 중 하나가 `GeometryReader` 이다. 괄호 내부를 보면 `geometry`가 선언되어있는 것을 볼 수 있고, `geometry.size.width`와 같이 크기를 받아오는 것을 확인할 수 있다. `GeometryReader`는 사용가능한 넓이와 높이를 알려주는 입력 값을 제공한다. View의 부모 View 사이즈를 기준으로 본인의 사이즈를 조절하고 싶을 때 사용한다.

### [BadgeSymbol.swift]

![BadgeSymbol]({{"assets/img/iOS/DrawingPathsandShapes/BadgeSymbol.png"| relative_url}})

심볼을 작성하는 코드이다.

```swift
import SwiftUI

struct BadgeSymbol: View {
    static let symbolColor = Color(red: 79.0 / 255, green: 79.0 / 255, blue: 191.0 / 255)
    
    var body: some View {
        GeometryReader { geometry in
            Path { path in
                let width = min(geometry.size.width, geometry.size.height)
                let height = width * 0.75
                let spacing = width * 0.030
                let middle = width * 0.5
                let topWidth = width * 0.226
                let topHeight = height * 0.488
                
                path.addLines([
                    CGPoint(x: middle, y: spacing),
                    CGPoint(x: middle - topWidth, y: topHeight - spacing),
                    CGPoint(x: middle, y: topHeight / 2 + spacing),
                    CGPoint(x: middle + topWidth, y: topHeight - spacing),
                    CGPoint(x: middle, y: spacing)
                ])
                
                path.move(to: CGPoint(x: middle, y: topHeight / 2 + spacing * 3))
                path.addLines([
                    CGPoint(x: middle - topWidth, y: topHeight + spacing),
                    CGPoint(x: spacing, y: height - spacing),
                    CGPoint(x: width - spacing, y: height - spacing),
                    CGPoint(x: middle + topWidth, y: topHeight + spacing),
                    CGPoint(x: middle, y: topHeight / 2 + spacing * 3)
                ])
                
            }
            .fill(Self.symbolColor)
        }
    }
}

struct BadgeSymbol_Previews: PreviewProvider {
    static var previews: some View {
        BadgeSymbol()
    }
}
```

### [RotatedBadgeSymbol.swift]

위 코드에서 작성된 심볼을 Rotate하기 위한 코드이다. Preview로 degrees에 5를 넣은 값에 맞게 5도 돌아간 심볼을 확인할 수 있다.

```swift
import SwiftUI

struct RotatedBadgeSymbol: View {
    let angle: Angle
    var body: some View {
        BadgeSymbol()
            .padding(-60)
            .rotationEffect(angle, anchor: .bottom)
    }
}

struct RotatedBadgeSymbol_Previews: PreviewProvider {
    static var previews: some View {
        RotatedBadgeSymbol(angle: Angle(degrees: 5))
    }
}
```

### [Badge.swift]

```swift
import SwiftUI

struct Badge: View {
    static let rotationCount = 8
    // rotationCount값에 맞게 각도 계산해서 돌려가며 ForEach로 반복해서 그리기
    var badgeSymbols: some View {
        ForEach(0..<Badge.rotationCount) { i in
            RotatedBadgeSymbol(
                angle: .degrees(Double(i) / Double(Badge.rotationCount)) * 360.0
            )
        }
        .opacity(0.5)
    }
    
    var body: some View {
        ZStack {
            BadgeBackground()
            
            GeometryReader { geometry in
                badgeSymbols
                    .scaleEffect(1.0 / 4.0, anchor: .top)
                    .position(x: geometry.size.width / 2.0, y: (3.0 / 4.0) * geometry.size.height)
            }
        }
        .scaledToFit()
    }
}

struct Badge_Previews: PreviewProvider {
    static var previews: some View {
        Badge()
    }
}
```

## 결과물

![UI]({{"assets/img/iOS/DrawingPathsandShapes/UI.jpg"| relative_url}}){: width="50%" height="50%"}

Preview로 다음과 같은 결과를 확인할 수 있다.

