# MVVM-C Architecture

Here's a snipped code of the MVVM-C that we're going to use in our MVP. 

### Coordinator

```swift
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
final class BrowserViewModel  {
    /// Bindable property. 
    ///It can also be used as an observer and receive input from the UI.
    @Published var webText: String? = "www.duckduckgo.com"
}
```
