---
layout: post
title: "Less Code, More Creation: The Power of Declarative UI"
date: 2026-11-22 13:55:00 -0500
categories: [Development, iOS, Android]
description: "What declarative UI means, how SwiftUI and Jetpack Compose work, and why you don't need to rewrite your existing mobile app to adopt them."
---

![Heres Jonny, blog introduction image]({{ '/images/BlogLogo.png' | relative_url }})

*Why understanding declarative programming can help you develop new mobile features without starting your application over.*

## Why are we writing so much UI code?
Building a great mobile application often means writing a lot of code to create, update, and maintain the interface. As the application grows, keeping the UI in sync with the application’s state can become challenging. 

For years, UIKit and Android’s traditional view system have provided powerful ways to build applications. They aren’t outdated because newer frameworks exist, and many production applications rely on them for good reasons. However developers have increasingly adopted another way of thinking about user interfaces: Declarative UI. 

Instead of having to describe every single update that needs to happen on a screen, we describe what the screen should look like for its current state. Frameworks such as Apple’s UIKit and Google’s Jetpack Compose then coordinate the UI updates for us. While this difference may sound subtle, it changes how we organize screens and reuse componets. The best part about using declarative programming is that you can slowly adopt and you do not need to automatically rewrite an entire application. 

## Imperative vs. declarative: what's the difference?
The distinction starts with intent. 

With imperative programming, we explicitly tell the interface what to do. The user specifically tells the compiler how to handle an event. 

With a declarative approach, we express the relationship between state and the view’s appearance. It gives the compiler permission to make decisions.

![Diagram comparing a series of manual UI updates with state flowing into a declarative user interface]({{ '/images/declarative-ui/imperative-vs-declarative.svg' | relative_url }})

Consider a small feature: a button for following another user.

### The imperative version: UIKit

Here's a simplified UIKit view controller that creates a button and manually updates its title when tapped:

```swift
import UIKit

final class FollowViewController: UIViewController {
    private var isFollowing = false
    private let followButton = UIButton(type: .system)

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground

        followButton.setTitle("Follow", for: .normal)
        followButton.addTarget(
            self,
            action: #selector(toggleFollow),
            for: .touchUpInside
        )

        followButton.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(followButton)
        NSLayoutConstraint.activate([
            followButton.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            followButton.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
    }

    @objc private func toggleFollow() {
        isFollowing.toggle()
        followButton.setTitle(
            isFollowing ? "Following" : "Follow",
            for: .normal
        )
    }
}
```

There's nothing inherently wrong with this approach. UIKit gives us precise control, and the logic is easy to understand. Notice, though, that our code must explicitly update the button after the value changes. As more controls depend on the same state, we have more places to keep synchronized.

### The declarative version: SwiftUI

Now look at a similar interaction in SwiftUI:

```swift
import SwiftUI

struct FollowButton: View {
    @State private var isFollowing = false

    var body: some View {
        Button(isFollowing ? "Following" : "Follow") {
            isFollowing.toggle()
        }
        .buttonStyle(.borderedProminent)
    }
}
```

Instead of holding a reference to a button and changing its title manually, the code declares what title should be displayed for each state. When `isFollowing` changes, SwiftUI refreshes the affected UI. This declarative approach removes boilerplate needed to manage UI updates and allows developers to achieve the same functionality with fewer lines of code.

This doesn't mean SwiftUI never runs imperative code. The button's action still changes state, and your app still needs networking, persistence, and business logic. **Declarative describes how the UI is expressed—not how every part of the application must be written.**

### The Android version: Jetpack Compose

Jetpack Compose follows the same general idea, using Kotlin:

```kotlin
@Composable
fun FollowButton() {
    var isFollowing by remember { mutableStateOf(false) }

    Button(onClick = { isFollowing = !isFollowing }) {
        Text(if (isFollowing) "Following" else "Follow")
    }
}
```

`remember` keeps this example's local state across recompositions. When that state changes, Compose can re-execute the relevant composable functions to update what users see—a process called **recomposition**. In a real feature, a ViewModel may own the state, rather than the button itself.

The three examples intentionally use a small interaction; line counts alone aren't a useful benchmark because the UIKit example also includes view setup and layout. The important difference is **who takes responsibility for synchronizing the visible UI with application state**.

## Two platforms, a shared philosophy

Apple's **SwiftUI** and Google's **Jetpack Compose** are distinct frameworks built for different ecosystems, but they share several useful ideas.
Both Frameworks: 
- Encourage developers to assemble complex screens from smaller, reusable pieces. 
- Make interface output depend on observable state. 
- Support previews and fast visual iteration in their respective development tools. 
- Can live alongside their platforms' established UI frameworks.

That doesn't mean SwiftUI code can be copied directly into Compose, or that both systems behave identically. You will find that their languages, lifecycle models, navigation APIs, styling conventions, and state-management tools differ. Yet once you understand the declarative model, the underlying mental approach will feel familiar across both platforms.

![Side-by-side diagram of incremental UI integration on iOS and Android]({{ '/images/declarative-ui/incremental-adoption.svg' | relative_url }})

## You don't have to rebuild your app to get started

This is the part I find most useful for developers working on established applications: **declarative UI can be introduced incrementally**.

Imagine your team maintains a UIKit application that already has authentication, networking, navigation, and years of tested functionality. Replacing everything with SwiftUI could require a lot of work with uncertain payoff. But you could build a new profile feature in SwiftUI and display it through a `UIHostingController`:

```swift
import UIKit
import SwiftUI

// Inside an existing UIKit UIViewController:
let profile = ProfileView()
let hostingController = UIHostingController(rootView: profile)
navigationController?.pushViewController(hostingController, animated: true)
```

The existing UIKit navigation controller can continue coordinating navigation while the new screen is written in SwiftUI. You can also embed a hosting controller inside an existing UIKit screen, using proper view-controller containment, when only part of the screen needs to be declarative.

Android offers a comparable path. If an Activity or Fragment already uses Android Views, a `ComposeView` can display composable content inside that existing hierarchy:

```kotlin
val composeView = ComposeView(requireContext()).apply {
    setViewCompositionStrategy(
        ViewCompositionStrategy.DisposeOnViewTreeLifecycleDestroyed
    )
    setContent {
        ProfileCard()
    }
}
```

This Android snippet is intended for use inside a Fragment and assumes the required Compose dependencies and imports are configured. You would still attach `composeView` to the Fragment's layout or return it from the appropriate view-creation method. The lifecycle-aware disposal strategy is particularly important when Compose is hosted inside a Fragment view.

Neither approach automatically migrates your data layer or navigation architecture. You still need to decide who owns shared state, how data flows into the new screen, and how events travel back to the legacy code. **But it gives you a way to adopt new UI technology one feature at a time.**

## How declarative UI can make development more efficient

When the right feature is chosen, declarative frameworks can offer several practical advantages.

### Less routine UI synchronization

When a view directly expresses its state you spend less time writing code that manually sets labels, hides components, and adjusts UI properties after every event. This will reduce “the data changed but the screen didn't” bugs.

### Reusable building blocks

A SwiftUI `View` or a Compose `@Composable` can encapsulate a small part of the interface. Instead of rebuilding a profile card or status badge on every screen, you can reuse the same component with different inputs. This can improve consistency and make changes easier to apply throughout the product.

### Faster feedback during development

SwiftUI previews and Compose previews make it possible to inspect many UI changes without navigating through the entire application each time. Previews aren't a replacement for simulator or device testing, but they're useful for iterating on layout, states, and components.

### UI states become easier to describe

A feature can present loading, content, error, and empty states as outcomes of a state model. Rather than scattering update instructions across multiple callbacks, the view can express which interface belongs to each case.

![Four interface states displayed as reusable cards: loading, content, empty, and error]({{ '/images/declarative-ui/state-driven-ui.svg' | relative_url }})

## When should you introduce it?

Before adopting SwiftUI or Compose for a new feature, I would ask a few questions:

1. **Is the feature reasonably isolated?** A screen with clear inputs and actions is often easier to integrate than a flow coupled to many legacy controllers.
2. **Where does the state live?** Decide whether a UIKit controller, Android ViewModel, shared domain layer, or new feature model owns the source of truth.
3. **Who owns navigation?** Let the existing navigation system remain in control unless there's a deliberate reason to migrate it.
4. **Can the team test and maintain both approaches?** Plan for previews, accessibility checks, UI tests, and consistent code review practices.
5. **Does the benefit justify the integration cost?** A stable legacy screen may not need rewriting; new features are often better migration candidates.

For developers learning both frameworks, I'd recommend building the same small feature—such as a task card, profile screen, or follow button—once in the existing UI system and once declaratively. Seeing how state and UI are connected in each version is more useful than memorizing a list of syntax differences.

## Progress doesn't require starting over

Declarative programming isn't a magic solution for every development challenge. It won't automatically fix poor architecture, produce accessible interfaces, or make every app run faster. What it does offer is a more direct way to describe how an interface should respond to the state of an application.

For teams with existing UIKit or Android View-based applications, the most exciting part may be the freedom to experiment without abandoning what's already working. You can preserve proven systems, introduce modern UI components where they're useful, and learn from each feature you build.

As developers, we're often excited by the newest tools and frameworks. But progress doesn't always require a complete rewrite. Sometimes the better engineering decision is to improve what we have, one thoughtfully designed feature at a time.

**Writing better software isn't always about writing more code. Sometimes, it's about finding a clearer way to express what we want our code to do.**

### Continue learning

- [Apple: SwiftUI — Declaring a custom view](https://developer.apple.com/documentation/swiftui/declaring-a-custom-view)
- [Apple: Integrating SwiftUI with UIKit](https://developer.apple.com/documentation/swiftui/uikit-integration)
- [Google: Thinking in Compose](https://developer.android.com/develop/ui/compose/mental-model)
- [Google: Migration strategy for Jetpack Compose](https://developer.android.com/develop/ui/compose/migrate/strategy)
- [Google: Using Compose in Views](https://developer.android.com/develop/ui/compose/migrate/interoperability-apis/compose-in-views)
