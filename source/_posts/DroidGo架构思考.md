# DroidGo架构的建立与思考

* ## 起因
前段时间刀牌Artifact(划掉!刀塔自走棋才是唯一真刀牌!)发布,我就想根据V社的Api获取数据做一个卡牌浏览和卡组收藏建立的App,顺带熟悉下Room,LiveData,ViewModel等AndroidArchitectureComponent组件的使用,由于工(yao)作(da)原(dao)因(ta)一直没能完成,只是有了一个初步的架构
> 下面是~~预览版~~的图

![droidgo_demo.gif](https://github.com/apm29/DroidGo/blob/apm29-dev/art/droidgo_demo.gif)

* ### View层的状态管理
  * Activity/Fragment中抽象出一个页面状态的接口,暂时分为LOADING,NORMAL,ERROR状态
  * 考虑怎样将管理类注入到Activity/Fragment而不侵入,不直接持有Context对象实现监听,还要接受数据层或者中间层的loading信号,怎样暴露自己给其他层
  * 初步考虑分为IViewState和IViewStateData接口,前者由Activity或者Fragment实现,并代理给一个IViewStateDelegate对象,IViewStateData由数据管理层持有,包含几个LiveData数据来监听View的状态改变,所以基本结构就是数据层订阅IViewStateData-->IViewStateDelegate观察-->IViewState获得回调改变页面
  * IViewStateDelegate因为要有对view的操作所以得被Activity/Fragment持有了,数据层呢如果是RxJava形式的,就写个扩展方法订阅IViewStateData
