# learning wepy

- `wepy.$instance` 返回一个 app 的实例，里面包含在 app.wpy 的所有内容，如下：
  - `$addons`
    - `promisify`
    - `requestfix`
  - `$wxapp`
    - `onLaunch`
    - `onHide`
    - `onShow`
    ...
  - `$interceptors`
  - `$pages`
  - `$config`
  - `$globalData`

  可以通过 api `this.$wxapp`, `this.$interceptors` 和 `this.pages` 等获取以上内容。
  获取全局变量：
  ```
  wepy.$instance.globalData
  ```

  - # wepy 重要提示: 
  微信开发者工具-->项目-->关闭代码压缩上传 重要：开启后，会导致真机computed, props.sync 等等属性失效。#270[https://github.com/wepyjs/wepy/issues/270]

  - coolhwm 修复了不能真机不能显示弹框选择器的问题。

### 组件自定义事件

1.4.8新增

可以使用`@customEvent.user`绑定用户自定义组件事件。
其中，@表示事件修饰符，customEvent 表示事件名称，`.user`表示事件后缀。
目前有三种后缀：
`.default`: 绑定小程序冒泡事件事件，如bindtap。
`.stop`: 绑定小程序非冒泡事件，如catchtap。
`.user`: 绑定用户自定义组件事件，通过$emit触发。

示例如下：
```
// index.wpy
<template>
    <child @childFn.user="parentFn"></child>
</template>
<script>
    import wepy from 'wepy';
    import Child from './coms/child';
    export default class Index extends wepy.page {
        components = {
            child: Child
        };

        methods = {
            parentFn (num, evt) {
                console.log('parent received emit event, number is: ' + num)
            }
        }
    }
</script>


// child.wpy
<template>
    <view @tap="tap">Click me</view>
</template>
<script>
    import wepy from 'wepy';
    export default class Child extends wepy.component {
        methods = {
            tap () {
                console.log('child is clicked');
                this.$emit('childFn', 100);
            }
        }
    }
</script>
```