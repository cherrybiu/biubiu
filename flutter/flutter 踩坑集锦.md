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

