---
url: custom-modifier
title: 在 SwiftUI 中自定义修饰器
date: 2021-07-13 10:41:14
categories: [技术]
tags: [SwiftUI]
---

学习 swiftUI 的记录。

<!--more-->

首先定义结构体，在其中对内容修饰：

```swift
struct Watermark: ViewModifier {
    var text: String

    func body(content: Content) -> some View {
        ZStack(alignment: .bottomTrailing) {
            content
            Text(text)
                .font(.caption)
                .foregroundColor(.white)
                .padding(5)
                .background(Color.black)
        }
    }
}
```

然后根据此结构体，定义 `extension`，在其中定义修饰器函数：

```swift
extension View {
    func watermarked(with text: String) -> some View {
        self.modifier(Watermark(text: text))
    }
}
```

最终只需调用 `extension` 中的函数即可实现：

```swift
Color.green
    .frame(width: 300, height: 200)
    .watermarked(with: "Hacking with Swift")
```

效果图：

<img src="https://i0.hdslb.com/bfs/album/e411afa5889eabb69f3512b4f2a6771cfbb5bd97.png" alt="image-20210713104903030" style="zoom:50%;" />
