**场景：**
你的 Pod 组件库同时包含 Swift 和 Objective-C 文件，并已集成到主工程中。现在需要在主工程的 Objective-C 代码中调用 Pod 内部的 Swift 代码。

**核心原理：**
Xcode 会为 Pod 中的 `@objc` Swift 代码自动生成一个 Objective-C 兼容的头文件，通过导入这个头文件，Objective-C 就能识别并调用 Swift 代码。

**关键步骤：**

1.  **标记 Pod 中的 Swift 代码：**
    *   在 Pod 的 Swift 文件中，所有需要从 Objective-C 访问的类和方法必须：
        *   继承自 `NSObject` (或其子类)。
        *   使用 `@objc` 属性进行标记。
        *   访问权限应为 `public` 或 `internal`。
    *   **示例 Swift 代码 (在 Pod 中):**
        ```swift
        import Foundation
        @objc public class MySwiftClassInPod: NSObject {
            @objc public func doSomethingInSwift() { /* ... */ }
            @objc public func calculateSum(_ a: Int, _ b: Int) -> Int { return a + b }
        }
        ```

2.  **导入自动生成的 Swift 兼容头文件：**
    *   Xcode 会为你的 Pod 自动生成一个名为 `<YourPodName>-Swift.h` 的头文件。
    *   **注意：** `YourPodName` 是你的 Pod 在 `podspec` 文件中定义的名称 (`s.name`)，而不是主工程的名称。
    *   在主工程中需要调用 Pod Swift 代码的 Objective-C `.m` 或 `.mm` 文件中，导入此头文件。
    *   **示例 Objective-C 导入 (在主工程中):**
        ```objective-c
        #import "YourPodName-Swift.h" // 替换 YourPodName 为你的Pod实际名称
        ```

3.  **在 Objective-C 中调用 Swift 代码：**
    *   导入头文件后，你就可以像使用任何其他 Objective-C 类一样，实例化 Pod 中的 Swift 类并调用其 `@objc` 方法。
    *   **示例 Objective-C 调用 (在主工程中):**
        ```objective-c
        // ...
        MySwiftClassInPod *swiftObject = [[MySwiftClassInPod alloc] init];
        [swiftObject doSomethingInSwift];
        NSInteger result = [swiftObject calculateSum:10 :20];
        NSLog(@"Result from Pod Swift: %ld", (long)result);
        // ...
        ```

**注意事项：**
*   确保 Pod 的 Build Settings 中 `DEFINES_MODULE` 设置为 `YES` (CocoaPods 通常会自动处理)。
*   正确使用 Pod 的模块名称来导入 `import <YourPodName/YourPodName-Swift.h>`。

---