1. **使用tabbar时，点击其他tabbar会有涟漪效果，取消这种效果可以在包裹tabbar的container内设置个颜色属性，如下：**

   ```dart
   Container(
     color: Theme.of(context).canvasColor, // 取消涟漪效果
     alignment: Alignment.centerLeft,
     padding: EdgeInsets.only(left: 16),
     height: 56,
       child: TabBar(
         labelColor: Color(0xfff2f2f7),
         unselectedLabelColor: Color(0xff7b7b7b),
         tabs: titleTabs,
         isScrollable: true,
         labelPadding: EdgeInsets.symmetric(horizontal: 8),
         indicator: CustomIndicator(borderSide: BorderSide(width: 6.0, color: Color(0xfffcc800)), )
       ),
   ),
   ```

2. **评论框弹出以及收回时, 键盘跟着弹出的速度不一样, 采用了多种方法但是仍然会有卡顿的效果:**
   - 使用默认键盘弹出将布局顶起的方法, 在特定时间内, 将输入框距离bottom的距离从(-默认高度)→默认高度进行过渡;
   - 关闭键盘默认改变布局size方法, 直接将输入框的bottom的值在特定时间内从(-默认高度)→(默认高度+键盘高度)进行过渡

