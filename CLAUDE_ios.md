### 🔄 Project Awareness & Context

- **Always read `PLANNING.md`** at the start of a new conversation to understand the project's architecture, goals, style, and constraints.
- **Check `TASK.md`** before starting a new task. If the task isn't listed, add it with a brief description and today's date.
- **Use consistent naming conventions, file structure, and architecture patterns** as described in `PLANNING.md`.
- **Configure Xcode schemes** for different environments (Development, Staging, Production) with appropriate environment variables.

### 🧱 Code Structure & Modularity

- **Never create a file longer than 500 lines of code.** If a file approaches this limit, refactor by splitting it into smaller components or helper files.
- **Organize code into clearly separated modules**, grouped by feature or responsibility.

  **Recommended project structure:**

  ```
  ProjectName/
  ├── Application/           # AppDelegate, SceneDelegate, Info.plist
  ├── Features/             # Feature-based modules
  │   ├── FeatureName/
  │   │   ├── Views/       # SwiftUI Views or UIViewControllers
  │   │   ├── ViewModels/  # Business logic (MVVM)
  │   │   ├── Models/      # Data structures
  │   │   └── Services/    # Feature-specific services
  ├── Shared/
  │   ├── Extensions/      # Swift extensions
  │   ├── Utilities/       # Helper functions
  │   ├── Constants/       # Constants.swift for static values
  │   └── Services/        # Shared services (networking, etc.)
  ├── Resources/
  │   └── Assets.xcassets/
  └── Tests/               # Mirror main app structure
  ```

- **Use clear, consistent imports** (import only what you need; prefer whole module imports).
- **Follow MVVM or MVVM-C architecture pattern** for better separation of concerns and testability.
- **Use dependency injection** via initializers to make code testable.
- **Prefer protocols over concrete types** for better abstraction and testability.

### 🧪 Testing & Reliability

- **Always create XCTest unit tests for new features** (functions, classes, view models, services, etc).
- **After updating any logic**, check whether existing unit tests need to be updated. If so, do it.
- **Tests should live in a `/Tests` folder** mirroring the main app structure.
- **Follow the Arrange-Act-Assert (AAA) pattern** for test structure:

  ```swift
  func testFeatureName_whenCondition_shouldExpectedResult() {
      // Arrange
      let sut = SystemUnderTest()

      // Act
      let result = sut.performAction()

      // Assert
      XCTAssertEqual(result, expectedValue)
  }
  ```

- Include at least:
  - 1 test for expected use
  - 1 edge case
  - 1 failure case
- **Use the most specialized assertion** (e.g., `XCTAssertNil` instead of `XCTAssert` for nil checks).
- **Never weaken encapsulation for testing** – don't make things public just for tests; use protocols instead.

### ✅ Task Completion

- **Mark completed tasks in `TASK.md`** immediately after finishing them.
- Add new sub-tasks or TODOs discovered during development to `TASK.md` under a "Discovered During Work" section.

### 📎 Style & Conventions

- **Use Swift** as the primary language.
- **Follow Apple's Swift API Design Guidelines** (https://swift.org/documentation/api-design-guidelines/).
- **Use SwiftFormat and SwiftLint** for code formatting and linting:
  - SwiftFormat for automatic code formatting (like black for Python)
  - SwiftLint for style checking and API misuse detection
  - Configure both in your project with `.swiftformat` and `.swiftlint.yml` files
- **Naming conventions:**
  - Types and protocols: `UpperCamelCase`
  - Everything else: `lowerCamelCase`
  - ViewControllers: `FeatureNameViewController` (UIKit)
  - Views: `FeatureNameView` (SwiftUI)
  - ViewModels: `FeatureNameViewModel`
  - Services: `FeatureNameService`
- **Type safety:**
  - Avoid force unwrapping (`!`) – use safe unwrapping methods
  - Avoid optional collections – prefer empty collections instead
  - Use explicit type annotations when type inference is ambiguous
- **Swift concurrency:**
  - Prefer `async`/`await` over completion handlers
  - Use `@MainActor` for UI updates
  - Understand and use structured concurrency
- **Write documentation comments for every public function/type** using Swift's documentation format:
  ```swift
  /// Brief summary of what this function does.
  ///
  /// Detailed description providing more context about the function's
  /// purpose and usage.
  func exampleFunction(parameter1: String, parameter2: Int) throws -> Bool {
      // Implementation
  }
  ```

### 📦 Package Management

- **Use Swift Package Manager (SPM)** as the preferred dependency manager.
- SPM is built into Xcode – no external tools needed.
- For legacy projects with CocoaPods, both can coexist, but migrate to SPM when possible.

### 🎨 SwiftUI vs UIKit

- **SwiftUI:** Modern, declarative UI framework (iOS 13+)
  - Use for new features and projects
  - Combines well with Combine framework
  - Natural fit for MVVM architecture
- **UIKit:** Traditional, imperative UI framework
  - Still widely used in production
  - May be required for legacy code or advanced customization
- **Both can coexist** in the same project. Prefer SwiftUI, but use UIKit when SwiftUI can't be used.

### 📚 Documentation & Explainability

- **Update `README.md`** when new features are added, dependencies change, or setup steps are modified.
- **Comment non-obvious code** and ensure everything is understandable to a mid-level iOS developer.
- When writing complex logic, **add an inline `// Reason:` comment** explaining the why, not just the what.

### 🔐 Security & Best Practices

- **Never commit secrets** (API keys, tokens) to the repository.
- **Use Xcode schemes and environment variables** for configuration:
  - Product → Scheme → Edit Scheme → Run → Arguments
  - Environment Variables for runtime configuration
  - Build configurations for compile-time settings
- **Use Keychain** for storing sensitive data locally.
- **Validate all user input** and sanitize data before use.
- **Follow iOS security best practices** for authentication, data storage, and networking.

### 🧠 AI Behavior Rules

- **Never assume missing context. Ask questions if uncertain.**
- **Never hallucinate libraries, frameworks, or APIs** – only use known, verified Swift packages and iOS frameworks.
- **Always confirm file paths and type names** exist before referencing them in code or tests.
- **Never delete or overwrite existing code** unless explicitly instructed to or if part of a task from `TASK.md`.
- **Be aware of iOS version compatibility** – check minimum deployment target before using APIs.
- **Understand the difference between SwiftUI and UIKit** – don't mix concepts inappropriately.
