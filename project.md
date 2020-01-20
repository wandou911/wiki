## 项目知识点总结

#### 1 IQKeyboardManager

项目中用到了IQKeyboardManager，它会在有文本框输入，键盘弹出后自动实现页面的滚动，但是在有的页面需要禁用IQKeyboardManager自动滚动，比如collectionView有文本框的时候，需要手动计算collectionView的上移或者下移，就需要禁用掉IQKeyboardManager

```
[[IQKeyboardManager sharedManager] setEnable:NO];
```
#### 2 [子控制器事件失效](https://blog.csdn.net/zhaoxy_thu/article/details/50826190)

子控制器 [self addChildViewController:childController];

没有这行代码会导致子控制器的事件无法生效

```
SecondViewController *childController = [[SecondViewController alloc] init];
[self addChildViewController:childController];
[self.view addSubview:childController.view];
```
#### 3 [viewWillAppear失效](https://www.jianshu.com/p/823135dc742a)

 将UINavigationController的view作为subview添加到了其他viewController的view中。或者把UINavigationController添加到UITabbarController中了;
此时，NavigationController的stack里面的viewController就收不到-(void)viewWillAppear:(BOOL)animated等4个方法的调用

解决方法:在viewWillAppear与viewWillDisappear把你的subCntlr的里面这两方法在调用一次即可

```
-(void)viewWillAppear:(BOOL)animated
{
 [super viewWillAppear:animated];
[subCntlr viewWillAppear:animated];

}

-(void)viewWillDisappear:(BOOL)animated
{
 [super viewWillDisappear:animated];
 [subCntlr viewWillDisappear:animated];
}
```

