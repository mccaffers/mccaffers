---
sidebar_position: 4
title: "Working with Objective C and Swift"
description: "In the process of taking an older iOS application, I've been mixing Objective C with Swift and SwiftUI"
keywords: ["Software Engineering","Swift","iOS","ObjectiveC"]
tags: ["Software Engineering"]
---

## Objective C and Swift

Objective C is a high level programming lanaguage, which became the main programming language for developing applications on macOS and iOS over the last 30 years. It's primarily influenced by C, with all objects represented as pointers and prior to v2.0 you would have to maintain references to retain and release objects. 

In 2014 Apple introduced Swift characterised as "Objective-C without the C". A multi-paradigm, open source, compiled language which uses the Objective-C runtime library, enabling the mixture of C, Objective C, C++ and Swift in one application.

## Removing the Storyboard
I had an older iOS application written in Objective C with a Storyboard that I wanted to remove. Storyboard are a visual representation of your UI but (in my opinion) are hard to deal with. You can get megre conflict easily, that are hard to fix and you can't programmatically fix issues (unless you enjoy editing XML).

To remove the storyboard, I looked at changing the `rootViewController` in the `AppDelegate` or the `SceneDelegate` (for >iOS 13.0), but the cleanest way was to use a `UIViewControllerRepresentable` and call it directly within the Swift Main App.

> Use a UIViewControllerRepresentable instance to create and manage a UIViewController object in your SwiftUI interface. Adopt this protocol in one of your app’s custom instances, and use its methods to create, update, and tear down your view controller.
> https://developer.apple.com/documentation/swiftui/uiviewcontrollerrepresentable

## UIViewController in Objective C

Lets say you've got a `UIViewController` (called `ObjectiveCViewController`) written in Objective C, using a Storyboard as the main entry point, like so:

**ObjectiveCViewController.m**
```objectivec {linenos=table,linenostart=1}
#import "ObjectiveCViewController.h"
#import "Masterview.h"

@interface ObjectiveCViewController ()

@end

@implementation ObjectiveCViewController

- (void)viewDidLoad
{
    [super viewDidLoad];

    // If you've setup the view to be called from a storyboard
    // you may need to initiate the first view inside the UIViewController
    self.view = [[MasterView alloc] init];

}

- (void)viewDidUnload
{
    [super viewDidUnload];
    // Release any retained subviews of the main view.
}

- (BOOL)shouldAutorotateToInterfaceOrientation:(UIInterfaceOrientation)interfaceOrientation {
    return (interfaceOrientation == UIInterfaceOrientationLandscapeLeft || 
            interfaceOrientation == UIInterfaceOrientationLandscapeRight);
}

@end
```

With a header file:

**ObjectiveCViewController.h**
```objectivec {linenos=table,linenostart=1}
#import <UIKit/UIKit.h>

@interface ObjectiveCViewController : UIViewController
@end

```

If you create a new Swift iOS application, you'll get an an `App.Swift` with a `@main` entrypoint, like so:

**App.Swift**
```swift {linenos=table,linenostart=1}
@main
struct MyApp: App {

      var body: some Scene {
          WindowGroup {
            Content()
          }
      }
}
```

## Final result

You can build a custom UIViewController entry point for the Swift Application

**MyViewControllerWrapper.swift**
```swift {linenos=table,linenostart=1}
struct MyViewControllerWrapper: UIViewControllerRepresentable {
    typealias UIViewControllerType = ObjectiveCViewController
    
    func makeUIViewController(context: Context) -> ObjectiveCViewController {
        return ObjectiveCViewController()
    }
    
    func updateUIViewController(_ uiViewController: ObjectiveCViewController, context: Context) {
        // Updates the state of the specified view controller with new information from SwiftUI.
    }
}
```

Then you can directly reference this `MyViewControllerWrapper` in the Swift app main, like so:

**App.Swift**
```swift {linenos=table,linenostart=1}
@main
struct MyApp: App {

      var body: some Scene {
          WindowGroup {
            MyViewControllerWrapper()
          }
      }
}
```

## Bridging Header

Xcode should generate a `project-Bridging-Header.h` in the root of the project, but just incase it doesn't you need this file to contain:

```objectivec {linenos=table,linenostart=1}
//
//  Use this file to import your target's public headers that you would like to expose to Swift.
//

#import "ObjectiveCViewController.h"
```

Thats it! You can now refactor older Objective C methods to use Swift and SwiftUI, moving away from square brackets!