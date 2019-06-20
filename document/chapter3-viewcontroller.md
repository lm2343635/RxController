# Chapter 3: View controller and view model

This chapter introduces the rule of view controller and view model classes.

## 3.1 Using RxViewController

RxController provides a basic view controller `RxViewController` (generic classes) and a basic view model `RxViewModel`.

We recommend to create a BaseViewController which extends RxViewController and a BaseViewModel which extends RxViewController.
Developers can customized something in the BaseViewController and BaseViewModel class.

```swift
class BaseViewController<ViewModel: BaseViewModel>: RxViewController<ViewModel> {
    // Customize somthing here...
}

class BaseViewModel: RxViewModel {
    // Customize somthing here...
}
```

**All the view controller classes should extend the BaseViewController.**

**All the view model classes should extend the BaseViewModel**

## 3.2 Structure of view controller

The code in the view controller should follow the order:

#### Define views

**Views should be defined with `private lazy var`.**

```swift
private lazy var nameLabel: UILabel = {
    let label = UILabel()
    label.textColor = UIColor(hex: 0x222222)
    label.font = UIFont.systemFont(ofSize: 14)
    return label
}()
```

**A closure should be used when some properties of this view need to be set.**
Otherwise, we omit the type and invoke the init method directly.

```swift
private lazy var nameLabel = UILabel()
```

**The order of the views should be same as the design.** 
The left views and top views should be in the top of the right views and the bottom views.

When we need a parent view a container view, **the container view should be defined after its subviews.**

```swift
private lazy var nameLabel = UILabel()

private lazy var iconImage = UIImageView()

private lazy var containerView: UILabel = {
    let view = UIView()
    view.addSubview(nameLabel)
    view.addSubview(iconImage)
    return view
}()
```

**The view definition closure does not contains the definition of subviews.** 
The following code is not recommended.

```swift
// NOT recommended.
private lazy var containerView: UILabel = {
    let view = UIView()
    let nameLabel = UILabel()
    view.addSubview(nameLabel)
    return view
}()
```

**The view definition closurecontains the properties and methods of this view only.**

### Define child view controllers

We define child view controllers using `private lazy var` after the definition of views.

```swift
private lazy var childViewController = ChildViewController(viewModel: init())
```

### Define data source of RxDataSources

We define the data source of RxDataSources using `private lazy var` after the definition of child view controllers.

```swift
private lazy var dataSource = DemoTableViewCell.tableViewSingleSectionDataSource()
```

### Define other private properties

Private properties are not recommended in the view controller.
The state properties are recommened to define in the view model class.
However, if needed, define them here.

### Init method

Set some properties of the view controller in `init` method if needed.

```swift
override init(viewModel: DemoViewModel) {
    super.init(viewModel: viewModel)
    
    modalPresentationStyle = .overCurrentContext
    modalTransitionStyle = .crossDissolve
}

required init?(coder aDecoder: NSCoder) {
    fatalError("init(coder:) has not been implemented")
}
```

Just invoke `fatalError` method in `required init?(coder aDecoder: NSCoder)` because we do not support storyboard.

### viewDidLoad method

The following steps should be contained in the `viewDidLoad` method.

- Set navigation bar and navigation items.
- Set peroperties of root view.
- Add subviews.
- Create constraints.
- Add child view controllers.
- Bind data.

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    // Set navigation bar and navigation items.
    navigationItem.leftBarButtonItem = closeBarButtonItem
    
    // Set peroperties of root view.
    view.backgroundColor = .white
    // Add subviews.
    view.addSubview(titleLabel)
    view.addSubview(tableView)
    // Create constraints.
    createConstraints()
    
    //Add child view controllers.
    addChild(topViewController, to: headerView)
    
    // Bind data.
    disposeBag ~ [
        viewModel.title ~> titleLabel.rx.text,
        viewModel.cartSection ~> tableView.rx.items(dataSource: dataSource)
    ]
}
```

We use the operator `~>` and `<~>` of RxBinding to bind data.
**The order of the bind code should be same as the order of the definition of the views.**

### Other lifecycle methods.

Add other lifecycle methods here if needed.

### Create constraints method.

A contraints method contains the SnapKit constraint creators of all the views in this view controller.

**The constants used in the closure should be define in a private enum `Const` before the view controller**

```swift
private func createConstraints() {
    
    titleLabel.snp.makeConstraints {
        $0.centerX.equalToSuperview()
        $0.top.equalToSuperview().offset(Const.Title.marginTop)
    }
    
    closeButton.snp.makeConstraints {
        $0.centerY.equalTo(titleLabel.snp.centerY)
        $0.right.equalToSuperview().offset(-Const.Close.marginRight)
        $0.size.equalTo(Const.Close.size)
    }

}
```

**The sub enum in the `Const` enum is corresponding to the subview, and it name is the prefix of the subview's name.**

```swift
demoView -> enum Demo
demoLabel -> enum Demo
```

The demo code of `Const` enum:

```swift
private enum Const {
    
    enum Title {
        static let marginTop = 11
        static let marginBottom = 17
    }
    
    enum Close {
        static let size = 24
        static let marginRight = 11
    }
    
}
```

## 3.3 Using child view controller or customized view

A view controller needs multiple child view controller or customized view to reduce the complexity.
**Cusztomized view is recommended for showing data only, or handling a simple action like tap.**
We don't receommend to use RxSwift directly in a customized view.
**To handle complex actions, a child view controller is receommened.**
A child view controller also extends the BaseViewController, so it can take advantage of view model and RxSwift.

![Platform](https://raw.githubusercontent.com/lm2343635/RxController/master/images/child_view_controllers.jpg)

**Using a container view is recommended for a child view controller.**
For example, to add the `childViewController1` into the parent view controller, a container view `containerView1` should be prepared at first.
The constraints is applied to the containerView1.
When we invoke the `addChild(childViewController1, to: containerView1)`, the root view of the child view controller will be added to `containerView1`.
The edges of the root view is same as `containerView1`.

A child view controller (`childViewController2`) may have its own child view controllers (`childViewController3`).