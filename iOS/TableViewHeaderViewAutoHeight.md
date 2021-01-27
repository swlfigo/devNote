## 在 TableHeaderView 中使用 AutoLayout

没有显式的 frame 计算。

> 使用 `UIViewController` 和 `UITableView` 作为子视图，而不是`UITableViewController`。

要点:

1. 设置 `table header view`
2. 将 `header view` 的 `centerX`，`width` 和`top` 锚点固定到 `table view`.
3. 用 `tableHeaderView` 调用 `layoutIfNeeded` 更新他的大小。
4. **重新赋值 `tableHeaderView`**

## Code

```swift
// ...In viewDidLoad()
// 1.
let containerView = UIView()
containerView.translatesAutoresizingMaskIntoConstraints = false
// headerView is your actual content.
containerView.addSubview(headerView)

// 2.
self.tableView.tableHeaderView = containerView
// 3. 设置锚点
containerView.centerXAnchor.constraint(equalTo: self.tableView.centerXAnchor).isActive = true
containerView.widthAnchor.constraint(equalTo: self.tableView.widthAnchor).isActive = true
containerView.topAnchor.constraint(equalTo: self.tableView.topAnchor).isActive = true
// 4.
self.tableView.tableHeaderView?.layoutIfNeeded()
self.tableView.tableHeaderView = self.tableView.tableHeaderView


//注意:
//需要创建一个 DummyView
//真正的 HeaderView 是添加在DummyView上面
```

## To update the header frame on device rotation

处理设备旋转需要重新布局的情况：

```swift
override func viewWillTransition(to size: CGSize, with coordinator: UIViewControllerTransitionCoordinator) {
  super.viewWillTransition(to: size, with: coordinator)
  DispatchQueue.main.async {
    self.tableView.tableHeaderView?.layoutIfNeeded()
    self.tableView.tableHeaderView = self.tableView.tableHeaderView
  }
}
```



## Extension

```swift
extension UITableView {
    //set the tableHeaderView so that the required height can be determined, update the header's frame and set it again
    func setAndLayoutTableHeaderView(header: UIView) {
        self.tableHeaderView = header
        header.setNeedsLayout()
        header.layoutIfNeeded()
        header.frame.size = header.systemLayoutSizeFittingSize(UILayoutFittingCompressedSize)
        self.tableHeaderView = header
    }
}
```



使用:

```swift
let header = SCAMessageView()
header.titleLabel.text = "Warning"
header.subtitleLabel.text = "Warning message here."
tableView.setAndLayoutTableHeaderView(header)
```