# Flutter自定义动态长度TabBar样式后与TabBarView联动过慢解决



如图中 TabBarView 滑动后 TabBar 联动滞后

<img src="https://sylarimage.oss-cn-shenzhen.aliyuncs.com/uPic/737db6c9823b1e175cadda3a4d3a7c62.gif" alt="img" style="zoom:50%;" />

修改前源码



```
  _controller = TabController(
        length: titleTabs.length,
        vsync: _tickerProvider,
        initialIndex: currentIndex)
      ..addListener(() {
        //TODO 监听滑动/点选位置
            
      });


  _buildBody(BuildContext context) {
    return MediaQuery.removePadding(
        context: context,
        removeTop: true,
        child: Builder(builder: (_) {
          return Container(
              color: appWhite,
              width: MediaQuery.of(context).size.width,
              child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    _buildTop(),
                    _buildTabBar(), // TabBar
                    Expanded(child: _buildDataList()),
                  ]));
        }));
  }

  _buildTabBar() {
    return Consumer<RegularListModel>(builder: (_, model, __) {
      List<String> dataList = model.titleTabs; //动态Tab，后台取值
      if (model.isBusy || model.isError) {
        return Container();
      }
      return Container(
        color: appWhite,
        padding: EdgeInsets.fromLTRB(0, 16, 16, 0),
        child: TabBar(
          labelPadding: EdgeInsets.fromLTRB(14, 0, 0, 0),
          tabs: List.generate(
              dataList.length,
              (index) => Tab( //自定义TabBar
                    child: Container(
                      height: 22,
                      width: 42,
                      padding: EdgeInsets.only(
                          bottom: (Platform.isAndroid && index == 0) ? 1.5 : 0),
                      alignment: Alignment.center,
                      decoration: BoxDecoration(
                          borderRadius: BorderRadius.all(Radius.circular(20.0)),
                          color: index == model.currentIndex
                              ? colorFF4180FF
                              : colorFF5F8FF),
                      child: Text(
                        dataList[index],
                        style: TextStyle(
                            color: index == model.currentIndex
                                ? Colors.white
                                : colorFF4180FF,
                            fontSize: 12.0),
                      ),
                    ),
                  )),
          isScrollable: true,
          controller: model.controller,
          indicator: const BoxDecoration(),
        ),
      );
    });
  }

  _buildDataList() {
    return Consumer<RegularListModel>(builder: (_, model, __) {
        return TabBarView(
          controller: model.controller,
          children: model.titleTabs.map((item) {
            return RegularListRecordPage(symbol: item);
          }).toList());
    });
  }
```



**修改方法 PageView 取代 TabBarView 解决**



```
  final PageController _pageController = PageController(initialPage: 0);

  _buildDataList() {
    return Consumer<RegularListModel>(builder: (_, model, __) {
      return PageView.builder(
          itemCount: model.titleTabs.length,
          onPageChanged: (index) {
            model.controller.animateTo(index);
          },
          controller: _pageController,
          itemBuilder: (_, int index) =>
              RegularListRecordPage(symbol: model.titleTabs[index]));
    });
  }
```



运行如下

<img src="https://sylarimage.oss-cn-shenzhen.aliyuncs.com/uPic/1d0a7722b422e398ec79a975ecdbd35b.gif" alt="img" style="zoom:50%;" />

