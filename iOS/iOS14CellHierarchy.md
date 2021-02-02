# iOS14 Cell不规范添加脚本检测



在 iOS14 bate 中，UITableViewCell 中如果有直接添加在 cell 上的控件，也就是使用 `[self addSubview:]` 方式添加的控件，会显示在 contentView 的下层。
 contentView 会阻挡事件交互，使所有事件都响应 `tableView:didSelectRowAtIndexPath:` 方法，如果 customView 存在交互事件将无法响应。如果 contentView 设置了背景色，还会影响界面显示。





```shell
//注: 默认Cell的文件名以 xxxCell 做命名做规范
//如果项目有另外特殊命名规则自行修改


find (项目根目录) -name "*Cell.m" -print | xargs grep "self addSubview"
```

