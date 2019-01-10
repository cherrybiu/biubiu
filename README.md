# biubiu
最近接触的项目都是要填很多输入框的需求，而且在微信端显示，然后发现ios设备上是会出现很多兼容问题的

在解决一个问题的同时可能又会ios的另一个问题，很是头疼，最后终于在同事的帮助下找到了完美的解决办法。

1 在全局对所有的输入框监听两个事件，聚焦和失去焦点事件：  
```
  document.body.addEventListener('focusout', () => {
      // 软键盘收起的事件处理
      if (this.$util.isIphone()) {
        this.isPopup = false;
        this.scrollToEvent();
      }
    });
    document.body.addEventListener('focusin', () => {
      if (this.$util.isIphone()) {
        this.isPopup = true;
      }
    });
```

isPopup用来标记键盘是否收起，用于后面判断

```
// IOS系统下微信平台收起键盘后控制网页复原位置的事件
    scrollToEvent() {
      let self = this;
      let timer = setTimeout(() => {
        if (!self.isPopup) {
          window.scrollTo({ left: 0, top: self.offsetTop, behavior: 'smooth' });
        }
        clearTimeout(timer);
      }, 0);
    },
    scrollAction(e) {
      this.offsetTop = e.target.scrollTop;
    },
```

如图的scrollAction事件监控包含表单的scroll事件，只要滚动条有变化就会触发
