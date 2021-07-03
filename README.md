# MVVM-C Architecture

### Architecture

![img](https://i.imgur.com/fPSF4XX.png)


Here's a snipped code of the MVVM-C that we're going to use in our MVP. 


### BaseCoordinator

```swift
import Combine

/// Base abstract coordinator generic over the return type of the `start` method.
open class BaseCoordinator<ResultType> {
    
    /// Typealias which will allows to access a ResultType of the Coordainator by `CoordinatorName.CoordinationResult`.
    typealias CoordinationResult = ResultType
    
    /// Utility `Set<AnyCancellable>` used by the subclasses.
    public let cancellable = Set<AnyCancellable>()
    
    /// Unique identifier.
    private let identifier = UUID()
    
    /// Dictionary of the child coordinators. Every child coordinator should be added
    /// to that dictionary in order to keep it in memory.
    /// Key is an `identifier` of the child coordinator and value is the coordinator itself.
    /// Value type is `Any` because Swift doesn't allow to store generic types in the array.
    private var childCoordinators = [UUID: Any]()
    
    /// Initializer
    public init() {}
    
    /// Stores coordinator to the `childCoordinators` dictionary.
    ///
    /// - Parameter coordinator: Child coordinator to store.
    private func store<T>(coordinator: BaseCoordinator<T>) {
        childCoordinators[coordinator.identifier] = coordinator
    }
    
    /// Release coordinator from the `childCoordinators` dictionary.
    ///
    /// - Parameter coordinator: Coordinator to release.
    private func free<T>(coordinator: BaseCoordinator<T>) {
        childCoordinators[coordinator.identifier] = nil
    }
    
    /// 1. Stores coordinator in a dictionary of child coordinators.
    /// 2. Calls method `start()` on that coordinator.
    /// 3. On the `handleEvents:` of returning observable of method `start()` removes coordinator from the dictionary.
    ///
    /// - Parameter coordinator: Coordinator to start.
    /// - Returns: Result of `start()` method.
    open func coordinate<T>(to coordinator: BaseCoordinator<T>) -> AnyPublisher<T, Never> {
        store(coordinator: coordinator)
        return coordinator.start()
            .handleEvents(receiveCompletion: { _ in self.free(coordinator: coordinator)})
            .eraseToAnyPublisher()
    }
    
    /// Starts job of the coordinator.
    ///
    /// - Returns: Result of coordinator job.
    open  func start() -> AnyPublisher<ResultType, Never> {
        fatalError("Start method should be implemented.")
    }
}
```

### Coordinator

```swift
import UIKit
import Combine

final class BrowserCoordinator: BaseCoordinator<Void> {
    
    private let window: UIWindow
    
    init(window: UIWindow) {
        self.window = window
    }
    
    override func start() -> AnyPublisher<Void, Never> {
        // setup view controller
        let viewModel = BrowserViewModel()
        let viewController = BrowserViewController(viewModel: viewModel)
        let navigationController = navigationController(with: viewController)
        
        // setup window
        window.rootViewController = navigationController
        window.makeKeyAndVisible()
        
        /// We don't have any parent coordinator to receive events
        /// from this coordinator. Therefore we will return an empty value. 
        return Empty<Void, Never>().eraseToAnyPublisher()
    }
    
    private func navigationController(with viewController: BrowserViewController) -> UINavigationController {
        let navigationController = UINavigationController(rootViewController: viewController)
        return navigationController
    }
}
```

### ViewController

```swift
import UIKit
import Combine

final class BrowserViewController: UIViewController {
    public var cancellables = Set<AnyCancellable>()
    private let mainView = BrowserView()
    let viewModel: BrowserViewModel
    
    required init(viewModel: BrowserViewModel) {
        self.viewModel = viewModel
        super.init(nibName: nil, bundle: nil)
    }
    
    /// Using Combine to bind the view and viewModel
    func bindViewModel() {
        viewModel.$webText
            .assign(to: \.text, on: mainView.webTextLabel)
            .store(in: &cancellables)
    }
    
    required init?(coder: NSCoder) { fatalError("init(coder:) has not been implemented") }
}
```

### View

```swift
import UIKit

final class BrowserView: UIView {
    /// Raw UILabel that will receive the value from the ViewModel.
    lazy var webTextLabel: UILabel = {
       let label = UILabel()
       return label
    }()
    
    required init?(coder: NSCoder) { fatalError("init(coder:) has not been implemented") }
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        addSubview(webTextLabel)
    }
}
```

### ViewModel

```swift
import Combine

final class BrowserViewModel  {
    /// Bindable property. 
    ///It can also be used as an observer and receive input from the UI.
    @Published var webText: String? = "www.duckduckgo.com"
}
```
