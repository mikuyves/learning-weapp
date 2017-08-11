# learning wepy


## WEPY API
wepy 提供方便的 api。带参数跳转更方便。

### 结构
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

### 获取全局变量
  ```
  wepy.$instance.globalData
  ```

  - # wepy 重要提示: 
  微信开发者工具-->项目-->关闭代码压缩上传 重要：开启后，会导致真机computed, props.sync 等等属性失效。#270[https://github.com/wepyjs/wepy/issues/270]

  - coolhwm 修复了不能真机不能显示弹框选择器的问题。

### 页面跳转
用法如下
```javascript
// 即时只穿一个参数，也建议 用结构的方式放在对象中。
this.$root.$navigate('relativeUrl', {goodsId});
```



## 组件方法

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


### 数据操作 
####案例一：undefinded 有坑
```html
<view class="weui-cell"  id="isCustomerDisplay" @tap="showInner">
  <view class="weui-cell__hd">
    <view class="weui-label">选择客户</view>
  </view>
  <view class="weui-cell__hd" style="margin-right: 15rpx;">
    <image style="width: 50rpx; height: 50rpx" class="weui-cell__hd" src="{{customer.avatarUrl ? customer.avatarUrl : ''}}" />
  </view>
  <view class="weui-cell__bd" style="color: {{customer ? 'default' : 'grey'}};" >
    {{customer ? (customer.nickName ? customer.nickName : '未注册') : '未选择'}}
  </view>
  <view class="weui-cell__ft weui-cell__ft_in-access"></view>
</view>
```

```javascript
def = {
    input: {
      customerId: ''
    },
    customers: [],
    customer: null,
}
data = {...this.def};
methods = {
  customer() {
    // 用户未能清空问题，出在直接 return find 的结果，如果是 undefined，
    // 则会把 data 中的 customer 删除，这会造成选择结果中在显示上次的结果。
    // 按道理，整个 customer 被删除，应该读取不了任何数据，但是模板居然还是
    // 保留了上次的数据，未知道是什么原因造成的。如果返回一个空对象 {} 也可以，
    // 但会造成模板条件判断复杂，因为 {} ? true : false 返回 true。
    // 解决方案是 return null。
    if (!this.customers && this.input.cunstomerId) {
      return
    }
    let customer = this.customers.find(item => item.objectId === this.input.customerId)
    return customer ? customer : null
  }
}
```


### 子组件 问题
同一个父模板 A
一级子模板 B C (文件内容一样)
二级子模板 D
    A
   /\
  B  C
  \ /
   D
A.wpy
```javascript
components = {
  OrderItem: OrderItem,
  NotesItem: NotesItem,
};
```

B.wpy
```
components = {
  OrderGoods: OrderGoods
};
```

这种结构会冲突?
还是因为 B 和 C 的文件内容完全一样造成的冲突?
```console
thirdScriptError
Cannot read property 'orderGoodsInfos' of undefined;at "pages/order/scart" page lifeCycleMethod onLoad function
TypeError: Cannot read property 'orderGoodsInfos' of undefined
```


### 模板 事件 传参
wepy 改进了传参方式，可以传 _对象_ 等。传多个参数也更方便。
```html
<!-- 模板
传多个参数，需要用 `,` 分格开两个 `{{xxx}}` 的参数。
 -->
 <view @tap="detail({{prod.prodKey.objectId}}, {{prod.skuList}})">
  <!-- ... -->
</view>
```

接收参数时，默认有 `event` 事件参数。
如果有其他参数，`event` 参数将在 _最后_ 位置。因此如果不用捕捉事件，可以省略 `event` 参数。
```javascript
// 接收参数
methods = {
  detail(goodsId, skuList) {
    console.log(goodsId)  // 59862372a0bb9f005884e0bc
    console.log(skuList)  // [Object, Object]
    this.$root.$navigate('../../pages/goods/prod', {goodsId, skuList});
  }
}

// 接收 event 参数
methods = {
  detail(goodsId, skuList, e) {
    console.log(e) // 结果： e {active: true, name: "system", source: e, type: "tap", timeStamp: 2634…}
    this.$root.$navigate('../../pages/goods/prod', {goodsId, skuList, e});
  }
}
```


